# Memória, Arrays e Listas em Java ? Guia Prático com League of Legends

## 1. Tipos Primitivos vs Tipos Referência

A diferença fundamental: como Java gerencia dados na memória.

### Tipos Primitivos (Tipo Valor) ? CAIXAS

São como **slots de inventário**. Cada variável é uma caixa física que guarda o valor.

```java
int meuDano = 150;
int seuDano = meuDano; // Cópia do valor

meuDano = 200;
System.out.println(seuDano); // Ainda 150 - cópia independente
```

**Na memória (Stack):**
```
meuDano: [150]
seuDano: [150]  ? Cópia separada
```

Quando você atribui `seuDano = meuDano`, ele **copia o conteúdo**, não compartilha a referência.

### Tipos Referência (Classes) ? PONTEIROS

São como **nomes de campeões na tela**. A variável não é o objeto, é um "tentáculo" apontando para ele.

```java
class Campeao {
    String nome;
    int hp;
}

Campeao p1 = new Campeao();
p1.nome = "Garen";
p1.hp = 650;

Campeao p2 = p1; // Ambas apontam para o MESMO objeto

p2.nome = "Changed";
System.out.println(p1.nome); // "Changed" - mesma referência!
```

**Na memória:**
```
Stack              Heap
p1 ???         [Campeao]
p2 ???????????? nome: "Changed"
     ?          hp: 650
```

Ambas `p1` e `p2` apontam para o mesmo Campeao no Heap.

---

### Resumo: A Diferença

| Aspecto | Primitivo | Referência |
|---------|-----------|------------|
| O que é | Valor direto | Ponteiro para objeto |
| Armazenamento | Stack | Heap |
| Cópia | Cria novo valor | Cria nova referência (mesmo obj) |
| Pode ser null? | Não | Sim |
| Exemplo | `int x = 5;` | `Campeao c = new Campeao();` |

---

## 2. Stack e Heap ? A Memória do Java

### Stack ? Rápido, Organizado

- Guarda **variáveis locais** e **tipos primitivos**
- Desaloca **automaticamente** quando sai do escopo
- Primeiro a entrar, último a sair (LIFO)
- Pequeno, mas rápido

### Heap ? Flexível, Lento

- Guarda **objetos** (referência)
- Não desaloca automaticamente
- Precisa do **Garbage Collector** para limpar
- Grande, mas mais lento

### Exemplo Visual: Laning Phase

```java
void jogarLaning() {
    int minuto = 10;              // Stack
    int ouro = 500;               // Stack
    
    Campeao meuChampion = new Campeao();  // Stack: referência
                                           // Heap: objeto
    meuChampion.ouro = ouro;
    
} // Ao sair: minuto, ouro, meuChampion desaparecem
  // Mas o objeto no Heap pode permanecer se alguém apontar para ele
```

---

## 3. Valor NULL ? O Vazio

Só tipos referência podem ser null.

```java
Campeao campeao = null; // OK - ninguém está usando

int numero = null; // ERRO - primitivos não aceitam null
```

### Analogia: Quando um campeão não foi escolhido

```java
class TimesGame {
    Campeao top = null;      // Vago
    Campeao mid = null;      // Vago
    Campeao adc = new Campeao(); // Escolhido
    
    void validar() {
        if (top == null) {
            System.out.println("Pick top!");
        }
    }
}
```

---

## 4. Garbage Collector ? Limpeza Automática

Java **monitora objetos** no Heap. Se ninguém aponta para um objeto, ele é deletado.

### Exemplo: Troca de Campeão

```java
Campeao p1 = new Campeao(); // Garen no Heap
p1.nome = "Garen";

Campeao p2 = new Campeao(); // Darius no Heap
p2.nome = "Darius";

p1 = p2; // p1 agora aponta para Darius

// Garen ficou sem referência
// Garbage Collector vai deletar Garen do Heap em breve
```

**Você não precisa fazer nada** ? Java cuida disso.

---

## 5. Escopo ? Quando Variáveis Morrem

Variáveis locais são desalocadas quando saem do escopo.

```java
void exemplo() {
    int x = 10; // Existe aqui
    
    if (x > 0) {
        int y = 20; // Existe apenas aqui
    } // y morre aqui
    
    System.out.println(y); // ERRO - y não existe mais
}
```

**Analogia:** Como um buff de curta duração. Quando o timer acaba, o efeito desaparece.

---

## 6. Arrays (Vetores) ? Estrutura Fixa

Array é uma **coleção de elementos do mesmo tipo**, **tamanho fixo**, em posições sequenciais.

### Declaração e Instanciação

```java
// Declaração
int[] notas;

// Instanciação
notas = new int[5]; // Array com 5 posições

// Os valores padrão são atribuídos automaticamente:
// int: 0
// double: 0.0
// boolean: false
// objeto: null
```

### Acesso aos Elementos

```java
int[] notas = new int[5];

notas[0] = 95;
notas[1] = 87;
notas[2] = 92;
notas[3] = 88;
notas[4] = 91;

System.out.println(notas[2]); // 92

// Propriedade length
System.out.println(notas.length); // 5
```

### Analogia LoL: Slots de Inventário

```java
// Inventário com 6 slots
Item[] inventario = new Item[6];

inventario[0] = new Item("Bota");
inventario[1] = new Item("Espada Longa");
inventario[2] = new Item("Cálice");
inventario[3] = null; // Vazio
inventario[4] = null;
inventario[5] = null;

// Acessar item do slot 1
System.out.println(inventario[1].nome); // Espada Longa
```

---

### Exemplo Prático: Média de Alturas

```java
import java.util.Locale;
import java.util.Scanner;

public class MediaAlturas {
    public static void main(String[] args) {
        Locale.setDefault(Locale.US);
        Scanner sc = new Scanner(System.in);
        
        int n = sc.nextInt();
        double[] alturas = new double[n];
        
        for (int i = 0; i < n; i++) {
            alturas[i] = sc.nextDouble();
        }
        
        double soma = 0.0;
        for (int i = 0; i < alturas.length; i++) {
            soma += alturas[i];
        }
        
        double media = soma / alturas.length;
        System.out.printf("MEDIA: %.2f%n", media);
        
        sc.close();
    }
}
```

---

## 7. Arrays de Objetos

Arrays podem conter referências a objetos.

### Exemplo: Array de Campeões

```java
class Campeao {
    String nome;
    int hp;
    
    Campeao(String nome, int hp) {
        this.nome = nome;
        this.hp = hp;
    }
}

public class Main {
    public static void main(String[] args) {
        Campeao[] time = new Campeao[5];
        
        time[0] = new Campeao("Garen", 650);
        time[1] = new Campeao("Lee Sin", 570);
        time[2] = new Campeao("Ahri", 550);
        time[3] = new Campeao("Ashe", 500);
        time[4] = new Campeao("Thresh", 490);
        
        for (int i = 0; i < time.length; i++) {
            System.out.println(time[i].nome + " - HP: " + time[i].hp);
        }
    }
}
```

### Inicialização com Valores Padrão

```java
Campeao[] time = new Campeao[3];
System.out.println(time[0]); // null - nenhum objeto ainda
```

---

## 8. For-Each ? Iteração Simplificada

Sintaxe mais limpa para percorrer arrays.

```java
String[] campeoes = {"Garen", "Darius", "Mordekaiser"};

// For tradicional
for (int i = 0; i < campeoes.length; i++) {
    System.out.println(campeoes[i]);
}

// For-each (sem índice)
for (String champ : campeoes) {
    System.out.println(champ);
}

// Ambos imprimem a mesma coisa
```

**Use for-each quando não precisa do índice. Use for tradicional quando precisa da posição.**

---

## 9. Boxing e Unboxing ? Conversão Automática

### Wrapper Classes

Cada tipo primitivo tem uma classe equivalente:

| Primitivo | Wrapper |
|-----------|---------|
| `int` | `Integer` |
| `double` | `Double` |
| `boolean` | `Boolean` |
| `char` | `Character` |

### Boxing ? Primitivo ? Referência

```java
int dano = 150;
Integer danoPack = dano; // Boxing automático

// Ou explícito
Integer danoPack = Integer.valueOf(dano);
```

### Unboxing ? Referência ? Primitivo

```java
Integer danoPack = 150;
int dano = danoPack; // Unboxing automático

// Ou explícito
int dano = danoPack.intValue();
```

### Por que usar Wrapper?

Em **banco de dados e entidades**, campos podem ser null:

```java
class Item {
    String nome;
    Double preco;      // Pode ser null (sem preço)
    Integer quantidade; // Pode ser null (desconhecido)
    
    // Em vez de:
    // double preco; (obrigado a ter valor)
}
```

---

## 10. Listas (List, ArrayList)

Array com **tamanho variável**. Ideal quando não sabe quantos elementos terá.

### Declaração e Instanciação

```java
import java.util.ArrayList;
import java.util.List;

List<String> campeoes = new ArrayList<>();

campeoes.add("Garen");
campeoes.add("Lee Sin");
campeoes.add("Ahri");

System.out.println(campeoes.size()); // 3
```

### Operações Principais

```java
List<String> campeoes = new ArrayList<>();

// Adicionar
campeoes.add("Garen");          // Adiciona no final
campeoes.add(1, "Lee Sin");     // Adiciona na posição 1

// Acessar
System.out.println(campeoes.get(0)); // Garen

// Encontrar posição
System.out.println(campeoes.indexOf("Lee Sin")); // 1

// Remover
campeoes.remove(0);             // Remove posição 0
campeoes.remove("Ahri");        // Remove por valor

// Verificar
System.out.println(campeoes.contains("Garen")); // true/false

// Tamanho
System.out.println(campeoes.size());

// Limpar
campeoes.clear();
```

---

### Array vs List ? Quando Usar

| Situação | Use |
|----------|-----|
| Sabe exatamente quantos elementos | Array |
| Vai adicionar/remover elementos | List |
| Quer acesso rápido por índice | Array |
| Quer facilidade | List |
| Quer performance | Array |

---

### Exemplo Prático: Lista de Campeões do Time

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

class Campeao {
    String nome;
    
    Campeao(String nome) {
        this.nome = nome;
    }
    
    public String toString() {
        return nome;
    }
}

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        List<Campeao> time = new ArrayList<>();
        
        System.out.print("Quantos campeões? ");
        int n = sc.nextInt();
        
        for (int i = 0; i < n; i++) {
            sc.nextLine();
            System.out.print("Nome: ");
            String nome = sc.nextLine();
            time.add(new Campeao(nome));
        }
        
        System.out.println("\nTime:");
        for (Campeao c : time) {
            System.out.println(c);
        }
        
        sc.close();
    }
}
```

---

## 11. Operações Avançadas em Listas

### RemoveIf ? Remover com Condição

```java
List<Integer> danos = new ArrayList<>();
danos.add(100);
danos.add(50);
danos.add(75);
danos.add(120);

// Remove danos menores que 60
danos.removeIf(x -> x < 60);

// Lista agora: [100, 75, 120]
```

### Stream e Filter ? Buscar com Critério

```java
List<String> campeoes = new ArrayList<>();
campeoes.add("Garen");
campeoes.add("Darius");
campeoes.add("Ahri");
campeoes.add("Annie");

// Encontra primeira que começa com 'A'
String resultado = campeoes.stream()
    .filter(x -> x.charAt(0) == 'A')
    .findFirst()
    .orElse(null);

System.out.println(resultado); // Ahri
```

---

## 12. Matrizes (Arrays Bidimensionais)

Matriz é um **"vetor de vetores"** ? tem linhas E colunas.

### Declaração e Acesso

```java
int[][] mapa = new int[3][4]; // 3 linhas, 4 colunas

mapa[0][0] = 1;
mapa[0][1] = 2;
mapa[1][0] = 5;

System.out.println(mapa[1][0]); // 5
```

### Propriedades

```java
int[][] mapa = new int[3][4];

System.out.println(mapa.length);      // 3 (linhas)
System.out.println(mapa[0].length);   // 4 (colunas da linha 0)
```

### Iteração com For Duplo

```java
int[][] terreno = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

for (int i = 0; i < terreno.length; i++) {
    for (int j = 0; j < terreno[i].length; j++) {
        System.out.print(terreno[i][j] + " ");
    }
    System.out.println();
}

// Saída:
// 1 2 3
// 4 5 6
// 7 8 9
```

---

### Analogia LoL: Mapa

```java
class MapaJogo {
    int[][] terreno = new int[12][12]; // Mapa de Summoner's Rift simplificado
    
    void inicializar() {
        terreno[0][0] = 1; // Torre aliada
        terreno[11][11] = 2; // Torre inimiga
    }
    
    void mostrarMapa() {
        for (int i = 0; i < terreno.length; i++) {
            for (int j = 0; j < terreno[i].length; j++) {
                System.out.print(terreno[i][j] + " ");
            }
            System.out.println();
        }
    }
}
```

---

### Exemplo: Diagonal Principal e Negativos

```java
int n = 3;
int[][] matriz = new int[n][n];

matriz[0][0] = 5;   matriz[0][1] = -3;  matriz[0][2] = 10;
matriz[1][0] = 15;  matriz[1][1] = 8;   matriz[1][2] = 2;
matriz[2][0] = 7;   matriz[2][1] = 9;   matriz[2][2] = -4;

// Diagonal principal (i == j)
System.out.println("Diagonal: ");
for (int i = 0; i < n; i++) {
    System.out.print(matriz[i][i] + " ");
}
System.out.println(); // 5 8 -4

// Contar negativos
int negativos = 0;
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
        if (matriz[i][j] < 0) {
            negativos++;
        }
    }
}
System.out.println("Negativos: " + negativos); // 2
```

---

## 13. Exercício Final: Sistema de Inventário Completo

Sistema de itens com array, lista e operações.

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

class Item {
    int id;
    String nome;
    double preco;
    
    Item(int id, String nome, double preco) {
        this.id = id;
        this.nome = nome;
        this.preco = preco;
    }
    
    public String toString() {
        return id + " - " + nome + ": R$ " + String.format("%.2f", preco);
    }
}

public class Inventario {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        List<Item> itens = new ArrayList<>();
        
        System.out.print("Quantos itens? ");
        int n = sc.nextInt();
        
        for (int i = 0; i < n; i++) {
            System.out.println("\nItem " + (i + 1));
            System.out.print("ID: ");
            int id = sc.nextInt();
            System.out.print("Nome: ");
            sc.nextLine();
            String nome = sc.nextLine();
            System.out.print("Preço: ");
            double preco = sc.nextDouble();
            
            itens.add(new Item(id, nome, preco));
        }
        
        System.out.println("\n--- INVENTÁRIO ---");
        for (Item item : itens) {
            System.out.println(item);
        }
        
        System.out.print("\nProcurar item por ID: ");
        int procura = sc.nextInt();
        Item encontrado = itens.stream()
            .filter(x -> x.id == procura)
            .findFirst()
            .orElse(null);
        
        if (encontrado != null) {
            System.out.println("Encontrado: " + encontrado);
        } else {
            System.out.println("Item não encontrado");
        }
        
        sc.close();
    }
}
```

---

## 14. Resumo Rápido

### Memória
- **Stack**: variáveis locais, primitivos, rápido
- **Heap**: objetos, gerenciado pelo GC

### Arrays
- Tamanho **fixo**, índice começa em **0**
- Acesso **rápido** mas sem flexibilidade

### Listas
- Tamanho **variável**, adicione/remova à vontade
- Mais flexível que array

### Tipos Primitivos vs Referência
- **Primitivo**: cópia de valor
- **Referência**: cópia de ponteiro (mesmo objeto)

### For-each
- Use quando não precisa do índice
- Mais limpo e legível

### Matrizes
- 2D = linhas + colunas
- Use loops aninhados para percorrer

