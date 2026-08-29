# Estruturas de Controle em Java - LoL Edition

Dominar fluxo de controle é essencial: qualquer programa precisa decidir e repetir ações.

---

## 1. If-Else: Decisões Simples

### Estrutura Básica

```java
if (condicao) {
    // Executado se condicao for true
}
```

### Exemplo: Campeão pode atacar?

```java
int hp = 450;

if (hp > 0) {
    System.out.println("Campeão está vivo, pode atacar!");
}
```

### If-Else: Dois Caminhos

```java
int hp = 0;

if (hp > 0) {
    System.out.println("Está vivo!");
} else {
    System.out.println("Desmaiado!");
}
```

### If-Else-If: Múltiplos Caminhos

Quando há **mais de dois cenários**, use `else if`:

```java
int nivel = 5;

if (nivel < 6) {
    System.out.println("Não tem acesso à ultimate!");
} else if (nivel >= 6 && nivel < 11) {
    System.out.println("Ultimate nível 1!");
} else if (nivel >= 11 && nivel < 16) {
    System.out.println("Ultimate nível 2!");
} else {
    System.out.println("Ultimate nível 3!");
}
```

### Problema: Aninhamento Profundo

```java
// ❌ Evitar: muito aninhado
if (estaVivo) {
    if (temMana) {
        if (temAlvo) {
            if (estaNoRange) {
                atacar();
            }
        }
    }
}

// ✓ Melhor: early return ou guards
if (!estaVivo) return;
if (!temMana) return;
if (!temAlvo) return;
if (!estaNoRange) return;

atacar();
```

---

## 2. Switch: Decisões por Valor

### Quando Usar Switch?

Quando você compara **o mesmo valor contra múltiplas opções**:

```java
// ❌ Ruim: muitos if-else
String tipo = "Mago";

if (tipo.equals("Tank")) {
    System.out.println("Resistência alta!");
} else if (tipo.equals("Mago")) {
    System.out.println("Poder mágico alto!");
} else if (tipo.equals("ADC")) {
    System.out.println("Dano físico alto!");
} else if (tipo.equals("Suporte")) {
    System.out.println("Cura alta!");
}

// ✓ Melhor: switch
switch (tipo) {
    case "Tank":
        System.out.println("Resistência alta!");
        break;
    case "Mago":
        System.out.println("Poder mágico alto!");
        break;
    case "ADC":
        System.out.println("Dano físico alto!");
        break;
    case "Suporte":
        System.out.println("Cura alta!");
        break;
    default:
        System.out.println("Tipo desconhecido!");
}
```

### Importante: `break`

```java
String tipo = "Mago";

// ❌ SEM break: continua executando!
switch (tipo) {
    case "Mago":
        System.out.println("Mago!");
        // Falta break aqui
    case "Tank":
        System.out.println("Tank!");  // Vai executar também!
        break;
}
// Output: "Mago!" "Tank!" (indesejado!)

// ✓ COM break: para no primeiro match
switch (tipo) {
    case "Mago":
        System.out.println("Mago!");
        break;  // Para aqui
    case "Tank":
        System.out.println("Tank!");
        break;
}
// Output: "Mago!" (correto!)
```

### Switch com Múltiplos Cases

```java
int dia = 1;

switch (dia) {
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
        System.out.println("Dia útil!");
        break;
    case 6:
    case 7:
        System.out.println("Fim de semana!");
        break;
}
```

### Switch Moderno (Java 12+)

```java
String tipo = "Mago";

// Sintaxe moderna com ->
String resultado = switch (tipo) {
    case "Tank" -> "Resistência alta!";
    case "Mago" -> "Poder mágico alto!";
    case "ADC" -> "Dano físico alto!";
    case "Suporte" -> "Cura alta!";
    default -> "Tipo desconhecido!";
};

System.out.println(resultado);  // "Poder mágico alto!"
```

---

## 3. Operador Ternário: If em Uma Linha

### Sintaxe

```java
condicao ? valor_se_verdadeiro : valor_se_falso
```

### Exemplos

```java
int hp = 450;

// Forma longa
String status;
if (hp > 0) {
    status = "Vivo";
} else {
    status = "Morto";
}

// Forma curta (ternário)
String status = (hp > 0) ? "Vivo" : "Morto";

// Aninhado (não recomendado)
String descricao = (hp > 250) ? "Saudável" : (hp > 100) ? "Ferido" : "Crítico";
```

### Caso Prático

```java
Campeao alvo = null;

// Procura por alvo
String mensagem = (alvo != null) ? 
    "Atacando " + alvo.getNome() : 
    "Sem alvo!";
```

---

## 4. For: Loops Fixos

### For Tradicional

```java
for (inicializacao; condicao; incremento) {
    // Executa enquanto condicao for true
}
```

### Exemplo: Listar Campeões

```java
String[] campeoes = {"Garen", "Lux", "Ahri", "Darius"};

for (int i = 0; i < campeoes.length; i++) {
    System.out.println(campeoes[i]);
}

// Output:
// Garen
// Lux
// Ahri
// Darius
```

### For-Each (Enhanced For)

Melhor quando você não precisa do índice:

```java
String[] campeoes = {"Garen", "Lux", "Ahri", "Darius"};

// ✓ Simples e legível
for (String campeao : campeoes) {
    System.out.println(campeao);
}

// Funciona com Arrays e Collections
List<String> times = new ArrayList<>();
times.add("Blue Team");
times.add("Red Team");

for (String time : times) {
    System.out.println(time);
}
```

### Comparação: Quando Usar Qual?

```java
// Use FOR TRADICIONAL quando precisa do índice
for (int i = 0; i < campeoes.length; i++) {
    System.out.println(i + ": " + campeoes[i]);
}
// Output: 0: Garen, 1: Lux, 2: Ahri...

// Use FOR-EACH quando só quer iterar
for (String campeao : campeoes) {
    System.out.println(campeao);
}
```

### Controle de Loop

```java
// BREAK: Sai do loop
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;  // Sai do loop
    }
    System.out.println(i);
}
// Output: 0 1 2 3 4

// CONTINUE: Pula para próxima iteração
for (int i = 0; i < 5; i++) {
    if (i == 2) {
        continue;  // Pula i=2
    }
    System.out.println(i);
}
// Output: 0 1 3 4
```

### Loops Aninhados: Cuidado!

```java
// ❌ Cuidado: O(n²) complexidade
for (int i = 0; i < 100; i++) {
    for (int j = 0; j < 100; j++) {
        // Executa 10.000 vezes!
    }
}

// ✓ Quando necessário (ex: matriz)
int[][] mapa = new int[5][5];
for (int linha = 0; linha < mapa.length; linha++) {
    for (int coluna = 0; coluna < mapa[linha].length; coluna++) {
        mapa[linha][coluna] = 0;
    }
}
```

---

## 5. While: Loops Condicionais

### Estrutura

```java
while (condicao) {
    // Executa enquanto condicao for true
}
```

### Exemplo: Simular Batalha

```java
int hpGaren = 650;
int hpLux = 450;
int danoGaren = 100;
int danoLux = 80;

while (hpGaren > 0 && hpLux > 0) {
    // Garen ataca
    hpLux -= danoGaren;
    System.out.println("Garen ataca! Lux agora tem " + hpLux + " HP");
    
    if (hpLux <= 0) break;
    
    // Lux ataca
    hpGaren -= danoLux;
    System.out.println("Lux ataca! Garen agora tem " + hpGaren + " HP");
}

if (hpGaren > 0) {
    System.out.println("Garen venceu!");
} else {
    System.out.println("Lux venceu!");
}
```

### Do-While: Executa Pelo Menos Uma Vez

```java
// WHILE: Verifica ANTES de executar
int tentativas = 0;
while (tentativas > 10) {
    System.out.println("Executar");  // Nunca executa
}

// DO-WHILE: Executa, DEPOIS verifica
int tentativas2 = 0;
do {
    System.out.println("Executar");  // Executa 1 vez
} while (tentativas2 > 10);
```

### Caso Prático: Menu

```java
Scanner scanner = new Scanner(System.in);
int opcao = 0;

do {
    System.out.println("1. Jogar");
    System.out.println("2. Configurações");
    System.out.println("3. Sair");
    System.out.print("Escolha: ");
    
    opcao = scanner.nextInt();
    
    switch (opcao) {
        case 1:
            System.out.println("Iniciando jogo...");
            break;
        case 2:
            System.out.println("Abrir configurações...");
            break;
        case 3:
            System.out.println("Saindo...");
            break;
    }
} while (opcao != 3);  // Continua até sair
```

---

## 6. Comparação: For vs While vs Do-While

```java
// FOR: Quando sabe quantas iterações
for (int i = 0; i < 5; i++) {
    System.out.println("Iteração " + i);
}

// WHILE: Quando a condição pode mudar dinamicamente
int hp = 100;
while (hp > 0) {
    hp -= 25;  // Pode mudar dentro do loop
    System.out.println("HP: " + hp);
}

// DO-WHILE: Quando quer garantir pelo menos uma execução
int escolha = 0;
do {
    System.out.println("Digite um número de 1-5");
    escolha = 5;  // Simular input
} while (escolha < 1 || escolha > 5);
```

---

## 7. Padrões Comuns

### Buscar em Array

```java
String[] campeoes = {"Garen", "Lux", "Ahri", "Darius"};
String buscado = "Lux";

for (int i = 0; i < campeoes.length; i++) {
    if (campeoes[i].equals(buscado)) {
        System.out.println("Encontrado no índice " + i);
        break;
    }
}
```

### Contar Ocorrências

```java
int[] kills = {10, 15, 8, 12, 20};
int contador = 0;

for (int kill : kills) {
    if (kill > 10) {
        contador++;
    }
}

System.out.println("Campeões com mais de 10 kills: " + contador);
```

### Somar Valores

```java
int[] kills = {10, 15, 8, 12, 20};
int total = 0;

for (int kill : kills) {
    total += kill;
}

System.out.println("Total de kills: " + total);
```

### Encontrar Máximo e Mínimo

```java
int[] danos = {100, 250, 180, 320, 90};

int maximo = danos[0];
int minimo = danos[0];

for (int dano : danos) {
    if (dano > maximo) maximo = dano;
    if (dano < minimo) minimo = dano;
}

System.out.println("Máximo: " + maximo);  // 320
System.out.println("Mínimo: " + minimo);  // 90
```

---

## 8. Erros Comuns

### Condição Infinita

```java
// ❌ PERIGOSO: Loop infinito!
while (true) {
    System.out.println("Spam!");
    // Nunca sai!
}

// ✓ Com uma condição de saída
int contador = 0;
while (true) {
    System.out.println("Iteração " + contador);
    contador++;
    if (contador >= 5) break;
}
```

### Incremento Esquecido

```java
// ❌ Loop infinito (i nunca muda)
for (int i = 0; i < 5;) {
    System.out.println(i);
    // Falta i++
}

// ✓ Correto
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}
```

### Off-by-One

```java
String[] campeoes = {"Garen", "Lux", "Ahri"};

// ❌ Array Out of Bounds (índice 3 não existe)
for (int i = 0; i <= campeoes.length; i++) {
    System.out.println(campeoes[i]);  // Erro quando i=3
}

// ✓ Correto
for (int i = 0; i < campeoes.length; i++) {
    System.out.println(campeoes[i]);
}
```

---

## 9. Fluxo Completo: Exemplo Prático

```java
public class Jogo {
    public static void main(String[] args) {
        String[] campeoes = {"Garen", "Lux", "Ahri", "Darius"};
        int[] kills = {15, 22, 18, 25};
        
        System.out.println("=== Placar da Partida ===");
        
        for (int i = 0; i < campeoes.length; i++) {
            String campeao = campeoes[i];
            int kill = kills[i];
            
            // If-else para status
            String status = (kill > 20) ? "PENTAKILL!" : 
                           (kill > 15) ? "Ótimo jogo!" : 
                           "Bom desempenho!";
            
            System.out.println(campeao + ": " + kill + " kills - " + status);
        }
        
        // Encontrar melhor campeão
        int melhorIndice = 0;
        for (int i = 1; i < kills.length; i++) {
            if (kills[i] > kills[melhorIndice]) {
                melhorIndice = i;
            }
        }
        
        System.out.println("\nMVP: " + campeoes[melhorIndice] + 
                          " com " + kills[melhorIndice] + " kills!");
    }
}

// Output:
// === Placar da Partida ===
// Garen: 15 kills - Bom desempenho!
// Lux: 22 kills - Ótimo jogo!
// Ahri: 18 kills - Ótimo jogo!
// Darius: 25 kills - PENTAKILL!
//
// MVP: Darius com 25 kills!
```

---

## Resumo Rápido

| Estrutura | Melhor Para | Exemplo |
|-----------|-----------|---------|
| **If-Else** | Decisões simples | `if (hp > 0)` |
| **Switch** | Múltiplas opções de um valor | `switch (tipo)` |
| **Ternário** | Uma linha | `hp > 0 ? "Vivo" : "Morto"` |
| **For** | Iterações fixas/conhecidas | `for (int i = 0; i < 5; i++)` |
| **For-Each** | Iterar sem índice | `for (String c : campeoes)` |
| **While** | Condição dinâmica | `while (hp > 0)` |
| **Do-While** | Pelo menos uma execução | Menu, validação |

---

## Dicas Finais

✓ Prefira **for-each** quando possível (mais seguro)  
✓ Use **switch** para múltiplas opções de 1 valor  
✓ Evite **loops infinitos** sem break  
✓ Lembre de **i < tamanho** (não <=)  
✓ Use **break** e **continue** com moderação  

Bora controlar o fluxo! 🎮
