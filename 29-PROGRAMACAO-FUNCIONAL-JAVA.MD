# Programação Funcional, Lambda e Streams em Java 


---

## Sumário

1. [Modelo de domínio usado nos exemplos](#1-modelo-de-domínio-usado-nos-exemplos)
2. [Paradigmas de programação](#2-paradigmas-de-programação)
3. [Transparência referencial](#3-transparência-referencial)
4. [Funções como objetos de primeira classe e method references](#4-funções-como-objetos-de-primeira-classe-e-method-references)
5. [Expressões lambda](#5-expressões-lambda)
6. [Interface funcional](#6-interface-funcional)
7. [Comparator e `List.sort`](#7-comparator-e-listsort)
8. [Predicate e `removeIf`](#8-predicate-e-removeif)
9. [Consumer e `forEach`](#9-consumer-e-foreach)
10. [Function e `map`](#10-function-e-map)
11. [Outras interfaces funcionais (Supplier, BiFunction, UnaryOperator...)](#11-outras-interfaces-funcionais)
12. [Criando métodos que recebem funções](#12-criando-métodos-que-recebem-funções)
13. [Streams: conceito](#13-streams-conceito)
14. [Criando streams](#14-criando-streams)
15. [Operações intermediárias e terminais](#15-operações-intermediárias-e-terminais)
16. [Pipeline na prática](#16-pipeline-na-prática)
17. [Streams primitivas e estatísticas](#17-streams-primitivas-e-estatísticas)
18. [Collectors](#18-collectors)
19. [Optional](#19-optional)
20. [Exercícios resolvidos](#20-exercícios-resolvidos)
21. [Armadilhas e boas práticas](#21-armadilhas-e-boas-práticas)
22. [Cheat sheet](#22-cheat-sheet)

---

## 1. Modelo de domínio usado nos exemplos

```java
public class Entrega {
    private String cliente;
    private String cidade;
    private double pesoKg;
    private double frete;
    private StatusEntrega status;

    public Entrega(String cliente, String cidade, double pesoKg, double frete, StatusEntrega status) {
        this.cliente = cliente;
        this.cidade = cidade;
        this.pesoKg = pesoKg;
        this.frete = frete;
        this.status = status;
    }

    public String getCliente() { return cliente; }
    public String getCidade() { return cidade; }
    public double getPesoKg() { return pesoKg; }
    public double getFrete() { return frete; }
    public void setFrete(double frete) { this.frete = frete; }
    public StatusEntrega getStatus() { return status; }

    @Override
    public String toString() {
        return cliente + " | " + cidade + " | " + pesoKg + "kg | R$ " + String.format("%.2f", frete) + " | " + status;
    }
}

public enum StatusEntrega { PENDENTE, EM_ROTA, ENTREGUE, CANCELADA }
```

Lista base usada nos exemplos:

```java
List<Entrega> entregas = new ArrayList<>();
entregas.add(new Entrega("Mercado Sol",  "Taubaté",  120.0, 450.00, StatusEntrega.EM_ROTA));
entregas.add(new Entrega("Farmácia Vida","Lorena",    15.5,  80.90, StatusEntrega.PENDENTE));
entregas.add(new Entrega("Auto Peças JK","Guará",     60.0, 290.00, StatusEntrega.ENTREGUE));
entregas.add(new Entrega("Tech Store",   "Taubaté",   8.0,   50.00, StatusEntrega.CANCELADA));
entregas.add(new Entrega("Construtora X","Lorena",   900.0, 1200.00, StatusEntrega.EM_ROTA));
```

---

## 2. Paradigmas de programação

| Paradigma | Exemplos de linguagem | Ideia |
|---|---|---|
| Imperativo | C, Pascal, Cobol | Sequência de comandos |
| Orientado a objetos | Java (< 8), C++ | Objetos com estado e comportamento |
| Funcional | Haskell, Erlang | Funções e expressões |
| Lógico | Prolog | Regras e fatos |
| Multiparadigma | Java 8+, JS, Python, C# | Mistura de tudo |

**Java a partir do 8 é multiparadigma**: continua OO, mas ganha ferramentas funcionais.

### Imperativo vs Funcional (analogia logística)

- **Imperativo ("como")**: você dá o passo a passo ao motorista: *"vire à direita, siga 2 km, vire à esquerda..."*
- **Funcional/declarativo ("o quê")**: você diz o destino: *"leve até o Centro de Distribuição de Lorena"*.

```java
// Imperativo: descreve COMO somar os fretes
double total = 0;
for (Entrega e : entregas) {
    total += e.getFrete();
}

// Funcional: descreve O QUÊ quero
double total = entregas.stream().mapToDouble(Entrega::getFrete).sum();
```

| Característica | Imperativa | Funcional |
|---|---|---|
| Como se descreve o que computar | Comandos ("como") | Expressões ("o quê") |
| Transparência referencial | Fraca | Forte |
| Objetos imutáveis | Raro | Comum |
| Funções são objetos de primeira classe | Não | Sim |
| Expressividade / código conciso | Baixa | Alta |
| Inferência de tipos | Raro | Comum |

> Base teórica: **Cálculo Lambda** (Alonzo Church, 1930).

---

## 3. Transparência referencial

Uma função tem **transparência referencial** quando, para as mesmas entradas, **sempre** devolve o mesmo resultado e **não altera nada fora dela** (sem efeitos colaterais).

### Contra-exemplo: função NÃO referencialmente transparente

```java
public class Program {
    public static double taxaCombustivel = 0.10; // estado global mutável

    public static double calcularFrete(double base) {
        return base + base * taxaCombustivel; // depende de algo externo
    }
}
```

`calcularFrete(100)` retorna 110 hoje. Se alguém mudar `taxaCombustivel` para 0.20, retorna 120. **Mesma entrada, resultado diferente.** Isso dificulta testes e paralelismo.

### Versão transparente

```java
public static double calcularFrete(double base, double taxaCombustivel) {
    return base + base * taxaCombustivel;
}
```

`calcularFrete(100, 0.10)` **sempre** dá 110.

**Benefícios:** simplicidade, previsibilidade, testabilidade e segurança em multithread.

---

## 4. Funções como objetos de primeira classe e method references

Funções de primeira classe podem ser **passadas como parâmetro**, **retornadas** por métodos e **guardadas em variáveis**.

```java
public class Program {

    public static int compararPorFrete(Entrega e1, Entrega e2) {
        return Double.compare(e1.getFrete(), e2.getFrete());
    }

    public static void main(String[] args) {
        entregas.sort(Program::compararPorFrete);   // method reference
        entregas.forEach(System.out::println);      // method reference
    }
}
```

### Operador `::` (method reference)

Sintaxe: `Classe::método`

| Tipo | Exemplo | Equivale a lambda |
|---|---|---|
| Método estático | `Program::compararPorFrete` | `(a, b) -> Program.compararPorFrete(a, b)` |
| Método de instância de objeto específico | `System.out::println` | `x -> System.out.println(x)` |
| Método de instância de tipo arbitrário | `Entrega::getFrete` | `e -> e.getFrete()` |
| Construtor | `ArrayList::new` | `() -> new ArrayList<>()` |

---

## 5. Expressões lambda

**Expressão lambda = função anônima de primeira classe.**

### Sintaxe

```
(parâmetros) -> corpo
```

```java
// Sem parâmetros
() -> System.out.println("Iniciando rota")

// Um parâmetro (parênteses opcionais)
e -> e.getFrete()

// Dois parâmetros
(e1, e2) -> Double.compare(e1.getFrete(), e2.getFrete())

// Corpo com várias linhas (chaves + return)
(e1, e2) -> {
    System.out.println("Comparando...");
    return Double.compare(e1.getFrete(), e2.getFrete());
}
```

### Inferência de tipos

O compilador deduz os tipos pelo contexto. Não é preciso escrever `(Entrega e1, Entrega e2)`:

```java
entregas.sort((e1, e2) -> Double.compare(e1.getFrete(), e2.getFrete()));
```

### Captura de variáveis (regra importante)

Lambdas só acessam variáveis locais **`final` ou *effectively final*** (que nunca são reatribuídas):

```java
double taxa = 0.10;                                     // effectively final
entregas.forEach(e -> e.setFrete(e.getFrete() * (1 + taxa)));   // OK

double taxa2 = 0.10;
taxa2 = 0.20;                                           // reatribuída
entregas.forEach(e -> e.setFrete(e.getFrete() * taxa2));        // ERRO de compilação
```

Campos de instância/estáticos podem ser alterados, mas isso quebra a transparência referencial. Evite.

---

## 6. Interface funcional

**Interface com exatamente um método abstrato.** Suas implementações podem ser escritas como lambdas.

```java
@FunctionalInterface
public interface CalculadoraFrete {
    double calcular(Entrega entrega);
}
```

```java
CalculadoraFrete porPeso = e -> e.getPesoKg() * 2.5;
CalculadoraFrete fixo    = e -> 50.0;

System.out.println(porPeso.calcular(entregas.get(0)));   // 300.0
```

- `@FunctionalInterface` é opcional, mas faz o compilador **reclamar** se você adicionar um segundo método abstrato.
- Podem ter métodos `default` e `static` à vontade (não contam como abstratos).
- `Comparator`, `Runnable`, `Callable`, `Predicate`, `Function`, `Consumer` são interfaces funcionais.

### Principais interfaces funcionais (`java.util.function`)

| Interface | Método | Recebe → Devolve | Uso típico |
|---|---|---|---|
| `Predicate<T>` | `test` | `T → boolean` | filtrar |
| `Function<T,R>` | `apply` | `T → R` | transformar |
| `Consumer<T>` | `accept` | `T → void` | executar ação |
| `Supplier<T>` | `get` | `() → T` | fornecer/criar valor |
| `BiFunction<T,U,R>` | `apply` | `(T,U) → R` | transformar com 2 entradas |
| `UnaryOperator<T>` | `apply` | `T → T` | transformar mesmo tipo |
| `BinaryOperator<T>` | `apply` | `(T,T) → T` | combinar/reduzir |

---

## 7. Comparator e `List.sort`

### O problema

Se `Entrega implements Comparable<Entrega>`, a ordenação fica **dentro da classe**. Se amanhã o critério mudar (por frete, por cidade, por peso), é preciso **alterar a classe**. Ela não fica "fechada para alteração" (princípio Open/Closed).

Solução: `List.sort(Comparator<? super E> c)`, com o critério **fora** da classe.

### As 5 formas de escrever um Comparator

**1) Classe separada**

```java
public class ComparadorFrete implements Comparator<Entrega> {
    @Override
    public int compare(Entrega e1, Entrega e2) {
        return Double.compare(e1.getFrete(), e2.getFrete());
    }
}

entregas.sort(new ComparadorFrete());
```

**2) Classe anônima**

```java
Comparator<Entrega> comp = new Comparator<Entrega>() {
    @Override
    public int compare(Entrega e1, Entrega e2) {
        return Double.compare(e1.getFrete(), e2.getFrete());
    }
};
entregas.sort(comp);
```

**3) Lambda com chaves**

```java
Comparator<Entrega> comp = (e1, e2) -> {
    return Double.compare(e1.getFrete(), e2.getFrete());
};
entregas.sort(comp);
```

**4) Lambda sem chaves**

```java
Comparator<Entrega> comp = (e1, e2) -> Double.compare(e1.getFrete(), e2.getFrete());
entregas.sort(comp);
```

**5) Lambda direto no argumento**

```java
entregas.sort((e1, e2) -> Double.compare(e1.getFrete(), e2.getFrete()));
```

### Complemento: `Comparator.comparing` (jeito idiomático)

```java
// Por frete crescente
entregas.sort(Comparator.comparingDouble(Entrega::getFrete));

// Por frete decrescente
entregas.sort(Comparator.comparingDouble(Entrega::getFrete).reversed());

// Por cidade (A-Z) e, em empate, por frete decrescente
entregas.sort(
    Comparator.comparing(Entrega::getCidade)
              .thenComparing(Entrega::getFrete, Comparator.reverseOrder())
);

// Ignorando maiúsculas/minúsculas
entregas.sort(Comparator.comparing(Entrega::getCliente, String.CASE_INSENSITIVE_ORDER));

// Nulls por último
entregas.sort(Comparator.comparing(Entrega::getCidade, Comparator.nullsLast(Comparator.naturalOrder())));
```

> Use `comparingDouble/Int/Long` para tipos primitivos (evita autoboxing).

---

## 8. Predicate e `removeIf`

```java
public interface Predicate<T> {
    boolean test(T t);
}
```

**Analogia:** o "portão de triagem" do galpão. Cada pacote passa e o portão responde **sim/não**.

### Problema

Remover da lista todas as entregas **canceladas**.

### 5 versões

**1) Implementação da interface**

```java
public class EntregaCancelada implements Predicate<Entrega> {
    @Override
    public boolean test(Entrega e) {
        return e.getStatus() == StatusEntrega.CANCELADA;
    }
}

entregas.removeIf(new EntregaCancelada());
```

**2) Method reference com método estático**

```java
public class EntregaUtil {
    public static boolean isCancelada(Entrega e) {
        return e.getStatus() == StatusEntrega.CANCELADA;
    }
}

entregas.removeIf(EntregaUtil::isCancelada);
```

**3) Method reference com método não estático**

```java
// Na classe Entrega:
public boolean isCancelada() { return status == StatusEntrega.CANCELADA; }

entregas.removeIf(Entrega::isCancelada);
```

**4) Lambda declarada**

```java
Predicate<Entrega> cancelada = e -> e.getStatus() == StatusEntrega.CANCELADA;
entregas.removeIf(cancelada);
```

**5) Lambda inline**

```java
entregas.removeIf(e -> e.getStatus() == StatusEntrega.CANCELADA);
```

### Complemento: combinando predicates

```java
Predicate<Entrega> pesada   = e -> e.getPesoKg() > 100;
Predicate<Entrega> emRota   = e -> e.getStatus() == StatusEntrega.EM_ROTA;

Predicate<Entrega> pesadaEmRota = pesada.and(emRota);
Predicate<Entrega> pesadaOuEmRota = pesada.or(emRota);
Predicate<Entrega> leve = pesada.negate();

entregas.removeIf(pesadaEmRota);
```

> `removeIf` **modifica a lista original**. Se ela for imutável (`List.of`, `.toList()`), lança `UnsupportedOperationException`.

---

## 9. Consumer e `forEach`

```java
public interface Consumer<T> {
    void accept(T t);
}
```

**Analogia:** um operador que **executa uma ação** em cada pacote (etiquetar, pesar, reajustar). Não devolve nada.

> Única interface funcional em que **efeitos colaterais são esperados**.

### Problema

Aumentar o frete de todas as entregas em 10%.

```java
// Lambda inline
entregas.forEach(e -> e.setFrete(e.getFrete() * 1.1));

// Method reference com método estático
public static void reajustar(Entrega e) {
    e.setFrete(e.getFrete() * 1.1);
}
entregas.forEach(EntregaUtil::reajustar);

// Lambda declarada
Consumer<Entrega> reajuste = e -> e.setFrete(e.getFrete() * 1.1);
entregas.forEach(reajuste);
```

### Imprimir a lista

```java
entregas.forEach(System.out::println);
```

### Encadeando consumers com `andThen`

```java
Consumer<Entrega> reajusta = e -> e.setFrete(e.getFrete() * 1.1);
Consumer<Entrega> loga     = e -> System.out.println("Reajustado: " + e);

entregas.forEach(reajusta.andThen(loga));
```

### `forEach` também existe em `Map`

```java
Map<String, Double> tarifaPorCidade = Map.of("Lorena", 1.5, "Taubaté", 2.0);
tarifaPorCidade.forEach((cidade, tarifa) -> System.out.println(cidade + " -> " + tarifa));
```

---

## 10. Function e `map`

```java
public interface Function<T, R> {
    R apply(T t);
}
```

**Analogia:** uma **esteira de transformação**. Entra um objeto, sai outro (ex.: entra `Entrega`, sai o nome do cliente).

### Problema

Gerar uma nova lista com os nomes dos clientes em caixa alta.

```java
// Lambda
List<String> nomes = entregas.stream()
    .map(e -> e.getCliente().toUpperCase())
    .collect(Collectors.toList());

// Method reference (com Function declarada)
Function<Entrega, String> nomeMaiusculo = e -> e.getCliente().toUpperCase();
List<String> nomes2 = entregas.stream().map(nomeMaiusculo).toList();
```

### Conversões

| De → Para | Como |
|---|---|
| `List` → `Stream` | `.stream()` |
| `Stream` → `List` | `.collect(Collectors.toList())` ou `.toList()` (Java 16+, imutável) |

> `map` (função de stream) **não é** `Map` (estrutura de dados chave/valor).

### Composição: `andThen` e `compose`

```java
Function<Entrega, Double> pegaFrete  = Entrega::getFrete;
Function<Double, Double>  comImposto = v -> v * 1.12;

Function<Entrega, Double> freteFinal = pegaFrete.andThen(comImposto); // pegaFrete e depois comImposto
```

---

## 11. Outras interfaces funcionais

### Supplier: fornece um valor (sem receber nada)

```java
Supplier<Entrega> entregaPadrao = () -> new Entrega("Cliente Balcão", "Lorena", 0, 0, StatusEntrega.PENDENTE);
Entrega e = entregaPadrao.get();
```

Muito usado em `Optional.orElseGet`, `Stream.generate` e criação lazy de objetos.

### BiFunction: 2 entradas

```java
BiFunction<Double, Double, Double> freteBase = (peso, tarifaKg) -> peso * tarifaKg;
double frete = freteBase.apply(120.0, 2.5);   // 300.0
```

### UnaryOperator: entrada e saída do mesmo tipo

```java
UnaryOperator<String> normaliza = s -> s.trim().toUpperCase();
```

### BinaryOperator: combina dois do mesmo tipo

```java
BinaryOperator<Double> soma = Double::sum;
double total = soma.apply(450.0, 80.9);
```

### Versões primitivas (evitam boxing)

`IntPredicate`, `ToDoubleFunction<T>`, `DoubleUnaryOperator`, `IntFunction`, etc.

```java
ToDoubleFunction<Entrega> pegaFrete = Entrega::getFrete;
```

---

## 12. Criando métodos que recebem funções

Você pode escrever seus próprios métodos aceitando funções (Strategy pattern sem classes extras).

### Problema

Somar o frete **somente** das entregas que satisfazem um critério (ex.: cidade começa com "T").

```java
public class EntregaService {

    public static double somaFrete(List<Entrega> lista, Predicate<Entrega> criterio) {
        double soma = 0.0;
        for (Entrega e : lista) {
            if (criterio.test(e)) {
                soma += e.getFrete();
            }
        }
        return soma;
    }
}
```

Uso:

```java
double totalT = EntregaService.somaFrete(entregas, e -> e.getCidade().startsWith("T"));
double totalRota = EntregaService.somaFrete(entregas, e -> e.getStatus() == StatusEntrega.EM_ROTA);
double totalTudo = EntregaService.somaFrete(entregas, e -> true);
```

Um único método, infinitos critérios. Sem `if` espalhado, sem novas classes.

### Versão com stream

```java
public static double somaFrete(List<Entrega> lista, Predicate<Entrega> criterio) {
    return lista.stream()
                .filter(criterio)
                .mapToDouble(Entrega::getFrete)
                .sum();
}
```

### Retornando funções

```java
public static Function<Double, Double> reajustador(double percentual) {
    return valor -> valor * (1 + percentual / 100);
}

Function<Double, Double> mais10 = reajustador(10);
mais10.apply(100.0);   // 110.0
```

---

## 13. Streams: conceito

**Stream** = sequência de elementos vinda de uma fonte de dados que suporta **operações agregadas**.

- **Fontes:** coleção, array, função geradora, arquivo/IO.
- **Analogia:** uma **esteira do centro de distribuição**. Os pacotes passam por estações (filtrar, classificar, etiquetar) e no final vão para um destino (caminhão, relatório, contagem).

### Características

| Característica | Significado |
|---|---|
| Declarativa | Iteração interna, escondida do programador |
| Parallel-friendly | Sem estado mutável compartilhado, pode usar `parallelStream()` |
| Sem efeitos colaterais | Não altera a fonte original |
| Lazy evaluation | Só executa quando há operação terminal |
| Acesso sequencial | Não há índices |
| Single-use | Só pode ser consumida **uma vez** |
| Pipeline | Operações intermediárias retornam nova stream, permitindo encadear |

### Stream ≠ Coleção

Uma coleção **armazena** dados. Uma stream **descreve o processamento** deles. A stream não guarda nada.

### Single-use na prática

```java
Stream<Entrega> st = entregas.stream();
st.forEach(System.out::println);
st.forEach(System.out::println);   // IllegalStateException: stream has already been operated upon or closed
```

Se precisar de novo, crie outra: `entregas.stream()`.

---

## 14. Criando streams

```java
// A partir de qualquer Collection
Stream<Entrega> s1 = entregas.stream();
Stream<Entrega> s2 = entregas.parallelStream();

// Stream.of
Stream<String> s3 = Stream.of("Lorena", "Taubaté", "Guará");

// Array
String[] cidades = {"Lorena", "Taubaté", "Guará"};
Stream<String> s4 = Arrays.stream(cidades);

// Stream.iterate (infinita, precisa de limit)
Stream<Integer> paradas = Stream.iterate(0, x -> x + 5);       // km 0, 5, 10, 15...
System.out.println(Arrays.toString(paradas.limit(6).toArray()));
// [0, 5, 10, 15, 20, 25]

// iterate com condição (Java 9+)
Stream.iterate(0, x -> x <= 30, x -> x + 10).forEach(System.out::println);   // 0 10 20 30

// Stream.generate
Stream<String> codigos = Stream.generate(() -> UUID.randomUUID().toString()).limit(3);

// Stream.ofNullable (Java 9+)
Stream<String> s5 = Stream.ofNullable(null);   // stream vazia

// Faixa de inteiros
IntStream.rangeClosed(1, 5).forEach(System.out::println);   // 1..5

// Linhas de arquivo
try (Stream<String> linhas = Files.lines(Path.of("entregas.csv"))) {
    linhas.forEach(System.out::println);
}
```

### Exemplo do curso: Fibonacci com `iterate`

```java
Stream<Long> fib = Stream.iterate(new long[]{0L, 1L}, p -> new long[]{p[1], p[0] + p[1]})
                         .map(p -> p[0]);
System.out.println(Arrays.toString(fib.limit(10).toArray()));
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

---

## 15. Operações intermediárias e terminais

O **pipeline** = *fonte → 0..N operações intermediárias → 1 operação terminal*.

### Intermediárias (retornam Stream, são lazy)

| Operação | O que faz | Exemplo logístico |
|---|---|---|
| `filter` | Mantém quem passa no teste | Só entregas EM_ROTA |
| `map` | Transforma cada elemento | Entrega → frete |
| `flatMap` | Transforma e "achata" | Lista de rotas → todas as paradas |
| `peek` | Espia (debug) | Log de cada item |
| `distinct` | Remove repetidos | Cidades únicas |
| `sorted` | Ordena | Por frete |
| `skip(n)` | Pula os n primeiros | Paginação |
| `limit(n)` | Pega só n (short-circuit) | Top 3 |
| `takeWhile` / `dropWhile` | Enquanto condição vale (Java 9+) | Até estourar capacidade |
| `mapToInt/Double/Long` | Para stream primitiva | Somar pesos |

### Terminais (produzem resultado e encerram a stream)

| Operação | Retorno | Exemplo logístico |
|---|---|---|
| `forEach` / `forEachOrdered` | void | Imprimir |
| `toArray` | array | |
| `toList` (Java 16+) | List imutável | |
| `collect` | coleção/Map/String... | Agrupar por cidade |
| `reduce` | valor único | Somar fretes |
| `min` / `max` | `Optional` | Entrega mais cara |
| `count` | long | Quantas pendentes |
| `anyMatch` (*) | boolean | Existe alguma cancelada? |
| `allMatch` (*) | boolean | Todas têm peso > 0? |
| `noneMatch` (*) | boolean | Nenhuma sem cliente? |
| `findFirst` / `findAny` (*) | `Optional` | Primeira em rota |

(*) **short-circuit**: pode parar antes de percorrer tudo.

### Lazy evaluation: exemplo que prova

```java
entregas.stream()
    .filter(e -> {
        System.out.println("filtrando " + e.getCliente());
        return e.getFrete() > 100;
    });
// Nada é impresso! Sem operação terminal, nada executa.

entregas.stream()
    .filter(e -> {
        System.out.println("filtrando " + e.getCliente());
        return e.getFrete() > 100;
    })
    .findFirst();
// Imprime só até achar o primeiro (short-circuit)
```

---

## 16. Pipeline na prática

```java
// Fretes com +10%
Stream<Double> fretes = entregas.stream().map(e -> e.getFrete() * 1.1);
System.out.println(Arrays.toString(fretes.toArray()));

// Soma total com reduce
double total = entregas.stream()
    .map(Entrega::getFrete)
    .reduce(0.0, (x, y) -> x + y);          // ou Double::sum

// Pipeline completo: entregas EM_ROTA, frete +10%, coletar em lista
List<Double> resultado = entregas.stream()
    .filter(e -> e.getStatus() == StatusEntrega.EM_ROTA)
    .map(e -> e.getFrete() * 1.1)
    .collect(Collectors.toList());
```

### Mais exemplos

```java
// Top 3 fretes mais caros
List<Entrega> top3 = entregas.stream()
    .sorted(Comparator.comparingDouble(Entrega::getFrete).reversed())
    .limit(3)
    .toList();

// Cidades atendidas (sem repetição, ordenadas)
List<String> cidades = entregas.stream()
    .map(Entrega::getCidade)
    .distinct()
    .sorted()
    .toList();

// Existe entrega cancelada?
boolean temCancelada = entregas.stream().anyMatch(e -> e.getStatus() == StatusEntrega.CANCELADA);

// Todas com peso positivo?
boolean pesosOk = entregas.stream().allMatch(e -> e.getPesoKg() > 0);

// Quantas pendentes?
long pendentes = entregas.stream().filter(e -> e.getStatus() == StatusEntrega.PENDENTE).count();

// Paginação (página 2, tamanho 2)
List<Entrega> pagina2 = entregas.stream().skip(2).limit(2).toList();
```

### `flatMap`: quando cada elemento vira uma lista

```java
public class Rota {
    private List<String> paradas;
    public List<String> getParadas() { return paradas; }
}

List<Rota> rotas = ...;

List<String> todasParadas = rotas.stream()
    .flatMap(r -> r.getParadas().stream())
    .distinct()
    .toList();
```

`map` daria `Stream<List<String>>`. `flatMap` "achata" para `Stream<String>`.

---

## 17. Streams primitivas e estatísticas

`Stream<Double>` faz *boxing* (objeto `Double`). Para números, prefira `DoubleStream`, `IntStream`, `LongStream`.

```java
double total = entregas.stream().mapToDouble(Entrega::getFrete).sum();

OptionalDouble media = entregas.stream().mapToDouble(Entrega::getFrete).average();
System.out.println(media.orElse(0.0));

double maxPeso = entregas.stream().mapToDouble(Entrega::getPesoKg).max().orElse(0);
```

### `summaryStatistics`: tudo de uma vez

```java
DoubleSummaryStatistics stats = entregas.stream()
    .mapToDouble(Entrega::getFrete)
    .summaryStatistics();

System.out.println(stats.getCount());
System.out.println(stats.getSum());
System.out.println(stats.getMin());
System.out.println(stats.getMax());
System.out.println(stats.getAverage());
```

---

## 18. Collectors

`collect(Collectors.xxx())` é a operação terminal mais poderosa.

```java
// Lista / Set
List<String> l = entregas.stream().map(Entrega::getCidade).collect(Collectors.toList());
Set<String> s  = entregas.stream().map(Entrega::getCidade).collect(Collectors.toSet());

// Juntar texto
String texto = entregas.stream()
    .map(Entrega::getCliente)
    .collect(Collectors.joining(", ", "[", "]"));
// [Mercado Sol, Farmácia Vida, ...]

// Agrupar por cidade
Map<String, List<Entrega>> porCidade = entregas.stream()
    .collect(Collectors.groupingBy(Entrega::getCidade));

// Contar por status
Map<StatusEntrega, Long> qtdPorStatus = entregas.stream()
    .collect(Collectors.groupingBy(Entrega::getStatus, Collectors.counting()));

// Somar frete por cidade
Map<String, Double> freteporCidade = entregas.stream()
    .collect(Collectors.groupingBy(Entrega::getCidade, Collectors.summingDouble(Entrega::getFrete)));

// Média de peso por cidade
Map<String, Double> pesoMedio = entregas.stream()
    .collect(Collectors.groupingBy(Entrega::getCidade, Collectors.averagingDouble(Entrega::getPesoKg)));

// Particionar em 2 grupos (true/false)
Map<Boolean, List<Entrega>> pesadas = entregas.stream()
    .collect(Collectors.partitioningBy(e -> e.getPesoKg() > 100));

// Virar Map (chave única obrigatória, senão IllegalStateException)
Map<String, Double> freteCliente = entregas.stream()
    .collect(Collectors.toMap(Entrega::getCliente, Entrega::getFrete));

// toMap com merge para chaves duplicadas
Map<String, Double> freteCidade = entregas.stream()
    .collect(Collectors.toMap(Entrega::getCidade, Entrega::getFrete, Double::sum));

// Nomes por cidade (mapping)
Map<String, List<String>> clientesPorCidade = entregas.stream()
    .collect(Collectors.groupingBy(Entrega::getCidade,
             Collectors.mapping(Entrega::getCliente, Collectors.toList())));
```

---

## 19. Optional

Contêiner que representa "**pode ter valor ou não**". Evita `NullPointerException`.

`min`, `max`, `findFirst`, `findAny`, `reduce` (sem identidade) e `average` retornam `Optional`.

```java
Optional<Entrega> maisCara = entregas.stream()
    .max(Comparator.comparingDouble(Entrega::getFrete));

// Formas de consumir
maisCara.ifPresent(e -> System.out.println("Mais cara: " + e));

Entrega e1 = maisCara.orElse(null);
Entrega e2 = maisCara.orElseGet(() -> new Entrega("N/A", "-", 0, 0, StatusEntrega.PENDENTE));
Entrega e3 = maisCara.orElseThrow(() -> new IllegalStateException("Sem entregas"));

// Transformar sem desembrulhar
String cliente = maisCara.map(Entrega::getCliente).orElse("Nenhum");

// Filtrar
boolean caraEPesada = maisCara.filter(e -> e.getPesoKg() > 500).isPresent();
```

> Use `Optional` como **retorno** de método. Evite em campos de classe e parâmetros.

---

## 20. Exercícios resolvidos

### 20.1 Média de fretes e destinos abaixo da média (filter, sorted, map, reduce)

**Enunciado (adaptado):** ler fretes de um `.csv`, mostrar o valor médio e, depois, os destinos (em ordem decrescente) cujo valor seja inferior à média.

**Arquivo `fretes.csv`**

```
Taubaté,900.00
Lorena,50.00
Guará,350.50
Cachoeira,80.90
Pindamonhangaba,850.00
Aparecida,290.00
```

**Classe**

```java
public class Frete {
    private String destino;
    private Double valor;

    public Frete(String destino, Double valor) {
        this.destino = destino;
        this.valor = valor;
    }

    public String getDestino() { return destino; }
    public Double getValor() { return valor; }
}
```

**Programa**

```java
import java.io.*;
import java.util.*;
import java.util.stream.*;

public class Program {
    public static void main(String[] args) {
        Locale.setDefault(Locale.US);
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter full file path: ");
        String path = sc.nextLine();

        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            List<Frete> fretes = new ArrayList<>();
            String line = br.readLine();
            while (line != null) {
                String[] f = line.split(",");
                fretes.add(new Frete(f[0], Double.parseDouble(f[1])));
                line = br.readLine();
            }

            double media = fretes.stream()
                .map(Frete::getValor)
                .reduce(0.0, Double::sum) / fretes.size();

            System.out.println("Average price: " + String.format("%.2f", media));

            Comparator<String> comp = (s1, s2) -> s1.toUpperCase().compareTo(s2.toUpperCase());

            List<String> nomes = fretes.stream()
                .filter(f -> f.getValor() < media)
                .map(Frete::getDestino)
                .sorted(comp.reversed())
                .collect(Collectors.toList());

            nomes.forEach(System.out::println);
        }
        catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
        sc.close();
    }
}
```

**Passo a passo do pipeline**

1. `reduce(0.0, Double::sum)` soma todos os valores.
2. `filter` deixa só os abaixo da média.
3. `map` extrai o destino.
4. `sorted(comp.reversed())` ordena de Z→A.
5. `collect` gera a lista.

**Saída esperada**

```
Average price: 420.23
Lorena
Guará
Cachoeira
Aparecida
```

> Média: (900 + 50 + 350.5 + 80.9 + 850 + 290) / 6 = 420,23. Abaixo dela: Lorena, Guará, Cachoeira, Aparecida. Em ordem decrescente: Lorena, Guará, Cachoeira, Aparecida.

---

### 20.2 Motoristas: e-mails acima de um salário + soma dos que começam com "M" (exercício de fixação)

**Enunciado:** ler nome, e-mail e salário de motoristas de um `.csv`. Mostrar, em ordem alfabética, o e-mail dos que ganham mais que um valor informado. Mostrar também a soma dos salários dos motoristas cujo nome começa com "M".

**Arquivo `motoristas.csv`**

```
Maria,maria@gmail.com,3200.00
Alex,alex@gmail.com,1900.00
Marco,marco@gmail.com,1700.00
Bob,bob@gmail.com,3500.00
Anna,anna@gmail.com,2800.00
```

**Classe**

```java
public class Motorista {
    private String nome;
    private String email;
    private Double salario;

    public Motorista(String nome, String email, Double salario) {
        this.nome = nome;
        this.email = email;
        this.salario = salario;
    }

    public String getNome() { return nome; }
    public String getEmail() { return email; }
    public Double getSalario() { return salario; }
}
```

**Programa**

```java
import java.io.*;
import java.util.*;
import java.util.stream.*;

public class Program {
    public static void main(String[] args) {
        Locale.setDefault(Locale.US);
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter full file path: ");
        String path = sc.nextLine();
        System.out.print("Enter salary: ");
        double salarioMin = sc.nextDouble();

        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            List<Motorista> lista = new ArrayList<>();
            String line = br.readLine();
            while (line != null) {
                String[] f = line.split(",");
                lista.add(new Motorista(f[0], f[1], Double.parseDouble(f[2])));
                line = br.readLine();
            }

            List<String> emails = lista.stream()
                .filter(m -> m.getSalario() > salarioMin)
                .map(Motorista::getEmail)
                .sorted()
                .collect(Collectors.toList());

            System.out.println("Email of people whose salary is more than " + String.format("%.2f", salarioMin) + ":");
            emails.forEach(System.out::println);

            double soma = lista.stream()
                .filter(m -> m.getNome().charAt(0) == 'M')
                .mapToDouble(Motorista::getSalario)
                .sum();

            System.out.println("Sum of salary of people whose name starts with 'M': " + String.format("%.2f", soma));
        }
        catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
        sc.close();
    }
}
```

**Saída esperada (salário = 2000)**

```
Email of people whose salary is more than 2000.00:
anna@gmail.com
bob@gmail.com
maria@gmail.com
Sum of salary of people whose name starts with 'M': 4900.00
```

> `charAt(0)` quebra em nome vazio. Em produção prefira `m.getNome().startsWith("M")`.

---

### 20.3 Extra: relatório gerencial de entregas

```java
// Frete total por cidade, da maior para a menor
entregas.stream()
    .collect(Collectors.groupingBy(Entrega::getCidade, Collectors.summingDouble(Entrega::getFrete)))
    .entrySet().stream()
    .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
    .forEach(en -> System.out.printf("%s -> R$ %.2f%n", en.getKey(), en.getValue()));

// Cliente da entrega mais pesada ainda não entregue
String cliente = entregas.stream()
    .filter(e -> e.getStatus() != StatusEntrega.ENTREGUE && e.getStatus() != StatusEntrega.CANCELADA)
    .max(Comparator.comparingDouble(Entrega::getPesoKg))
    .map(Entrega::getCliente)
    .orElse("Nenhuma");
```

---

## 21. Armadilhas e boas práticas

| Armadilha | Explicação | Solução |
|---|---|---|
| Reusar stream | `IllegalStateException` | Criar nova stream a cada uso |
| Alterar a fonte durante o pipeline | `ConcurrentModificationException` | Coletar em nova lista ou usar `removeIf` |
| Efeitos colaterais em `map`/`filter` | Quebra previsibilidade e paralelismo | Reservar efeitos para `forEach` |
| `peek` para lógica de negócio | Pode nem executar (lazy/otimização) | Usar só para debug |
| `parallelStream()` por padrão | Overhead e ordem não garantida; ruim para IO | Só em CPU-bound com muitos dados; medir antes |
| `.toList()` e depois modificar | Lista imutável, `UnsupportedOperationException` | `Collectors.toList()` ou `new ArrayList<>(...)` |
| `Collectors.toMap` com chave duplicada | `IllegalStateException` | Informar *merge function* |
| Autoboxing em `Stream<Double>` | Lento em grandes volumes | `mapToDouble`, `mapToInt` |
| Lambdas gigantes | Ilegíveis | Extrair para método e usar method reference |
| `Optional.get()` sem checar | `NoSuchElementException` | `orElse`, `orElseThrow`, `ifPresent` |
| Exceção checada dentro de lambda | Não compila | `try/catch` dentro do lambda ou método auxiliar |
| Capturar variável não *effectively final* | Erro de compilação | Usar array/`AtomicReference` ou refatorar (preferir refatorar) |
| Stream sobre `null` | `NullPointerException` | Retornar lista vazia, `Stream.ofNullable`, `Optional` |

### Regras de ouro

1. **Prefira method reference** quando o lambda só chama um método (`Entrega::getFrete`).
2. **Um pipeline legível** tem uma operação por linha.
3. **Não force stream** onde um `for` simples é mais claro (ex.: loops com índice ou early exit complexo).
4. **Prefira funções puras** (sem estado externo).
5. **Use tipos primitivos** (`mapToDouble`) em cálculos numéricos.
6. **Nomeie predicates reutilizáveis** (`emRota`, `pesada`) e combine com `and/or/negate`.

---

## 22. Cheat sheet

### Interface funcional → onde aparece

| Interface | API que consome |
|---|---|
| `Comparator<T>` | `List.sort`, `stream.sorted`, `max`, `min` |
| `Predicate<T>` | `removeIf`, `filter`, `anyMatch`, `allMatch`, `noneMatch` |
| `Consumer<T>` | `forEach`, `peek`, `Optional.ifPresent` |
| `Function<T,R>` | `map`, `flatMap`, `Comparator.comparing`, `groupingBy` |
| `Supplier<T>` | `orElseGet`, `Stream.generate`, `Collectors.toCollection` |
| `BinaryOperator<T>` | `reduce`, `toMap` (merge) |

### Receita rápida

```java
lista.stream()
     .filter(...)        // quais entram?
     .map(...)           // em que viram?
     .sorted(...)        // em que ordem?
     .limit(...)         // quantos?
     .collect(...)       // para onde vão?
```

### Lambda → Method reference

```java
e -> e.getFrete()               →  Entrega::getFrete
x -> System.out.println(x)      →  System.out::println
() -> new ArrayList<>()         →  ArrayList::new
(a, b) -> Double.sum(a, b)      →  Double::sum
s -> s.toUpperCase()            →  String::toUpperCase
```

### Referências

- Docs Java: [`Comparator`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Comparator.html), [`java.util.function`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/function/package-summary.html), [`Stream`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html), [`Collectors`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Collectors.html), [`Optional`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Optional.html)
- Repositórios do curso: `acenelio/lambda1-java` a `lambda7-java`