# Threads e Concorrência em Java


---

## Sumário

1. [Modelo de domínio usado nos exemplos](#1-modelo-de-domínio-usado-nos-exemplos)
2. [O que é uma Thread](#2-o-que-é-uma-thread)
3. [Criando threads: `Thread` vs `Runnable`](#3-criando-threads-thread-vs-runnable)
4. [Ciclo de vida de uma thread](#4-ciclo-de-vida-de-uma-thread)
5. [`join` e `sleep`](#5-join-e-sleep)
6. [Condição de corrida (race condition)](#6-condição-de-corrida-race-condition)
7. [`synchronized`: métodos e blocos](#7-synchronized-métodos-e-blocos)
8. [`Lock` explícito (`ReentrantLock`)](#8-lock-explícito-reentrantlock)
9. [Deadlock](#9-deadlock)
10. [Variáveis atômicas (`java.util.concurrent.atomic`)](#10-variáveis-atômicas-javautilconcurrentatomic)
11. [Coleções thread-safe](#11-coleções-thread-safe)
12. [`ExecutorService` e pool de threads](#12-executorservice-e-pool-de-threads)
13. [`Callable` e `Future`](#13-callable-e-future)
14. [`CompletableFuture`](#14-completablefuture)
15. [Virtual Threads (Java 21 — contexto)](#15-virtual-threads-java-21--contexto)
16. [Exercício resolvido](#16-exercício-resolvido)
17. [Armadilhas e boas práticas](#17-armadilhas-e-boas-práticas)
18. [Cheat sheet](#18-cheat-sheet)

---

## 1. Modelo de domínio usado nos exemplos

```java
public class Entrega {
    private String codigo;
    private String destino;
    private double pesoKg;
    private volatile String status;

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

---

## 2. O que é uma Thread

**Thread** é um fluxo independente de execução dentro de um processo. Um processo Java sempre tem pelo menos a **thread main**; outras podem ser criadas para rodar tarefas em paralelo.

**Analogia logística:** um centro de distribuição tem várias **docas de carregamento** funcionando ao mesmo tempo. Cada doca (thread) carrega um caminhão (tarefa) independentemente das outras, mas todas compartilham o **mesmo estoque** (memória do processo) — e é aí que os problemas de concorrência aparecem.

| Conceito | Analogia |
|---|---|
| Processo | O centro de distribuição inteiro |
| Thread | Uma doca de carregamento |
| Memória compartilhada | O estoque comum a todas as docas |
| Condição de corrida | Duas docas pegando a última caixa ao mesmo tempo |

---

## 3. Criando threads: `Thread` vs `Runnable`

### Estendendo `Thread`

```java
public class ThreadCarregamento extends Thread {
    private String doca;

    public ThreadCarregamento(String doca) {
        this.doca = doca;
    }

    @Override
    public void run() {
        System.out.println("Carregando caminhão na " + doca);
    }
}

ThreadCarregamento t1 = new ThreadCarregamento("Doca 1");
t1.start();   // nunca chame run() diretamente, perde a concorrência
```

### Implementando `Runnable` (preferível)

```java
Runnable tarefaCarregamento = () -> System.out.println("Carregando caminhão na Doca 2");

Thread t2 = new Thread(tarefaCarregamento);
t2.start();
```

| Abordagem | Vantagem | Desvantagem |
|---|---|---|
| `extends Thread` | Simples | Já "gasta" a única herança da classe |
| `implements Runnable` / lambda | Permite herdar de outra classe, reuso em `ExecutorService` | Nenhuma relevante |

> `start()` agenda a execução em **nova thread**. `run()` apenas chama o método como código comum, na thread atual.

---

## 4. Ciclo de vida de uma thread

```
NEW -> RUNNABLE -> (BLOCKED / WAITING / TIMED_WAITING) -> TERMINATED
```

| Estado | Significado |
|---|---|
| `NEW` | Criada, `start()` ainda não chamado |
| `RUNNABLE` | Pronta ou executando |
| `BLOCKED` | Esperando um lock (`synchronized`) |
| `WAITING` / `TIMED_WAITING` | Esperando notificação ou tempo (`sleep`, `join`, `wait`) |
| `TERMINATED` | `run()` terminou |

```java
Thread t = new Thread(() -> {});
System.out.println(t.getState());   // NEW
t.start();
System.out.println(t.getState());   // RUNNABLE
```

---

## 5. `join` e `sleep`

```java
Thread t1 = new Thread(() -> {
    System.out.println("Carregando Doca 1...");
    try { Thread.sleep(1000); } catch (InterruptedException e) { }
    System.out.println("Doca 1 finalizada.");
});

t1.start();
t1.join();   // thread main espera t1 terminar antes de continuar
System.out.println("Todos os caminhões liberados.");
```

- `sleep(ms)` pausa a thread atual pelo tempo informado.
- `join()` faz a thread chamadora **esperar** a thread-alvo terminar.
- Ambos lançam `InterruptedException` (checada) — tratamento obrigatório.

---

## 6. Condição de corrida (race condition)

### Problema

Duas docas tentam baixar o **mesmo estoque** compartilhado ao mesmo tempo.

```java
public class Estoque {
    private int quantidade = 100;

    public void retirar(int qtd) {
        if (quantidade >= qtd) {
            quantidade -= qtd;   // operação NÃO atômica: ler + subtrair + gravar
        }
    }

    public int getQuantidade() { return quantidade; }
}
```

```java
Estoque estoque = new Estoque();

Runnable retirada = () -> {
    for (int i = 0; i < 1000; i++) {
        estoque.retirar(1);
    }
};

Thread doca1 = new Thread(retirada);
Thread doca2 = new Thread(retirada);
doca1.start();
doca2.start();
doca1.join();
doca2.join();

System.out.println(estoque.getQuantidade());
// Esperado: 0 (duas docas de 1000 cada, estoque de 100 só permite as primeiras 100)
// Na prática: valor inconsistente e variável entre execuções
```

As duas threads leem `quantidade`, decidem que pode retirar, e **ambas** subtraem — gerando valores inconsistentes ou até negativos. Isso é uma **condição de corrida**.

---

## 7. `synchronized`: métodos e blocos

### Método sincronizado

```java
public class Estoque {
    private int quantidade = 100;

    public synchronized void retirar(int qtd) {
        if (quantidade >= qtd) {
            quantidade -= qtd;
        }
    }

    public int getQuantidade() { return quantidade; }
}
```

Agora só **uma thread por vez** executa `retirar` para uma mesma instância de `Estoque`. O resultado passa a ser sempre `0`.

### Bloco sincronizado (sincroniza só o trecho crítico)

```java
public class Estoque {
    private int quantidade = 100;
    private final Object lock = new Object();

    public void retirar(int qtd) {
        synchronized (lock) {
            if (quantidade >= qtd) {
                quantidade -= qtd;
            }
        }
    }
}
```

> Prefira sincronizar o **menor trecho possível** (bloco) a um método inteiro, para não travar operações que não precisam de exclusão mútua.

---

## 8. `Lock` explícito (`ReentrantLock`)

Mais flexível que `synchronized`: permite tentar o lock com timeout, verificar se está travado, e liberar em `finally`.

```java
import java.util.concurrent.locks.ReentrantLock;

public class Estoque {
    private int quantidade = 100;
    private final ReentrantLock lock = new ReentrantLock();

    public void retirar(int qtd) {
        lock.lock();
        try {
            if (quantidade >= qtd) {
                quantidade -= qtd;
            }
        } finally {
            lock.unlock();   // sempre no finally, senão o lock trava para sempre em caso de exceção
        }
    }
}
```

```java
if (lock.tryLock()) {
    try {
        // conseguiu o lock sem bloquear
    } finally {
        lock.unlock();
    }
} else {
    System.out.println("Doca ocupada, tentando depois.");
}
```

---

## 9. Deadlock

Ocorre quando duas threads esperam **mutuamente** um recurso que a outra detém.

```java
Object docaA = new Object();
Object docaB = new Object();

Thread t1 = new Thread(() -> {
    synchronized (docaA) {
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        synchronized (docaB) {
            System.out.println("t1 carregou nas duas docas");
        }
    }
});

Thread t2 = new Thread(() -> {
    synchronized (docaB) {
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        synchronized (docaA) {
            System.out.println("t2 carregou nas duas docas");
        }
    }
});

t1.start();
t2.start();
// t1 trava esperando docaB; t2 trava esperando docaA -> DEADLOCK
```

**Solução:** sempre adquirir os locks **na mesma ordem**, em todas as threads (ex.: sempre `docaA` antes de `docaB`).

---

## 10. Variáveis atômicas (`java.util.concurrent.atomic`)

Para contadores simples, `AtomicInteger`/`AtomicLong` evitam `synchronized` com melhor performance.

```java
import java.util.concurrent.atomic.AtomicInteger;

public class ContadorEntregas {
    private AtomicInteger entregasFinalizadas = new AtomicInteger(0);

    public void finalizarEntrega() {
        entregasFinalizadas.incrementAndGet();   // operação atômica
    }

    public int getTotal() {
        return entregasFinalizadas.get();
    }
}
```

| Método | Faz |
|---|---|
| `incrementAndGet()` / `getAndIncrement()` | +1 (atômico) |
| `addAndGet(n)` | soma `n` |
| `compareAndSet(esperado, novo)` | troca só se o valor atual for o esperado |

---

## 11. Coleções thread-safe

```java
import java.util.concurrent.*;

// Map seguro para concorrência, melhor que Hashtable/synchronizedMap
ConcurrentHashMap<String, Integer> pacotesPorCidade = new ConcurrentHashMap<>();
pacotesPorCidade.put("Lorena", 10);
pacotesPorCidade.merge("Lorena", 1, Integer::sum);   // incremento atômico

// Fila segura para produtor/consumidor
BlockingQueue<String> filaDeRotas = new LinkedBlockingQueue<>();
filaDeRotas.put("Rota-Lorena");          // bloqueia se a fila estiver cheia (com capacidade limitada)
String proximaRota = filaDeRotas.take(); // bloqueia se estiver vazia
```

> Evite `Collections.synchronizedList/Map` em código novo — `java.util.concurrent` oferece estruturas mais eficientes e com operações atômicas compostas (`merge`, `putIfAbsent`, `computeIfAbsent`).

---

## 12. `ExecutorService` e pool de threads

Criar uma `Thread` para cada tarefa é caro. O **pool de threads** reaproveita um número fixo de threads para muitas tarefas.

```java
import java.util.concurrent.*;

ExecutorService pool = Executors.newFixedThreadPool(3);   // 3 docas fixas

List<String> entregas = List.of("E001", "E002", "E003", "E004", "E005");

for (String codigo : entregas) {
    pool.submit(() -> {
        System.out.println(Thread.currentThread().getName() + " processando " + codigo);
    });
}

pool.shutdown();                                   // não aceita novas tarefas
pool.awaitTermination(5, TimeUnit.SECONDS);         // espera as pendentes terminarem
```

| Fábrica | Uso |
|---|---|
| `newFixedThreadPool(n)` | N threads fixas, boa para carga previsível |
| `newCachedThreadPool()` | Cria/reaproveita threads conforme demanda |
| `newSingleThreadExecutor()` | Uma thread só, tarefas em fila, ordem garantida |
| `newScheduledThreadPool(n)` | Tarefas agendadas/periódicas |

---

## 13. `Callable` e `Future`

`Runnable` não devolve valor. `Callable<T>` devolve e pode lançar exceção checada; o resultado chega via `Future<T>`.

```java
ExecutorService pool = Executors.newFixedThreadPool(2);

Callable<Double> calcularFrete = () -> {
    Thread.sleep(200);
    return 450.0 * 1.1;
};

Future<Double> futuro = pool.submit(calcularFrete);

System.out.println("Calculando frete em paralelo...");
double freteComImposto = futuro.get();   // bloqueia até o resultado ficar pronto
System.out.println("Frete final: " + freteComImposto);

pool.shutdown();
```

- `future.get()` bloqueia a thread chamadora até o resultado existir (ou aceita timeout: `get(2, TimeUnit.SECONDS)`).
- `future.isDone()` verifica sem bloquear.
- `future.cancel(true)` tenta cancelar a tarefa.

---

## 14. `CompletableFuture`

Permite compor tarefas assíncronas sem bloquear manualmente com `get()`.

```java
import java.util.concurrent.CompletableFuture;

CompletableFuture<Double> freteBase = CompletableFuture.supplyAsync(() -> {
    // simula cálculo de frete em outra thread
    return 450.0;
});

CompletableFuture<String> resultado = freteBase
    .thenApply(frete -> frete * 1.1)                       // transforma o valor (ainda assíncrono)
    .thenApply(freteComImposto -> "Frete final: R$ " + freteComImposto)
    .exceptionally(ex -> "Erro ao calcular frete: " + ex.getMessage());

System.out.println(resultado.join());   // espera e devolve o resultado (ou lança se não tratado)
```

```java
// Combinando dois cálculos independentes
CompletableFuture<Double> frete = CompletableFuture.supplyAsync(() -> 450.0);
CompletableFuture<Double> seguro = CompletableFuture.supplyAsync(() -> 30.0);

CompletableFuture<Double> total = frete.thenCombine(seguro, (f, s) -> f + s);
System.out.println(total.join());   // 480.0
```

---

## 15. Virtual Threads (Java 21 — contexto)

A partir do Java 21, existem **virtual threads** (Project Loom): threads leves gerenciadas pela JVM, muito mais baratas que threads tradicionais do sistema operacional, ideais para aplicações com muitas tarefas bloqueantes de I/O (ex.: milhares de chamadas simultâneas a APIs de rastreamento de entregas).

```java
// Disponível apenas em Java 21+, incluído aqui apenas como contexto evolutivo
try (ExecutorService pool = Executors.newVirtualThreadPerTaskExecutor()) {
    pool.submit(() -> System.out.println("Rastreando entrega em virtual thread"));
}
```

> Em **Java 17**, use `ExecutorService` com pools tradicionais (seção 12) — virtual threads não estão disponíveis nessa versão.

---

## 16. Exercício resolvido

**Problema:** processar o cálculo de frete de uma lista de entregas em paralelo, usando um pool de threads, e somar o total de forma thread-safe.

```java
import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicReference;

public class Program {
    public static void main(String[] args) throws InterruptedException {
        List<Entrega> entregas = List.of(
            new Entrega("E001", "Lorena", 15.0, "PENDENTE"),
            new Entrega("E002", "Taubaté", 120.0, "PENDENTE"),
            new Entrega("E003", "Guará", 60.0, "PENDENTE")
        );

        ExecutorService pool = Executors.newFixedThreadPool(3);
        AtomicReference<Double> totalFrete = new AtomicReference<>(0.0);
        CountDownLatch latch = new CountDownLatch(entregas.size());

        for (Entrega e : entregas) {
            pool.submit(() -> {
                try {
                    double frete = e.getPesoKg() * 2.5;
                    totalFrete.updateAndGet(valor -> valor + frete);
                    e.setStatus("CALCULADO");
                    System.out.println(Thread.currentThread().getName() + " calculou " + e);
                } finally {
                    latch.countDown();
                }
            });
        }

        latch.await();   // espera as 3 tarefas terminarem
        pool.shutdown();

        System.out.println("Frete total do lote: R$ " + totalFrete.get());
    }
}
```

**Saída esperada (ordem das threads pode variar):**

```
pool-1-thread-1 calculou E001 -> Lorena (15.0kg) [CALCULADO]
pool-1-thread-2 calculou E002 -> Taubaté (120.0kg) [CALCULADO]
pool-1-thread-3 calculou E003 -> Guará (60.0kg) [CALCULADO]
Frete total do lote: R$ 487.5
```

> `CountDownLatch` é usado aqui para esperar múltiplas tarefas sem precisar guardar e chamar `join()`/`get()` em cada uma individualmente.

---

## 17. Armadilhas e boas práticas

| Armadilha | Explicação | Solução |
|---|---|---|
| Chamar `run()` em vez de `start()` | Executa na thread atual, sem concorrência real | Sempre `start()` |
| Esquecer `synchronized`/`Lock` em estado compartilhado | Condição de corrida, resultados inconsistentes | Proteger a seção crítica |
| `unlock()` fora do `finally` | Lock nunca liberado se houver exceção | Sempre liberar em `finally` |
| Ordem de locks diferente entre threads | Deadlock | Padronizar a ordem de aquisição |
| Criar uma `Thread` por tarefa em alto volume | Alto custo de memória/contexto | Usar `ExecutorService` com pool |
| Esquecer `pool.shutdown()` | Threads do pool nunca terminam, JVM não finaliza | Sempre `shutdown()`/`awaitTermination` |
| `future.get()` sem timeout | Pode bloquear para sempre se a tarefa travar | `get(timeout, unidade)` |
| Campo mutável compartilhado sem `volatile`/sincronização | Uma thread pode nunca ver a atualização de outra | `volatile` para visibilidade, lock para atomicidade |
| Ignorar `InterruptedException` | Pode mascarar cancelamento/encerramento do programa | Tratar ou repropagar (`Thread.currentThread().interrupt()`) |

### Regras de ouro

1. **Minimize estado compartilhado mutável** — menos compartilhamento, menos sincronização necessária.
2. **Prefira estruturas de `java.util.concurrent`** a sincronizar manualmente coleções comuns.
3. **Use `ExecutorService`**, não gerencie `Thread` manualmente em aplicações reais.
4. **Sempre libere locks em `finally`.**
5. **Teste com carga concorrente real** — bugs de concorrência raramente aparecem em execução única.

---

## 18. Cheat sheet

```java
// Criar e iniciar
new Thread(() -> tarefa()).start();

// Esperar terminar
thread.join();

// Seção crítica
synchronized (lock) { /* ... */ }

// Lock explícito
lock.lock();
try { /* ... */ } finally { lock.unlock(); }

// Pool de threads
ExecutorService pool = Executors.newFixedThreadPool(n);
pool.submit(() -> tarefa());
pool.shutdown();

// Tarefa com retorno
Future<T> f = pool.submit(callable);
T resultado = f.get();

// Assíncrono encadeado
CompletableFuture.supplyAsync(() -> valor)
    .thenApply(v -> transforma(v))
    .thenAccept(v -> usa(v));

// Contador atômico
AtomicInteger contador = new AtomicInteger(0);
contador.incrementAndGet();
```

### Referências

- Docs Java: [`Thread`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Thread.html), [`java.util.concurrent`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/package-summary.html), [`ExecutorService`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/ExecutorService.html), [`CompletableFuture`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/CompletableFuture.html)
- Livro de referência: **Java Concurrency in Practice** (Brian Goetz)
