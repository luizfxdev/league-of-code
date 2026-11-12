# Programação Síncrona e Assíncrona em Java

---

## Sumário

1. [Conceitos Fundamentais](#1-conceitos-fundamentais)
2. [Bloqueio e Não-Bloqueio](#2-bloqueio-e-não-bloqueio)
3. [Callbacks](#3-callbacks)
4. [`Future` e `Callable`](#4-future-e-callable)
5. [`CompletableFuture`](#5-completablefuture)
6. [Composição e Encadeamento de Operações Assíncronas](#6-composição-e-encadeamento-de-operações-assíncronas)
7. [Tratamento de Erros em Operações Assíncronas](#7-tratamento-de-erros-em-operações-assíncronas)
8. [Programação Reativa: Streams e Observables](#8-programação-reativa-streams-e-observables)
9. [Virtual Threads (Java 21+): Simplificando a Assincronia](#9-virtual-threads-java-21-simplificando-a-assincronia)
10. [Síncrono vs Assíncrono: Quando Usar](#10-síncrono-vs-assíncrono-quando-usar)
11. [Armadilhas e Boas Práticas](#11-armadilhas-e-boas-práticas)
12. [Exercício Resolvido](#12-exercício-resolvido)
13. [Cheat Sheet](#13-cheat-sheet)

---

## 1. Conceitos Fundamentais

### Execução Síncrona

**Síncrono** significa que operações ocorrem **sequencialmente**, uma depois da outra. A thread chamadora **bloqueia** (aguarda) até que a operação termine e o resultado esteja disponível.

```
Tempo: |--Op1--|--Op2--|--Op3--|
       início   fim     início  fim
```

```java
public class SincronoExemplo {
    public static void main(String[] args) {
        System.out.println("Início");
        
        // Busca dados do banco — bloqueia até retorno
        List<String> pedidos = buscarPedidos();
        System.out.println("Pedidos: " + pedidos);
        
        System.out.println("Fim");  // Só executa depois de buscarPedidos() terminar
    }
    
    static List<String> buscarPedidos() {
        try {
            Thread.sleep(2000);  // simula I/O lento
            return List.of("PED001", "PED002");
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}

// Saída:
// Início
// (espera 2 segundos...)
// Pedidos: [PED001, PED002]
// Fim
```

**Vantagem:** código linear e fácil de entender.  
**Desvantagem:** se uma operação lenta ocorrer, toda a thread fica bloqueada, desperdiçando recursos.

---

### Execução Assíncrona

**Assíncrono** significa que a operação é iniciada, mas a thread **não bloqueia** esperando o resultado. O resultado chega **posteriormente**, via callback, promise ou outro mecanismo de notificação.

```
Tempo: |--Op1----|
       Inicia    (resultado chega depois, em outra thread)
       |--Op2--(em paralelo)
       |--Op3--(em paralelo)
```

```java
public class AsincronoExemplo {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Início");
        
        // Inicia busca de forma assíncrona, sem bloquear
        buscarPedidosAsync((pedidos) -> {
            System.out.println("Pedidos: " + pedidos);
        });
        
        System.out.println("Continuando enquanto busca ocorre em paralelo...");
        Thread.sleep(3000);  // dá tempo para a operação assíncrona terminar
        System.out.println("Fim");
    }
    
    static void buscarPedidosAsync(Consumer<List<String>> callback) {
        new Thread(() -> {
            try {
                Thread.sleep(2000);  // simula I/O lento
                callback.accept(List.of("PED001", "PED002"));
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).start();
    }
}

// Saída (imediata):
// Início
// Continuando enquanto busca ocorre em paralelo...
// (após 2 segundos)
// Pedidos: [PED001, PED002]
// Fim
```

**Vantagem:** thread não fica bloqueada, melhor aproveitamento de recursos.  
**Desvantagem:** código mais complexo, difícil de rastrear fluxo de execução.

| Aspecto | Síncrono | Assíncrono |
|---|---|---|
| **Bloqueio** | Sim, aguarda resultado | Não, continua imediatamente |
| **Complexidade** | Simples, linear | Complexa, callbacks/promises |
| **Uso de recursos** | Ineficiente em I/O intensivo | Eficiente, economiza threads |
| **Rastreamento** | Fácil (stack trace direto) | Difícil (stack traces fragmentados) |

---

## 2. Bloqueio e Não-Bloqueio

### Operações Bloqueantes

Operações de **I/O** (ler arquivo, chamar API, banco de dados) são naturalmente bloqueantes: enquanto aguarda resposta do servidor, a thread fica parada.

```java
public void processarPedidoSync(String id) throws IOException {
    String resposta = buscarPedidoBD(id);  // BLOQUEIA aqui, esperando BD
    System.out.println("Pedido: " + resposta);
}

String buscarPedidoBD(String id) throws IOException {
    Thread.sleep(5000);  // simula latência de rede/BD
    return "Pedido-" + id;
}
```

Se temos **100 requisições simultâneas**, precisaríamos de **100 threads** — um peso para o sistema operacional.

### Operações Não-Bloqueantes (Assíncronas)

A operação é disparada, e a thread retorna imediatamente. O resultado chega via callback/promise posteriormente.

```java
public void processarPedidoAsync(String id) {
    buscarPedidoBDAsync(id, (resposta) -> {
        System.out.println("Pedido: " + resposta);
    });
    
    System.out.println("Não bloqueou, continuamos aqui!");
}

void buscarPedidoBDAsync(String id, Consumer<String> onSuccess) {
    Executors.newFixedThreadPool(10).submit(() -> {
        try {
            Thread.sleep(5000);
            onSuccess.accept("Pedido-" + id);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    });
}
```

Agora, com **um pool de 10 threads**, podemos servir **centenas** de requisições.

---

## 3. Callbacks

Um **callback** é uma função passada como argumento, que será chamada quando uma operação assíncrona terminar.

### Callback Básico

```java
interface PedidoCallback {
    void onSucesso(String pedido);
    void onErro(Exception e);
}

public class ServidorPedidos {
    public void buscarPedido(String id, PedidoCallback callback) {
        new Thread(() -> {
            try {
                Thread.sleep(1000);
                if (id.startsWith("P")) {
                    callback.onSucesso("Pedido: " + id);
                } else {
                    callback.onErro(new IllegalArgumentException("ID inválido"));
                }
            } catch (InterruptedException e) {
                callback.onErro(e);
            }
        }).start();
    }
}

// Uso
ServidorPedidos servidor = new ServidorPedidos();
servidor.buscarPedido("P001", new PedidoCallback() {
    @Override
    public void onSucesso(String pedido) {
        System.out.println("✓ " + pedido);
    }
    
    @Override
    public void onErro(Exception e) {
        System.err.println("✗ Erro: " + e.getMessage());
    }
});
```

### Callback Hell

Quando callbacks são aninhados, o código fica ilegível — conhecido como **"Callback Hell"** ou **"Pyramid of Doom"**.

```java
servidor.buscarPedido("P001", new PedidoCallback() {
    @Override
    public void onSucesso(String pedido) {
        servidor.calcularFrete("P001", new FreteCallback() {
            @Override
            public void onSucesso(double frete) {
                servidor.processarPagamento("P001", frete, new PagamentoCallback() {
                    @Override
                    public void onSucesso() {
                        System.out.println("✓ Tudo pronto!");
                    }
                    
                    @Override
                    public void onErro(Exception e) {
                        System.err.println("✗ Pagamento falhou: " + e);
                    }
                });
            }
            
            @Override
            public void onErro(Exception e) { /* ... */ }
        });
    }
    
    @Override
    public void onErro(Exception e) { /* ... */ }
});
```

Soluções para esse problema: **`Future`**, **`CompletableFuture`**, **promises** — veremos a seguir.

---

## 4. `Future` e `Callable`

**`Future`** representa o resultado de uma operação assíncrona que **eventualmente** estará disponível. Permite verificar se terminou, obter o resultado e até cancelar.

```java
import java.util.concurrent.*;

ExecutorService executor = Executors.newFixedThreadPool(2);

// Callable: tipo Runnable, mas retorna resultado
Callable<String> tarefaBuscarPedido = () -> {
    Thread.sleep(1000);
    return "Pedido-P001";
};

Future<String> futuro = executor.submit(tarefaBuscarPedido);

System.out.println("Tarefa disparada...");

// Diferentes formas de consultar:
if (!futuro.isDone()) {
    System.out.println("Ainda não terminou.");
}

try {
    String resultado = futuro.get();  // BLOQUEIA até resultado ficar pronto
    System.out.println("Resultado: " + resultado);
} catch (ExecutionException e) {
    System.err.println("Erro na tarefa: " + e.getCause());
} catch (InterruptedException e) {
    System.err.println("Interrompido: " + e);
}

executor.shutdown();
```

### Com Timeout

```java
try {
    String resultado = futuro.get(2, TimeUnit.SECONDS);
    System.out.println("Resultado: " + resultado);
} catch (TimeoutException e) {
    System.err.println("Operação demorou demais!");
    futuro.cancel(true);  // tenta cancelar
}
```

| Método | O que faz |
|---|---|
| `isDone()` | Retorna `true` se terminou |
| `isCancelled()` | Retorna `true` se foi cancelada |
| `get()` | **Bloqueia** até resultado chegar |
| `get(timeout, unit)` | **Bloqueia** com timeout |
| `cancel(mayInterruptIfRunning)` | Tenta cancelar |

> **Problema com `Future`:** ainda requer bloqueio com `get()` para obter o resultado. Se quisermos compor múltiplos `Future`s sem bloquear, fica complexo.

---

## 5. `CompletableFuture`

Introduzido no **Java 8**, `CompletableFuture` permite compor operações assíncronas de forma elegante **sem bloquear manualmente**.

### Criando um `CompletableFuture`

```java
// Completar manualmente
CompletableFuture<String> futuro = new CompletableFuture<>();

new Thread(() -> {
    try {
        Thread.sleep(1000);
        futuro.complete("Pedido processado!");
    } catch (InterruptedException e) {
        futuro.completeExceptionally(e);
    }
}).start();

System.out.println("Aguardando resultado...");
System.out.println(futuro.join());  // espera sem lançar checked exception
```

### Criando com `supplyAsync`

```java
CompletableFuture<String> futuro = CompletableFuture.supplyAsync(() -> {
    Thread.sleep(1000);
    return "Pedido-P001";
});

System.out.println("Disparado em thread assíncrona, continuamos aqui.");
System.out.println("Resultado: " + futuro.join());
```

---

## 6. Composição e Encadeamento de Operações Assíncronas

A vantagem do `CompletableFuture` é poder encadear operações **sem callback hell** e **sem bloqueio manual**.

### `thenApply` — Transformação

Aplica uma função ao resultado e retorna novo `CompletableFuture`.

```java
CompletableFuture<String> buscarPedido = CompletableFuture.supplyAsync(() -> {
    Thread.sleep(500);
    return "Pedido-P001";
});

CompletableFuture<Double> calcularFrete = buscarPedido
    .thenApply(pedidoId -> {
        System.out.println("Transformando: " + pedidoId);
        Thread.sleep(500);
        return 100.0;  // retorna Double
    });

double frete = calcularFrete.join();
System.out.println("Frete: R$ " + frete);
```

### `thenAccept` — Consumo (Sem Retorno)

```java
buscarPedido
    .thenApply(pedidoId -> pedidoId.toUpperCase())
    .thenAccept(pedidoFormatado -> {
        System.out.println("Pedido formatado: " + pedidoFormatado);
        // sem retorno
    })
    .join();
```

### `thenRun` — Execução (Descarta Resultado)

```java
buscarPedido
    .thenRun(() -> {
        System.out.println("Operação concluída!");
        System.out.println("Limpando recursos...");
    })
    .join();
```

### `thenCombine` — Combinando Dois `CompletableFuture`s

```java
CompletableFuture<Double> frete = CompletableFuture.supplyAsync(() -> {
    Thread.sleep(500);
    return 100.0;
});

CompletableFuture<Double> seguro = CompletableFuture.supplyAsync(() -> {
    Thread.sleep(300);
    return 25.0;
});

CompletableFuture<Double> total = frete.thenCombine(seguro, (f, s) -> f + s);
System.out.println("Total: R$ " + total.join());  // 125.0
```

### `thenCompose` — Encadeamento Assíncrono Sequencial

Quando a próxima operação depende do resultado da anterior e também é assíncrona.

```java
CompletableFuture<String> etapa1 = CompletableFuture.supplyAsync(() -> {
    Thread.sleep(500);
    return "P001";
});

CompletableFuture<Double> etapa2 = etapa1.thenCompose(pedidoId -> {
    // esta operação retorna um CompletableFuture
    return CompletableFuture.supplyAsync(() -> {
        Thread.sleep(500);
        return 150.0;
    });
});

System.out.println("Resultado final: " + etapa2.join());
```

### `anyOf` e `allOf` — Múltiplos `CompletableFuture`s

```java
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "Resultado 1");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "Resultado 2");
CompletableFuture<String> f3 = CompletableFuture.supplyAsync(() -> "Resultado 3");

// Primeiro a terminar
CompletableFuture<Object> primeiro = CompletableFuture.anyOf(f1, f2, f3);
System.out.println("Primeiro: " + primeiro.join());

// Todos terminam
CompletableFuture<Void> todos = CompletableFuture.allOf(f1, f2, f3);
todos.join();
System.out.println("Todos terminaram!");
```

---

## 7. Tratamento de Erros em Operações Assíncronas

### `exceptionally` — Fallback em Caso de Erro

```java
CompletableFuture<Double> calcularFrete = CompletableFuture.supplyAsync(() -> {
    if (Math.random() > 0.5) {
        throw new RuntimeException("API de frete indisponível");
    }
    return 100.0;
})
.exceptionally(ex -> {
    System.err.println("Erro capturado: " + ex.getMessage());
    return 50.0;  // valor padrão em caso de erro
});

System.out.println("Frete: R$ " + calcularFrete.join());
```

### `handle` — Tratamento Explícito (Sucesso e Erro)

```java
CompletableFuture<String> operacao = CompletableFuture.supplyAsync(() -> {
    if (System.currentTimeMillis() % 2 == 0) {
        throw new RuntimeException("Erro!");
    }
    return "Sucesso!";
})
.handle((resultado, erro) -> {
    if (erro != null) {
        return "Tratado erro: " + erro.getMessage();
    }
    return "Resultado: " + resultado;
});

System.out.println(operacao.join());
```

### `whenComplete` — Finalization (sem transformar resultado)

```java
CompletableFuture.supplyAsync(() -> "Operação concluída")
    .whenComplete((resultado, erro) -> {
        if (erro == null) {
            System.out.println("✓ " + resultado);
        } else {
            System.err.println("✗ " + erro.getMessage());
        }
    })
    .join();
```

---

## 8. Programação Reativa: Streams e Observables

**Programação Reativa** vai além: combina assincronia com fluxos de dados contínuos. Bibliotecas como **RxJava** e **Project Reactor** (Spring Boot) implementam esse padrão.

### Conceito: Observable

```
Observable (Produtor) ---|-> Subscriber (Consumidor)
   emite dados         continua emitindo enquanto houver
   unsubscribe         ambos vivem em threads diferentes
```

### Exemplo com RxJava (simplificado, contexto educativo)

```java
// Pseudocódigo: em projeto real, adicionar dependência RxJava
import io.reactivex.rxjava3.core.*;
import io.reactivex.rxjava3.schedulers.*;

Observable<String> pedidos = Observable.create(emitter -> {
    new Thread(() -> {
        for (int i = 0; i < 5; i++) {
            emitter.onNext("Pedido-P00" + i);
            try { Thread.sleep(500); } catch (InterruptedException e) { }
        }
        emitter.onComplete();
    }).start();
});

pedidos
    .subscribeOn(Schedulers.io())  // emissão em thread I/O
    .map(p -> p.toUpperCase())     // transforma
    .observeOn(Schedulers.mainThread())  // observação na main
    .subscribe(
        resultado -> System.out.println("✓ " + resultado),
        erro -> System.err.println("✗ " + erro),
        () -> System.out.println("Concluído!")
    );
```

**Diferenciais:**

| Aspecto | CompletableFuture | Reactive (RxJava) |
|---|---|---|
| **Fluxo único** | Sim | Sim ou múltiplos eventos |
| **Múltiplos valores** | Não (um resultado) | Sim (stream) |
| **Backpressure** | Não | Sim (controla velocidade) |
| **Curva de aprendizado** | Moderada | Íngreme |

> Para a maioria dos casos em Java moderno, **`CompletableFuture` é suficiente**. Reatividade é adicionada apenas quando necessário (ex.: **Spring WebFlux**, **gRPC streaming**).

---

## 9. Virtual Threads (Java 21+): Simplificando a Assincronia

**Virtual Threads** são threads leves gerenciadas pela JVM. Uma single thread pode suportar **centenas de milhares** delas, eliminando a necessidade de callbacks e promises em muitos casos.

### Antes (Java 17): Assíncrono Obrigatório

```java
// Java 17: Múltiplas requisições = usar callbacks/CompletableFuture
ExecutorService pool = Executors.newFixedThreadPool(10);

for (int i = 0; i < 1000; i++) {
    CompletableFuture.supplyAsync(() -> buscarPedido(), pool)
        .thenAccept(p -> System.out.println(p));
}
```

### Agora (Java 21+): Virtual Threads Permitem Código Síncrono Elegante

```java
// Java 21+: Syntax síncrono, mas escala como assíncrono
try (ExecutorService pool = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1000; i++) {
        pool.submit(() -> {
            String pedido = buscarPedido();  // síncrono, mas não bloqueia threads do SO
            System.out.println(pedido);
        });
    }
}
```

**Vantagens:**

- Código **síncrono (simples)** mas com **escalabilidade assíncrona**.
- Sem callbacks, sem promises, sem callback hell.
- Menor memória por thread (~kilobytes vs megabytes).
- Mudanças mínimas no código existente.

**Limitações:**

- Requer **Java 21+** (ainda não está em LTS 17 ou anterior).
- Operações **CPU-bound** não ganham nada (ainda precisam de pool real).

---

## 10. Síncrono vs Assíncrono: Quando Usar

| Cenário | Recomendação | Por quê |
|---|---|---|
| **I/O lento, alta carga** (API, BD, FS) | Assíncrono | Evita bloquear threads, economiza recursos |
| **Processamento CPU-intensivo** | Síncrono (em pool) | Threads CPU não ganham com assincronia |
| **Código simples, baixa carga** | Síncrono | Complexidade desnecessária |
| **Streaming contínuo de dados** | Reativo (RxJava, Reactor) | Backpressure, processamento em pipeline |
| **Escalabilidade massiva (10k+ reqs/s)** | Assíncrono ou Virtual Threads | Reduz overhead de threads |
| **Legacy code (Java < 21)** | CompletableFuture | Melhor que callbacks puro |
| **Java 21+ disponível** | Virtual Threads (preferir) | Simplicidade + escalabilidade |

---

## 11. Armadilhas e Boas Práticas

| Armadilha | Problema | Solução |
|---|---|---|
| Não tratar exceções em `CompletableFuture` | Erros silenciosos, difíceis de debugar | Usar `exceptionally`, `handle` ou `whenComplete` |
| Usar `get()` sem timeout | Pode bloquear para sempre | `get(timeout, unidade)` ou preferir `join` se já sabe que completa |
| Criar uma thread por requisição | Alto overhead | Usar `ExecutorService` ou **virtual threads** (Java 21+) |
| Misturar `thenApply` com `thenCompose` sem razão | Código confuso | `thenApply`: transformação simples; `thenCompose`: aninhamento assíncrono |
| Ignorar propagação de `InterruptedException` | Cancelamentos silenciosos | Sempre retratar ou repropagar |
| Pool de threads sem `shutdown()` | Threads nunca terminam | `shutdown()` + `awaitTermination()` ou **try-with-resources** (Java 19+) |
| Operações bloqueantes dentro de `supplyAsync` | Simula assincronia mas bloqueia | Usar callbacks do I/O real (ex.: drivers async de BD) |

### Regras de Ouro

1. **Prefira composição a bloqueio:** use `thenApply`, `thenCompose` em vez de `get()`.
2. **Sempre trate exceções:** `exceptionally` ou `handle`.
3. **Escolha o modelo certo:** síncrono para lógica simples, assíncrono para I/O, virtual threads (Java 21+) para simplicidade em escala.
4. **Teste com carga real:** problemas de concorrência aparecem sob stress.
5. **Use `try-with-resources` com executors** para garantir limpeza.

---

## 12. Exercício Resolvido

**Problema:** Buscar dados de 3 APIs diferentes em paralelo, combinar resultados, e exibir. Se alguma falhar, usar valor padrão.

```java
import java.util.concurrent.*;

public class ExercicioAsync {
    static class Dados {
        String pedido;
        double frete;
        String status;
        
        Dados(String pedido, double frete, String status) {
            this.pedido = pedido;
            this.frete = frete;
            this.status = status;
        }
        
        @Override
        public String toString() {
            return String.format("Pedido: %s, Frete: R$ %.2f, Status: %s", 
                pedido, frete, status);
        }
    }
    
    public static void main(String[] args) throws Exception {
        long inicio = System.currentTimeMillis();
        
        CompletableFuture<String> buscarPedido = CompletableFuture.supplyAsync(() -> {
            simulaLatencia(800);
            return "PED-12345";
        });
        
        CompletableFuture<Double> buscarFrete = CompletableFuture.supplyAsync(() -> {
            simulaLatencia(600);
            if (System.nanoTime() % 10 > 7) {
                throw new RuntimeException("API de frete indisponível");
            }
            return 150.0;
        })
        .exceptionally(ex -> 100.0);  // fallback
        
        CompletableFuture<String> buscarStatus = CompletableFuture.supplyAsync(() -> {
            simulaLatencia(1000);
            return "PROCESSANDO";
        });
        
        // Combinar os 3 resultados
        CompletableFuture<Dados> resultado = buscarPedido
            .thenCombine(buscarFrete, (pedido, frete) -> 
                new SimpleEntry<>(pedido, frete))
            .thenCombine(buscarStatus, (entry, status) -> 
                new Dados(entry.getKey(), entry.getValue(), status));
        
        // Aguardar
        Dados dados = resultado.join();
        
        long duracao = System.currentTimeMillis() - inicio;
        System.out.println(dados);
        System.out.println("Tempo total: " + duracao + "ms (vs ~2400ms se síncrono)");
    }
    
    static void simulaLatencia(long ms) {
        try { Thread.sleep(ms); } 
        catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }
    
    static class SimpleEntry<K, V> {
        K key; V value;
        SimpleEntry(K k, V v) { key = k; value = v; }
        K getKey() { return key; }
        V getValue() { return value; }
    }
}

// Saída (aproximada):
// Pedido: PED-12345, Frete: R$ 100.00, Status: PROCESSANDO
// Tempo total: 1050ms (vs ~2400ms se síncrono)
```

> Note que o tempo total é **~1000ms** (o máximo entre as 3 operações), não ~2400ms (soma).

---

## 13. Cheat Sheet

### Callbacks Simples

```java
void buscarDados(Consumer<String> onSuccess, Consumer<Exception> onError) {
    new Thread(() -> {
        try {
            Thread.sleep(1000);
            onSuccess.accept("Dados");
        } catch (Exception e) {
            onError.accept(e);
        }
    }).start();
}

buscarDados(
    dados -> System.out.println("✓ " + dados),
    erro -> System.err.println("✗ " + erro)
);
```

### Future

```java
ExecutorService executor = Executors.newFixedThreadPool(2);
Future<String> futuro = executor.submit(() -> {
    Thread.sleep(1000);
    return "Resultado";
});

String resultado = futuro.get(2, TimeUnit.SECONDS);
executor.shutdown();
```

### CompletableFuture — Composição

```java
CompletableFuture.supplyAsync(() -> "Etapa 1")
    .thenApply(r -> r + " -> Etapa 2")           // transforma
    .thenAccept(r -> System.out.println(r))      // consome
    .join();                                      // aguarda
```

### CompletableFuture — Múltiplas Operações

```java
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "A");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "B");

CompletableFuture<String> resultado = f1.thenCombine(f2, (a, b) -> a + b);
System.out.println(resultado.join());  // AB
```

### CompletableFuture — Tratamento de Erros

```java
CompletableFuture.supplyAsync(() -> {
    throw new RuntimeException("Erro!");
})
.exceptionally(ex -> "Fallback: " + ex.getMessage())
.thenAccept(System.out::println)
.join();
```

### Virtual Threads (Java 21+)

```java
try (ExecutorService pool = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10000; i++) {
        pool.submit(() -> {
            System.out.println("Virtual thread " + Thread.currentThread().getName());
        });
    }
}  // automaticamente aguarda e fecha
```

### RxJava (Pseudocódigo, contexto educativo)

```java
Observable.interval(1, TimeUnit.SECONDS)
    .take(5)
    .map(x -> x * 2)
    .subscribe(
        v -> System.out.println(v),
        e -> System.err.println(e),
        () -> System.out.println("Completo")
    );
```

---

## Referências

- [Java Docs: CompletableFuture](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/CompletableFuture.html)
- [Java Docs: ExecutorService](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/ExecutorService.html)
- [Java 21 Virtual Threads Documentation](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.html)
- [RxJava Documentation](https://github.com/ReactiveX/RxJava)
- [Projeto Loom (Virtual Threads)](https://openjdk.org/projects/loom/)
- [Livro: Java Concurrency in Practice (Brian Goetz)](https://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601)
- Ver também: [30-THREADS-CONCORRENCIA-JAVA.MD](30-THREADS-CONCORRENCIA-JAVA.MD), [31-PARALELISMO-JAVA.MD](31-PARALELISMO-JAVA.MD)
