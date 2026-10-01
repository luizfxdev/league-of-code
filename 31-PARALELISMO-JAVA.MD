# Paralelismo em Java


---

## Sumário

1. [Modelo de domínio usado nos exemplos](#1-modelo-de-domínio-usado-nos-exemplos)
2. [Concorrência vs Paralelismo](#2-concorrência-vs-paralelismo)
3. [`parallelStream()`](#3-parallelstream)
4. [Quando paralelizar compensa (e quando não)](#4-quando-paralelizar-compensa-e-quando-não)
5. [`ForkJoinPool` e work-stealing](#5-forkjoinpool-e-work-stealing)
6. [`RecursiveTask` e divisão de trabalho (fork/join manual)](#6-recursivetask-e-divisão-de-trabalho-forkjoin-manual)
7. [Paralelismo com `ExecutorService`](#7-paralelismo-com-executorservice)
8. [Idempotência](#8-idempotência)
9. [Idempotência na prática: retry seguro](#9-idempotência-na-prática-retry-seguro)
10. [Exercício resolvido](#10-exercício-resolvido)
11. [Armadilhas e boas práticas](#11-armadilhas-e-boas-práticas)
12. [Cheat sheet](#12-cheat-sheet)

---

## 1. Modelo de domínio usado nos exemplos

```java
public class Entrega {
    private String codigo;
    private String destino;
    private double pesoKg;
    private String status;

    public Entrega(String codigo, String destino, double pesoKg, String status) {
        this.codigo = codigo;
        this.destino = destino;
        this.pesoKg = pesoKg;
        this.status = status;
    }

    public String getCodigo() { return codigo; }
    public String getDestino() { return destino; }
    public double getPesoKg() { return pesoKg; }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }

    @Override
    public String toString() {
        return codigo + " -> " + destino + " (" + pesoKg + "kg) [" + status + "]";
    }
}
```

Lista base (grande, para simular carga de processamento):

```java
List<Entrega> entregas = new ArrayList<>();
for (int i = 1; i <= 1_000_000; i++) {
    entregas.add(new Entrega("E" + i, "Cidade" + (i % 50), (i % 200) + 1.0, "PENDENTE"));
}
```

---

## 2. Concorrência vs Paralelismo

Os termos são próximos, mas não são sinônimos:

| Conceito | Significado | Analogia logística |
|---|---|---|
| **Concorrência** | Várias tarefas progridem no mesmo período, mas não necessariamente ao mesmo instante (podem se alternar em 1 núcleo) | Um único atendente alternando entre 3 clientes na janela de atendimento |
| **Paralelismo** | Várias tarefas executam **literalmente ao mesmo tempo**, em núcleos de CPU diferentes | 3 docas de carregamento, cada uma com seu próprio operador, carregando 3 caminhões simultaneamente |

Paralelismo **exige** múltiplos núcleos reais; concorrência não.

```java
// Concorrente (pode rodar em 1 núcleo, alternando) e também paralelo se houver núcleos livres
Thread t1 = new Thread(() -> processar(entregas.subList(0, 500_000)));
Thread t2 = new Thread(() -> processar(entregas.subList(500_000, 1_000_000)));
t1.start();
t2.start();
```

> Threads e `ExecutorService` (vistos no tópico anterior) dão **concorrência**. Paralelismo de verdade depende de o hardware ter núcleos disponíveis e a carga de trabalho ser dividida entre eles — é isso que este tópico aborda.

---

## 3. `parallelStream()`

Forma mais simples de paralelizar processamento de coleções em Java, usando o `ForkJoinPool` comum internamente.

```java
// Sequencial
double totalSequencial = entregas.stream()
    .mapToDouble(Entrega::getPesoKg)
    .sum();

// Paralelo
double totalParalelo = entregas.parallelStream()
    .mapToDouble(Entrega::getPesoKg)
    .sum();
```

O pipeline é **dividido em pedaços**, cada pedaço processado em uma thread diferente do pool, e os resultados parciais são combinados no final.

```java
// Contagem de entregas pesadas (>100kg) por cidade, em paralelo
Map<String, Long> pesadasPorCidade = entregas.parallelStream()
    .filter(e -> e.getPesoKg() > 100)
    .collect(Collectors.groupingBy(Entrega::getDestino, Collectors.counting()));
```

> `Collectors.groupingBy` com `parallelStream()` precisa de um `Map` **concorrente** internamente (o JDK já cuida disso ao usar `Collectors.groupingByConcurrent` quando apropriado) — veja a seção 11 para a armadilha de usar coleções não thread-safe manualmente dentro do pipeline.

---

## 4. Quando paralelizar compensa (e quando não)

| Cenário | Paralelizar? | Por quê |
|---|---|---|
| Volume grande (centenas de milhares+) de dados, operação CPU-bound (cálculo pesado) | **Sim** | Ganho real ao dividir trabalho entre núcleos |
| Volume pequeno (dezenas, centenas) | **Não** | Overhead de dividir/combinar supera o ganho |
| Operação I/O-bound (chamada de API, leitura de arquivo, banco) | **Não** (use assíncrono, não `parallelStream`) | Threads ficam bloqueadas esperando I/O, não há trabalho de CPU para paralelizar |
| Fonte é `LinkedList` ou estrutura sem acesso indexado eficiente | **Não** | Divisão (`spliterator`) é cara nessas estruturas |
| Operação com efeitos colaterais/estado compartilhado mutável | **Não**, a menos que sincronizado | Risco de condição de corrida (ver [[30-THREADS-CONCORRENCIA-JAVA]]) |

```java
// Exemplo de MEDIÇÃO antes de decidir (nunca assuma, meça)
long inicio = System.nanoTime();
double r1 = entregas.stream().mapToDouble(Entrega::getPesoKg).sum();
long tempoSequencial = System.nanoTime() - inicio;

inicio = System.nanoTime();
double r2 = entregas.parallelStream().mapToDouble(Entrega::getPesoKg).sum();
long tempoParalelo = System.nanoTime() - inicio;

System.out.println("Sequencial: " + tempoSequencial + "ns | Paralelo: " + tempoParalelo + "ns");
```

> Regra prática: para listas pequenas (menos de ~10.000 elementos) com operações simples, o `stream()` sequencial costuma ser **mais rápido** que `parallelStream()`.

---

## 5. `ForkJoinPool` e work-stealing

`parallelStream()` usa, por padrão, o **`ForkJoinPool.commonPool()`**, compartilhado por toda a JVM. Ele implementa **work-stealing**: threads ociosas "roubam" tarefas da fila de threads ocupadas, equilibrando a carga automaticamente.

```java
import java.util.concurrent.ForkJoinPool;

System.out.println(ForkJoinPool.commonPool().getParallelism());
// Por padrão: número de núcleos disponíveis - 1
```

```java
// Pool dedicado, isolado do commonPool (evita disputar threads com outras partes do sistema)
ForkJoinPool poolDedicado = new ForkJoinPool(4);

double total = poolDedicado.submit(() ->
    entregas.parallelStream().mapToDouble(Entrega::getPesoKg).sum()
).get();

poolDedicado.shutdown();
```

> Em aplicações de produção com múltiplos módulos usando `parallelStream()`, considere um `ForkJoinPool` dedicado para tarefas críticas — assim elas não competem pelo `commonPool` com outras partes do sistema.

---

## 6. `RecursiveTask` e divisão de trabalho (fork/join manual)

Para controle fino sobre como o trabalho é dividido (em vez de deixar o `parallelStream` decidir), use `RecursiveTask<T>` diretamente.

```java
import java.util.concurrent.RecursiveTask;
import java.util.concurrent.ForkJoinPool;
import java.util.List;

public class SomaPesoTask extends RecursiveTask<Double> {
    private static final int LIMITE = 10_000;
    private final List<Entrega> lista;

    public SomaPesoTask(List<Entrega> lista) {
        this.lista = lista;
    }

    @Override
    protected Double compute() {
        if (lista.size() <= LIMITE) {
            double soma = 0;
            for (Entrega e : lista) soma += e.getPesoKg();
            return soma;
        }

        int meio = lista.size() / 2;
        SomaPesoTask esquerda = new SomaPesoTask(lista.subList(0, meio));
        SomaPesoTask direita  = new SomaPesoTask(lista.subList(meio, lista.size()));

        esquerda.fork();                      // executa em outra thread
        double resultadoDireita = direita.compute();   // computa na thread atual
        double resultadoEsquerda = esquerda.join();    // espera o resultado da outra

        return resultadoEsquerda + resultadoDireita;
    }
}
```

```java
ForkJoinPool pool = new ForkJoinPool();
double total = pool.invoke(new SomaPesoTask(entregas));
System.out.println("Peso total: " + total);
pool.shutdown();
```

- **Fork**: divide a tarefa em subtarefas menores.
- **Join**: espera e combina os resultados.
- Útil quando a lógica de divisão não é tão simples quanto um `map`/`reduce` de stream (ex.: algoritmos recursivos como merge sort paralelo, busca em árvore de rotas).

---

## 7. Paralelismo com `ExecutorService`

Quando o trabalho não é homogêneo (tarefas diferentes, não "a mesma operação em cada elemento"), use um pool fixo dimensionado para o número de núcleos.

```java
import java.util.concurrent.*;

int nucleos = Runtime.getRuntime().availableProcessors();
ExecutorService pool = Executors.newFixedThreadPool(nucleos);

List<List<Entrega>> lotes = particionar(entregas, nucleos);   // divide a lista em N partes

List<Future<Double>> futuros = new ArrayList<>();
for (List<Entrega> lote : lotes) {
    futuros.add(pool.submit(() ->
        lote.stream().mapToDouble(Entrega::getPesoKg).sum()
    ));
}

double totalGeral = 0;
for (Future<Double> f : futuros) {
    totalGeral += f.get();   // combina os resultados parciais
}

pool.shutdown();
```

> `Runtime.getRuntime().availableProcessors()` informa quantos núcleos a JVM vê — use-o para dimensionar pools de tarefas CPU-bound, em vez de um número fixo arbitrário.

---

## 8. Idempotência

**Idempotência** não é um recurso da linguagem Java — é uma propriedade de **operações/APIs**: uma operação é idempotente quando **executá-la várias vezes produz o mesmo efeito final** que executá-la uma única vez.

**Analogia logística:** registrar a baixa de uma entrega como "ENTREGUE" é idempotente — não importa quantas vezes o sistema reenvie essa confirmação (por retry de rede, por exemplo), o status final continua "ENTREGUE". Já **incrementar** um contador de entregas (`total++`) a cada confirmação **não é** idempotente — reenviar a mesma confirmação duas vezes contaria a entrega duas vezes.

```java
// NÃO idempotente: reexecutar 2x gera efeito duplicado
public void confirmarEntrega(Entrega entrega) {
    totalEntregasConcluidas++;              // soma de novo a cada chamada
    entrega.setStatus("ENTREGUE");
}

// Idempotente: reexecutar N vezes tem o mesmo efeito que executar 1 vez
public void confirmarEntrega(Entrega entrega) {
    if (!"ENTREGUE".equals(entrega.getStatus())) {
        totalEntregasConcluidas++;
        entrega.setStatus("ENTREGUE");
    }
}
```

| Operação | Idempotente? | Por quê |
|---|---|---|
| `entrega.setStatus("ENTREGUE")` | Sim | Atribuição direta sempre resulta no mesmo estado final |
| `estoque.retirar(5)` chamado 2x | Não | Retira 10 no total, não 5 |
| `PUT /entregas/123` (substitui o recurso inteiro) | Sim (por convenção REST) | Reenviar o mesmo PUT deixa o recurso igual |
| `POST /entregas` (cria um novo recurso) | Não (por padrão) | Reenviar cria uma segunda entrega duplicada |

> Relação com [[30-THREADS-CONCORRENCIA-JAVA]]: operações idempotentes facilitam **retry seguro** em cenários concorrentes/distribuídos — se uma thread ou serviço falha e tenta de novo, não há risco de duplicar efeito.

---

## 9. Idempotência na prática: retry seguro

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.Set;

public class ProcessadorEntregas {
    private final Set<String> processadas = ConcurrentHashMap.newKeySet();

    public void processar(Entrega entrega) {
        // chave de idempotência: evita reprocessar o mesmo código mesmo sob retries concorrentes
        if (!processadas.add(entrega.getCodigo())) {
            System.out.println(entrega.getCodigo() + " já processado, ignorando repetição.");
            return;
        }
        entrega.setStatus("PROCESSADO");
        System.out.println("Processado: " + entrega);
    }
}
```

```java
ProcessadorEntregas processador = new ProcessadorEntregas();
Entrega e = new Entrega("E001", "Lorena", 20.0, "PENDENTE");

ExecutorService pool = Executors.newFixedThreadPool(4);
for (int i = 0; i < 4; i++) {
    pool.submit(() -> processador.processar(e));   // simula 4 retries concorrentes do mesmo evento
}
pool.shutdown();
// Resultado: processado exatamente 1 vez, mesmo com 4 chamadas concorrentes
```

`Set.add()` do `ConcurrentHashMap.newKeySet()` é atômico: só a **primeira** chamada com um dado código retorna `true`; as demais (mesmo concorrentes) retornam `false` e são ignoradas.

---

## 10. Exercício resolvido

**Problema:** calcular o frete total de 1 milhão de entregas usando paralelismo, comparando tempo sequencial vs paralelo, e garantir que o registro de "lote processado" seja idempotente mesmo se o método for chamado mais de uma vez para o mesmo lote.

```java
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.ConcurrentHashMap;

public class Program {
    private static final Set<String> lotesProcessados = ConcurrentHashMap.newKeySet();

    public static void main(String[] args) {
        List<Entrega> entregas = gerarEntregas(1_000_000);

        long t0 = System.nanoTime();
        double totalSequencial = entregas.stream()
            .mapToDouble(e -> e.getPesoKg() * 2.5)
            .sum();
        long tempoSequencial = System.nanoTime() - t0;

        long t1 = System.nanoTime();
        double totalParalelo = entregas.parallelStream()
            .mapToDouble(e -> e.getPesoKg() * 2.5)
            .sum();
        long tempoParalelo = System.nanoTime() - t1;

        System.out.printf("Sequencial: %.2fms | Paralelo: %.2fms%n",
            tempoSequencial / 1_000_000.0, tempoParalelo / 1_000_000.0);

        registrarLoteProcessado("LOTE-2025-10");
        registrarLoteProcessado("LOTE-2025-10");   // chamada duplicada (ex.: retry), não deve duplicar efeito
    }

    private static void registrarLoteProcessado(String idLote) {
        if (lotesProcessados.add(idLote)) {
            System.out.println("Lote " + idLote + " registrado como processado.");
        } else {
            System.out.println("Lote " + idLote + " já estava registrado, ignorando.");
        }
    }

    private static List<Entrega> gerarEntregas(int n) {
        List<Entrega> lista = new ArrayList<>(n);
        for (int i = 1; i <= n; i++) {
            lista.add(new Entrega("E" + i, "Cidade" + (i % 50), (i % 200) + 1.0, "PENDENTE"));
        }
        return lista;
    }
}
```

**Saída esperada (tempos variam por máquina):**

```
Sequencial: 42.10ms | Paralelo: 11.35ms
Lote LOTE-2025-10 registrado como processado.
Lote LOTE-2025-10 já estava registrado, ignorando.
```

---

## 11. Armadilhas e boas práticas

| Armadilha | Explicação | Solução |
|---|---|---|
| Usar `parallelStream()` em listas pequenas | Overhead de divisão supera o ganho | Medir antes; para poucos elementos, usar `stream()` sequencial |
| `parallelStream()` em operação I/O-bound | Threads do `ForkJoinPool` ficam bloqueadas esperando I/O, prejudicando o pool inteiro (até outras partes do sistema) | Usar `ExecutorService`/`CompletableFuture` dedicado para I/O, não `parallelStream` |
| Coletar em `ArrayList` comum dentro de `forEach` paralelo | Condição de corrida, itens perdidos ou `ArrayIndexOutOfBoundsException` | Usar `collect(Collectors.toList())` (thread-safe internamente) ou coleção concorrente |
| Assumir que paralelismo preserva ordem | `parallelStream().forEach` não garante ordem de execução | Usar `forEachOrdered` se a ordem importa (com custo de desempenho) |
| Tratar "concorrente" como sinônimo de "paralelo" | Confunde intenção e expectativa de ganho de desempenho | Concorrência é sobre estrutura; paralelismo é sobre execução simultânea real |
| Esquecer idempotência em operações reexecutáveis (retry, mensageria, múltiplas threads chamando o mesmo evento) | Efeitos duplicados (cobrança duplicada, contagem duplicada, entrega duplicada) | Chave de idempotência (ex.: `codigo` único) + verificação atômica antes de aplicar o efeito |
| Esgotar o `ForkJoinPool.commonPool()` com tarefas bloqueantes | Afeta todo `parallelStream()` da aplicação, inclusive de outros módulos | Pool dedicado para tarefas longas/bloqueantes |

### Regras de ouro

1. **Meça antes de paralelizar** — intuição sobre ganho de desempenho costuma estar errada.
2. **Paralelismo é para CPU-bound**; I/O-bound pede assincronia, não paralelismo de stream.
3. **Projete operações críticas para serem idempotentes**, especialmente as que podem ser reexecutadas por retry, mensageria ou concorrência.
4. **Evite estado mutável compartilhado** dentro de pipelines paralelos.
5. **Isole pools dedicados** para cargas de trabalho sensíveis, não dependa só do `commonPool`.

---

## 12. Cheat sheet

```java
// Paralelo simples via stream
lista.parallelStream()
     .filter(...)
     .map(...)
     .collect(Collectors.toList());

// Fork/Join manual
ForkJoinPool pool = new ForkJoinPool();
pool.invoke(new MinhaRecursiveTask(lista));

// Núcleos disponíveis
int nucleos = Runtime.getRuntime().availableProcessors();

// Idempotência com chave única (thread-safe)
Set<String> processados = ConcurrentHashMap.newKeySet();
if (processados.add(chave)) {
    // primeira vez: aplica o efeito
}
```

| Cenário | Ferramenta recomendada |
|---|---|
| Transformar/filtrar coleção grande, CPU-bound | `parallelStream()` |
| Divisão de trabalho customizada (ex.: algoritmo recursivo) | `ForkJoinPool` + `RecursiveTask` |
| Tarefas heterogêneas, controle fino | `ExecutorService` + `Future` |
| Evitar efeito duplicado em reexecução/retry | Chave de idempotência + estrutura atômica |

### Referências

- Docs Java: [`Stream#parallel`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html#parallel()), [`ForkJoinPool`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/ForkJoinPool.html), [`RecursiveTask`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/RecursiveTask.html)
- Ver também: [[30-THREADS-CONCORRENCIA-JAVA]] (base de concorrência usada aqui)
