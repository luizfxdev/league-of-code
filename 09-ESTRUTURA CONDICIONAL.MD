# Estrutura Condicional em Java ? Guia Prático com League of Legends

## 1. Operadores Comparativos

Imagine você monitorando o **HP do campeão inimigo**. Você precisa tomar decisões baseadas em quanto de vida ele tem.

### Operadores Disponíveis

| Operador | Significado | Exemplo | Resultado |
|----------|-------------|---------|-----------|
| `>` | maior | `7 > 3` | `true` |
| `<` | menor | `7 < 3` | `false` |
| `>=` | maior ou igual | `7 >= 7` | `true` |
| `<=` | menor ou igual | `3 <= 7` | `true` |
| `==` | igual | `5 == 5` | `true` |
| `!=` | diferente | `5 != 3` | `true` |

### Analogia LoL

```java
int hpInimigo = 250;

// Está abaixo de 30% de vida? Pode "all-in"
if (hpInimigo < 300) {
    System.out.println("Engaje agora!");
}

// Está com mais vida que a minha? Espera cooldown
if (hpInimigo >= meuHp) {
    System.out.println("Aguarde...");
}
```

---

## 2. Operadores Lógicos

Agora você não toma decisões por apenas UM fator. Você precisa checar **múltiplas condições** antes de agir.

### Tipos de Operadores Lógicos

#### `&&` (E) ? TODAS as condições devem ser verdadeiras

**Analogia:** Para fazer **All-in** você precisa:
- **E** não estar sozinho (aliados perto)
- **E** ter cooldowns disponíveis
- **E** inimigo estar isolado

```java
boolean temaliadosPerto = true;
boolean cooldownDisponivel = true;
boolean inimigoIsolado = true;

if (temaliadosPerto && cooldownDisponivel && inimigoIsolado) {
    System.out.println("All-in! 5v5!");
}
```

**Tabela verdade do `&&`:**

| A | B | A && B |
|---|---|--------|
| V | V | **V**  |
| V | F | F      |
| F | V | F      |
| F | F | F      |

---

#### `||` (OU) ? PELO MENOS UMA condição deve ser verdadeira

**Analogia:** Você faz **recall** se:
- **OU** tiver mana baixa
- **OU** estiver com muita vida perdida
- **OU** a lane estiver segura

```java
boolean manaAlta = false;
boolean vidaAlta = true;
boolean laneSeguradoInimigo = false;

if (!manaAlta || !vidaAlta || laneSeguradoInimigo) {
    System.out.println("Fazendo recall...");
}
```

**Tabela verdade do `||`:**

| A | B | A \|\| B |
|---|---|----------|
| V | V | **V**    |
| V | F | **V**    |
| F | V | **V**    |
| F | F | F        |

---

#### `!` (NÃO) ? Inverte a condição

**Analogia:** Você dá **engage** se o inimigo **NÃO** está com ult disponível.

```java
boolean ultimateDisponivel = true;

if (!ultimateDisponivel) {
    System.out.println("Engaje! Ult dele não está pronta.");
}

// Com x = 5:
int x = 5;
System.out.println(!(x == 10)); // true (5 é diferente de 10)
System.out.println(!(x >= 2));  // false (5 é >= 2)
```

**Tabela verdade do `!`:**

| A | !A |
|---|-----|
| V | **F** |
| F | **V** |

---

### Exemplo Combinado: Decisão de Equipe

```java
int ouroEquipe = 8500;
int ouroInimigos = 7200;
boolean temaliadosDisponiveis = true;
boolean ultimatesDisponiveis = true;
boolean objetivo = true; // Baron/Dragon próximo

if ((ouroEquipe > ouroInimigos) && 
    (temaliadosDisponiveis || ultimatesDisponiveis) && 
    objetivo) {
    System.out.println("Vamos tomar o objetivo!");
} else {
    System.out.println("Farmar mais...");
}
```

---

## 3. Estrutura `if` e `if-else`

### `if` Simples ? Executa OU não executa

```java
int hpInimigo = 200;

if (hpInimigo < 300) {
    System.out.println("Engaje agora!");
}
```

**Fluxo:**
- Condição **verdadeira** ? executa o bloco
- Condição **falsa** ? pula o bloco

---

### `if-else` ? Executa um OU outro

```java
int ouroMeu = 2800;

if (ouroMeu >= 2800) {
    System.out.println("Comprar item legendário!");
} else {
    System.out.println("Ainda não tenho ouro suficiente.");
}
```

**Analogia LoL:**
- **if** = Ult carregada? Ataque com ult
- **else** = Ult não carregada? Ataque sem ult

---

## 4. `if-else if-else` ? Múltiplas Decisões

**Analogia:** No laning phase, sua ação depende da **composição inimiga**:

```java
String campeoninimigo = "Yasuo";

if (campeoninimigo.equals("Yasuo")) {
    System.out.println("Dodge com cuidado, ele knocka!");
} else if (campeoninimigo.equals("Blitzcrank")) {
    System.out.println("Dodge dos hooks!");
} else if (campeoninimigo.equals("Zed")) {
    System.out.println("Cuidado com o poke!");
} else {
    System.out.println("Jogue normal.");
}
```

---

### Exemplo Realista: Decisão de Objetivos

```java
int minuto = 25;

if (minuto < 10) {
    System.out.println("Foco: farm e harass");
} else if (minuto >= 10 && minuto < 15) {
    System.out.println("Prepare-se para primeiros teamfights");
} else if (minuto >= 15 && minuto < 25) {
    System.out.println("Rotacione, tome objetivos, farm");
} else if (minuto >= 25) {
    System.out.println("TEAMFIGHT por Baron/Nexo");
} else {
    System.out.println("Game over");
}
```

---

## 5. Estrutura `switch-case` ? Alternativa ao `if-else if`

Quando você tem **muitas opções baseadas em UM valor**, `switch` é mais limpo.

### Comparação: `if-else if` vs `switch`

**Com `if-else if`:**
```java
int role = 2; // 1=top, 2=jg, 3=mid, 4=adc, 5=sup
String funcao;

if (role == 1) {
    funcao = "Top - Farm e ganhe teamfights";
} else if (role == 2) {
    funcao = "Jungle - Gank e controle de mapa";
} else if (role == 3) {
    funcao = "Mid - Poke e teamfights centrais";
} else if (role == 4) {
    funcao = "ADC - Farm e dano no teamfight";
} else if (role == 5) {
    funcao = "Support - Proteção e engaje";
} else {
    funcao = "Role inválido";
}
```

**Com `switch`:**
```java
int role = 2;
String funcao;

switch (role) {
    case 1:
        funcao = "Top - Farm e ganhe teamfights";
        break;
    case 2:
        funcao = "Jungle - Gank e controle de mapa";
        break;
    case 3:
        funcao = "Mid - Poke e teamfights centrais";
        break;
    case 4:
        funcao = "ADC - Farm e dano no teamfight";
        break;
    case 5:
        funcao = "Support - Proteção e engaje";
        break;
    default:
        funcao = "Role inválido";
}

System.out.println(funcao);
```

### Regras Importantes do `switch`

1. **`break`** é obrigatório ? sem ele, "cai" para o próximo case
2. **`default`** é opcional ? executa se nenhum `case` bate
3. Funciona com `int`, `String`, `enum` e outros tipos primitivos

### Exemplo com `break` errado (Fall-through)

```java
int tier = 2; // 1=Iron, 2=Bronze, 3=Silver

switch (tier) {
    case 1:
        System.out.println("Iron");
    case 2:
        System.out.println("Bronze");  // Vai executar isto
    case 3:
        System.out.println("Silver");  // E isto também!
}

// Saída:
// Bronze
// Silver
```

**Solução:** Adicione `break`

```java
switch (tier) {
    case 1:
        System.out.println("Iron");
        break;
    case 2:
        System.out.println("Bronze");
        break;
    case 3:
        System.out.println("Silver");
        break;
}
```

---

## 6. Operador Ternário ? Uma linha para decisão simples

Use quando precisa **atribuir um valor** com base em uma condição.

### Sintaxe

```
(condição) ? valor_se_verdadeiro : valor_se_falso
```

### Exemplo

```java
int meuHp = 450;
int hpMaximo = 600;

// Com if-else
double percentualVida;
if (meuHp < hpMaximo / 2) {
    percentualVida = 0.3; // 30%
} else {
    percentualVida = 1.0; // 100%
}

// Com ternário (uma linha)
double percentualVida = (meuHp < hpMaximo / 2) ? 0.3 : 1.0;
```

### Analogia LoL: Efeito de Poção

```java
int hpAtual = 250;
int hpMaximo = 600;

// Se HP estiver abaixo de 40%, restaura 150
// Se não, restaura 100
int restauracao = (hpAtual < hpMaximo * 0.4) ? 150 : 100;

System.out.println("Restaurando: " + restauracao + " HP");
```

### Ternário Aninhado (use com cuidado!)

```java
int minuto = 12;

String fase = (minuto < 10) ? "Early Game" 
            : (minuto < 20) ? "Mid Game"
            : "Late Game";

System.out.println(fase); // Mid Game
```

---

## 7. Caso Prático Completo: Sistema de Compra de Items

Recriando a lógica da loja do LoL:

```java
import java.util.Locale;
import java.util.Scanner;

public class LojaItens {
    public static void main(String[] args) {
        Locale.setDefault(Locale.US);
        Scanner sc = new Scanner(System.in);
        
        int ouro = sc.nextInt();
        int itemDesejado = sc.nextInt();
        
        // 1 = Cálice (850), 2 = Bota (300), 3 = Espada Longa (1100)
        int precoItem = 0;
        String nomeItem = "";
        
        switch (itemDesejado) {
            case 1:
                precoItem = 850;
                nomeItem = "Cálice";
                break;
            case 2:
                precoItem = 300;
                nomeItem = "Bota";
                break;
            case 3:
                precoItem = 1100;
                nomeItem = "Espada Longa";
                break;
            default:
                nomeItem = "Item inválido";
        }
        
        // Pode comprar?
        if (ouro >= precoItem && precoItem > 0) {
            System.out.println("Item comprado: " + nomeItem);
            System.out.println("Ouro restante: " + (ouro - precoItem));
        } else {
            int ouroFaltante = precoItem - ouro;
            System.out.println("Ouro insuficiente!");
            System.out.println("Faltam: " + ouroFaltante + " ouro");
        }
        
        sc.close();
    }
}
```

**Entrada/Saída:**
```
Entrada: 1200 (ouro), 1 (item)
Saída: Item comprado: Cálice
       Ouro restante: 350

Entrada: 500 (ouro), 3 (item)
Saída: Ouro insuficiente!
       Faltam: 600 ouro
```

---

## 8. Operadores de Atribuição Cumulativa

Atalhos para operações comuns:

| Operador | Equivalente |
|----------|-------------|
| `a += b` | `a = a + b` |
| `a -= b` | `a = a - b` |
| `a *= b` | `a = a * b` |
| `a /= b` | `a = a / b` |
| `a %= b` | `a = a % b` |

### Exemplo: Consumo de Mana

```java
int mana = 300;
int manaPorHabilidade = 45;

// Usa abilidade Q
mana -= manaPorHabilidade; // Equivalente: mana = mana - 45
System.out.println("Mana restante: " + mana); // 255

// Regenera durante o recall (soma 100 mana por segundo)
mana += 100;
System.out.println("Mana após recall: " + mana); // 355
```

---

## 9. Escopo e Inicialização

**Regra de ouro:** Uma variável só existe **dentro do bloco** onde foi declarada.

### ? Correto

```java
double preco = sc.nextDouble();

if (preco > 100.0) {
    double desconto = preco * 0.1;
    System.out.println("Desconto: " + desconto);
}
```

### ? Errado

```java
double preco = sc.nextDouble();

if (preco > 100.0) {
    double desconto = preco * 0.1;
}

System.out.println(desconto); // ERRO! desconto não existe aqui
```

### Solução: Declare fora do `if`

```java
double preco = sc.nextDouble();
double desconto;

if (preco > 100.0) {
    desconto = preco * 0.1;
} else {
    desconto = preco * 0.05;
}

System.out.println("Desconto: " + desconto); // OK!
```

---

## 10. Resumo Rápido ? Quando Usar Cada Uma

| Situação | Use |
|----------|-----|
| Sim/Não simples | `if` |
| Sim ou Não, com alternativa | `if-else` |
| Múltiplas condições sequenciais | `if-else if-else` |
| Muitos valores de UMA variável | `switch-case` |
| Atribuir valor em uma linha | Ternário `?:` |
| Verificar VÁRIAS condições | `&&` ou `\|\|` |

---

## Exercício Final: Sistema de Runes

Implemente um programa que leia um tipo de runa (1=Ofensiva, 2=Defensiva, 3=Utilitária) e mostre qual é o benefício:

```java
// Dicas:
// - Use switch para cada tipo
// - Combine && ou || se precisar verificar múltiplas condições
// - Use ternário para cálculos rápidos
```

**Resposta sugerida:**

<details>
<summary>Clique para ver a solução</summary>

```java
import java.util.Scanner;

public class SistemaRunes {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("Digite o tipo de runa (1=Ofensiva, 2=Defensiva, 3=Utilitária): ");
        int tipoRuna = sc.nextInt();
        
        String beneficio;
        
        switch (tipoRuna) {
            case 1:
                beneficio = "Ataque: +15% dano";
                break;
            case 2:
                beneficio = "Defesa: +10% armadura";
                break;
            case 3:
                beneficio = "Utilidade: +5% velocidade de movimento";
                break;
            default:
                beneficio = "Runa inválida!";
        }
        
        System.out.println("Benefício: " + beneficio);
        sc.close();
    }
}
```

</details>

---

## Referências Rápidas

**Tabela de Prioridade de Operadores:**
1. `!` (NÃO)
2. `&&` (E)
3. `||` (OU)

**Use parênteses para deixar claro:**
```java
// Evite ambiguidade
if (a || b && c) { } // Qual tem prioridade?

// Deixe explícito
if (a || (b && c)) { } // Agora está claro!
```

---

**Última dica:** Debug suas condições imprimindo os valores intermediários. Use `System.out.println()` para ver o que está acontecendo.

```java
boolean condicao1 = meuHp < 300;
boolean condicao2 = ultimateDisponivel;

System.out.println("HP baixo? " + condicao1);
System.out.println("Ult pronta? " + condicao2);

if (condicao1 && condicao2) {
    System.out.println("Engaje!");
}
```