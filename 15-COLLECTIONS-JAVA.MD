# Collections em Java - LoL Edition

Collections são estruturas de dados muito mais poderosas e flexíveis que arrays. São a ferramenta ideal para trabalhar com grupos de objetos.

---

## 1. Arrays vs. Collections

### O Problema com Arrays

```java
// ❌ Problemas:
String[] campeoes = new String[3];
campeoes[0] = "Garen";
campeoes[1] = "Lux";
campeoes[2] = "Ahri";

// 1. Tamanho fixo
// Quer adicionar um 4º campeão? Cria novo array inteiro!

// 2. Sem métodos úteis
// Quer remover um elemento? Faz manualmente!

// 3. Sem type safety
Object[] misturado = {"Garen", 123, true};  // Sem erro!
```

### A Solução: Collections

```java
// ✓ Solução:
List<String> campeoes = new ArrayList<>();
campeoes.add("Garen");       // Tamanho dinâmico
campeoes.add("Lux");
campeoes.add("Ahri");

campeoes.remove("Lux");      // Remove facilmente
campeoes.size();             // Tamanho
campeoes.contains("Garen");  // Busca

// ✓ Type safety
// List<String> só aceita String
```

---

## 2. List<T>: Coleção Ordenada

### ArrayList: Implementação Mais Comum

```java
// Criar
List<String> campeoes = new ArrayList<>();

// Adicionar (ordem é preservada)
campeoes.add("Garen");
campeoes.add("Lux");
campeoes.add("Ahri");

// Acessar (tem índice como array)
System.out.println(campeoes.get(0));  // "Garen"
System.out.println(campeoes.get(1));  // "Lux"

// Remover
campeoes.remove(1);  // Remove "Lux"

// Verificar tamanho
System.out.println(campeoes.size());  // 2

// Verificar se contém
if (campeoes.contains("Garen")) {
    System.out.println("Garen está no time!");
}

// Iterar
for (String campeao : campeoes) {
    System.out.println(campeao);
}
```

### ArrayList com Objetos

```java
List<Campeao> time = new ArrayList<>();

time.add(new Campeao("Garen", 650));
time.add(new Campeao("Lux", 450));
time.add(new Campeao("Ahri", 495));

// Encontrar campeão por nome
for (Campeao c : time) {
    if (c.getNome().equals("Lux")) {
        System.out.println("Encontrado: " + c);
    }
}

// Remover
time.removeIf(c -> c.getNome().equals("Lux"));

// Modificar
for (Campeao c : time) {
    c.ganharExp(100);
}
```

### LinkedList: Melhor para Inserção/Remoção

```java
// ArrayList: bom para acesso (O(1))
List<String> lista1 = new ArrayList<>();

// LinkedList: bom para inserção/remoção (O(1) no começo/fim)
List<String> lista2 = new LinkedList<>();
lista2.add("Garen");
lista2.addFirst("Raiz");  // Adiciona no começo
lista2.addLast("Fim");    // Adiciona no fim
```

---

## 3. Set<T>: Coleção Sem Duplicatas

### HashSet: Ordem Aleatória

```java
Set<String> campeoes = new HashSet<>();

campeoes.add("Garen");
campeoes.add("Lux");
campeoes.add("Lux");  // Duplicata ignorada

System.out.println(campeoes.size());  // 2, não 3

// Iteração (ordem não garantida)
for (String c : campeoes) {
    System.out.println(c);
}

// Operações de conjunto
Set<String> set2 = new HashSet<>(Arrays.asList("Lux", "Ahri"));

// Interseção
campeoes.retainAll(set2);  // Mantém apenas elementos em set2

// União
Set<String> uniao = new HashSet<>(campeoes);
uniao.addAll(set2);

// Diferença
campeoes.removeAll(set2);  // Remove elementos que estão em set2
```

### TreeSet: Ordem Alfabética

```java
Set<String> campeoes = new TreeSet<>();

campeoes.add("Garen");
campeoes.add("Lux");
campeoes.add("Ahri");

// Iteração em ordem alfabética
for (String c : campeoes) {
    System.out.println(c);
}
// Output: Ahri, Garen, Lux
```

---

## 4. Map<K, V>: Pares Chave-Valor

### HashMap: Mais Comum

```java
// Criar
Map<String, Integer> kills = new HashMap<>();

// Adicionar (chave -> valor)
kills.put("Garen", 15);
kills.put("Lux", 22);
kills.put("Ahri", 18);

// Acessar
System.out.println(kills.get("Lux"));  // 22

// Verificar se tem chave
if (kills.containsKey("Garen")) {
    System.out.println("Garen está aqui!");
}

// Remover
kills.remove("Lux");

// Tamanho
System.out.println(kills.size());  // 2

// Iterar pelas chaves
for (String campeao : kills.keySet()) {
    System.out.println(campeao + ": " + kills.get(campeao));
}

// Iterar pelos valores
for (Integer kill : kills.values()) {
    System.out.println("Kills: " + kill);
}

// Iterar por pares
for (Map.Entry<String, Integer> entry : kills.entrySet()) {
    System.out.println(entry.getKey() + " - " + entry.getValue());
}
```

### TreeMap: Chaves Ordenadas

```java
Map<String, Integer> stats = new TreeMap<>();

stats.put("Garen", 650);
stats.put("Lux", 450);
stats.put("Ahri", 495);

// Iteração em ordem alfabética (chaves)
for (String campeao : stats.keySet()) {
    System.out.println(campeao + ": " + stats.get(campeao));
}
```

### Exemplo Prático: Time com Stats

```java
// Armazenar stats de cada campeão
Map<String, Map<String, Integer>> time = new HashMap<>();

// Garen: kills, deaths, assists
Map<String, Integer> statsGaren = new HashMap<>();
statsGaren.put("kills", 15);
statsGaren.put("deaths", 3);
statsGaren.put("assists", 8);

time.put("Garen", statsGaren);

// Acessar
System.out.println(time.get("Garen").get("kills"));  // 15

// Ou mais simples: Map<String, Campeao>
Map<String, Campeao> timePorNome = new HashMap<>();
timePorNome.put("Garen", new Campeao("Garen", 650));
timePorNome.put("Lux", new Campeao("Lux", 450));

Campeao garen = timePorNome.get("Garen");
```

---

## 5. Queue<T>: Fila (FIFO)

### LinkedList como Queue

```java
Queue<String> fila = new LinkedList<>();

// Adicionar (enqueue)
fila.add("Garen");
fila.add("Lux");
fila.add("Ahri");

// Remover primeiro (dequeue)
String primeiro = fila.remove();  // "Garen"

// Ver primeiro sem remover
String proximo = fila.peek();  // "Lux"

// Tamanho
System.out.println(fila.size());
```

### PriorityQueue: Fila com Prioridade

```java
// Ordem natural (menor primeiro)
Queue<Integer> prioridades = new PriorityQueue<>();

prioridades.add(50);  // Média prioridade
prioridades.add(10);  // Alta prioridade
prioridades.add(100); // Baixa prioridade

// Remove em ordem de prioridade
System.out.println(prioridades.remove());  // 10 (maior prioridade)
System.out.println(prioridades.remove());  // 50
System.out.println(prioridades.remove());  // 100
```

---

## 6. Collections Framework Completo

```
Collection
├── List (ordenada, permite duplicata)
│   ├── ArrayList
│   └── LinkedList
├── Set (sem duplicata)
│   ├── HashSet
│   └── TreeSet
└── Queue (FIFO, LIFO)
    ├── LinkedList
    └── PriorityQueue

Map (chave-valor, sem duplicata de chave)
├── HashMap
└── TreeMap
```

---

## 7. Métodos Úteis de Collections

### Converter Entre Tipos

```java
// Array → List
String[] array = {"Garen", "Lux"};
List<String> lista = Arrays.asList(array);

// List → Array
List<String> lista2 = new ArrayList<>();
String[] array2 = lista2.toArray(new String[0]);

// Outras conversões
Set<String> set = new HashSet<>(lista);
List<String> lista3 = new ArrayList<>(set);
```

### Ordenar

```java
List<String> campeoes = new ArrayList<>();
campeoes.add("Garen");
campeoes.add("Lux");
campeoes.add("Ahri");

// Ordenar alfabeticamente
Collections.sort(campeoes);

// Ordenar invertido
Collections.sort(campeoes, Collections.reverseOrder());

// Ordenar objetos customizados
List<Campeao> times = new ArrayList<>();
// Collections.sort(times);  // Precisa implementar Comparable

// Embaralhar
Collections.shuffle(campeoes);
```

### Buscar e Procurar

```java
List<Integer> kills = Arrays.asList(10, 20, 15, 25, 30);

// Elemento máximo
int max = Collections.max(kills);  // 30

// Elemento mínimo
int min = Collections.min(kills);  // 10

// Frequência
List<String> nomes = Arrays.asList("Garen", "Lux", "Garen");
int freq = Collections.frequency(nomes, "Garen");  // 2
```

---

## 8. Exemplo Prático Completo

```java
public class Time {
    private List<Campeao> integrantes;
    private Map<String, Integer> bancos;  // Nome -> Ouro
    
    public Time() {
        this.integrantes = new ArrayList<>();
        this.bancos = new HashMap<>();
    }
    
    // Adicionar campeão
    public void adicionarCampeao(Campeao c) {
        if (integrantes.size() >= 5) {
            System.out.println("Time cheio!");
            return;
        }
        integrantes.add(c);
        bancos.put(c.getNome(), 0);  // Começar com 0 ouro
    }
    
    // Remover campeão
    public void removerCampeao(String nome) {
        integrantes.removeIf(c -> c.getNome().equals(nome));
        bancos.remove(nome);
    }
    
    // Adicionar ouro
    public void adicionarOuro(String nome, int quantidade) {
        if (bancos.containsKey(nome)) {
            bancos.put(nome, bancos.get(nome) + quantidade);
        }
    }
    
    // Listar time
    public void listarTime() {
        System.out.println("=== Time ===");
        for (Campeao c : integrantes) {
            int ouro = bancos.getOrDefault(c.getNome(), 0);
            System.out.println(c.getNome() + " - Ouro: " + ouro);
        }
    }
    
    // Campeão com mais ouro
    public String getCampeaoMaisRico() {
        return bancos.entrySet()
            .stream()
            .max((e1, e2) -> e1.getValue().compareTo(e2.getValue()))
            .map(e -> e.getKey())
            .orElse("Nenhum");
    }
    
    // Ouro total
    public int getOuroTotal() {
        return bancos.values().stream().mapToInt(Integer::intValue).sum();
    }
    
    public static void main(String[] args) {
        Time time = new Time();
        
        time.adicionarCampeao(new Campeao("Garen", 650));
        time.adicionarCampeao(new Campeao("Lux", 450));
        time.adicionarCampeao(new Campeao("Ahri", 495));
        
        time.adicionarOuro("Garen", 1500);
        time.adicionarOuro("Lux", 2300);
        time.adicionarOuro("Ahri", 1800);
        
        time.listarTime();
        
        System.out.println("\nCampeão mais rico: " + time.getCampeaoMaisRico());
        System.out.println("Ouro total do time: " + time.getOuroTotal());
    }
}

// Output:
// === Time ===
// Garen - Ouro: 1500
// Lux - Ouro: 2300
// Ahri - Ouro: 1800
//
// Campeão mais rico: Lux
// Ouro total do time: 5600
```

---

## 9. Performance: Qual Usar?

| Estrutura | Acesso | Inserção | Remoção | Duplicata | Ordem |
|-----------|--------|----------|---------|-----------|-------|
| **ArrayList** | O(1) | O(n) | O(n) | Sim | Sim |
| **LinkedList** | O(n) | O(1) | O(1) | Sim | Sim |
| **HashSet** | O(1)* | O(1)* | O(1)* | Não | Não |
| **TreeSet** | O(log n) | O(log n) | O(log n) | Não | Sim |
| **HashMap** | O(1)* | O(1)* | O(1)* | Não (chaves) | Não |
| **TreeMap** | O(log n) | O(log n) | O(log n) | Não (chaves) | Sim |

**Regra prática:**
- Precisa de **ordem**? → **ArrayList ou LinkedList**
- Precisa de **sem duplicatas**? → **HashSet ou TreeSet**
- Precisa de **pares chave-valor**? → **HashMap ou TreeMap**

---

## 10. Erros Comuns

### ConcurrentModificationException

```java
// ❌ Remover enquanto itera
List<String> campeoes = new ArrayList<>();
campeoes.add("Garen");
campeoes.add("Lux");

for (String c : campeoes) {
    if (c.equals("Lux")) {
        campeoes.remove(c);  // Erro!
    }
}

// ✓ Usar Iterator ou removeIf
campeoes.removeIf(c -> c.equals("Lux"));
```

### NullPointerException com Collections

```java
// ❌ Esqueceu de inicializar
List<String> campeoes = null;
campeoes.add("Garen");  // NullPointerException!

// ✓ Inicializar antes
List<String> campeoes = new ArrayList<>();
campeoes.add("Garen");
```

### Misturar Tipos em Map

```java
// ❌ Ruim: sem genéricos
Map map = new HashMap();
map.put("kills", 15);
map.put("mana", 400.5);
Object valor = map.get("kills");
int kills = (int) valor;  // Possível ClassCastException

// ✓ Bom: com genéricos
Map<String, Integer> stats = new HashMap<>();
stats.put("kills", 15);
int kills = stats.get("kills");  // Type-safe
```

---

## Resumo

| Estrutura | Melhor Para |
|-----------|-----------|
| **ArrayList** | Acesso frequente |
| **LinkedList** | Muita inserção/remoção |
| **HashSet** | Sem duplicata, ordem não importa |
| **TreeSet** | Sem duplicata + ordenado |
| **HashMap** | Pares chave-valor |
| **TreeMap** | Pares chave-valor + ordenado |

---

## Dicas Finais

✓ **Sempre use Collections** em vez de arrays  
✓ Use **genéricos** para type-safety  
✓ **ArrayList** é seu amigo (padrão)  
✓ Use **Map** para buscar por chave (muito mais rápido)  
✓ Escolha a estrutura certa para a **tarefa**  

Collections transformam seu código em um profissional! 🎮
