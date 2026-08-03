# Operadores em Java ? LoL Edition

## Exemplo Prático: Validação de Veículo em OS

```java
public class Veiculo {
    // Atributos de instância = Stats do campeão
    private String placa;           // Health
    private int kmAtual;            // Mana
    private int kmMilhao;           // Attack Damage
    private boolean emManutencao;   // Estado: stunned
    private double valorCustodia;   // Gold
    
    // Membro estático = Buff global (todos os veículos herdam)
    private static final int KM_LIMITE_MANUTENCAO = 50000;
    private static final double TAXA_OPERACAO = 0.15;
    
    public boolean podeUsarEmOS(int kmRequisicao) {
        // Validação antes do teamfight
        
        // Guard 1: está em manutenção? (CC aplicado?)
        if (emManutencao) {
            return false;  // Não pode jogar
        }
        
        // Guard 2: km total excede limite?
        int kmTotal = (kmMilhao * 1_000_000) + kmAtual;
        if (kmTotal > 500_000_000) {
            return false;  // Cooldown muito alto
        }
        
        // Guard 3: km da requisição é válido?
        if (kmRequisicao <= 0 || kmRequisicao > 10_000) {
            return false;  // Parâmetro fora do range
        }
        
        // Guard 4: próxima manutenção está perto?
        int proxikaManutencao = (kmMilhao * 1_000_000) + KM_LIMITE_MANUTENCAO;
        if ((kmTotal + kmRequisicao) >= proxikaManutencao) {
            return false;  // Vai ficar com CC antes de acabar
        }
        
        // Passou em todas as validações = win condition
        return true;
    }
    
    public double calcularCustoOperacao() {
        // Usa membro estático (TAXA_OPERACAO é buff global)
        return valorCustodia * TAXA_OPERACAO;
    }
    
    // Getters
    public String getPlaca() { return placa; }
    public int getKmAtual() { return kmAtual; }
    public boolean isEmManutencao() { return emManutencao; }
}
```

---

## Operadores Comparativos

| Operador | Significado | LoL | Exemplo |
|----------|------------|-----|---------|
| `==` | Igual a | Mesmo nível | `if (kmMilhao == 1)` ? checkLevel |
| `!=` | Diferente de | Inimigos | `if (!emManutencao)` ? pode atacar |
| `>` | Maior que | Mais HP | `if (kmTotal > limite)` ? precisa heal |
| `<` | Menor que | Menos ouro | `if (valorCustodia < 1000)` ? broke |
| `>=` | Maior ou igual | HP >= necessário | `if (kmAtual >= KM_LIMITE)` ? está pronto |
| `<=` | Menor ou igual | Mana <= cap | `if (tempo <= 30)` ? early game |

### Exemplos:
```java
int nivel = 5;
int experiencia = 3200;

if (nivel > 3) {
    System.out.println("Pode fazer double buff");  // ?
}

if (experiencia == 3200) {
    System.out.println("Quase level up");  // ?
}

if (!(nivel < 2)) {
    System.out.println("Não é level 1");  // ? Negação
}
```

---

## Operadores Lógicos

| Operador | Significado | LoL | Quando Usar |
|----------|------------|-----|------------|
| `&&` | E (AND) | **Ambos** tem que estar certos | Validações em série |
| `\|\|` | OU (OR) | **Um ou outro** vale | Alternativas aceitáveis |
| `!` | NÃO (NOT) | Inverte condição | Negar um estado |

### AND (`&&`) ? Validação em Série

```java
boolean podeAtacar = temMana && temVida && naoCcAplicado && arquivoAberto;
//                   ^^^^^^    ^^^^^^^^  ^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^
//                   Tudo tem que ser TRUE, senão false

if (podeAtacar) {
    System.out.println("Solte a ult!");  // Só executa se TUDO passar
}
```

**Analogia:** Você precisa estar vivo **E** ter mana **E** não estar stun **E** ter cooldown down. Se um for false, não solta nada.

**Comportamento Short-circuit:** Para na primeira condição false.
```java
int kmAtual = 100;

if (kmAtual > 50 && kmAtual < 200) {  // TRUE && TRUE = TRUE
    System.out.println("Range válido");  // ?
}

if (kmAtual < 50 && valorCustodia > 5000) {  // FALSE && (não verifica) = FALSE
    System.out.println("Nunca vai rodar");  // ?
}
```

### OR (`||`) ? Alternativas

```java
boolean podeUsar = temVaga || temFila || temReserva;
//                 ^^^^^^^^  ^^^^^^^^^  ^^^^^^^^^^^^
//                 Se UMA for TRUE, tudo TRUE

if (podeUsar) {
    System.out.println("Conseguiu espaço!");  // Executa se pelo menos uma passar
}
```

**Analogia:** Você ganha se destrói inibidor **OU** mata barão **OU** destrói nexo. Só precisa de um.

**Comportamento Short-circuit:** Para na primeira condição true.
```java
boolean temProblema = statusVeiculo.equals("INATIVO") || kmAtual > 500_000 || emManutencao;

if (temProblema) {
    alertarSM();  // Executado se alguma condição for true
}
```

### NOT (`!`) ? Negação

```java
boolean podeUsar = !emManutencao;  // Inverte: se true ? false, se false ? true

if (!temProblema) {
    System.out.println("Está tudo bem");  // Executa se temProblema for FALSE
}
```

**Analogia:** É o oposto do que o buff diz. Se buff "Stun" está ativo, `!stun` é false (você PODE se mexer).

---

## Classes, Atributos e Estáticos ? LoL Mindset

### Classes = Campeões Únicos

```java
public class Veiculo {
    // Cada instância é um campeão diferente
    private String placa;      // Nome único (SummName)
    private int kmAtual;       // HP desta instância
    private int kmMilhao;      // Bonus AD deste champ
}

Veiculo v1 = new Veiculo("ABC1234", 100_000, 1);
Veiculo v2 = new Veiculo("XYZ9876", 250_000, 2);

// v1 e v2 são campeões DIFERENTES
// cada um tem seus próprios stats
```

### Atributos = Stats do Campeão

```java
public class Veiculo {
    private String placa;          // Identificador único
    private int kmAtual;           // Health
    private double valorCustodia;  // Armor
    private boolean emManutencao;  // É uma flag: true/false
}

// Cada Veiculo tem seus stats INDEPENDENTES
v1.kmAtual = 150_000;
v2.kmAtual = 200_000;
// v1 != v2, mesmo que sejam ambos Veiculo
```

### Membros Estáticos = Buff Global

```java
public class Veiculo {
    private static final int KM_LIMITE_MANUTENCAO = 50_000;
    //                       ^^^^^^^^^^^^^^^^^^^^^^^^^^
    //                       Todos os veículos compartilham isso
    
    private static double TAXA_OPERACAO = 0.15;
    //     ^^^^^^ Pode ser alterado, mas é GLOBAL
    
    public double calcularCustoOperacao() {
        return valorCustodia * TAXA_OPERACAO;
        //                    ^^^^^^^^^^^^^^^
        //                    Mesmo valor para TODO mundo
    }
}

// Acesso estático (não precisa de instância)
Veiculo.TAXA_OPERACAO = 0.18;  // Alterou para TODOS
int limite = Veiculo.KM_LIMITE_MANUTENCAO;  // Igual para todos
```

**Analogia:**
- **Atributo:** Garen tem 100 HP, Darius tem 120 HP (diferentes)
- **Estático:** Ambos têm o mesmo cd de Flash (5 min) ? é um buff do jogo, não do champ

---

## Combinando Operadores ? Lógica Complexa

```java
public boolean validarOS(Veiculo veiculo, int kmSolicitado, String status) {
    
    boolean temProblema = 
        veiculo.isEmManutencao() ||              // Problema 1: Em manutenção
        kmSolicitado <= 0 ||                    // Problema 2: KM inválido
        kmSolicitado > 50_000;                  // Problema 3: KM demais
    
    if (temProblema) {
        return false;  // Reject OS
    }
    
    boolean statusValido = 
        status.equals("ATIVO") ||
        status.equals("DISPONIVEL") ||
        status.equals("EMERGENCIA");
    
    boolean podeExecutar = 
        !temProblema &&              // Sem problemas E
        statusValido &&              // Status é válido E
        veiculo.getValorCustodia() > 0;  // Tem custo registrado
    
    return podeExecutar;  // TRUE = Pode fazer OS
}
```

**Trace com valores:**
```java
validarOS(
    veiculo("ABC1234", emManutencao=false),
    kmSolicitado=10_000,
    status="ATIVO"
)

temProblema = false || false || false = FALSE  ?
statusValido = false || false || true = TRUE   ?
podeExecutar = !FALSE && TRUE && TRUE = TRUE && TRUE && TRUE = TRUE  ?
? Resultado: true (Pode fazer OS)
```

---

## Operadores Especiais em Java

### Ternário (`? :`) ? Quick Decision

```java
String status = emManutencao ? "Indisponível" : "Disponível";
//               ^^^^^^^^^^^^   ^^^^^^^^^^^^^^   ^^^^^^^^^^^
//               Condição       Se true          Se false

int custoFinal = isPrioritario ? (valor * 2) : valor;
```

**Analogia:** Se tem ulti ready ? ataca; senão, recua.

### instanceof ? Verifica Tipo

```java
if (veiculo instanceof VeiculoFrota) {
    VeiculoFrota frota = (VeiculoFrota) veiculo;
    System.out.println("É frota: " + frota.getFrotaId());
}
// Verificou se é "da classe VeiculoFrota"
```

---

## Boas Práticas ? Guard Clauses

```java
// ? Ruim: Aninhamento profundo
public void processarOS(Veiculo v, int km) {
    if (!v.isEmManutencao()) {
        if (km > 0) {
            if (km <= 50_000) {
                if (v.getValorCustodia() > 0) {
                    // 4 níveis deep ? callback hell
                    criarOS(v, km);
                }
            }
        }
    }
}

// ? Bom: Guard clauses (rejeição rápida)
public void processarOS(Veiculo v, int km) {
    if (v.isEmManutencao()) return;
    if (km <= 0 || km > 50_000) return;
    if (v.getValorCustodia() <= 0) return;
    
    // Aqui é seguro, tudo passou
    criarOS(v, km);
}
```

**Analogia:** Guard clause é recuar se a lane não tem visão. Não entra no jogo até ter segurança.