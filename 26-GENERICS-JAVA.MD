# Generics em Java 

## Sumário

1. [O que são Generics?](#1-o-que-são-generics)
2. [Por que usar Generics?](#2-por-que-usar-generics)
3. [Evolução: do tipo específico ao genérico](#3-evolução-do-tipo-específico-ao-genérico)
4. [Sintaxe básica de Generics](#4-sintaxe-básica-de-generics)
5. [Generics delimitados (Bounded Types)](#5-generics-delimitados-bounded-types)
6. [Tipos Wildcard](#6-tipos-wildcard)
7. [Wildcards delimitados](#7-wildcards-delimitados)
8. [Covariância e Contravariância](#8-covariância-e-contravariância)
9. [Princípio Get/Put](#9-princípio-getput)
10. [Hierarquia de tipos Wrapper](#10-hierarquia-de-tipos-wrapper)
11. [Generics em métodos](#11-generics-em-métodos)
12. [Generics em interfaces](#12-generics-em-interfaces)
13. [Type Erasure](#13-type-erasure)
14. [Restrições dos Generics](#14-restrições-dos-generics)
15. [Casos de uso na Logística](#15-casos-de-uso-na-logística)
16. [Boas práticas](#16-boas-práticas)
17. [Tabela comparativa](#17-tabela-comparativa)
18. [Exercícios](#18-exercícios)
19. [Resumo](#19-resumo)

---

## 1. O que são Generics?

Generics permitem que classes, interfaces e métodos operem sobre **tipos parametrizados**. Em vez de escrever código fixo para `String`, `Integer` ou `Veiculo`, você escreve uma vez usando `<T>` e reutiliza para qualquer tipo.

**Analogia logística:** Imagine um **contêiner de carga**. Ele não foi fabricado para carregar apenas frutas ou apenas eletrônicos — foi projetado para carregar **qualquer tipo de mercadoria**. O contêiner é genérico; o conteúdo é parametrizado no momento do uso.

```
Contêiner<Frutas>     → carrega apenas frutas
Contêiner<Eletrônicos> → carrega apenas eletrônicos
Contêiner<?>           → carrega qualquer coisa (mas você não sabe o quê)
```

---

## 2. Por que usar Generics?

### Três pilares

| Pilar | Sem Generics | Com Generics |
|-------|-------------|-------------|
| **Reuso** | Uma classe para cada tipo | Uma classe para todos os tipos |
| **Segurança de tipo** | Erros em tempo de execução (`ClassCastException`) | Erros em tempo de compilação |
| **Performance** | Casting manual (boxing/unboxing) | Sem casting desnecessário |

### Exemplo sem Generics

```java
public class RegistroDeEntrega {
    private Object item;
    
    public void setItem(Object item) {
        this.item = item;
    }
    
    public Object getItem() {
        return item;
    }
}

// Uso — perigoso!
RegistroDeEntrega registro = new RegistroDeEntrega();
registro.setItem("Caixa 42");
Integer numero = (Integer) registro.getItem(); // ClassCastException em runtime!
```

### Exemplo com Generics

```java
public class RegistroDeEntrega<T> {
    private T item;
    
    public void setItem(T item) {
        this.item = item;
    }
    
    public T getItem() {
        return item;
    }
}

// Uso — seguro!
RegistroDeEntrega<String> registro = new RegistroDeEntrega<>();
registro.setItem("Caixa 42");
// registro.setItem(123); // ERRO DE COMPILAÇÃO! Não compila.
String item = registro.getItem(); // Sem casting
```

---

## 3. Evolução: do tipo específico ao genérico

Vamos acompanhar a evolução de um serviço de impressão de etiquetas na logística.

### Etapa 1 — Tipo específico

```java
public class ServicoEtiqueta {
    private String[] etiquetas = new String[10];
    private int count = 0;
    
    public void adicionarEtiqueta(String etiqueta) {
        etiquetas[count++] = etiqueta;
    }
    
    public String primeiraEtiqueta() {
        return etiquetas[0];
    }
}
```

**Problema:** só funciona com `String`. Se precisarmos de etiquetas com código de barras (`Long`) ou com objetos `Etiqueta`, precisamos criar outra classe.

### Etapa 2 — Usando Object

```java
public class ServicoEtiqueta {
    private Object[] etiquetas = new Object[10];
    private int count = 0;
    
    public void adicionarEtiqueta(Object etiqueta) {
        etiquetas[count++] = etiqueta;
    }
    
    public Object primeiraEtiqueta() {
        return etiquetas[0];
    }
}
```

**Problema:** aceita qualquer coisa (String, Integer, Veiculo...), sem verificação. Casting manual é necessário e pode falhar em runtime.

```java
ServicoEtiqueta servico = new ServicoEtiqueta();
servico.adicionarEtiqueta("ETQ-001");
servico.adicionarEtiqueta(42); // Compila, mas não faz sentido!

String etiqueta = (String) servico.primeiraEtiqueta(); // OK
String segunda = (String) servico.segundaEtiqueta();   // ClassCastException!
```

### Etapa 3 — Usando Generics

```java
public class ServicoEtiqueta<T> {
    private T[] etiquetas;
    private int count = 0;
    
    @SuppressWarnings("unchecked")
    public ServicoEtiqueta(int capacidade) {
        etiquetas = (T[]) new Object[capacidade];
    }
    
    public void adicionarEtiqueta(T etiqueta) {
        etiquetas[count++] = etiqueta;
    }
    
    public T primeiraEtiqueta() {
        return etiquetas[0];
    }
}
```

```java
ServicoEtiqueta<String> servico = new ServicoEtiqueta<>(10);
servico.adicionarEtiqueta("ETQ-001");
// servico.adicionarEtiqueta(42); // ERRO DE COMPILAÇÃO!

String etiqueta = servico.primeiraEtiqueta(); // Sem casting, seguro
```

---

## 4. Sintaxe básica de Generics

### Convenção de nomes para parâmetros de tipo

| Letra | Significado | Exemplo logístico |
|-------|------------|-------------------|
| `T` | Type (tipo genérico) | `Armazem<T>` |
| `E` | Element (elemento de coleção) | `List<E>` |
| `K` | Key (chave) | `Map<K, V>` |
| `V` | Value (valor) | `Map<K, V>` |
| `N` | Number (tipo numérico) | `Calculadora<N>` |

### Classe genérica

```java
public class CaixaDeEntrega<T> {
    private T conteudo;
    
    public CaixaDeEntrega(T conteudo) {
        this.conteudo = conteudo;
    }
    
    public T getConteudo() {
        return conteudo;
    }
    
    public void setConteudo(T conteudo) {
        this.conteudo = conteudo;
    }
}
```

### Múltiplos parâmetros de tipo

```java
public class Remessa<T, U> {
    private T remetente;
    private U destinatario;
    
    public Remessa(T remetente, U destinatario) {
        this.remetente = remetente;
        this.destinatario = destinatario;
    }
    
    public T getRemetente() { return remetente; }
    public U getDestinatario() { return destinatario; }
}

// Uso
Remessa<Fornecedor, Cliente> remessa = new Remessa<>(fornecedor, cliente);
```

### Diamond Operator `<>`

A partir do Java 7, o compilador infere o tipo:

```java
// Antes do Java 7
CaixaDeEntrega<String> caixa = new CaixaDeEntrega<String>("Pacote A");

// A partir do Java 7
CaixaDeEntrega<String> caixa = new CaixaDeEntrega<>("Pacote A");
```

---

## 5. Generics delimitados (Bounded Types)

Às vezes você precisa restringir quais tipos podem ser usados. Para isso, use `extends`.

### Problema: encontrar o veículo com maior capacidade

```java
public class ServicoCalculo {

    // SEM delimitação — não compila, pois T pode não ser Comparable
    public static <T> T maximo(List<T> lista) {
        T max = lista.get(0);
        for (T item : lista) {
            if (item.compareTo(max) > 0) { // ERRO! T não tem compareTo
                max = item;
            }
        }
        return max;
    }
}
```

### Solução: delimitação com `extends`

```java
public class ServicoCalculo {

    public static <T extends Comparable<T>> T maximo(List<T> lista) {
        T max = lista.get(0);
        for (T item : lista) {
            if (item.compareTo(max) > 0) {
                max = item;
            }
        }
        return max;
    }
}
```

Agora `T` é obrigatoriamente um tipo que implementa `Comparable`.

### Exemplo completo com logística

```java
public class Veiculo implements Comparable<Veiculo> {
    private String placa;
    private double capacidadeKg;
    
    public Veiculo(String placa, double capacidadeKg) {
        this.placa = placa;
        this.capacidadeKg = capacidadeKg;
    }
    
    @Override
    public int compareTo(Veiculo outro) {
        return Double.compare(this.capacidadeKg, outro.capacidadeKg);
    }
    
    @Override
    public String toString() {
        return placa + " (" + capacidadeKg + " kg)";
    }
}
```

```java
List<Veiculo> frota = Arrays.asList(
    new Veiculo("ABC-1234", 1500.0),
    new Veiculo("DEF-5678", 3000.0),
    new Veiculo("GHI-9012", 800.0)
);

Veiculo maior = ServicoCalculo.maximo(frota);
System.out.println("Maior capacidade: " + maior);
// Saída: Maior capacidade: DEF-5678 (3000.0 kg)
```

### Delimitação com hierarquia de classes

Quando existe herança (A → B → C), usar `<T extends Comparable<T>>` pode ser restritivo demais:

```java
public class VeiculoBase implements Comparable<VeiculoBase> {
    protected double capacidadeKg;
    
    @Override
    public int compareTo(VeiculoBase outro) {
        return Double.compare(this.capacidadeKg, outro.capacidadeKg);
    }
}

public class Caminhao extends VeiculoBase {
    private int eixos;
}
```

`Caminhao` implementa `Comparable<VeiculoBase>`, não `Comparable<Caminhao>`. A solução:

```java
public static <T extends Comparable<? super T>> T maximo(List<T> lista) {
    // Agora aceita Caminhao, pois Comparable<VeiculoBase> é "super" de Caminhao
}
```

---

## 6. Tipos Wildcard

O wildcard `?` representa um **tipo desconhecido**.

### Por que não usar `<Object>`?

```java
List<Integer> inteiros = Arrays.asList(1, 2, 3);
List<Object> objetos = inteiros; // ERRO DE COMPILAÇÃO!
```

`List<Integer>` **NÃO** é subtipo de `List<Object>`. Generics são **invariantes**.

### Usando wildcard `<?>`

```java
public static void imprimirManifesto(List<?> itens) {
    for (Object item : itens) {
        System.out.println(item);
    }
}

// Funciona com qualquer tipo de lista!
List<String> nomes = Arrays.asList("Carga A", "Carga B");
List<Integer> codigos = Arrays.asList(101, 102, 103);

imprimirManifesto(nomes);   // OK
imprimirManifesto(codigos); // OK
```

### Restrição do wildcard: não pode adicionar

```java
List<?> lista = new ArrayList<Integer>();
// lista.add(3);   // ERRO! Não sabe qual tipo a lista aceita
// lista.add("x"); // ERRO!

// Só pode ler como Object
Object obj = lista.get(0); // OK
```

**Analogia:** É como um contêiner lacrado — você pode **olhar** o que tem dentro, mas **não pode colocar** nada novo porque não sabe se é compatível.

---

## 7. Wildcards delimitados

### `<? extends T>` — Upper Bounded (Covariância)

Aceita `T` ou qualquer **subtipo** de `T`. Permite **leitura** (GET), mas não escrita (PUT).

```java
// Aceita List<Integer>, List<Double>, List<Number>...
public static double somarPesos(List<? extends Number> pesos) {
    double total = 0;
    for (Number peso : pesos) {
        total += peso.doubleValue();
    }
    return total;
}
```

```java
List<Integer> pesosInteiros = Arrays.asList(100, 200, 350);
List<Double> pesosDecimais = Arrays.asList(150.5, 275.3);

System.out.println(somarPesos(pesosInteiros));  // 650.0
System.out.println(somarPesos(pesosDecimais));  // 425.8
```

**Analogia:** Uma esteira de **descarga** — você pode **retirar** qualquer carga (GET), mas não pode **colocar** carga nova porque não sabe o tipo exato que a esteira aceita.

### `<? super T>` — Lower Bounded (Contravariância)

Aceita `T` ou qualquer **supertipo** de `T`. Permite **escrita** (PUT), mas leitura retorna `Object`.

```java
// Aceita List<Integer>, List<Number>, List<Object>...
public static void adicionarPesosBase(List<? super Integer> lista) {
    lista.add(100);
    lista.add(200);
    lista.add(350);
}
```

```java
List<Number> numeros = new ArrayList<>();
adicionarPesosBase(numeros); // OK — Number é supertipo de Integer

List<Object> objetos = new ArrayList<>();
adicionarPesosBase(objetos); // OK — Object é supertipo de Integer
```

**Analogia:** Uma esteira de **carga** — você pode **colocar** caixas de um certo tamanho (PUT), mas ao retirar, não sabe exatamente o que vai sair (pode ser Number, Object...).

---

## 8. Covariância e Contravariância

### Invariância (padrão dos Generics)

```java
List<Caminhao> caminhoes = new ArrayList<>();
List<Veiculo> veiculos = caminhoes; // ERRO! Generics são invariantes
```

Mesmo que `Caminhao` seja subtipo de `Veiculo`, `List<Caminhao>` **NÃO** é subtipo de `List<Veiculo>`.

### Covariância com `<? extends T>`

```java
List<Caminhao> caminhoes = new ArrayList<>();
List<? extends Veiculo> veiculos = caminhoes; // OK! Covariância

Veiculo v = veiculos.get(0); // OK — pode ler como Veiculo
// veiculos.add(new Caminhao()); // ERRO — não pode escrever
```

### Contravariância com `<? super T>`

```java
List<Veiculo> veiculos = new ArrayList<>();
List<? super Caminhao> lista = veiculos; // OK! Contravariância

lista.add(new Caminhao()); // OK — pode escrever Caminhao
Object obj = lista.get(0); // Leitura retorna Object
```

### Tabela resumo

| Tipo | Sintaxe | Leitura (GET) | Escrita (PUT) |
|------|---------|---------------|---------------|
| Invariante | `List<T>` | `T` | `T` |
| Covariante | `List<? extends T>` | `T` | Proibido |
| Contravariante | `List<? super T>` | `Object` | `T` |

---

## 9. Princípio Get/Put

> **GET → `extends`** | **PUT → `super`**

Esse princípio (também chamado de PECS — *Producer Extends, Consumer Super*) define quando usar cada wildcard.

### Exemplo prático: copiar cargas entre listas

```java
public static void copiarCargas(
        List<? extends Carga> origem,    // Producer — lê (GET)
        List<? super Carga> destino) {   // Consumer — escreve (PUT)
    for (Carga c : origem) {
        destino.add(c);
    }
}
```

```java
List<CargaPerigosa> perigosas = Arrays.asList(
    new CargaPerigosa("Químicos"),
    new CargaPerigosa("Inflamáveis")
);
List<Object> deposito = new ArrayList<>();

copiarCargas(perigosas, deposito);
// perigosas → Producer (extends) → lê CargaPerigosa como Carga
// deposito  → Consumer (super)   → escreve Carga em List<Object>
```

### Exemplo completo de logística

```java
public class TransferenciaEstoque {

    public static <T> void transferir(
            List<? extends T> origem,
            List<? super T> destino) {
        for (T item : origem) {
            destino.add(item);
        }
    }
    
    public static void main(String[] args) {
        List<Integer> codigosOrigem = Arrays.asList(1001, 1002, 1003);
        List<Number> codigosDestino = new ArrayList<>();
        
        transferir(codigosOrigem, codigosDestino);
        
        System.out.println(codigosDestino); // [1001, 1002, 1003]
    }
}
```

---

## 10. Hierarquia de tipos Wrapper

Entender a hierarquia ajuda a usar wildcards corretamente:

```
         Object
           |
         Number
        /  |  \  \  \  \
   Byte Short Integer Long Float Double
```

Exemplos práticos:

```java
List<Integer> inteiros = Arrays.asList(100, 200, 300);
List<Double> decimais = Arrays.asList(1.5, 2.7, 3.9);

// Ambos funcionam porque Integer e Double são subtipos de Number
double totalInt = somarPesos(inteiros);    // 600.0
double totalDec = somarPesos(decimais);    // 8.1
```

---

## 11. Generics em métodos

Métodos podem ter seus próprios parâmetros de tipo, independentes da classe.

```java
public class UtilLogistica {

    public static <T> void imprimirArray(T[] array) {
        for (T elemento : array) {
            System.out.print(elemento + " ");
        }
        System.out.println();
    }
    
    public static <T extends Comparable<T>> T encontrarMenor(T[] array) {
        T menor = array[0];
        for (int i = 1; i < array.length; i++) {
            if (array[i].compareTo(menor) < 0) {
                menor = array[i];
            }
        }
        return menor;
    }
}
```

```java
String[] rotas = {"SP-RJ", "RJ-BH", "BH-BA"};
UtilLogistica.imprimirArray(rotas); // SP-RJ RJ-BH BH-BA

Integer[] distancias = {430, 340, 490};
Integer menor = UtilLogistica.encontrarMenor(distancias);
System.out.println("Menor distância: " + menor); // 340
```

---

## 12. Generics em interfaces

Interfaces genéricas são a base do Collections Framework e de padrões como Repository.

```java
public interface Repositorio<T> {
    void salvar(T entidade);
    T buscarPorId(int id);
    List<T> listarTodos();
    void remover(int id);
}
```

### Implementação concreta

```java
public class RepositorioVeiculo implements Repositorio<Veiculo> {
    private List<Veiculo> veiculos = new ArrayList<>();
    
    @Override
    public void salvar(Veiculo veiculo) {
        veiculos.add(veiculo);
    }
    
    @Override
    public Veiculo buscarPorId(int id) {
        return veiculos.stream()
            .filter(v -> v.getId() == id)
            .findFirst()
            .orElse(null);
    }
    
    @Override
    public List<Veiculo> listarTodos() {
        return Collections.unmodifiableList(veiculos);
    }
    
    @Override
    public void remover(int id) {
        veiculos.removeIf(v -> v.getId() == id);
    }
}
```

### Implementação genérica (mantendo o tipo aberto)

```java
public class RepositorioEmMemoria<T> implements Repositorio<T> {
    private Map<Integer, T> dados = new HashMap<>();
    private int proximoId = 1;
    
    @Override
    public void salvar(T entidade) {
        dados.put(proximoId++, entidade);
    }
    
    @Override
    public T buscarPorId(int id) {
        return dados.get(id);
    }
    
    @Override
    public List<T> listarTodos() {
        return new ArrayList<>(dados.values());
    }
    
    @Override
    public void remover(int id) {
        dados.remove(id);
    }
}
```

---

## 13. Type Erasure

Em tempo de execução, o Java **apaga** as informações de tipo genérico. Isso é chamado de *type erasure*.

```java
// O que você escreve:
List<String> rotas = new ArrayList<>();
rotas.add("SP-RJ");

// O que o compilador gera (após erasure):
List rotas = new ArrayList();
rotas.add("SP-RJ");
```

### Consequências práticas

```java
// Isso NÃO funciona:
public class Armazem<T> {
    // private T[] itens = new T[10];          // ERRO! Não pode instanciar T
    // if (item instanceof T) { }              // ERRO! Não pode usar instanceof com T
}

// Soluções:
public class Armazem<T> {
    @SuppressWarnings("unchecked")
    private T[] itens = (T[]) new Object[10];  // Cast de Object[]
}
```

---

## 14. Restrições dos Generics

| Restrição | Exemplo que NÃO compila |
|-----------|------------------------|
| Não pode instanciar tipo genérico | `new T()` |
| Não pode criar array de tipo genérico | `new T[10]` |
| Não pode usar tipos primitivos | `List<int>` → use `List<Integer>` |
| Não pode usar `instanceof` com genérico | `obj instanceof List<String>` |
| Não pode ter `static` com tipo da classe | `static T campo;` |
| Não pode fazer catch/throw de tipo genérico | `catch (T e)` |

---

## 15. Casos de uso na Logística

### 1. Serviço genérico de rastreamento

```java
public class Rastreamento<T extends Rastreavel> {
    private List<T> itensRastreados = new ArrayList<>();
    
    public void registrar(T item) {
        itensRastreados.add(item);
    }
    
    public T buscarPorCodigo(String codigo) {
        return itensRastreados.stream()
            .filter(item -> item.getCodigo().equals(codigo))
            .findFirst()
            .orElse(null);
    }
    
    public List<T> listarTodos() {
        return Collections.unmodifiableList(itensRastreados);
    }
}
```

Funciona para `Pacote`, `Encomenda`, `Container` — qualquer coisa que implemente `Rastreavel`.

### 2. Conversor genérico de unidades

```java
public interface ConversorUnidade<T extends Number> {
    double converter(T valor);
}

public class KgParaLibras implements ConversorUnidade<Double> {
    @Override
    public double converter(Double kg) {
        return kg * 2.20462;
    }
}

public class KmParaMilhas implements ConversorUnidade<Integer> {
    @Override
    public double converter(Integer km) {
        return km * 0.621371;
    }
}
```

### 3. Par genérico para mapeamento origem/destino

```java
public class ParOrigemDestino<O, D> {
    private O origem;
    private D destino;
    
    public ParOrigemDestino(O origem, D destino) {
        this.origem = origem;
        this.destino = destino;
    }
    
    public O getOrigem() { return origem; }
    public D getDestino() { return destino; }
    
    @Override
    public String toString() {
        return origem + " → " + destino;
    }
}
```

```java
ParOrigemDestino<String, String> rota = new ParOrigemDestino<>("São Paulo", "Rio de Janeiro");
ParOrigemDestino<Armazem, Cliente> entrega = new ParOrigemDestino<>(armazem, cliente);
```

### 4. Fila de processamento genérica

```java
public class FilaDeProcessamento<T> {
    private Queue<T> fila = new LinkedList<>();
    
    public void enfileirar(T item) {
        fila.add(item);
    }
    
    public T processar() {
        return fila.poll();
    }
    
    public boolean temPendentes() {
        return !fila.isEmpty();
    }
    
    public int tamanho() {
        return fila.size();
    }
}
```

```java
FilaDeProcessamento<OrdemDeEntrega> filaEntregas = new FilaDeProcessamento<>();
filaEntregas.enfileirar(new OrdemDeEntrega("OE-001", "SP", "RJ"));
filaEntregas.enfileirar(new OrdemDeEntrega("OE-002", "RJ", "BH"));

while (filaEntregas.temPendentes()) {
    OrdemDeEntrega ordem = filaEntregas.processar();
    System.out.println("Processando: " + ordem);
}
```

---

## 16. Boas práticas

### Faça

- Use nomes de parâmetro significativos (`T`, `E`, `K`, `V`)
- Prefira `List<? extends T>` para parâmetros de entrada somente leitura
- Prefira `List<? super T>` para parâmetros de saída (destino)
- Aplique o princípio PECS (Producer Extends, Consumer Super)
- Delimite tipos quando possível (`<T extends Serializable>`)
- Use diamond operator `<>` para evitar redundância

### Evite

- Não misture raw types (`List`) com parameterizados (`List<String>`)
- Não ignore warnings de unchecked — resolva-os
- Não use `instanceof` para verificar tipo genérico
- Não crie arrays de tipo genérico diretamente
- Evite wildcards em tipos de retorno — dificulta o uso pelo chamador

### Quando usar cada abordagem

| Situação | Use |
|----------|-----|
| Classe/interface reutilizável | `<T>` |
| Método independente da classe | `<T>` no método |
| Restrição de tipo | `<T extends X>` |
| Parâmetro somente leitura | `<? extends T>` |
| Parâmetro somente escrita | `<? super T>` |
| Tipo completamente desconhecido | `<?>` |

---

## 17. Tabela comparativa

| Conceito | Sintaxe | Leitura | Escrita | Uso típico |
|----------|---------|---------|---------|-----------|
| Tipo concreto | `List<Veiculo>` | `Veiculo` | `Veiculo` | Uso direto |
| Wildcard ilimitado | `List<?>` | `Object` | Proibido | Leitura genérica |
| Upper bounded | `List<? extends Veiculo>` | `Veiculo` | Proibido | Producer (PECS) |
| Lower bounded | `List<? super Veiculo>` | `Object` | `Veiculo` | Consumer (PECS) |
| Tipo parametrizado | `<T>` | `T` | `T` | Classes/métodos genéricos |
| Bounded type | `<T extends Comparable<T>>` | `T` | `T` | Restrição de capacidade |

---

## 18. Exercícios

### Exercício 1 — Armazém genérico

Crie uma classe genérica `Armazem<T>` que armazene itens em uma lista interna. Implemente:
- `adicionar(T item)` — adiciona item ao armazém
- `remover(T item)` — remove item do armazém
- `buscar(Predicate<T> filtro)` — retorna lista de itens que atendem ao filtro
- `listar()` — retorna todos os itens

Teste com `Armazem<Produto>` e `Armazem<String>`.

### Exercício 2 — Serviço de cálculo de frete

Crie um método genérico que encontre a entrega mais cara de uma lista:

```java
public static <T extends Comparable<T>> T maisCaro(List<T> entregas)
```

Crie uma classe `Entrega` que implemente `Comparable<Entrega>` comparando por valor do frete. Teste com uma lista de entregas.

### Exercício 3 — Transferência entre armazéns

Usando o princípio Get/Put, crie um método que transfira itens de um armazém de origem para um armazém de destino:

```java
public static <T> void transferir(
    List<? extends T> origem,
    List<? super T> destino
)
```

Teste transferindo uma `List<CargaRefrigerada>` para uma `List<Carga>`.

### Exercício 4 — Conversor genérico

Crie uma interface genérica `Conversor<S, D>` (Source → Destination) com um método `D converter(S source)`. Implemente:
- `VeiculoParaDTO` — converte `Veiculo` para `VeiculoDTO`
- `DTOParaVeiculo` — converte `VeiculoDTO` para `Veiculo`

---

## 19. Resumo

```
Generics em Java
│
├── O quê? → Tipos parametrizados (<T>, <E>, <K,V>)
│
├── Por quê?
│   ├── Reuso de código
│   ├── Segurança de tipo (erros em compilação)
│   └── Performance (sem casting)
│
├── Tipos
│   ├── Classe genérica: class Armazem<T>
│   ├── Interface genérica: interface Repositorio<T>
│   ├── Método genérico: <T> void metodo(T param)
│   └── Bounded: <T extends Comparable<T>>
│
├── Wildcards
│   ├── <?> — tipo desconhecido
│   ├── <? extends T> — covariância (GET)
│   └── <? super T> — contravariância (PUT)
│
├── PECS
│   ├── Producer → extends (leitura)
│   └── Consumer → super (escrita)
│
├── Restrições
│   ├── Type erasure em runtime
│   ├── Não pode instanciar T
│   └── Não aceita primitivos
│
└── Na logística
    ├── Repositórios genéricos
    ├── Filas de processamento
    ├── Conversores de unidade
    └── Rastreamento parametrizado
```