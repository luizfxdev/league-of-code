# Tratamento de Exceções em Java — Contexto de Logística

## 1. O que é uma exceção

Exceção = evento anormal durante a execução do programa. Em uma transportadora, seria como: tentar entregar um pacote com endereço inválido, ou consultar um veículo que não existe na frota.

Hierarquia:

```
Throwable
├── Error                  (erro grave de JVM, não tratamos)
└── Exception
    ├── RuntimeException   (unchecked — compilador NÃO obriga tratar)
    └── outras Exception   (checked — compilador OBRIGA tratar ou propagar)
```

- **Checked (`Exception`)**: compilador exige `try/catch` ou `throws`. Ex: `IOException` ao ler um arquivo de rotas.
- **Unchecked (`RuntimeException`)**: compilador não exige nada. Ex: `NullPointerException`, `ArrayIndexOutOfBoundsException`.

---

## 2. Por que usar exceções (e não `if` espalhado)

Cenário: validar uma entrega antes de despachar.

### Solução ruim — retorno de string/código de erro

```java
public String despachar(Entrega entrega) {
    if (entrega.getPeso() > veiculo.getCapacidade()) {
        return "ERRO: peso excede capacidade";
    }
    if (entrega.getDestino() == null) {
        return "ERRO: destino inválido";
    }
    // ... lógica misturada com validação
    return "OK";
}
```

Problemas: quem chama pode esquecer de checar o retorno; a lógica de despacho se mistura com validação; não dá pra saber o tipo do erro sem parsear string.

### Solução boa — exceção

```java
public void despachar(Entrega entrega) throws DespachoException {
    if (entrega.getPeso() > veiculo.getCapacidade()) {
        throw new CapacidadeExcedidaException("Peso excede capacidade do veículo");
    }
    if (entrega.getDestino() == null) {
        throw new EnderecoInvalidoException("Destino não informado");
    }
    // lógica de despacho limpa, sem condicionais de erro
}
```

Vantagens: a lógica de negócio fica limpa; o erro carrega dados (peso, capacidade); o compilador ajuda (checked); dá pra tratar por hierarquia.

---

## 3. `try-catch` — capturando o problema

```java
public class ControleFrota {
    public static void main(String[] args) {
        Veiculo[] frota = new Veiculo[5];
        Scanner sc = new Scanner(System.in);

        try {
            int indice = sc.nextInt();
            Veiculo v = frota[indice]; // pode não existir
            System.out.println(v.getPlaca());
        }
        catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Veículo não encontrado na frota");
        }
        catch (InputMismatchException e) {
            System.out.println("Índice inválido, digite um número");
        }
        finally {
            sc.close();
        }
    }
}
```

Regras:
- Pode haver vários `catch` — Java testa na ordem, do topo pra baixo.
- Catch de superclasse (`Exception`) sempre por último, senão os `catch` mais específicos ficam inalcançáveis (erro de compilação).
- `finally` roda sempre — ideal para fechar `Scanner`, conexão com banco (ex: liberar conexão após consultar status de uma entrega).

---

## 4. Pilha de chamadas (call stack)

Quando uma exceção é lançada e não é tratada no método atual, ela **sobe** na pilha até achar um `catch` compatível ou o programa encerrar.

```java
public class RastreamentoEntrega {

    public static void main(String[] args) {
        processarRota();
        System.out.println("Fim do processamento");
    }

    static void processarRota() {
        System.out.println("Iniciando rota...");
        calcularDistancia();
        System.out.println("Rota processada");
    }

    static void calcularDistancia() {
        try {
            int[] coordenadas = new int[3];
            System.out.println(coordenadas[5]); // explode aqui
        }
        catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Coordenada de rota inválida");
            e.printStackTrace(); // mostra o caminho: calcularDistancia -> processarRota -> main
        }
    }
}
```

Se `calcularDistancia()` não tivesse `catch`, a exceção subiria para `processarRota()`, e se lá também não houvesse tratamento, subiria para `main()`. Sem tratamento em nenhum nível, o programa encerra com stack trace impresso.

---

## 5. `throws` vs `throw`

| Palavra | Uso | Exemplo |
|---|---|---|
| `throws` | Declaração no método — avisa "esse método pode lançar isso, quem chamar decide tratar" | `void entregar() throws RotaInvalidaException` |
| `throw` | Ação — lança a exceção de fato, interrompe o método imediatamente | `throw new RotaInvalidaException("Sem rota até o destino")` |

```java
public void iniciarEntrega(Entrega e) throws PrazoExpiradoException {
    if (e.getPrazo().isBefore(LocalDate.now())) {
        throw new PrazoExpiradoException("Prazo de entrega já expirou");
    }
    // segue o fluxo normal
}
```

---

## 6. Criando exceções personalizadas

Padrão de pacotes sugerido (igual ao slide do curso, adaptado):

```
model
├── entities      (Entrega, Veiculo, Motorista)
├── enums         (StatusEntrega, TipoVeiculo)
├── exceptions    (RotaInvalidaException, CapacidadeExcedidaException...)
└── services      (LogisticaService)
```

Exemplo de exceção customizada, extendendo `RuntimeException` (unchecked, mais comum em regras de negócio):

```java
package model.exceptions;

public class CapacidadeExcedidaException extends RuntimeException {

    public CapacidadeExcedidaException(String msg) {
        super(msg);
    }
}
```

Extendendo `Exception` (checked — força quem chama a tratar), útil quando o erro é "esperável" e recuperável, como falha de comunicação com o rastreador:

```java
package model.exceptions;

public class RastreadorIndisponivelException extends Exception {

    public RastreadorIndisponivelException(String msg) {
        super(msg);
    }
}
```

Uso:

```java
public void atualizarLocalizacao(Veiculo v) throws RastreadorIndisponivelException {
    if (!v.getRastreador().estaOnline()) {
        throw new RastreadorIndisponivelException("Rastreador do veículo " + v.getPlaca() + " offline");
    }
    v.setLocalizacaoAtual(v.getRastreador().getCoordenadas());
}
```

Chamando no `main`:

```java
try {
    servico.atualizarLocalizacao(veiculo);
}
catch (RastreadorIndisponivelException e) {
    System.out.println("Erro: " + e.getMessage());
}
```

---

## 7. Exceção carregando dados extras

Uma exceção é um objeto — pode carregar mais que uma mensagem.

```java
public class CapacidadeExcedidaException extends RuntimeException {

    private double capacidadeMaxima;
    private double pesoTentado;

    public CapacidadeExcedidaException(double capacidadeMaxima, double pesoTentado) {
        super("Peso " + pesoTentado + "kg excede capacidade de " + capacidadeMaxima + "kg");
        this.capacidadeMaxima = capacidadeMaxima;
        this.pesoTentado = pesoTentado;
    }

    public double getExcedente() {
        return pesoTentado - capacidadeMaxima;
    }
}
```

Quem captura pode usar `getExcedente()` para, por exemplo, sugerir dividir a carga em dois veículos.

---

## 8. Exemplo completo — Reserva de veículo (adaptado do slide)

Regras: só pode atualizar reserva para datas futuras; data de devolução deve ser maior que a de retirada.

```java
package model.entities;

import java.time.LocalDate;
import model.exceptions.ReservaException;

public class ReservaVeiculo {

    private Integer placaVeiculo;
    private LocalDate dataRetirada;
    private LocalDate dataDevolucao;

    public ReservaVeiculo(Integer placaVeiculo, LocalDate dataRetirada, LocalDate dataDevolucao) {
        this.placaVeiculo = placaVeiculo;
        this.dataRetirada = dataRetirada;
        this.dataDevolucao = dataDevolucao;
    }

    public long duracaoDias() {
        return dataRetirada.until(dataDevolucao).getDays();
    }

    public void atualizarDatas(LocalDate novaRetirada, LocalDate novaDevolucao) {
        LocalDate hoje = LocalDate.now();

        if (novaRetirada.isBefore(hoje) || novaDevolucao.isBefore(hoje)) {
            throw new ReservaException("Datas de atualização devem ser futuras");
        }
        if (!novaDevolucao.isAfter(novaRetirada)) {
            throw new ReservaException("Data de devolução deve ser posterior à retirada");
        }

        dataRetirada = novaRetirada;
        dataDevolucao = novaDevolucao;
    }
}
```

```java
package model.exceptions;

public class ReservaException extends RuntimeException {
    public ReservaException(String msg) {
        super(msg);
    }
}
```

Uso no `main`:

```java
try {
    reserva.atualizarDatas(novaRetirada, novaDevolucao);
    System.out.println("Reserva atualizada: " + reserva.duracaoDias() + " dias");
}
catch (ReservaException e) {
    System.out.println("Erro na reserva: " + e.getMessage());
}
```

---

## 9. Exercício de fixação — Saque de conta vinculada a motorista (adaptado)

Regra: motorista tem um saldo de "vale-combustível"; não pode sacar mais que o saldo nem mais que o limite diário.

```java
package model.entities;

import model.exceptions.SaqueException;

public class ContaMotorista {

    private Integer numero;
    private String motorista;
    private Double saldo;
    private Double limiteDiario;

    public ContaMotorista(Integer numero, String motorista, Double saldo, Double limiteDiario) {
        this.numero = numero;
        this.motorista = motorista;
        this.saldo = saldo;
        this.limiteDiario = limiteDiario;
    }

    public void sacarCombustivel(Double valor) {
        if (valor > limiteDiario) {
            throw new SaqueException("Valor excede o limite diário de vale-combustível");
        }
        if (valor > saldo) {
            throw new SaqueException("Saldo insuficiente");
        }
        saldo -= valor;
    }

    public Double getSaldo() {
        return saldo;
    }
}
```

```java
package model.exceptions;

public class SaqueException extends RuntimeException {
    public SaqueException(String msg) {
        super(msg);
    }
}
```

Teste:

```java
ContaMotorista conta = new ContaMotorista(8021, "João Silva", 500.0, 300.0);

try {
    conta.sacarCombustivel(400.0);
}
catch (SaqueException e) {
    System.out.println("Erro no saque: " + e.getMessage());
}
```

Saída: `Erro no saque: Valor excede o limite diário de vale-combustível`

---

## 10. Resumo

| Conceito | Significado |
|---|---|
| `try/catch` | Bloco que tenta executar e captura erro se ocorrer |
| `finally` | Sempre executa — usar para liberar recursos (conexão, arquivo, scanner) |
| `throw` | Lança a exceção, corta a execução do método |
| `throws` | Declara que o método pode lançar, delega a decisão pra quem chama |
| `Exception` | Checked — compilador obriga tratar/propagar (ex: falha de comunicação) |
| `RuntimeException` | Unchecked — regra de negócio violada (ex: peso excedido) |
| Exceção customizada | Carrega dados específicos do domínio, deixa o erro semanticamente claro |

**Vantagens do modelo:**
- Lógica de validação fica na classe responsável (ex: `ReservaVeiculo` valida suas próprias datas).
- Código sem `if` aninhado — no momento que a exceção é lançada, a execução pula direto pro `catch`.
- Dá pra capturar por hierarquia (`catch (ReservaException e)` pega qualquer subtipo).
- A exceção pode carregar dados (peso excedente, saldo faltante) pra quem trata decidir o que fazer.