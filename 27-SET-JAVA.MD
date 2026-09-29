# Set em Java 

## Sumário

1. [O que é Set?](#1-o-que-é-set)
2. [Características fundamentais](#2-características-fundamentais)
3. [Implementações de Set](#3-implementações-de-set)
4. [HashSet](#4-hashset)
5. [TreeSet](#5-treeset)
6. [LinkedHashSet](#6-linkedhashset)
7. [Métodos principais](#7-métodos-principais)
8. [Operações de conjunto](#8-operações-de-conjunto)
9. [hashCode e equals](#9-hashcode-e-equals)
10. [Como Set testa igualdade](#10-como-set-testa-igualdade)
11. [Comparable e Comparator com TreeSet](#11-comparable-e-comparator-com-treeset)
12. [Iteração sobre Set](#12-iteração-sobre-set)
13. [Casos de uso na Logística](#13-casos-de-uso-na-logística)
14. [Boas práticas](#14-boas-práticas)
15. [Tabela comparativa](#15-tabela-comparativa)
16. [Exercícios](#16-exercícios)
17. [Resumo](#17-resumo)

---

## 1. O que é Set?

`Set<T>` é uma coleção que representa um **conjunto de elementos** (similar ao conceito de conjunto da Álgebra). A principal característica: **não admite elementos repetidos**.

**Analogia logística:** Pense no **cadastro de placas de veículos** de uma frota. Não faz sentido ter a mesma placa cadastrada duas vezes. Se você tentar cadastrar "ABC-1234" duas vezes, o sistema ignora a segunda tentativa. Isso é um Set.

```java
Set<String> placas = new HashSet<>();
placas.add("ABC-1234");
placas.add("DEF-5678");
placas.add("ABC-1234"); // Ignorado! Já existe

System.out.println(placas.size()); // 2
```

---

## 2. Características fundamentais

| Característica | Descrição | Analogia logística |
|---------------|-----------|-------------------|
| **Sem repetições** | Não aceita elementos duplicados | Cadastro de CNPJs de fornecedores |
| **Sem posição** | Elementos não possuem índice | Diferente de uma lista de entregas ordenada |
| **Operações rápidas** | Acesso, inserção e remoção são rápidos | Consulta rápida de placa na frota |
| **Operações de conjunto** | União, interseção, diferença | Comparar frotas de filiais |

### Set vs List

```java
// List — aceita duplicatas, tem posição
List<String> listaRotas = new ArrayList<>();
listaRotas.add("SP-RJ");
listaRotas.add("SP-RJ"); // OK, lista aceita
System.out.println(listaRotas.size()); // 2
System.out.println(listaRotas.get(0)); // OK, tem índice

// Set — rejeita duplicatas, sem posição
Set<String> setRotas = new HashSet<>();
setRotas.add("SP-RJ");
setRotas.add("SP-RJ"); // Ignorado
System.out.println(setRotas.size()); // 1
// setRotas.get(0); // ERRO! Set não tem get por índice
```

---

## 3. Implementações de Set

O Java oferece três implementações principais de `Set`:

```
        Set<T>  (interface)
       /    |    \
  HashSet  TreeSet  LinkedHashSet
```

| Implementação | Estrutura interna | Complexidade | Ordenação |
|--------------|-------------------|-------------|-----------|
| **HashSet** | Tabela hash | O(1) | Sem ordem definida |
| **TreeSet** | Árvore rubro-negra | O(log n) | Ordenado (natural ou Comparator) |
| **LinkedHashSet** | Hash + lista ligada | O(1)* | Ordem de inserção |

*\* Ligeiramente mais lento que HashSet por manter a lista ligada.*

**Analogia:** Imagine três formas de organizar **crachás de motoristas**:
- **HashSet** → jogados em uma caixa (rápido para buscar, sem ordem)
- **TreeSet** → organizados em ordem alfabética em um fichário
- **LinkedHashSet** → empilhados na ordem em que chegaram

---

## 4. HashSet

A implementação mais utilizada. Usa **tabela hash** internamente.

### Quando usar

- Quando a **ordem não importa**
- Quando você precisa de **máxima velocidade** em add/remove/contains
- Quando os elementos implementam `hashCode` e `equals` corretamente

### Exemplo: cadastro de motoristas ativos

```java
Set<String> motoristasAtivos = new HashSet<>();
motoristasAtivos.add("João Silva");
motoristasAtivos.add("Maria Santos");
motoristasAtivos.add("Pedro Lima");
motoristasAtivos.add("João Silva"); // Ignorado

System.out.println(motoristasAtivos);
// Saída: [Pedro Lima, Maria Santos, João Silva] — ordem NÃO garantida

System.out.println(motoristasAtivos.contains("Maria Santos")); // true
System.out.println(motoristasAtivos.size()); // 3
```

### Como HashSet funciona internamente

```
Elemento → hashCode() → posição no array (bucket) → equals() para confirmar
```

1. Calcula o `hashCode()` do elemento
2. Determina o bucket (posição no array interno)
3. Se o bucket estiver vazio → insere
4. Se o bucket tiver elemento(s) → usa `equals()` para verificar duplicata

---

## 5. TreeSet

Mantém os elementos **ordenados**. Usa **árvore rubro-negra** internamente.
**árvore rubro-negra** = Ou ***red-black tree*** é uma estrutura de dados de árvore binária de busca auto-balanceada que o **TreeSet** utiliza nos bastidores para armazenar e ordenar seus elementos.

### Quando usar

- Quando você precisa dos elementos **ordenados**
- Quando precisa de métodos como `first()`, `last()`, `headSet()`, `tailSet()`
- Aceita performance ligeiramente menor (O(log n) vs O(1))

### Exemplo: rotas ordenadas

```java
Set<String> rotas = new TreeSet<>();
rotas.add("SP-BH");
rotas.add("SP-RJ");
rotas.add("SP-BA");
rotas.add("SP-RJ"); // Ignorado

System.out.println(rotas);
// Saída: [SP-BA, SP-BH, SP-RJ] — ordenado alfabeticamente!
```

### Métodos exclusivos do TreeSet

```java
TreeSet<Integer> distancias = new TreeSet<>(Arrays.asList(430, 120, 340, 550, 210));

System.out.println(distancias.first());      // 120 (menor)
System.out.println(distancias.last());       // 550 (maior)
System.out.println(distancias.lower(340));   // 210 (menor que 340)
System.out.println(distancias.higher(340));  // 430 (maior que 340)
System.out.println(distancias.headSet(340)); // [120, 210] (menores que 340)
System.out.println(distancias.tailSet(340)); // [340, 430, 550] (>= 340)
```

### Requisito: Comparable ou Comparator

TreeSet **exige** que os elementos sejam comparáveis:

```java
// Opção 1: classe implementa Comparable
public class Veiculo implements Comparable<Veiculo> {
    private String placa;
    private double capacidadeKg;
    
    @Override
    public int compareTo(Veiculo outro) {
        return this.placa.compareToIgnoreCase(outro.placa);
    }
}

Set<Veiculo> frota = new TreeSet<>(); // OK, Veiculo é Comparable

// Opção 2: passar Comparator no construtor
Set<Veiculo> frotaPorCapacidade = new TreeSet<>(
    Comparator.comparingDouble(Veiculo::getCapacidadeKg)
);
```

---

## 6. LinkedHashSet

Mantém a **ordem de inserção**. Combina hash table com lista duplamente ligada.

### Quando usar

- Quando a **ordem de inserção** importa
- Quando precisa de velocidade similar ao HashSet
- Quando quer iterar na ordem em que os elementos foram adicionados

### Exemplo: histórico de paradas (sem repetição)

```java
Set<String> paradas = new LinkedHashSet<>();
paradas.add("São Paulo");
paradas.add("Campinas");
paradas.add("Ribeirão Preto");
paradas.add("Campinas"); // Ignorado
paradas.add("Uberlândia");

System.out.println(paradas);
// Saída: [São Paulo, Campinas, Ribeirão Preto, Uberlândia] — ordem de inserção!
```

---

## 7. Métodos principais

### Métodos básicos

```java
Set<String> cidades = new HashSet<>();

// Adicionar
cidades.add("São Paulo");    // true (adicionou)
cidades.add("São Paulo");    // false (já existia)

// Verificar
cidades.contains("São Paulo"); // true
cidades.isEmpty();              // false
cidades.size();                 // 1

// Remover
cidades.remove("São Paulo");   // true (removeu)
cidades.remove("Curitiba");    // false (não existia)

// Limpar
cidades.clear();
```

### removeIf — remoção condicional

```java
Set<Integer> pesosKg = new HashSet<>(Arrays.asList(100, 500, 1200, 50, 800));

// Remover pesos menores que 200 kg
pesosKg.removeIf(peso -> peso < 200);

System.out.println(pesosKg); // [500, 800, 1200]
```

**Analogia logística:** Remover automaticamente da frota todos os veículos com vistoria vencida:

```java
Set<Veiculo> frota = new HashSet<>(/* veículos */);
frota.removeIf(v -> v.getVistoriaVencida());
```

---

## 8. Operações de conjunto

Set oferece operações de conjunto da Álgebra.

### Dados de exemplo

```java
Set<Integer> frotaFilialSP = new TreeSet<>(Arrays.asList(101, 102, 103, 104, 105));
Set<Integer> frotaFilialRJ = new TreeSet<>(Arrays.asList(103, 104, 105, 106, 107));
```

### União (`addAll`)

Combina os elementos dos dois conjuntos, sem repetição.

```java
Set<Integer> uniao = new TreeSet<>(frotaFilialSP);
uniao.addAll(frotaFilialRJ);
System.out.println("União: " + uniao);
// União: [101, 102, 103, 104, 105, 106, 107]
```

**Analogia:** Listar **todos os veículos** que pertencem a SP **ou** RJ.

### Interseção (`retainAll`)

Mantém apenas os elementos presentes em **ambos** os conjuntos.

```java
Set<Integer> intersecao = new TreeSet<>(frotaFilialSP);
intersecao.retainAll(frotaFilialRJ);
System.out.println("Interseção: " + intersecao);
// Interseção: [103, 104, 105]
```

**Analogia:** Listar os veículos **compartilhados** entre SP e RJ.

### Diferença (`removeAll`)

Remove do primeiro conjunto os elementos que também estão no segundo.

```java
Set<Integer> diferenca = new TreeSet<>(frotaFilialSP);
diferenca.removeAll(frotaFilialRJ);
System.out.println("Diferença (SP - RJ): " + diferenca);
// Diferença (SP - RJ): [101, 102]
```

**Analogia:** Listar os veículos **exclusivos** de SP (que não estão em RJ).

### Visualização

```
SP:  {101, 102, 103, 104, 105}
RJ:  {103, 104, 105, 106, 107}

União:      {101, 102, 103, 104, 105, 106, 107}
Interseção: {103, 104, 105}
Diferença:  {101, 102}
```

---

## 9. hashCode e equals

### O que são?

São métodos herdados de `Object` usados para **comparar objetos**.

| Método | Velocidade | Precisão | Uso |
|--------|-----------|----------|-----|
| `hashCode()` | Rápido | Pode ter colisão | Pré-filtro rápido |
| `equals()` | Lento | 100% preciso | Confirmação final |

### equals

Compara se dois objetos são **logicamente iguais**, retornando `true` ou `false`.

```java
String a = "Maria";
String b = "Alex";
System.out.println(a.equals(b)); // false
```

### hashCode

Retorna um número inteiro que representa um **código gerado** a partir das informações do objeto.

```java
String a = "Maria";
String b = "Alex";
System.out.println(a.hashCode()); // 74113750
System.out.println(b.hashCode()); // 2043454
```

### Regra de ouro do hashCode

1. Se o hashCode de dois objetos for **diferente** → os objetos são **definitivamente diferentes**
2. Se o hashCode de dois objetos for **igual** → os objetos são **provavelmente iguais** (pode haver colisão)

> hashCode **diferente** = objetos **diferentes** (nunca falha)
> hashCode **igual** = objetos **provavelmente iguais** (pode haver colisão)

### Tipos que já implementam

`String`, `Integer`, `Double`, `Date`, `LocalDate` etc. já possuem `hashCode` e `equals`. **Classes personalizadas precisam sobrepô-los.**

### Implementação personalizada

```java
public class Motorista {
    private String cpf;
    private String nome;
    
    public Motorista(String cpf, String nome) {
        this.cpf = cpf;
        this.nome = nome;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(cpf);
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Motorista outro = (Motorista) obj;
        return Objects.equals(cpf, outro.cpf);
    }
}
```

Agora dois `Motorista` com o mesmo CPF são considerados iguais, independente do nome:

```java
Motorista m1 = new Motorista("123.456.789-00", "João");
Motorista m2 = new Motorista("123.456.789-00", "João Silva");

System.out.println(m1.equals(m2));    // true
System.out.println(m1.hashCode() == m2.hashCode()); // true

Set<Motorista> motoristas = new HashSet<>();
motoristas.add(m1);
motoristas.add(m2); // Ignorado — mesmo CPF
System.out.println(motoristas.size()); // 1
```

---

## 10. Como Set testa igualdade

### HashSet (e LinkedHashSet)

```
1. Calcula hashCode() do novo elemento
2. Compara com hashCode() dos elementos no mesmo bucket
3. Se hashCode diferente → elemento é novo → insere
4. Se hashCode igual → chama equals() para confirmar
5. Se equals() retorna true → duplicata → ignora
6. Se equals() retorna false → colisão → insere
```

### Sem hashCode e equals implementados

Se a classe **NÃO** sobrepõe `hashCode` e `equals`, o Set compara **referências (ponteiros)**:

```java
public class Produto {
    private String nome;
    private Double preco;
    // SEM hashCode e equals!
}

Set<Produto> estoque = new HashSet<>();
estoque.add(new Produto("Notebook", 1200.0));
estoque.add(new Produto("Notebook", 1200.0));

System.out.println(estoque.size()); // 2! São objetos diferentes na memória
```

### Com hashCode e equals implementados

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

Set<Produto> estoque = new HashSet<>();
estoque.add(new Produto("Notebook", 1200.0));
estoque.add(new Produto("Notebook", 1200.0));

System.out.println(estoque.size()); // 1! Agora reconhece como duplicata
```

### TreeSet

TreeSet **NÃO** usa `hashCode`/`equals`. Usa `compareTo()` (ou `Comparator`) para determinar igualdade:

```java
public class Produto implements Comparable<Produto> {
    private String nome;
    private Double preco;
    
    @Override
    public int compareTo(Produto outro) {
        return this.nome.toUpperCase().compareTo(outro.nome.toUpperCase());
    }
}

Set<Produto> estoque = new TreeSet<>();
estoque.add(new Produto("TV", 900.0));
estoque.add(new Produto("Notebook", 1200.0));
estoque.add(new Produto("Tablet", 400.0));

// Saída ordenada por nome:
// [Notebook, Tablet, TV]
```

---

## 11. Comparable e Comparator com TreeSet

### Comparable — ordenação natural

```java
public class Rota implements Comparable<Rota> {
    private String origem;
    private String destino;
    private double distanciaKm;
    
    @Override
    public int compareTo(Rota outra) {
        return Double.compare(this.distanciaKm, outra.distanciaKm);
    }
}

Set<Rota> rotas = new TreeSet<>();
rotas.add(new Rota("SP", "RJ", 430));
rotas.add(new Rota("SP", "BH", 580));
rotas.add(new Rota("SP", "CT", 410));
// Ordenado por distância: [SP-CT(410), SP-RJ(430), SP-BH(580)]
```

### Comparator — ordenação personalizada

```java
// Ordenar por destino ao invés de distância
Set<Rota> rotasPorDestino = new TreeSet<>(
    Comparator.comparing(Rota::getDestino)
);

// Ordenar por distância decrescente
Set<Rota> rotasMaisLongas = new TreeSet<>(
    Comparator.comparingDouble(Rota::getDistanciaKm).reversed()
);

// Ordenar por múltiplos critérios
Set<Rota> rotasMultiplo = new TreeSet<>(
    Comparator.comparing(Rota::getOrigem)
              .thenComparing(Rota::getDestino)
);
```

---

## 12. Iteração sobre Set

### for-each

```java
Set<String> centrosDistribuicao = new HashSet<>(
    Arrays.asList("CD-SP", "CD-RJ", "CD-BH")
);

for (String cd : centrosDistribuicao) {
    System.out.println(cd);
}
```

### Iterator

```java
Iterator<String> it = centrosDistribuicao.iterator();
while (it.hasNext()) {
    String cd = it.next();
    if (cd.equals("CD-BH")) {
        it.remove(); // Remoção segura durante iteração
    }
}
```

### Stream API

```java
centrosDistribuicao.stream()
    .filter(cd -> cd.startsWith("CD-S"))
    .forEach(System.out::println);
```

### Converter Set para List (quando precisa de índice)

```java
List<String> lista = new ArrayList<>(centrosDistribuicao);
String primeiro = lista.get(0);
```

---

## 13. Casos de uso na Logística

### 1. Registro de acessos únicos ao sistema de rastreamento

```java
public class RegistroAcessos {
    
    public static int contarUsuariosDistintos(String caminhoArquivo) throws IOException {
        Set<String> usuarios = new HashSet<>();
        
        try (Scanner sc = new Scanner(new File(caminhoArquivo))) {
            while (sc.hasNextLine()) {
                String linha = sc.nextLine();
                String usuario = linha.split(" ")[0];
                usuarios.add(usuario);
            }
        }
        
        return usuarios.size();
    }
}
```

Arquivo de entrada:
```
joao.motorista 2024-01-15T08:30:00Z
maria.despachante 2024-01-15T09:15:00Z
joao.motorista 2024-01-15T10:45:00Z
pedro.gerente 2024-01-15T11:00:00Z
maria.despachante 2024-01-15T14:30:00Z
```

```java
int total = RegistroAcessos.contarUsuariosDistintos("acessos.txt");
System.out.println("Usuários distintos: " + total); // 3
```

### 2. Controle de CNPJs de fornecedores por filial

```java
public class ControleFornecedores {
    
    public static void main(String[] args) {
        Set<String> fornecedoresSP = new HashSet<>(
            Arrays.asList("11.111.111/0001-01", "22.222.222/0001-02", "33.333.333/0001-03")
        );
        Set<String> fornecedoresRJ = new HashSet<>(
            Arrays.asList("22.222.222/0001-02", "33.333.333/0001-03", "44.444.444/0001-04")
        );
        
        // Fornecedores que atendem AMBAS as filiais
        Set<String> compartilhados = new HashSet<>(fornecedoresSP);
        compartilhados.retainAll(fornecedoresRJ);
        System.out.println("Compartilhados: " + compartilhados);
        // [22.222.222/0001-02, 33.333.333/0001-03]
        
        // Fornecedores exclusivos de SP
        Set<String> exclusivosSP = new HashSet<>(fornecedoresSP);
        exclusivosSP.removeAll(fornecedoresRJ);
        System.out.println("Exclusivos SP: " + exclusivosSP);
        // [11.111.111/0001-01]
        
        // TODOS os fornecedores (sem repetição)
        Set<String> todos = new HashSet<>(fornecedoresSP);
        todos.addAll(fornecedoresRJ);
        System.out.println("Total fornecedores: " + todos.size()); // 4
    }
}
```

### 3. Alunos em cursos de treinamento (contar únicos)

```java
public class TreinamentoMotoristas {
    
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        Set<Integer> alunosTotais = new HashSet<>();
        String[] cursos = {"Direção Defensiva", "Primeiros Socorros", "Carga Perigosa"};
        
        for (String curso : cursos) {
            System.out.print("Quantos alunos no curso " + curso + "? ");
            int n = sc.nextInt();
            
            for (int i = 0; i < n; i++) {
                System.out.print("Matrícula: ");
                int matricula = sc.nextInt();
                alunosTotais.add(matricula);
            }
        }
        
        System.out.println("Total de motoristas em treinamento: " + alunosTotais.size());
        sc.close();
    }
}
```

### 4. Cidades atendidas pela transportadora (ordenadas)

```java
public class CidadesAtendidas {
    
    public static void main(String[] args) {
        Set<String> cidades = new TreeSet<>();
        
        // Adiciona cidades de diversas rotas
        cidades.add("São Paulo");
        cidades.add("Rio de Janeiro");
        cidades.add("Belo Horizonte");
        cidades.add("São Paulo");     // duplicata ignorada
        cidades.add("Curitiba");
        cidades.add("Rio de Janeiro"); // duplicata ignorada
        cidades.add("Florianópolis");
        
        System.out.println("Cidades atendidas (ordem alfabética):");
        for (String cidade : cidades) {
            System.out.println("  - " + cidade);
        }
        // - Belo Horizonte
        // - Curitiba
        // - Florianópolis
        // - Rio de Janeiro
        // - São Paulo
    }
}
```

### 5. Validação de placas duplicadas em manifesto de carga

```java
public class ValidadorManifesto {
    
    public static List<String> encontrarDuplicatas(List<String> placas) {
        Set<String> vistas = new HashSet<>();
        Set<String> duplicatas = new LinkedHashSet<>();
        
        for (String placa : placas) {
            if (!vistas.add(placa)) {
                duplicatas.add(placa);
            }
        }
        
        return new ArrayList<>(duplicatas);
    }
    
    public static void main(String[] args) {
        List<String> manifesto = Arrays.asList(
            "ABC-1234", "DEF-5678", "ABC-1234", "GHI-9012", "DEF-5678"
        );
        
        List<String> duplicadas = encontrarDuplicatas(manifesto);
        System.out.println("Placas duplicadas: " + duplicadas);
        // [ABC-1234, DEF-5678]
    }
}
```

---

## 14. Boas práticas

### Faça

- Sempre implemente `hashCode` e `equals` em classes que serão usadas em `HashSet`/`LinkedHashSet`
- Implemente `Comparable` ou passe `Comparator` para classes que serão usadas em `TreeSet`
- Use `Set.of("a", "b", "c")` (Java 9+) para sets imutáveis
- Programe para a interface `Set<T>`, não para a implementação
- Use `Collections.unmodifiableSet()` para retornar sets somente leitura

### Evite

- Não modifique campos usados em `hashCode`/`equals` após inserir no Set
- Não assuma ordem em `HashSet`
- Não use `TreeSet` sem `Comparable`/`Comparator` — lança `ClassCastException`
- Não use `get(index)` — Set não tem índice

### Escolha rápida

| Precisa de... | Use |
|--------------|-----|
| Velocidade máxima | `HashSet` |
| Elementos ordenados | `TreeSet` |
| Ordem de inserção | `LinkedHashSet` |
| Set imutável | `Set.of()` ou `Collections.unmodifiableSet()` |

---

## 15. Tabela comparativa

| Aspecto | HashSet | TreeSet | LinkedHashSet |
|---------|---------|---------|---------------|
| **Estrutura** | Tabela hash | Árvore rubro-negra | Hash + lista ligada |
| **add/remove/contains** | O(1) | O(log n) | O(1) |
| **Ordenação** | Nenhuma | Natural ou Comparator | Inserção |
| **null** | Aceita (1x) | Não aceita* | Aceita (1x) |
| **Thread-safe** | Não | Não | Não |
| **Igualdade** | hashCode + equals | compareTo | hashCode + equals |
| **Quando usar** | Padrão, mais rápido | Precisa ordenar | Ordem de inserção |

*\* TreeSet não aceita null a partir do Java 7.*

---

## 16. Exercícios

### Exercício 1 — Log de rastreamento de entregas

Um sistema de rastreamento registra as entregas realizadas por cada motorista. O registro contém o nome do motorista e o timestamp. Faça um programa que leia o arquivo de log e informe **quantos motoristas distintos** realizaram entregas.

**Entrada (arquivo):**
```
carlos 2024-03-15T08:00:00Z
ana 2024-03-15T09:30:00Z
carlos 2024-03-15T11:00:00Z
marcos 2024-03-15T14:00:00Z
ana 2024-03-15T16:00:00Z
carlos 2024-03-15T17:30:00Z
```

**Saída esperada:**
```
Motoristas distintos: 3
```

### Exercício 2 — Rotas compartilhadas entre transportadoras

Duas transportadoras operam rotas identificadas por códigos. Crie um programa que:
- Leia as rotas da Transportadora A e da Transportadora B
- Mostre as rotas **exclusivas** de cada uma
- Mostre as rotas **compartilhadas**
- Mostre o **total de rotas** (sem repetição)

### Exercício 3 — Veículos por capacidade (TreeSet)

Crie uma classe `Veiculo` com placa e capacidade (kg). Implemente `Comparable` para ordenar por capacidade. Use `TreeSet` para manter a frota ordenada e:
- Exibir a frota em ordem crescente de capacidade
- Mostrar o veículo com menor capacidade
- Mostrar o veículo com maior capacidade
- Listar veículos com capacidade acima de 2000 kg

### Exercício 4 — Detecção de pedidos duplicados

Um sistema recebe pedidos de entrega que podem vir duplicados por falha de integração. Crie uma classe `Pedido` (com número e destino) que implemente `hashCode`/`equals` pelo número. Use `Set` para filtrar duplicatas e retornar apenas pedidos únicos.

---

## 17. Resumo

```
Set em Java
│
├── O quê? → Coleção SEM duplicatas e SEM posição
│
├── Implementações
│   ├── HashSet → O(1), sem ordem, usa hashCode/equals
│   ├── TreeSet → O(log n), ordenado, usa compareTo/Comparator
│   └── LinkedHashSet → O(1), ordem de inserção
│
├── hashCode e equals
│   ├── hashCode → código numérico rápido (pré-filtro)
│   ├── equals → comparação precisa (confirmação)
│   ├── Tipos padrão já implementam (String, Integer...)
│   └── Classes personalizadas DEVEM implementar
│
├── Operações de conjunto
│   ├── addAll → União
│   ├── retainAll → Interseção
│   └── removeAll → Diferença
│
├── Métodos principais
│   ├── add, remove, contains
│   ├── size, isEmpty, clear
│   └── removeIf (remoção condicional)
│
└── Na logística
    ├── Cadastro de placas/CNPJs (sem duplicatas)
    ├── Controle de acessos únicos
    ├── Comparação de frotas entre filiais
    ├── Cidades atendidas (ordenadas)
    └── Detecção de duplicatas em manifesto
```