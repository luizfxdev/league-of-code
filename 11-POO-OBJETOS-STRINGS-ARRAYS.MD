# Objetos, Strings e Arrays em Java - LoL Edition

Conteúdo aprofundado sobre instanciação de objetos, manipulação de strings, arrays e conceitos complementares essenciais para POO.

---

## 1. Instanciação de Objetos

### O que é Instanciação?

Instanciação é o processo de **criar um objeto concreto** a partir de uma classe. É como usar um molde para fabricar um produto real.

```
Classe (Molde) ─────> new ─────> Objeto (Produto Real)
```

### Processo em Detalhes

Quando você escreve `new Campeao()`, Java executa 4 passos:

```java
public class Campeao {
    private String nome;
    private int hp;
    
    // Construtor
    public Campeao(String nome, int hp) {
        this.nome = nome;
        this.hp = hp;
    }
}

// Instanciação: Criando um novo objeto Campeao
Campeao garen = new Campeao("Garen", 650);
```

**O que acontece nos bastidores:**

1. **`new`** → Aloca espaço na memória (Heap) para o novo objeto
2. **`Campeao()`** → Construtor é chamado para inicializar atributos
3. **Atribuição** → A referência do objeto é armazenada na variável `garen`
4. **Retorno** → O objeto está pronto para uso

### Referências vs. Objetos

Aqui está um conceito crítico: a **variável armazena uma referência**, não o objeto em si.

```java
// Criando dois campeões
Campeao garen = new Campeao("Garen", 650);
Campeao darius = new Campeao("Darius", 625);

// Agora garen e darius são duas REFERÊNCIAS diferentes
// apontando para dois OBJETOS diferentes na memória

System.out.println(garen.getNome());  // "Garen"
System.out.println(darius.getNome()); // "Darius"
```

### Copiar Referências vs. Criar Novos Objetos

```java
Campeao campeao1 = new Campeao("Lux", 450);
Campeao campeao2 = campeao1;  // ⚠ Cópia da REFERÊNCIA

campeao2.setHp(100);
System.out.println(campeao1.getHp());  // 100 (mudou!)

// Por quê? campeao1 e campeao2 apontam para o MESMO objeto
```

**Visualizando:**

```
Antes: campeao1 ────────────────> Objeto Lux (HP: 450)
       campeao2                   

Depois: campeao1 ────────────────> Objeto Lux (HP: 100)
        campeao2 ────────────────/

# Ambas as referências apontam para o MESMO objeto!
```

Para criar um objeto **realmente independente**, você precisa instanciar novamente:

```java
Campeao campeao1 = new Campeao("Lux", 450);
Campeao campeao2 = new Campeao("Lux", 450);  // Novo objeto, mesmo nome

campeao2.setHp(100);
System.out.println(campeao1.getHp());  // 450 (não mudou!)

// Agora são dois objetos diferentes apontados por duas referências diferentes
```

### Null: A Referência Vazia

Às vezes, uma variável não aponta para nenhum objeto:

```java
Campeao campeao = null;  // Não aponta para nada

System.out.println(campeao.getNome());  // ❌ NullPointerException!
```

**Sempre verifique antes de usar:**

```java
if (campeao != null) {
    System.out.println(campeao.getNome());  // ✓ Seguro
}

// Ou com operador ternário
String nome = (campeao != null) ? campeao.getNome() : "Desconhecido";
```

### Stack vs. Heap: Onde Tudo é Armazenado?

```java
public void criarTimeFinal() {
    Campeao garen = new Campeao("Garen", 650);
    int level = 18;
}

// Stack (armazena referências e primitivos)
// ┌─────────────────────────┐
// │ garen ──┐               │
// │ level 18│               │
// └─────────┼───────────────┘
//           │
//           │ aponta para
//           │
// Heap (armazena objetos)
// ┌─────────────────────────┐
// │ Campeao                 │
// │ nome: "Garen"           │
// │ hp: 650                 │
// └─────────────────────────┘
```

**Regras importantes:**

- **Tipos primitivos** (`int`, `double`, `boolean`) vivem na **Stack**
- **Objetos** e **Strings** vivem na **Heap**
- **Referências** (variáveis que apontam para objetos) vivem na **Stack**
- Quando a variável sai do escopo, a referência é deletada, mas o objeto na Heap pode ser coletado pelo **Garbage Collector**

---

## 2. Strings em Java

### O que é uma String?

Uma **String** é uma sequência de caracteres (texto). Em Java, é uma **classe**, não um tipo primitivo.

```java
// String é um objeto, não um primitivo
String nome = "Ahri";
int level = 18;  // primitivo

// nome aponta para um objeto String na Heap
// level está na Stack
```

### Características: Imutabilidade

Uma das características mais importantes de String é que **são imutáveis**: uma vez criada, não pode ser modificada.

```java
String campeao = "Ahri";
campeao = "Lux";  // ⚠ Você não MODIFICOU a String
                  // Você criou uma NOVA String e atribuiu à variável

// A String "Ahri" continua existindo na memória
// (até o Garbage Collector limpar)
```

**Por que isso importa?**

```java
String nome = "Garen";
String nomeAnterior = nome;

nome = "Darius";  // Apenas a referência mudou

System.out.println(nome);          // "Darius"
System.out.println(nomeAnterior);  // "Garen" ✓ Não foi afetado
```

### Criando Strings

```java
// Forma 1: Usando literal (mais eficiente)
String campeao1 = "Lux";

// Forma 2: Usando construtor (cria novo objeto na Heap)
String campeao2 = new String("Lux");

// Importante: campeao1 e campeao2 são diferentes na memória!
System.out.println(campeao1 == campeao2);         // false (referências diferentes)
System.out.println(campeao1.equals(campeao2));    // true (conteúdo igual)
```

**Use `==` para comparar referências, `.equals()` para comparar conteúdo:**

```java
String time1 = "Blue Team";
String time2 = "Blue Team";
String time3 = new String("Blue Team");

System.out.println(time1 == time2);       // true (mesma referência)
System.out.println(time1 == time3);       // false (referências diferentes)
System.out.println(time1.equals(time3));  // true (mesmo conteúdo)
```

### Principais Métodos de String

```java
String campeao = "Ahri";
String descricao = "Rainha Nórdica";

// Comprimento
int tamanho = campeao.length();  // 4

// Maiúsculas e minúsculas
String upper = campeao.toUpperCase();      // "AHRI"
String lower = descricao.toLowerCase();    // "rainha nórdica"

// Procurar caractere ou substring
int posicao = campeao.indexOf('h');        // 1
boolean contem = descricao.contains("Rei"); // true

// Extrair partes (substring)
String parte = campeao.substring(0, 2);    // "Ah"
String resto = campeao.substring(2);       // "ri"

// Remover espaços
String com_espacos = "  Lux  ";
String sem_espacos = com_espacos.trim();   // "Lux"

// Dividir string
String[] palavras = descricao.split(" ");  // ["Rainha", "Nórdica"]

// Substituir
String novo = campeao.replace("Ahri", "Lux");  // "Lux"

// Verificar começo e fim
boolean comeca = campeao.startsWith("Ah");     // true
boolean termina = campeao.endsWith("ri");      // true
```

### String Pool: Otimização de Memória

Java possui uma área especial chamada **String Pool** que armazena strings literais para reutilização:

```java
String campeao1 = "Garen";   // Criada no String Pool
String campeao2 = "Garen";   // Reutiliza a mesma do Pool

System.out.println(campeao1 == campeao2);  // true ✓ Mesma referência

// Mas com construtor, não:
String campeao3 = new String("Garen");
System.out.println(campeao1 == campeao3);  // false (nova na Heap)
```

---

## 3. Concatenação de Strings

### Operador +

A forma mais intuitiva é usar o operador `+`:

```java
String nome = "Ahri";
String tipo = "Mago";
int nivel = 18;

String resultado = nome + " é um " + tipo + " nível " + nivel;
System.out.println(resultado);  // "Ahri é um Mago nível 18"
```

**Como funciona nos bastidores:**

```java
String mensagem = "O campeão " + "Lux" + " tem " + 450 + " de HP";
```

Java converte isso para:

```java
// Java faz algo como:
String mensagem = new StringBuilder()
    .append("O campeão ")
    .append("Lux")
    .append(" tem ")
    .append(450)
    .append(" de HP")
    .toString();
```

### Método .concat()

Similar ao `+`, mas mais explícito:

```java
String nomeHeroi = "Garen";
String titulo = "O Campeão Desafiador";

String nomeCompleto = nomeHeroi.concat(" - ").concat(titulo);
System.out.println(nomeCompleto);  // "Garen - O Campeão Desafiador"
```

### StringBuilder: Para Muitas Concatenações

Quando você faz muitas concatenações em um loop, `StringBuilder` é **muito mais eficiente**:

```java
// ❌ Ineficiente
String resultado = "";
for (int i = 0; i < 1000; i++) {
    resultado = resultado + "Campeão " + i + "\n";
    // Cria 1000 Strings novas (desperdício!)
}

// ✓ Eficiente
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append("Campeão ").append(i).append("\n");
    // Apenas 1 StringBuilder
}
String resultado = sb.toString();
```

### String Formatting (Mais Legível)

Para mensagens complexas, use formatação:

```java
String campeao = "Ahri";
int kills = 25;
int deaths = 3;
double kda = (double) kills / deaths;

// Usando format()
String relatorio = String.format(
    "Campeão: %s | Kills: %d | Deaths: %d | KDA: %.2f",
    campeao, kills, deaths, kda
);
System.out.println(relatorio);
// "Campeão: Ahri | Kills: 25 | Deaths: 3 | KDA: 8.33"
```

**Especificadores:**
- `%s` = String
- `%d` = int
- `%f` = double
- `%.2f` = double com 2 casas decimais
- `%n` = quebra de linha (multiplataforma)

### Comparação de Abordagens

```java
// 1. Concatenação simples (OK para 1-2 strings)
String msg1 = campeao + " ganhou " + gold + " de ouro";

// 2. Método concat() (mais explícito)
String msg2 = "Campeão ".concat(campeao).concat(" venceu!");

// 3. StringBuilder (muitas concatenações)
StringBuilder sb = new StringBuilder();
sb.append("Campeão: ").append(campeao).append("\n");
String msg3 = sb.toString();

// 4. Format (mensagens estruturadas)
String msg4 = String.format("Campeão: %s | Ouro: %d", campeao, gold);
```

---

## 4. Arrays

### O que é um Array?

Um **array** é uma estrutura de dados que armazena **múltiplos valores do mesmo tipo** em posições sequenciais.

```
Array de Campeões:
┌──────┬──────┬──────┬──────┬──────┐
│Garen │Lux   │Ahri  │Darius│Akali │
│ [0]  │ [1]  │ [2]  │ [3]  │ [4]  │
└──────┴──────┴──────┴──────┴──────┘
  ↑
Índice começa do 0!
```

### Criando Arrays

```java
// Forma 1: Especificar tamanho
int[] niveis = new int[5];  // Array com 5 posições

// Forma 2: Inicializar com valores
String[] campeoes = {"Garen", "Lux", "Ahri", "Darius", "Akali"};

// Forma 3: Combinado
int[] kills = new int[]{10, 15, 8, 12, 20};

// Arrays de objetos
Campeao[] time = new Campeao[5];
time[0] = new Campeao("Garen", 650);
time[1] = new Campeao("Lux", 450);
```

### Acessando Elementos

```java
String[] campeoes = {"Garen", "Lux", "Ahri"};

// Leitura
System.out.println(campeoes[0]);  // "Garen"
System.out.println(campeoes[1]);  // "Lux"
System.out.println(campeoes[2]);  // "Ahri"

// Escrita
campeoes[1] = "Akali";
System.out.println(campeoes[1]);  // "Akali"

// Tamanho
System.out.println(campeoes.length);  // 3
```

### Iterando sobre Arrays

```java
String[] campeoes = {"Garen", "Lux", "Ahri", "Darius"};

// Forma 1: Loop tradicional (for)
for (int i = 0; i < campeoes.length; i++) {
    System.out.println(campeoes[i]);
}

// Forma 2: Enhanced for (for-each) - mais simples
for (String campeao : campeoes) {
    System.out.println(campeao);
}

// Forma 3: While
int i = 0;
while (i < campeoes.length) {
    System.out.println(campeoes[i]);
    i++;
}
```

### Arrays de Objetos: Exemplo Prático

```java
public class Campeao {
    private String nome;
    private int hp;
    
    public Campeao(String nome, int hp) {
        this.nome = nome;
        this.hp = hp;
    }
    
    public String getNome() { return nome; }
    public int getHp() { return hp; }
}

// Criando um time
public class Time {
    private Campeao[] integrantes;
    
    public Time(Campeao[] integrantes) {
        this.integrantes = integrantes;
    }
    
    public void listarCampeoes() {
        for (Campeao campeao : integrantes) {
            System.out.println(campeao.getNome() + " (" + campeao.getHp() + " HP)");
        }
    }
    
    public int getTotalHp() {
        int total = 0;
        for (Campeao campeao : integrantes) {
            total += campeao.getHp();
        }
        return total;
    }
}

// Usando:
Campeao[] teamBlue = {
    new Campeao("Garen", 650),
    new Campeao("Lux", 450),
    new Campeao("Ahri", 495)
};

Time time = new Time(teamBlue);
time.listarCampeoes();
System.out.println("HP total: " + time.getTotalHp());  // 1595
```

### Arrays Multidimensionais

Arrays de arrays! Úteis para matrizes:

```java
// Array 2D (matriz 3x3)
int[][] mapa = new int[3][3];

mapa[0][0] = 1;
mapa[0][1] = 2;
mapa[1][2] = 3;

// Ou inicializar direto:
String[][] tabuleiro = {
    {"Torre", "Inibidor", "Nexus"},
    {"Criadores", "Campeão", "Estruturas"},
    {"Jogo", "Estratégia", "Equipe"}
};

// Acessar
System.out.println(tabuleiro[0][2]);  // "Nexus"

// Iterar (for-each aninhado)
for (String[] linha : tabuleiro) {
    for (String celula : linha) {
        System.out.print(celula + " ");
    }
    System.out.println();
}
```

### Problemas Comuns com Arrays

```java
String[] campeoes = {"Garen", "Lux"};

// ❌ ArrayIndexOutOfBoundsException
System.out.println(campeoes[5]);  // Só tem índices 0 e 1

// ❌ Confundir tamanho com índice
int tamanho = campeoes.length;    // 2
int ultimoIndice = tamanho - 1;   // 1
System.out.println(campeoes[ultimoIndice]);  // "Lux" ✓

// ❌ Array null
String[] times = null;
System.out.println(times.length);  // NullPointerException
```

---

## 5. Conceitos Complementares Importantes

### Tipos Primitivos vs. Tipos de Referência

```java
// PRIMITIVOS (vivem na Stack)
int numero = 42;
double decimal = 3.14;
boolean ativo = true;
char letra = 'A';

// REFERÊNCIA (variável aponta para Heap)
String nome = "Lux";
int[] numeros = {1, 2, 3};
Campeao heroi = new Campeao("Garen", 650);

// Diferença crítica:
int a = 10;
int b = a;
b = 20;
System.out.println(a);  // 10 (não afetado)

Campeao c1 = new Campeao("Lux", 450);
Campeao c2 = c1;
c2.setHp(100);
System.out.println(c1.getHp());  // 100 (afetado!)
```

### Escopo de Variáveis

O escopo define **onde uma variável existe e pode ser acessada**:

```java
public class Game {
    private int roundas = 0;  // Escopo: toda a classe
    
    public void jogar() {
        int score = 100;  // Escopo: método jogar()
        
        if (true) {
            String mensagem = "Vitória!";  // Escopo: bloco if
        }
        
        // ❌ mensagem não existe aqui
        System.out.println(mensagem);  // Erro!
    }
    
    public void teste() {
        // ❌ score não existe aqui
        System.out.println(score);  // Erro!
    }
}
```

### Métodos: Entradas e Saídas

Um método recebe parâmetros (entrada) e retorna um valor (saída):

```java
public class Campeao {
    private int hp;
    
    // Método sem retorno (void)
    public void levarDano(int dano) {
        this.hp -= dano;
    }
    
    // Método com retorno
    public int getHp() {
        return this.hp;
    }
    
    // Método com múltiplos parâmetros
    public int atacar(Campeao alvo, int dano) {
        int danoFinal = dano;
        alvo.levarDano(danoFinal);
        return danoFinal;
    }
}

// Usando:
Campeao garen = new Campeao("Garen", 650);
Campeao lux = new Campeao("Lux", 450);

garen.atacar(lux, 100);
System.out.println(lux.getHp());  // 350
```

### Static vs. Instância

**Instância** = Cada objeto tem sua própria cópia

**Static** = Compartilhado por todos os objetos da classe

```java
public class Campeao {
    // Instância: cada campeão tem seu próprio nome
    private String nome;
    
    // Static: todos os campeões compartilham a mesma versão
    private static int totalCampeoesCriados = 0;
    
    public Campeao(String nome) {
        this.nome = nome;
        totalCampeoesCriados++;
    }
    
    public static int getTotalCriadosCriados() {
        return totalCampeoesCriados;
    }
}

// Usando:
Campeao garen = new Campeao("Garen");
Campeao lux = new Campeao("Lux");
Campeao ahri = new Campeao("Ahri");

System.out.println(Campeao.getTotalCampeoesCriados());  // 3
```

### This: Referência ao Próprio Objeto

`this` é uma palavra-chave especial que aponta para **o próprio objeto**:

```java
public class Campeao {
    private String nome;
    private int hp;
    
    public Campeao(String nome, int hp) {
        // this.nome = atributo da classe
        // nome = parâmetro do construtor
        this.nome = nome;
        this.hp = hp;
    }
    
    public void usarHabilidade(Campeao alvo) {
        // this = o campeão que está usando a habilidade
        // alvo = o alvo da habilidade
        System.out.println(this.nome + " atacou " + alvo.nome);
    }
}
```

---

## Resumo: Fluxo Completo

```java
// 1. CLASSE: O molde
public class Campeao {
    private String nome;
    private String[] habilidades;  // Array
    
    public Campeao(String nome) {
        this.nome = nome;
        this.habilidades = new String[]{"Q", "W", "E", "R"};
    }
    
    public String getDescricao() {
        // Concatenação de strings
        return this.nome + " tem " + habilidades.length + " habilidades";
    }
}

// 2. INSTANCIAÇÃO: Criar objetos reais
Campeao garen = new Campeao("Garen");
Campeao lux = new Campeao("Lux");

// 3. USANDO OBJETOS
System.out.println(garen.getDescricao());  // "Garen tem 4 habilidades"

// 4. ARRAYS: Múltiplos objetos
Campeao[] time = {garen, lux, new Campeao("Ahri")};

for (Campeao campeao : time) {
    System.out.println(campeao.getDescricao());
}
```

---

## Dicas Finais

✓ **Instanciação:** Sempre use `new` para criar objetos  
✓ **Null:** Sempre verifique antes de usar referências  
✓ **Strings:** Use `.equals()` para comparar, não `==`  
✓ **Arrays:** Lembre que índices começam do 0  
✓ **StringBuilder:** Use para múltiplas concatenações em loops  
✓ **Referências:** Entenda que cópias de referências apontam para o mesmo objeto  

Bora treinar! 🎮
