# Map em Java — Guia Completo com Exemplos de Logística

## Sumário

1. [O que é Map?](#1-o-que-é-map)
2. [Características fundamentais](#2-características-fundamentais)
3. [Implementações de Map](#3-implementações-de-map)
4. [HashMap](#4-hashmap)
5. [TreeMap](#5-treemap)
6. [LinkedHashMap](#6-linkedhashmap)
7. [Métodos principais](#7-métodos-principais)
8. [Iteração sobre Map](#8-iteração-sobre-map)
9. [Map com objetos personalizados como chave](#9-map-com-objetos-personalizados-como-chave)
10. [Métodos utilitários (Java 8+)](#10-métodos-utilitários-java-8)
11. [Casos de uso na Logística](#11-casos-de-uso-na-logística)
12. [Map vs Set vs List](#12-map-vs-set-vs-list)
13. [Boas práticas](#13-boas-práticas)
14. [Tabela comparativa](#14-tabela-comparativa)
15. [Exercícios](#15-exercícios)
16. [Resumo](#16-resumo)

---

## 1. O que é Map?

`Map<K, V>` é uma coleção de **pares chave/valor**. Cada chave é única e mapeia para exatamente um valor. É como um dicionário: você busca pela chave e obtém o valor associado.

**Analogia logística:** Pense em um **painel de rastreamento** onde cada **código de rastreio** (chave) está associado ao **status da entrega** (valor):

```
"BR123456789" → "Em trânsito"
"BR987654321" → "Entregue"
"BR111222333" → "Aguardando coleta"
```

```java
Map<String, String> rastreamento = new HashMap<>();
rastreamento.put("BR123456789", "Em trânsito");
rastreamento.put("BR987654321", "Entregue");
rastreamento.put("BR111222333", "Aguardando coleta");

String status = rastreamento.get("BR123456789");
System.out.println(status); // Em trânsito
```

---

## 2. Características fundamentais

| Característica | Descrição | Analogia logística |
|---------------|-----------|-------------------|
| **Pares chave/valor** | Cada entrada tem K e V | Código de rastreio → Status |
| **Chave única** | Não admite chaves duplicadas | Cada placa mapeia para um veículo |
| **Sem posição** | Elementos indexados pela chave, não por índice | Busca por código, não por posição |
| **Acesso rápido** | Inserção, remoção e busca são rápidas | Consulta instantânea de status |

### Map NÃO é Collection

`Map` não estende `Collection`. É uma estrutura separada na hierarquia do Java Collections Framework:

```
Iterable
  └── Collection
        ├── List
        ├── Set
        └── Queue

Map (hierarquia independente)
  ├── HashMap
  ├── TreeMap
  └── LinkedHashMap
```

### Chave duplicada sobrescreve o valor

```java
Map<String, String> status = new HashMap<>();
status.put("ABC-1234", "Disponível");
status.put("ABC-1234", "Em manutenção"); // Sobrescreve!

System.out.println(status.get("ABC-1234")); // Em manutenção
System.out.println(status.size()); // 1
```

---

## 3. Implementações de Map

```
       Map<K,V>  (interface)
       /    |    \
  HashMap  TreeMap  LinkedHashMap
```

| Implementação | Estrutura interna | Complexidade | Ordenação das chaves |
|--------------|-------------------|-------------|---------------------|
| **HashMap** | Tabela hash | O(1) | Sem ordem definida |
| **TreeMap** | Árvore rubro-negra | O(log n) | Ordenada (natural ou Comparator) |
| **LinkedHashMap** | Hash + lista ligada | O(1)* | Ordem de inserção |

**Analogia:** Três formas de organizar **etiquetas de encomendas**:
- **HashMap** → etiquetas jogadas numa gaveta (busca rápida por código, sem ordem)
- **TreeMap** → etiquetas em fichário alfabético por código
- **LinkedHashMap** → etiquetas empilhadas na ordem em que chegaram

---

## 4. HashMap

A implementação mais utilizada. Usa **tabela hash** internamente.

### Quando usar

- Quando a **ordem das chaves não importa**
- Quando precisa de **velocidade máxima** em put/get/remove/containsKey
- Quando as chaves implementam `hashCode` e `equals` corretamente

### Exemplo: estoque por produto

```java
Map<String, Integer> estoque = new HashMap<>();
estoque.put("Caixa Pequena", 500);
estoque.put("Caixa Média", 300);
estoque.put("Caixa Grande", 150);
estoque.put("Pallet", 80);

System.out.println(estoque.get("Caixa Média")); // 300
System.out.println(estoque.containsKey("Pallet")); // true
System.out.println(estoque.size()); // 4
```

### Aceita null

```java
Map<String, String> config = new HashMap<>();
config.put(null, "valor padrão");     // Chave null OK
config.put("chave", null);             // Valor null OK
```

---

## 5. TreeMap

Mantém as **chaves ordenadas**. Usa **árvore rubro-negra** internamente.

### Quando usar

- Quando precisa das chaves **em ordem**
- Quando precisa de métodos como `firstKey()`, `lastKey()`, `headMap()`, `tailMap()`
- Aceita performance ligeiramente menor (O(log n) vs O(1))

### Exemplo: cookies de sessão (ordenados por chave)

```java
Map<String, String> cookies = new TreeMap<>();
cookies.put("username", "maria");
cookies.put("email", "maria@transportadora.com");
cookies.put("phone", "99771122");

cookies.remove("email");
cookies.put("phone", "99771133"); // Sobrescreve

System.out.println("Contém 'phone': " + cookies.containsKey("phone")); // true
System.out.println("Telefone: " + cookies.get("phone")); // 99771133
System.out.println("Email: " + cookies.get("email")); // null (removido)
System.out.println("Tamanho: " + cookies.size()); // 2

System.out.println("Todos os cookies:");
for (String key : cookies.keySet()) {
    System.out.println(key + ": " + cookies.get(key));
}
// phone: 99771133
// username: maria
```

### Métodos exclusivos do TreeMap

```java
TreeMap<Integer, String> rotasPorKm = new TreeMap<>();
rotasPorKm.put(430, "SP-RJ");
rotasPorKm.put(580, "SP-BH");
rotasPorKm.put(410, "SP-CT");
rotasPorKm.put(1000, "SP-BA");

System.out.println(rotasPorKm.firstKey());        // 410
System.out.println(rotasPorKm.lastKey());         // 1000
System.out.println(rotasPorKm.lowerKey(430));     // 410
System.out.println(rotasPorKm.higherKey(430));    // 580
System.out.println(rotasPorKm.headMap(500));      // {410=SP-CT, 430=SP-RJ}
System.out.println(rotasPorKm.tailMap(500));      // {580=SP-BH, 1000=SP-BA}
```

### Não aceita null como chave

```java
Map<String, String> tree = new TreeMap<>();
// tree.put(null, "valor"); // NullPointerException!
```

---

## 6. LinkedHashMap

Mantém a **ordem de inserção**. Combina hash table com lista duplamente ligada.

### Quando usar

- Quando a **ordem de inserção** importa
- Quando precisa de velocidade similar ao HashMap
- Útil para implementar cache LRU (Least Recently Used)

### Exemplo: histórico de operações

```java
Map<String, String> historico = new LinkedHashMap<>();
historico.put("08:00", "Coleta em Guarulhos");
historico.put("10:30", "Chegada ao CD São Paulo");
historico.put("11:00", "Triagem concluída");
historico.put("14:00", "Saída para entrega");

for (Map.Entry<String, String> entry : historico.entrySet()) {
    System.out.println(entry.getKey() + " - " + entry.getValue());
}
// 08:00 - Coleta em Guarulhos
// 10:30 - Chegada ao CD São Paulo
// 11:00 - Triagem concluída
// 14:00 - Saída para entrega
```

---

## 7. Métodos principais

### Operações básicas

```java
Map<String, Double> fretes = new HashMap<>();

// Inserir/atualizar
fretes.put("SP-RJ", 350.00);    // Insere
fretes.put("SP-BH", 480.00);    // Insere
fretes.put("SP-RJ", 375.00);    // Atualiza (chave já existe)

// Consultar
Double valor = fretes.get("SP-RJ");            // 375.00
Double inexistente = fretes.get("SP-BA");       // null
Double comPadrao = fretes.getOrDefault("SP-BA", 0.0); // 0.0

// Verificar
fretes.containsKey("SP-RJ");   // true
fretes.containsValue(480.00);  // true
fretes.isEmpty();               // false
fretes.size();                  // 2

// Remover
fretes.remove("SP-BH");        // Remove e retorna 480.00
fretes.remove("SP-RJ", 999.0); // NÃO remove (valor não bate)

// Limpar
fretes.clear();
```

### keySet, values e entrySet

```java
Map<String, Integer> capacidades = new HashMap<>();
capacidades.put("Fiorino", 600);
capacidades.put("Sprinter", 1500);
capacidades.put("Truck", 8000);

// Conjunto de chaves
Set<String> veiculos = capacidades.keySet();
System.out.println(veiculos); // [Fiorino, Truck, Sprinter]

// Coleção de valores
Collection<Integer> caps = capacidades.values();
System.out.println(caps); // [600, 8000, 1500]

// Conjunto de entradas (pares chave-valor)
Set<Map.Entry<String, Integer>> entradas = capacidades.entrySet();
for (Map.Entry<String, Integer> entry : entradas) {
    System.out.println(entry.getKey() + " → " + entry.getValue() + " kg");
}
// Fiorino → 600 kg
// Truck → 8000 kg
// Sprinter → 1500 kg
```

---

## 8. Iteração sobre Map

### Iterar por chaves

```java
Map<String, Double> fretes = new HashMap<>();
fretes.put("SP-RJ", 350.0);
fretes.put("SP-BH", 480.0);
fretes.put("SP-CT", 290.0);

for (String rota : fretes.keySet()) {
    System.out.println(rota + ": R$ " + fretes.get(rota));
}
```

### Iterar por entradas (mais eficiente)

```java
for (Map.Entry<String, Double> entry : fretes.entrySet()) {
    System.out.println(entry.getKey() + ": R$ " + entry.getValue());
}
```

### forEach (Java 8+)

```java
fretes.forEach((rota, valor) -> 
    System.out.println(rota + ": R$ " + valor)
);
```

### Stream API

```java
// Filtrar rotas com frete acima de R$ 300
fretes.entrySet().stream()
    .filter(e -> e.getValue() > 300.0)
    .forEach(e -> System.out.println(e.getKey() + ": R$ " + e.getValue()));

// Somar todos os fretes
double total = fretes.values().stream()
    .mapToDouble(Double::doubleValue)
    .sum();
System.out.println("Total: R$ " + total); // 1120.0
```

---

## 9. Map com objetos personalizados como chave

Quando se usa um objeto customizado como chave do Map, é essencial implementar `hashCode` e `equals`.

### Sem hashCode e equals

```java
public class Produto {
    private String nome;
    private Double preco;
    // SEM hashCode e equals!
}

Map<Produto, Double> estoque = new HashMap<>();
Produto p1 = new Produto("TV", 900.0);
estoque.put(p1, 10000.0);

Produto ps = new Produto("TV", 900.0);
System.out.println(estoque.containsKey(ps)); // false! Objetos diferentes na memória
```

### Com hashCode e equals

```java
public class Produto {
    private String nome;
    private Double preco;
    
    @Override
    public int hashCode() {
        return Objects.hash(nome, preco);
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Produto outro = (Produto) obj;
        return Objects.equals(nome, outro.nome) && Objects.equals(preco, outro.preco);
    }
}

Map<Produto, Double> estoque = new HashMap<>();
Produto p1 = new Produto("TV", 900.0);
estoque.put(p1, 10000.0);

Produto ps = new Produto("TV", 900.0);
System.out.println(estoque.containsKey(ps)); // true!
```

---

## 10. Métodos utilitários (Java 8+)

### putIfAbsent — insere apenas se não existir

```java
Map<String, Integer> entregas = new HashMap<>();
entregas.put("João", 15);

entregas.putIfAbsent("João", 20); // Não sobrescreve (já existe)
entregas.putIfAbsent("Maria", 10); // Insere (não existia)

System.out.println(entregas); // {João=15, Maria=10}
```

### getOrDefault — valor padrão se chave não existir

```java
int entregasJoao = entregas.getOrDefault("João", 0);   // 15
int entregasPedro = entregas.getOrDefault("Pedro", 0);  // 0
```

### merge — combinar valores

```java
Map<String, Integer> votosRegiao1 = new HashMap<>();
votosRegiao1.put("Candidato A", 150);
votosRegiao1.put("Candidato B", 200);

// Somar votos da região 2
votosRegiao1.merge("Candidato A", 100, Integer::sum); // 150 + 100 = 250
votosRegiao1.merge("Candidato B", 50, Integer::sum);  // 200 + 50 = 250
votosRegiao1.merge("Candidato C", 80, Integer::sum);  // Novo: 80

System.out.println(votosRegiao1);
// {Candidato A=250, Candidato B=250, Candidato C=80}
```

### compute — calcular novo valor

```java
Map<String, Integer> contadorRotas = new HashMap<>();

// Incrementar contador de cada rota processada
List<String> rotasProcessadas = Arrays.asList("SP-RJ", "SP-BH", "SP-RJ", "SP-RJ", "SP-BH");

for (String rota : rotasProcessadas) {
    contadorRotas.compute(rota, (k, v) -> v == null ? 1 : v + 1);
}

System.out.println(contadorRotas); // {SP-RJ=3, SP-BH=2}
```

### replaceAll — transformar todos os valores

```java
Map<String, Double> fretes = new HashMap<>();
fretes.put("SP-RJ", 350.0);
fretes.put("SP-BH", 480.0);

// Aplicar reajuste de 10% em todos os fretes
fretes.replaceAll((rota, valor) -> valor * 1.10);

System.out.println(fretes); // {SP-RJ=385.0, SP-BH=528.0}
```

### Map.of e Map.ofEntries — maps imutáveis (Java 9+)

```java
// Até 10 pares
Map<String, String> statusCodigos = Map.of(
    "PEN", "Pendente",
    "TRA", "Em trânsito",
    "ENT", "Entregue",
    "CAN", "Cancelado"
);

// Mais de 10 pares
Map<String, String> grande = Map.ofEntries(
    Map.entry("SP", "São Paulo"),
    Map.entry("RJ", "Rio de Janeiro"),
    Map.entry("MG", "Minas Gerais")
    // ...
);

// statusCodigos.put("DEV", "Devolvido"); // UnsupportedOperationException!
```

---

## 11. Casos de uso na Logística

### 1. Contagem de entregas por motorista

```java
public class RelatorioEntregas {
    
    public static Map<String, Integer> contarEntregas(List<String> registros) {
        Map<String, Integer> contagem = new TreeMap<>();
        
        for (String motorista : registros) {
            contagem.merge(motorista, 1, Integer::sum);
        }
        
        return contagem;
    }
    
    public static void main(String[] args) {
        List<String> registros = Arrays.asList(
            "João", "Maria", "João", "Pedro", "Maria", "João", "Pedro", "Maria", "Maria"
        );
        
        Map<String, Integer> resultado = contarEntregas(registros);
        resultado.forEach((motorista, qtd) -> 
            System.out.println(motorista + ": " + qtd + " entregas")
        );
        // João: 3 entregas
        // Maria: 4 entregas
        // Pedro: 2 entregas
    }
}
```

### 2. Consolidação de votos/avaliações por filial

```java
public class ConsolidadorVotos {
    
    public static Map<String, Integer> consolidar(String caminhoArquivo) throws IOException {
        Map<String, Integer> resultado = new TreeMap<>();
        
        try (Scanner sc = new Scanner(new File(caminhoArquivo))) {
            while (sc.hasNextLine()) {
                String[] partes = sc.nextLine().split(",");
                String candidato = partes[0].trim();
                int votos = Integer.parseInt(partes[1].trim());
                
                resultado.merge(candidato, votos, Integer::sum);
            }
        }
        
        return resultado;
    }
}
```

Arquivo de entrada (formato CSV):
```
Alex Blue,15
Maria Green,22
Bob Brown,21
Alex Blue,30
Bob Brown,15
Maria Green,27
Maria Green,22
Bob Brown,25
Alex Blue,31
```

Saída:
```
Alex Blue: 76
Bob Brown: 61
Maria Green: 71
```

### 3. Cache de rotas calculadas

```java
public class CacheRotas {
    private final Map<String, Double> cache = new LinkedHashMap<>(16, 0.75f, true) {
        @Override
        protected boolean removeEldestEntry(Map.Entry<String, Double> eldest) {
            return size() > 100; // Mantém no máximo 100 rotas em cache
        }
    };
    
    public Double getDistancia(String origem, String destino) {
        String chave = origem + "-" + destino;
        return cache.computeIfAbsent(chave, k -> calcularDistancia(origem, destino));
    }
    
    private Double calcularDistancia(String origem, String destino) {
        // Simulação de cálculo pesado
        System.out.println("Calculando rota " + origem + " → " + destino);
        return Math.random() * 1000;
    }
}
```

### 4. Tabela de preços de frete por faixa de CEP

```java
public class TabelaFrete {
    private final Map<String, Double> precoPorFaixaCep = new TreeMap<>();
    
    public TabelaFrete() {
        precoPorFaixaCep.put("01000-09999", 25.00);  // São Paulo capital
        precoPorFaixaCep.put("10000-19999", 35.00);  // Interior SP
        precoPorFaixaCep.put("20000-29999", 45.00);  // Rio de Janeiro
        precoPorFaixaCep.put("30000-39999", 55.00);  // Minas Gerais
        precoPorFaixaCep.put("40000-49999", 70.00);  // Bahia
    }
    
    public Double consultarFrete(String cep) {
        int cepNum = Integer.parseInt(cep.replace("-", ""));
        
        for (Map.Entry<String, Double> entry : precoPorFaixaCep.entrySet()) {
            String[] faixa = entry.getKey().split("-");
            int inicio = Integer.parseInt(faixa[0]);
            int fim = Integer.parseInt(faixa[1]);
            
            if (cepNum >= inicio && cepNum <= fim) {
                return entry.getValue();
            }
        }
        
        return null;
    }
}
```

### 5. Estoque por produto com controle de entrada/saída

```java
public class ControleEstoque {
    private final Map<String, Integer> estoque = new HashMap<>();
    
    public void entrada(String produto, int quantidade) {
        estoque.merge(produto, quantidade, Integer::sum);
    }
    
    public boolean saida(String produto, int quantidade) {
        Integer atual = estoque.get(produto);
        if (atual == null || atual < quantidade) {
            return false;
        }
        int novo = atual - quantidade;
        if (novo == 0) {
            estoque.remove(produto);
        } else {
            estoque.put(produto, novo);
        }
        return true;
    }
    
    public int consultar(String produto) {
        return estoque.getOrDefault(produto, 0);
    }
    
    public void relatorio() {
        System.out.println("=== Estoque ===");
        estoque.entrySet().stream()
            .sorted(Map.Entry.comparingByKey())
            .forEach(e -> System.out.printf("%-20s %d unidades%n", e.getKey(), e.getValue()));
    }
}
```

```java
ControleEstoque estoque = new ControleEstoque();
estoque.entrada("Caixa Pequena", 100);
estoque.entrada("Caixa Grande", 50);
estoque.entrada("Fita Adesiva", 200);
estoque.saida("Caixa Pequena", 30);
estoque.entrada("Caixa Pequena", 20);

estoque.relatorio();
// === Estoque ===
// Caixa Grande         50 unidades
// Caixa Pequena        90 unidades
// Fita Adesiva         200 unidades
```

### 6. Mapeamento de veículos por região

```java
public class FrotaPorRegiao {
    private final Map<String, List<String>> veiculosPorRegiao = new HashMap<>();
    
    public void alocarVeiculo(String regiao, String placa) {
        veiculosPorRegiao.computeIfAbsent(regiao, k -> new ArrayList<>()).add(placa);
    }
    
    public List<String> getVeiculos(String regiao) {
        return veiculosPorRegiao.getOrDefault(regiao, Collections.emptyList());
    }
    
    public void relatorio() {
        veiculosPorRegiao.forEach((regiao, veiculos) -> {
            System.out.println(regiao + ": " + veiculos.size() + " veículos");
            veiculos.forEach(v -> System.out.println("  - " + v));
        });
    }
}
```

```java
FrotaPorRegiao frota = new FrotaPorRegiao();
frota.alocarVeiculo("Sudeste", "ABC-1234");
frota.alocarVeiculo("Sudeste", "DEF-5678");
frota.alocarVeiculo("Sul", "GHI-9012");
frota.alocarVeiculo("Nordeste", "JKL-3456");
frota.alocarVeiculo("Sudeste", "MNO-7890");

frota.relatorio();
// Sudeste: 3 veículos
//   - ABC-1234
//   - DEF-5678
//   - MNO-7890
// Sul: 1 veículos
//   - GHI-9012
// Nordeste: 1 veículos
//   - JKL-3456
```

---

## 12. Map vs Set vs List

| Aspecto | List | Set | Map |
|---------|------|-----|-----|
| **Estrutura** | Sequência ordenada | Conjunto sem duplicatas | Pares chave/valor |
| **Duplicatas** | Permite | Não permite | Chave única, valor duplica |
| **Acesso** | Por índice `get(i)` | Sem índice | Por chave `get(key)` |
| **Uso típico** | Lista de entregas | Placas únicas | Preço por produto |
| **Herança** | `Collection` | `Collection` | Independente |

### Quando usar cada um

| Situação | Use |
|----------|-----|
| Sequência ordenada com acesso por posição | `List` |
| Coleção sem duplicatas | `Set` |
| Associar chave a valor | `Map` |
| Contar ocorrências | `Map<T, Integer>` |
| Agrupar elementos | `Map<K, List<V>>` |
| Cache | `Map<K, V>` |
| Eliminar duplicatas de uma lista | Converter para `Set` |

### Conversões úteis

```java
// Map → Set de chaves
Set<String> chaves = mapa.keySet();

// Map → Collection de valores
Collection<Double> valores = mapa.values();

// Map → Set de entradas
Set<Map.Entry<String, Double>> entradas = mapa.entrySet();

// List → Map (contagem)
List<String> itens = Arrays.asList("A", "B", "A", "C", "B", "A");
Map<String, Long> contagem = itens.stream()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
// {A=3, B=2, C=1}

// Map → List (de entradas)
List<Map.Entry<String, Double>> lista = new ArrayList<>(mapa.entrySet());
```

---

## 13. Boas práticas

### Faça

- Sempre implemente `hashCode` e `equals` em classes usadas como chave em `HashMap`/`LinkedHashMap`
- Implemente `Comparable` ou passe `Comparator` para chaves de `TreeMap`
- Use `Map.of()` ou `Map.ofEntries()` (Java 9+) para maps imutáveis
- Programe para a interface `Map<K, V>`, não para a implementação
- Use `getOrDefault()` para evitar `NullPointerException`
- Prefira `merge()` ou `compute()` ao invés de `get` + `put` manual

### Evite

- Não modifique campos usados em `hashCode`/`equals` de chaves já inseridas
- Não assuma ordem em `HashMap`
- Não use `TreeMap` com chaves que não são `Comparable` (sem passar `Comparator`)
- Evite chaves mutáveis — prefira `String`, `Integer` ou objetos imutáveis

### Anti-pattern vs Best practice

```java
// Anti-pattern: get + null check + put
Integer count = mapa.get(chave);
if (count == null) {
    mapa.put(chave, 1);
} else {
    mapa.put(chave, count + 1);
}

// Best practice: merge
mapa.merge(chave, 1, Integer::sum);
```

```java
// Anti-pattern: get + null check para lista
List<String> lista = mapa.get(chave);
if (lista == null) {
    lista = new ArrayList<>();
    mapa.put(chave, lista);
}
lista.add(valor);

// Best practice: computeIfAbsent
mapa.computeIfAbsent(chave, k -> new ArrayList<>()).add(valor);
```

---

## 14. Tabela comparativa

| Aspecto | HashMap | TreeMap | LinkedHashMap |
|---------|---------|---------|---------------|
| **Estrutura** | Tabela hash | Árvore rubro-negra | Hash + lista ligada |
| **put/get/remove** | O(1) | O(log n) | O(1) |
| **Ordenação** | Nenhuma | Natural ou Comparator | Inserção |
| **Chave null** | Aceita (1x) | Não aceita* | Aceita (1x) |
| **Valor null** | Aceita | Aceita | Aceita |
| **Thread-safe** | Não | Não | Não |
| **Igualdade da chave** | hashCode + equals | compareTo | hashCode + equals |
| **Quando usar** | Padrão, mais rápido | Precisa ordenar chaves | Ordem de inserção |

*\* TreeMap não aceita chave null.*

### Versões thread-safe

| Precisa de thread-safety? | Use |
|--------------------------|-----|
| Acesso concorrente pesado | `ConcurrentHashMap` |
| Sincronização simples | `Collections.synchronizedMap(new HashMap<>())` |
| Leitura frequente, escrita rara | `ConcurrentHashMap` |

---

## 15. Exercícios

### Exercício 1 — Consolidação de votos de urnas

Na contagem de votos de uma eleição para "Melhor Motorista do Mês", são gerados vários registros de votação contendo o nome do candidato e a quantidade de votos (formato CSV) que ele obteve em cada setor. Faça um programa que leia os registros de votação a partir de um arquivo e gere um relatório consolidado com os totais de cada candidato, em ordem alfabética.

**Entrada (arquivo):**
```
João Silva,15
Maria Santos,22
Pedro Lima,21
João Silva,30
Pedro Lima,15
Maria Santos,27
Maria Santos,22
Pedro Lima,25
João Silva,31
```

**Saída esperada:**
```
João Silva: 76
Maria Santos: 71
Pedro Lima: 61
```

### Exercício 2 — Controle de estoque do armazém

Crie um programa que simule o controle de estoque de um armazém logístico. O programa deve:
- Permitir **entrada** de produtos (nome + quantidade)
- Permitir **saída** de produtos (verificar disponibilidade)
- Mostrar **relatório** do estoque atual (ordenado por nome)
- Mostrar produto com **maior** e **menor** quantidade

### Exercício 3 — Mapeamento de entregas por cidade

Um sistema recebe registros de entregas (cidade, valor). Crie um programa que:
- Acumule o valor total de entregas por cidade
- Mostre o ranking das cidades por valor total (do maior para o menor)
- Calcule a média de valor por cidade

### Exercício 4 — Agenda de motoristas por turno

Crie um `Map<String, Set<String>>` onde a chave é o turno (Manhã, Tarde, Noite) e o valor é o conjunto de motoristas alocados. Implemente:
- Alocar motorista a um turno
- Verificar se um motorista está em algum turno
- Listar todos os motoristas (sem repetição) e em quantos turnos cada um trabalha
- Remover motorista de um turno específico

---

## 16. Resumo

```
Map em Java
│
├── O quê? → Coleção de pares CHAVE/VALOR (chave única)
│
├── Implementações
│   ├── HashMap → O(1), sem ordem, usa hashCode/equals
│   ├── TreeMap → O(log n), chaves ordenadas, usa compareTo/Comparator
│   └── LinkedHashMap → O(1), ordem de inserção
│
├── Métodos principais
│   ├── put, get, remove, containsKey
│   ├── size, isEmpty, clear
│   ├── keySet, values, entrySet
│   ├── getOrDefault, putIfAbsent
│   ├── merge, compute, computeIfAbsent
│   └── replaceAll, forEach
│
├── Chaves: hashCode + equals
│   ├── Tipos padrão já implementam (String, Integer...)
│   ├── Classes personalizadas DEVEM implementar
│   └── Nunca modificar campos do hash após inserção
│
├── Iteração
│   ├── for-each com keySet() ou entrySet()
│   ├── forEach (Java 8+)
│   └── Stream API
│
├── Diferença para Set e List
│   ├── List → sequência com índice
│   ├── Set → conjunto sem duplicatas
│   └── Map → associação chave/valor
│
└── Na logística
    ├── Rastreamento (código → status)
    ├── Estoque (produto → quantidade)
    ├── Fretes (rota → preço)
    ├── Contagem de entregas por motorista
    ├── Cache de rotas calculadas
    └── Agrupamento por região/turno
```