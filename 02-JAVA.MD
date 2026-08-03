# Operadores em Java ? LoL Edition

## Exemplo Prático: Validação de Campeão em Teamfight

```java
public class Champion {
    // Atributos de instância = Stats do campeão
    private String nome;                // Garen, Lux, Ahri
    private double healthAtual;         // HP (varia durante o jogo)
    private double manaAtual;           // Mana (varia)
    private double attackDamage;        // AD (aumenta com itens)
    private double armorResist;         // Resistência física
    private boolean estaStunado;        // Estado: CC aplicado?
    private boolean ultimaAbilitadePronta;  // Ulti ready?
    private int nivel;                  // Level (1-18)
    private double ouro;                // Gold (para itens)
    
    // Membro estático = Buff global (regra do jogo, todos herdam)
    private static final double VELOCIDADE_MOVIMENTO_BASE = 330.0;
    private static final double DANO_MINION_BASE = 5.0;
    private static final int NIVEL_MAXIMO = 18;
    private static final double GOLD_KILL_BASE = 300.0;
    
    public boolean podeAtacar(Champion alvo) {
        // Validação antes de soltar combo no teamfight
        
        // Guard 1: está stunado ou CC aplicado?
        if (estaStunado) {
            return false;  // Sem permissão para agir
        }
        
        // Guard 2: tem mana suficiente pra soltar habilidade?
        if (manaAtual <= 0) {
            return false;  // Broke de mana
        }
        
        // Guard 3: alvo é válido (existe e não está inimigo)?
        if (alvo == null || alvo.estaVivo() == false) {
            return false;  // Alvo morto ou não existe
        }
        
        // Guard 4: alvo está muito longe (fora do range)?
        double distancia = calcularDistancia(alvo);
        if (distancia > 550.0) {
            return false;  // Fora do range de ataque
        }
        
        // Guard 5: cooldown da ulti venceu?
        if (ultimaAbilitadePronta && nivel < 6) {
            return false;  // Ulti não destravou ainda
        }
        
        // Passou em todas as validações = win condition
        return true;
    }
    
    public double calcularDanoFinal(Champion alvo) {
        // Usa membro estático (DANO_MINION_BASE é buff global)
        double danoBase = attackDamage + DANO_MINION_BASE;
        double armorInimigo = alvo.getArmorResist();
        
        // Redução por armor: quanto mais armor, menos dano
        double reducaoArmor = armorInimigo / (100.0 + armorInimigo);
        double danofinal = danoBase * (1.0 - reducaoArmor);
        
        return danofinal;
    }
    
    public void matarInimigo(Champion inimigo) {
        if (podeAtacar(inimigo)) {
            double dano = calcularDanoFinal(inimigo);
            inimigo.receberDano(dano);
            
            if (!inimigo.estaVivo()) {
                // Kill confirmado, recebe ouro
                this.ouro += GOLD_KILL_BASE;  // Buff global aplicado
            }
        }
    }
    
    // Getters
    public String getNome() { return nome; }
    public double getHealthAtual() { return healthAtual; }
    public boolean estaVivo() { return healthAtual > 0; }
    public boolean isEstaStunado() { return estaStunado; }
    public double getArmorResist() { return armorResist; }
}
```

---

## Operadores Comparativos

| Operador | Significado | LoL | Exemplo |
|----------|------------|-----|---------|
| `==` | Igual a | Mesmo level | `if (nivel == 6)` ? ulti destravou |
| `!=` | Diferente de | Campeões diferentes | `if (!nome.equals("Garen"))` ? não é Garen |
| `>` | Maior que | Mais HP | `if (healthAtual > 100)` ? tankeável |
| `<` | Menor que | Menos ouro | `if (ouro < 1300)` ? broke de components |
| `>=` | Maior ou igual | HP >= necessário | `if (manaAtual >= custoAbilidade)` ? pode cashar |
| `<=` | Menor ou igual | Mana <= cap | `if (nivel <= 3)` ? early game |

### Exemplos:
```java
Champion garen = new Champion("Garen", 590, 0, 80, 40, false, true, 5, 800);
double ouro = 800.0;
int nivel = 5;

if (nivel > 3) {
    System.out.println("Pode fazer double buff na jungla");  // ?
}

if (ouro >= 1300) {
    System.out.println("Tem gold pra completar item");  // ?
}

if (!garen.isEstaStunado()) {
    System.out.println("Pode atacar");  // ? Negação
}

if (garen.estaVivo()) {
    System.out.println("Ainda está na luta");  // ?
}
```

---

## Operadores Lógicos

| Operador | Significado | LoL | Quando Usar |
|----------|------------|-----|------------|
| `&&` | E (AND) | **Ambos** têm que estar certos | Validações em série |
| `\|\|` | OU (OR) | **Um ou outro** vale | Alternativas aceitáveis |
| `!` | NÃO (NOT) | Inverte condição | Negar um estado |

### AND (`&&`) ? Decisão no Teamfight

```java
boolean podeAtacar = temMana && estaVivo && !estaStunado && temAlvoVisivel;
//                   ^^^^^^^^  ^^^^^^^^  ^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^
//                   Tudo tem que ser TRUE, senão false

if (podeAtacar) {
    System.out.println("Go! Solte a combinação!");  // Só executa se TUDO passar
}
```

**Analogia:** Você precisa estar vivo **E** ter mana **E** não estar CC'd **E** ter alvo à vista. Se um falhar, não entra no teamfight.

**Comportamento Short-circuit:** Para na primeira condição false.
```java
Champion lux = new Champion("Lux", 500, 400, 50, 30, false, true, 9, 2500);
Champion inimigo = new Champion("Garen", 600, 100, 100, 50, true, false, 9, 3000);

if (lux.estaVivo() && lux.getManaAtual() > 100 && !inimigo.estaStunado()) {  
    // TRUE && TRUE && FALSE = FALSE
    System.out.println("Nunca vai rodar");  // ? Inimigo é stun
}

if (lux.estaVivo() && lux.getManaAtual() > 0) {  // TRUE && TRUE
    System.out.println("Pode Q!");  // ? Passou nas validações
}
```

### OR (`||`) ? Estratégia de Ganho

```java
boolean timeGanhou = destroiNexoInimigo || destroiTodasAsTorres || surrenderInimigo;
//                   ^^^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^
//                   Se UMA for TRUE, tudo TRUE

if (timeGanhou) {
    System.out.println("VICTORY!");  // Executa se pelo menos uma passar
}
```

**Analogia:** Você ganha se destrói Nexo **OU** destrói todas towers **OU** inimigo surrender. Só precisa de um.

**Comportamento Short-circuit:** Para na primeira condição true.
```java
boolean deveBanirChampion = banidoNoMeta || countraNoMeu || temPlayRate80Porcento;

if (deveBanirChampion) {
    System.out.println("Ban esse champion!");  // Executado se alguma condição for true
}
```

### NOT (`!`) ? Negação de Estado

```java
boolean podeRecuar = !estaEngajado;  // Inverte: se true ? false, se false ? true

if (!estaStunado) {
    System.out.println("Posso me mover!");  // Executa se estaStunado for FALSE
}
```

**Analogia:** É o oposto do estado. Se Stun está ativo, `!stun` é false (você NÃO pode se mexer). Se Stun é falso, `!stun` é true (VOCÊ PODE se mexer).

---

## Classes, Atributos e Estáticos ? LoL Mindset

### Classes = Campeões Únicos

```java
public class Champion {
    // Cada instância é um campeão diferente (como um summoner diferente)
    private String nome;           // Garen, Lux, Ahri (SummonerName)
    private double healthAtual;    // HP desta instância
    private double attackDamage;   // AD deste champ
}

Champion garen = new Champion("Garen", 590, 80);
Champion lux = new Champion("Lux", 520, 50);

// garen e lux são campeões DIFERENTES
// cada um tem seus próprios stats mesmo sendo ambos Champion
```

### Atributos = Stats da Instância

```java
public class Champion {
    private String nome;              // Identificador único
    private double healthAtual;       // Health desta instância
    private double attackDamage;      // AD (aumenta com itens)
    private boolean estaStunado;      // Flag: true/false
}

// Cada Champion tem stats INDEPENDENTES
garen.healthAtual = 590;      // Garen tem X HP
lux.healthAtual = 520;        // Lux tem Y HP (diferente!)

// garen != lux, mesmo que sejam ambos Champion
```

### Membros Estáticos = Buff Global do Jogo

```java
public class Champion {
    // Constante estática = Imutável, compartilhado por TODO mundo
    private static final double VELOCIDADE_MOVIMENTO_BASE = 330.0;
    //                          ^^^^^^^^^^^^^^^^^^^^^^^
    //                          Todos os campeões têm essa velocidade base
    
    // Variável estática = Pode mudar, mas afeta TODO mundo
    private static double DANO_MINION_BASE = 5.0;
    //     ^^^^^^ Buff dinâmico (muda para TODOS)
    
    public double calcularDanoFinal(Champion alvo) {
        return attackDamage + DANO_MINION_BASE;
        //                    ^^^^^^^^^^^^^^^^
        //                    Mesmo valor para TODO campeão
    }
}

// Acesso estático (não precisa instanciar)
double velocidade = Champion.VELOCIDADE_MOVIMENTO_BASE;  // 330.0 para TODOS
Champion.DANO_MINION_BASE = 7.0;  // Alterou o buff global

// Garen, Lux, Ahri, TODOS agora fazem +7 de dano base
```

**Analogia:**
- **Atributo:** Garen tem 590 HP, Lux tem 520 HP ? stats únicos de cada champ
- **Estático:** Todos têm velocidade 330 ? é regra do jogo, não do champ
  - Mudar `DANO_MINION_BASE` de 5 para 7 = patch que afeta TODO mundo simultaneamente

---

## Combinando Operadores ? Decisão no Teamfight

```java
public boolean podeJuntarNoTeamfight(Champion eu, Champion aliado1, Champion aliado2) {
    
    // Problemas que podem acontecer
    boolean temRiscoAlto = 
        !eu.estaVivo() ||                        // Problema 1: Morto
        eu.getManaAtual() < 100 ||               // Problema 2: Sem mana
        eu.estaStunado();                        // Problema 3: CC aplicado
    
    if (temRiscoAlto) {
        return false;  // Reject teamfight
    }
    
    // Aliados podem ajudar?
    boolean temSuporteAliados = 
        (aliado1.estaVivo() && aliado1.getManaAtual() > 0) ||  // Aliado 1 pronto
        (aliado2.estaVivo() && aliado2.getManaAtual() > 0);    // Aliado 2 pronto
    
    // Decisão final: entrar no teamfight?
    boolean podeEntraNoFight = 
        !temRiscoAlto &&              // Sem riscos críticos E
        temSuporteAliados &&          // Tem aliado vivo E pronto E
        eu.getNivel() >= 6;           // Tem ulti destravada
    
    return podeEntraNoFight;  // TRUE = Go teamfight!
}
```

**Trace com valores (cenário de game):**
```java
podeJuntarNoTeamfight(
    eu(Lux, alive=true, mana=350, stun=false),
    aliado1(Leona, alive=true, mana=200),
    aliado2(Janna, alive=false, mana=100)
)

temRiscoAlto = false || false || false = FALSE     ? Sem risco
temSuporteAliados = (true && true) || (false && true) = TRUE || FALSE = TRUE  ? Leona tá pronto
podeEntraNoFight = !FALSE && TRUE && TRUE = TRUE  ?
? Resultado: true (ENTRAR NO FIGHT!)
```

**Outro cenário:**
```java
podeJuntarNoTeamfight(
    eu(Lux, alive=true, mana=50, stun=false),     // Mana baixa!
    aliado1(Leona, alive=false, mana=0),          // Morto
    aliado2(Janna, alive=false, mana=0)           // Morto
)

temRiscoAlto = false || true || false = TRUE      ? Sem mana!
? Resultado: false (NÃO ENTRA) ? short-circuit, não precisou checar o resto
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

## Boas Práticas ? Guard Clauses no Engajamento

```java
// ? Ruim: Aninhamento profundo (nested hell)
public void realizarEngajamento(Champion eu, Champion inimigo, boolean temVantagem) {
    if (eu != null) {
        if (eu.estaVivo()) {
            if (inimigo != null) {
                if (inimigo.estaVivo()) {
                    if (temVantagem) {
                        if (eu.getManaAtual() > 100) {
                            // 6 níveis deep ? difícil de ler
                            atacarInimigo(eu, inimigo);
                        }
                    }
                }
            }
        }
    }
}

// ? Bom: Guard clauses (rejeição rápida = early escape)
public void realizarEngajamento(Champion eu, Champion inimigo, boolean temVantagem) {
    // Rejeita rápido se alguma condição crítica falhar
    if (eu == null || !eu.estaVivo()) return;       // Guard 1
    if (inimigo == null || !inimigo.estaVivo()) return;  // Guard 2
    if (!temVantagem) return;                       // Guard 3
    if (eu.getManaAtual() <= 100) return;           // Guard 4
    
    // Aqui é 100% seguro ? todas as validações passaram
    atacarInimigo(eu, inimigo);
}
```

**Analogia:** Guard clause é como recuar da lane se não tem visão. Não engaja até ter segurança total. Se algo tá errado, sai fácil e rápido (`return`).