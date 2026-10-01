# Idempotência em Java


---

## Sumário

1. [Modelo de domínio usado nos exemplos](#1-modelo-de-domínio-usado-nos-exemplos)
2. [O que é idempotência](#2-o-que-é-idempotência)
3. [Por que idempotência importa](#3-por-que-idempotência-importa)
4. [Idempotência x operações HTTP/REST](#4-idempotência-x-operações-httprest)
5. [Classificando operações: idempotente, não idempotente, segura](#5-classificando-operações-idempotente-não-idempotente-segura)
6. [❌ O que NÃO fazer: operações frágeis a reexecução](#6--o-que-não-fazer-operações-frágeis-a-reexecução)
7. [✅ Padrão: chave de idempotência](#7--padrão-chave-de-idempotência)
8. [✅ Padrão: idempotência via estado (upsert / set absoluto)](#8--padrão-idempotência-via-estado-upsert--set-absoluto)
9. [✅ Padrão: operações matematicamente idempotentes](#9--padrão-operações-matematicamente-idempotentes)
10. [Idempotência sob concorrência](#10-idempotência-sob-concorrência)
11. [Idempotência com banco de dados (constraint única)](#11-idempotência-com-banco-de-dados-constraint-única)
12. [Idempotência em filas e mensageria (at-least-once delivery)](#12-idempotência-em-filas-e-mensageria-at-least-once-delivery)
13. [Expiração de chaves de idempotência](#13-expiração-de-chaves-de-idempotência)
14. [Exercício resolvido](#14-exercício-resolvido)
15. [Armadilhas e boas práticas](#15-armadilhas-e-boas-práticas)
16. [Cheat sheet](#16-cheat-sheet)

---

## 1. Modelo de domínio usado nos exemplos

```java
public class Entrega {
    private String codigo;
    private String destino;
    private double pesoKg;
    private double frete;
    private StatusEntrega status;

    public Entrega(String codigo, String destino, double pesoKg, double frete, StatusEntrega status) {
        this.codigo = codigo;
        this.destino = destino;
        this.pesoKg = pesoKg;
        this.frete = frete;
        this.status = status;
    }

    public String getCodigo() { return codigo; }
    public String getDestino() { return destino; }
    public double getPesoKg() { return pesoKg; }
    public double getFrete() { return frete; }
    public StatusEntrega getStatus() { return status; }
    public void setStatus(StatusEntrega status) { this.status = status; }
    public void setFrete(double frete) { this.frete = frete; }

    @Override
    public String toString() {
        return codigo + " | " + destino + " | " + pesoKg + "kg | R$ " + String.format("%.2f", frete) + " | " + status;
    }
}

public enum StatusEntrega { PENDENTE, EM_ROTA, ENTREGUE, CANCELADA }
```

**Cenário usado nos exemplos:** um sistema de logística recebe **eventos** (confirmações de entrega, solicitações de cobrança de frete, pedidos de cancelamento) que podem chegar **mais de uma vez** — por reenvio de app do motorista com internet instável, por retry automático de uma integração, ou por dois operadores clicando "confirmar" ao mesmo tempo.

---

## 2. O que é idempotência

**Idempotência** é a propriedade de uma operação de produzir **o mesmo resultado final**, não importa quantas vezes seja executada com a mesma entrada.

> Formalmente: `f(x) == f(f(x)) == f(f(f(x)))`

**Analogia logística:** apertar o botão de **chamar o elevador de carga** no térreo é idempotente — apertar 1 vez ou 10 vezes seguidas tem o mesmo efeito final: o elevador vem ao térreo. Já apertar o botão de **"adicionar 1 volume ao caminhão"** não é idempotente — cada aperto soma mais um volume, e apertar 10 vezes carrega 10 volumes.

```java
// Idempotente: resultado final não muda com repetição
entrega.setStatus(StatusEntrega.ENTREGUE);
entrega.setStatus(StatusEntrega.ENTREGUE);   // sem efeito adicional, status já é ENTREGUE

// NÃO idempotente: cada chamada agrava o efeito
volumesNoCaminhao++;
volumesNoCaminhao++;   // agora são 2, não 1
```

Importante: idempotência fala do **resultado final do sistema**, não da ausência de efeitos colaterais. A operação pode gravar um log a cada chamada (isso não é idempotente em si), mas o **estado de negócio relevante** (status da entrega, saldo, volumes) precisa convergir para o mesmo valor.

---

## 3. Por que idempotência importa

| Cenário real | Problema sem idempotência |
|---|---|
| App do motorista perde sinal após enviar "entrega confirmada" e reenvia ao reconectar | Entrega contabilizada 2x nas métricas |
| Gateway de pagamento de frete faz retry automático por timeout | Frete cobrado 2x do cliente |
| Dois operadores clicam "cancelar pedido" quase ao mesmo tempo | Estoque devolvido 2x, criando saldo fantasma |
| Fila de mensagens (Kafka, SQS) redelivera a mesma mensagem (garantia *at-least-once*) | Processamento duplicado do evento |
| Load balancer reenvia requisição que não recebeu resposta a tempo (mas que já havia sido processada) | Efeito duplicado no backend |

Idempotência é o que torna **retry seguro**. Sem ela, qualquer mecanismo de resiliência (retry, redelivery, timeout + reenvio) se torna uma fonte de bugs de duplicação.

---

## 4. Idempotência x operações HTTP/REST

A especificação HTTP já classifica métodos quanto à idempotência — útil como referência mental, mesmo fora de REST:

| Método | Idempotente? | Por quê |
|---|---|---|
| `GET` | Sim | Só leitura, não altera estado |
| `PUT` | Sim (por convenção) | Substitui o recurso inteiro pelo mesmo conteúdo — repetir não muda o resultado |
| `DELETE` | Sim (por convenção) | Deletar algo já deletado continua "não existe" |
| `POST` | **Não**, por padrão | Cada chamada tende a **criar** um novo recurso |
| `PATCH` | Depende | Idempotente se o patch é um "set absoluto"; não idempotente se é um incremento relativo |

```java
// PUT idempotente: define o estado final, não importa quantas vezes seja chamado
public void atualizarStatus(String codigo, StatusEntrega novoStatus) {
    Entrega e = buscar(codigo);
    e.setStatus(novoStatus);   // PUT /entregas/{codigo} { status: "ENTREGUE" }
}

// POST não idempotente: cada chamada cria uma nova entrega
public Entrega criarEntrega(String destino, double pesoKg) {
    return repositorio.salvar(new Entrega(gerarCodigo(), destino, pesoKg, 0, StatusEntrega.PENDENTE));
}
```

> `POST` pode **se tornar** idempotente com o padrão de chave de idempotência (seção 7) — é exatamente para isso que o padrão existe.

---

## 5. Classificando operações: idempotente, não idempotente, segura

```java
public class EntregaService {

    // IDEMPOTENTE: define um valor absoluto
    public void definirFrete(Entrega e, double valorFrete) {
        e.setFrete(valorFrete);
    }

    // NÃO IDEMPOTENTE: aplica um delta/incremento
    public void aplicarDesconto(Entrega e, double percentual) {
        e.setFrete(e.getFrete() * (1 - percentual / 100));
    }

    // IDEMPOTENTE: transição de estado "absorvente" (uma vez ENTREGUE, continua ENTREGUE)
    public void confirmarEntrega(Entrega e) {
        e.setStatus(StatusEntrega.ENTREGUE);
    }

    // SEGURA (safe) e também idempotente: leitura não muda nada
    public StatusEntrega consultarStatus(Entrega e) {
        return e.getStatus();
    }
}
```

| Tipo | Repetir muda o resultado final? | Exemplo |
|---|---|---|
| **Segura (safe)** | Nunca muda nada (nem 1x) | Consultas/leituras |
| **Idempotente** | 1x ou Nx chegam ao mesmo estado final | `setStatus`, `PUT` |
| **Não idempotente** | Cada execução agrava o efeito | `aplicarDesconto`, incrementos, `POST` simples |

> Toda operação **segura** é idempotente (por vacuidade), mas nem toda idempotente é segura — `setStatus` muda o sistema na primeira chamada, só não agrava em chamadas seguintes.

---

## 6. ❌ O que NÃO fazer: operações frágeis a reexecução

### ❌ Incrementar contadores de negócio em um handler que pode ser reexecutado

```java
// ERRADO: cada retry da confirmação soma mais uma entrega
public class RelatorioService {
    private int totalEntregasConcluidas = 0;

    public void onEntregaConfirmada(String codigo) {
        totalEntregasConcluidas++;   // retry do evento = contagem duplicada
    }
}
```

### ❌ Criar um novo registro sem checar se o evento já foi processado

```java
// ERRADO: dois cliques/retries criam duas cobranças de frete
public class CobrancaService {
    public Cobranca cobrarFrete(String codigoEntrega, double valor) {
        return repositorio.salvar(new Cobranca(codigoEntrega, valor));   // sempre cria nova linha
    }
}
```

### ❌ Usar apenas timestamp/horário como proteção contra duplicidade

```java
// ERRADO: retries acontecem em milissegundos de diferença, timestamp não distingue nada
public void registrarEvento(String tipo) {
    if (System.currentTimeMillis() - ultimoRegistro > 1000) {   // janela arbitrária, não confiável
        processar(tipo);
    }
}
```

### ❌ Confiar só na ordem de chegada das mensagens

```java
// ERRADO: filas como Kafka/SQS não garantem exactly-once por padrão,
// e mensagens podem chegar fora de ordem ou duplicadas
public void onMensagemRecebida(EventoEntrega evento) {
    aplicarEfeito(evento);   // sem checagem de duplicidade nem de ordem
}
```

### ❌ Operação "ler, decidir, escrever" sem atomicidade

```java
// ERRADO: race condition entre leitura e escrita (TOCTOU - time of check to time of use)
public void retirarEstoque(String produto, int qtd) {
    int atual = estoque.get(produto);
    if (atual >= qtd) {
        estoque.put(produto, atual - qtd);   // outra thread pode ter alterado 'atual' nesse intervalo
    }
}
```

---

## 7. ✅ Padrão: chave de idempotência

A técnica mais geral: cada operação sensível carrega uma **chave única** (gerada pelo cliente ou pelo evento de origem). Antes de aplicar o efeito, o sistema verifica se aquela chave já foi processada.

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.Set;

public class ConfirmacaoEntregaService {
    // Em produção: tabela/cache (banco, Redis) com constraint única — aqui, em memória para o exemplo
    private final Set<String> chavesProcessadas = ConcurrentHashMap.newKeySet();

    public void confirmar(String chaveIdempotencia, Entrega entrega) {
        if (!chavesProcessadas.add(chaveIdempotencia)) {
            System.out.println("Evento " + chaveIdempotencia + " já processado. Ignorando repetição.");
            return;
        }
        entrega.setStatus(StatusEntrega.ENTREGUE);
        System.out.println("Entrega confirmada: " + entrega);
    }
}
```

```java
ConfirmacaoEntregaService servico = new ConfirmacaoEntregaService();
Entrega entrega = new Entrega("E001", "Lorena", 20.0, 80.0, StatusEntrega.EM_ROTA);

// O app do motorista reenvia o mesmo evento 3x (perda de conexão, retry automático etc.)
servico.confirmar("evt-entrega-e001-confirmacao", entrega);
servico.confirmar("evt-entrega-e001-confirmacao", entrega);
servico.confirmar("evt-entrega-e001-confirmacao", entrega);
// Resultado: confirmado 1 vez, 2 chamadas ignoradas
```

A chave deve ser:
- **Determinística** a partir do evento de origem (ex.: `codigoEntrega + ":" + tipoEvento`), nunca gerada aleatoriamente a cada chamada.
- **Única** o suficiente para não colidir com eventos diferentes.
- Gerada no **cliente/produtor do evento** quando possível (ex.: app do motorista gera um UUID por tentativa de envio e reusa o mesmo UUID em reenvios).

---

## 8. ✅ Padrão: idempotência via estado (upsert / set absoluto)

Em vez de "somar" ou "aplicar delta", a operação **define o estado final diretamente**. Repetir não agrava nada.

```java
public class EntregaRepository {
    private final Map<String, Entrega> dados = new ConcurrentHashMap<>();

    // upsert: idempotente por natureza, sempre sobrescreve com o mesmo valor final
    public void salvar(Entrega entrega) {
        dados.put(entrega.getCodigo(), entrega);
    }
}
```

```java
// Transição de status como "set absoluto", não como incremento de etapa
public void avancarParaEmRota(Entrega e) {
    e.setStatus(StatusEntrega.EM_ROTA);   // definir, não "avançar + 1 estado"
}
```

Compare com a alternativa não idempotente (uma máquina de estados baseada em "próximo passo"):

```java
// Frágil: se chamado 2x, tenta avançar 2 estados
public void avancarStatus(Entrega e) {
    StatusEntrega[] ordem = StatusEntrega.values();
    int proximo = e.getStatus().ordinal() + 1;
    e.setStatus(ordem[proximo]);   // 2 chamadas = PENDENTE -> EM_ROTA -> ENTREGUE (pulo indevido)
}
```

---

## 9. ✅ Padrão: operações matematicamente idempotentes

Algumas operações já são idempotentes por definição matemática — prefira-as quando o domínio permitir.

```java
// Idempotente: aplicar o mesmo "clamp" de peso máximo quantas vezes quiser
public void aplicarLimiteMaximoPeso(Entrega e, double limite) {
    e.setPesoKg(Math.min(e.getPesoKg(), limite));
}

// Idempotente: funções de arredondamento/normalização
public void normalizarFrete(Entrega e) {
    e.setFrete(Math.round(e.getFrete() * 100) / 100.0);
}

// Idempotente: operações de conjunto (adicionar a um Set)
Set<String> cidadesAtendidas = new HashSet<>();
cidadesAtendidas.add("Lorena");
cidadesAtendidas.add("Lorena");   // conjunto continua com 1 elemento
```

| Operação | Idempotente? |
|---|---|
| `Math.min/max(x, limite)` | Sim |
| `set.add(x)` | Sim (estado do conjunto) |
| `map.put(chave, valor)` | Sim (mesmo valor) |
| `lista.add(x)` | **Não** (cada chamada adiciona outro elemento) |
| `contador += 1` | **Não** |
| `contador = valorFixo` | Sim |

---

## 10. Idempotência sob concorrência

A checagem "existe? então ignora" **precisa ser atômica**, senão duas threads concorrentes podem passar pela checagem antes que qualquer uma registre a chave — reabrindo a janela de duplicidade que a idempotência deveria fechar.

### ❌ Checagem não atômica (ainda vulnerável a race condition)

```java
// ERRADO: "contains" e "add" são 2 operações separadas — outra thread pode intercalar entre elas
public void confirmar(String chave, Entrega e) {
    if (!chavesProcessadas.contains(chave)) {   // thread A e thread B podem ler "false" ao mesmo tempo
        chavesProcessadas.add(chave);
        aplicarEfeito(e);                       // as duas aplicam o efeito
    }
}
```

### ✅ Checagem atômica com `Set.add` de `ConcurrentHashMap`

```java
// CORRETO: add() do ConcurrentHashMap.newKeySet() é atômico —
// só a primeira chamada (entre threads concorrentes) recebe true
public void confirmar(String chave, Entrega e) {
    if (chavesProcessadas.add(chave)) {
        aplicarEfeito(e);
    }
}
```

### ✅ Alternativa com `putIfAbsent` para guardar também o resultado

```java
private final ConcurrentHashMap<String, Entrega> resultadosPorChave = new ConcurrentHashMap<>();

public Entrega confirmarComResultado(String chave, Entrega e) {
    return resultadosPorChave.computeIfAbsent(chave, k -> {
        e.setStatus(StatusEntrega.ENTREGUE);
        return e;
    });
}
```

`computeIfAbsent` garante que a função só executa **uma vez por chave**, mesmo sob concorrência — chamadas repetidas (concorrentes ou não) recebem o resultado já calculado.

> Para aprofundar os mecanismos de atomicidade e exclusão mútua usados aqui, veja [[30-THREADS-CONCORRENCIA-JAVA]].

---

## 11. Idempotência com banco de dados (constraint única)

Em memória, `ConcurrentHashMap` resolve o problema dentro de uma única instância da aplicação. Em sistemas distribuídos (múltiplas instâncias), a garantia precisa estar no **banco de dados**.

```java
// DDL (ilustrativo): chave de idempotência com constraint UNIQUE
// CREATE TABLE confirmacoes_entrega (
//     chave_idempotencia VARCHAR(100) PRIMARY KEY,
//     codigo_entrega VARCHAR(20) NOT NULL,
//     processado_em TIMESTAMP NOT NULL
// );

public class ConfirmacaoEntregaRepository {

    public boolean registrarSeNovo(String chaveIdempotencia, String codigoEntrega) {
        String sql = "INSERT INTO confirmacoes_entrega (chave_idempotencia, codigo_entrega, processado_em) "
                   + "VALUES (?, ?, NOW())";
        try (PreparedStatement stmt = conexao.prepareStatement(sql)) {
            stmt.setString(1, chaveIdempotencia);
            stmt.setString(2, codigoEntrega);
            stmt.executeUpdate();
            return true;   // inserção nova, pode aplicar o efeito de negócio
        } catch (SQLIntegrityConstraintViolationException e) {
            return false;  // chave já existia (violação de UNIQUE/PRIMARY KEY) -> já processado, ignorar
        }
    }
}
```

```java
public void confirmarEntrega(String chaveIdempotencia, Entrega entrega) {
    boolean novo = confirmacaoRepository.registrarSeNovo(chaveIdempotencia, entrega.getCodigo());
    if (novo) {
        entrega.setStatus(StatusEntrega.ENTREGUE);
        entregaRepository.salvar(entrega);
    }
}
```

> A constraint `UNIQUE`/`PRIMARY KEY` é quem garante atomicidade real entre múltiplas instâncias da aplicação — o banco rejeita a segunda inserção com a mesma chave, mesmo que duas instâncias tentem ao mesmo tempo.

---

## 12. Idempotência em filas e mensageria (at-least-once delivery)

Sistemas de mensageria (Kafka, SQS, RabbitMQ com certas configurações) costumam garantir **at-least-once delivery**: a mensagem **será** entregue, mas **pode ser entregue mais de uma vez**. O consumidor precisa ser idempotente.

```java
public class ConsumidorEventoEntrega {
    private final Set<String> idsMensagensProcessadas = ConcurrentHashMap.newKeySet();

    public void onMensagem(MensagemFila mensagem) {
        String idMensagem = mensagem.getId();   // id único da mensagem, gerado pelo produtor

        if (!idsMensagensProcessadas.add(idMensagem)) {
            System.out.println("Mensagem " + idMensagem + " já processada. Confirmando ack sem reprocessar.");
            mensagem.ack();   // confirma recebimento mesmo sem reprocessar, para a fila não reenviar de novo
            return;
        }

        try {
            processarEvento(mensagem);
            mensagem.ack();
        } catch (Exception ex) {
            mensagem.nack();   // falha real de processamento -> pede reentrega
        }
    }

    private void processarEvento(MensagemFila mensagem) {
        System.out.println("Processando: " + mensagem.getPayload());
    }
}
```

> Em produção, o conjunto de IDs processados deve persistir em algo compartilhado entre instâncias (banco, Redis com TTL), não em memória local — senão cada instância "esquece" o que já viu ao reiniciar.

---

## 13. Expiração de chaves de idempotência

Guardar chaves de idempotência **para sempre** não escala. A prática comum é expirar a chave após uma janela de tempo em que retries realistas ainda poderiam ocorrer.

```java
import java.time.Instant;
import java.util.concurrent.ConcurrentHashMap;

public class IdempotenciaComExpiracao {
    private record Registro(Instant expiraEm) {}

    private final ConcurrentHashMap<String, Registro> chaves = new ConcurrentHashMap<>();
    private final long janelaSegundos;

    public IdempotenciaComExpiracao(long janelaSegundos) {
        this.janelaSegundos = janelaSegundos;
    }

    public boolean processarSeNovo(String chave) {
        Instant agora = Instant.now();

        Registro existente = chaves.get(chave);
        if (existente != null && existente.expiraEm().isAfter(agora)) {
            return false;   // ainda dentro da janela: considerado duplicado
        }

        chaves.put(chave, new Registro(agora.plusSeconds(janelaSegundos)));
        return true;
    }
}
```

> A janela deve ser maior que o maior intervalo de retry esperado do sistema de origem (ex.: 24h para integrações com apps móveis que podem reenviar após reconectar depois de ficarem offline por horas).

---

## 14. Exercício resolvido

**Problema:** um serviço recebe eventos de confirmação de entrega que podem chegar duplicados (reenvio de app + retry de rede). Implementar um processador idempotente que: (1) ignora eventos repetidos, (2) atualiza o status corretamente na primeira vez, (3) é seguro sob chamadas concorrentes.

```java
import java.util.*;
import java.util.concurrent.*;

public class Program {
    public static void main(String[] args) throws InterruptedException {
        Map<String, Entrega> entregas = new ConcurrentHashMap<>();
        entregas.put("E001", new Entrega("E001", "Lorena", 20.0, 80.0, StatusEntrega.EM_ROTA));

        ProcessadorConfirmacao processador = new ProcessadorConfirmacao(entregas);

        // Simula 5 chamadas concorrentes com a MESMA chave de idempotência (retry duplicado)
        ExecutorService pool = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            pool.submit(() -> processador.processarConfirmacao("evt-E001-confirmacao", "E001"));
        }
        pool.shutdown();
        pool.awaitTermination(2, TimeUnit.SECONDS);

        System.out.println("Status final: " + entregas.get("E001").getStatus());
        System.out.println("Vezes que o efeito de negócio foi realmente aplicado: " + processador.getContadorAplicacoes());
    }
}

class ProcessadorConfirmacao {
    private final Map<String, Entrega> entregas;
    private final Set<String> chavesProcessadas = ConcurrentHashMap.newKeySet();
    private final java.util.concurrent.atomic.AtomicInteger contadorAplicacoes = new java.util.concurrent.atomic.AtomicInteger(0);

    public ProcessadorConfirmacao(Map<String, Entrega> entregas) {
        this.entregas = entregas;
    }

    public void processarConfirmacao(String chaveIdempotencia, String codigoEntrega) {
        if (!chavesProcessadas.add(chaveIdempotencia)) {
            System.out.println(Thread.currentThread().getName() + ": evento duplicado, ignorado.");
            return;
        }

        Entrega entrega = entregas.get(codigoEntrega);
        entrega.setStatus(StatusEntrega.ENTREGUE);
        contadorAplicacoes.incrementAndGet();
        System.out.println(Thread.currentThread().getName() + ": confirmação aplicada.");
    }

    public int getContadorAplicacoes() {
        return contadorAplicacoes.get();
    }
}
```

**Saída esperada (ordem das threads pode variar):**

```
pool-1-thread-1: confirmação aplicada.
pool-1-thread-2: evento duplicado, ignorado.
pool-1-thread-3: evento duplicado, ignorado.
pool-1-thread-4: evento duplicado, ignorado.
pool-1-thread-5: evento duplicado, ignorado.
Status final: ENTREGUE
Vezes que o efeito de negócio foi realmente aplicado: 1
```

---

## 15. Armadilhas e boas práticas

| Armadilha | Explicação | Solução |
|---|---|---|
| Confundir "idempotente" com "sem efeitos colaterais" | Idempotência é sobre o **estado final**, não sobre ausência total de efeito | Pode logar/auditar a cada chamada; o estado de negócio relevante é que não deve duplicar |
| Checar duplicidade com `contains` + `add` separados | Race condition entre as duas operações | Usar operação atômica (`Set.add`, `computeIfAbsent`, constraint `UNIQUE` no banco) |
| Gerar a chave de idempotência **no servidor, a cada tentativa** | Cada retry gera uma chave nova, perdendo a proteção | Chave deve vir do cliente/produtor do evento, ou ser determinística a partir do conteúdo do evento |
| Guardar chaves de idempotência para sempre, sem expiração | Crescimento ilimitado de memória/armazenamento | Definir janela de expiração compatível com o maior retry esperado |
| Tratar `POST` como automaticamente seguro para retry | `POST` cria recursos por padrão; retry sem proteção duplica | Aplicar padrão de chave de idempotência explicitamente |
| Usar apenas timestamp como defesa contra duplicidade | Retries podem ocorrer em milissegundos ou dias depois; timestamp não é confiável | Usar identificador único do evento, não tempo |
| Implementar idempotência só em memória em sistema com múltiplas instâncias | Cada instância "esquece" o que já processou | Persistir estado de idempotência em armazenamento compartilhado (banco, Redis) |
| Assumir que a fila garante exactly-once | Maioria das filas reais garante at-least-once | Consumidor deve ser idempotente independente da garantia da fila |

### Regras de ouro

1. **Prefira "definir o estado final" a "aplicar um delta"** sempre que o domínio permitir.
2. **Toda operação reexecutável por retry, fila ou concorrência precisa de uma chave de idempotência.**
3. **A checagem de duplicidade deve ser atômica** — nunca "verificar e depois agir" em passos separados.
4. **Em sistemas distribuídos, a garantia de idempotência vive no armazenamento compartilhado**, não na memória de uma instância.
5. **Documente explicitamente** quais operações da sua API são idempotentes — isso é um contrato para quem integra com ela.

---

## 16. Cheat sheet

```java
// Checagem atômica simples (processa uma única vez por chave)
Set<String> processados = ConcurrentHashMap.newKeySet();
if (processados.add(chave)) {
    aplicarEfeito();
}

// Checagem atômica com resultado memorizado
ConcurrentHashMap<String, Resultado> cache = new ConcurrentHashMap<>();
Resultado r = cache.computeIfAbsent(chave, k -> aplicarEfeitoEDevolverResultado());

// "Set absoluto" em vez de delta (idempotente por natureza)
entidade.setCampo(valorFinal);        // idempotente
// contador += valor;                 // NÃO idempotente

// Banco de dados: constraint única como fonte de verdade distribuída
// INSERT INTO idempotencia (chave) VALUES (?)  -- falha com UNIQUE violation se repetido
```

| Pergunta de verificação | Se a resposta for "sim" |
|---|---|
| Essa operação pode ser reexecutada pelo cliente, por retry ou por uma fila? | Precisa de proteção de idempotência |
| O efeito é "somar/incrementar" em vez de "definir"? | Reavaliar para um modelo de "set absoluto" quando possível |
| A checagem de duplicidade usa mais de uma operação separada (ler depois escrever)? | Trocar por uma operação atômica |
| O sistema roda em mais de uma instância? | A proteção precisa estar no armazenamento compartilhado, não em memória local |

### Referências

- Docs Java: [`ConcurrentHashMap`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html), [`Map#computeIfAbsent`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Map.html#computeIfAbsent(K,java.util.function.Function))
- RFC 9110 (HTTP Semantics) — seção sobre métodos seguros e idempotentes
- Ver também: [[30-THREADS-CONCORRENCIA-JAVA]] e [[31-PARALELISMO-JAVA]] (base de concorrência usada nos exemplos sob carga concorrente)
