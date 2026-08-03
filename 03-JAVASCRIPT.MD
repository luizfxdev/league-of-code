# Operadores em JavaScript ? LoL Edition

## Exemplo Prático: Game Loop e Detecção de Combate

```javascript
// Classe = Campeão (ES6)
class Champion {
    // Atributos de instância = Stats do campeão
    constructor(nome, healthAtual, manaAtual, attackDamage, level) {
        this.nome = nome;                   // "Garen", "Lux", "Ahri"
        this.healthAtual = healthAtual;     // HP desta instância
        this.manaAtual = manaAtual;         // Mana (varia)
        this.attackDamage = attackDamage;   // AD (aumenta com itens)
        this.level = level;                 // Level (1-18)
        this.estaStunado = false;           // Estado: CC aplicado?
        this.ultimaProntaEm = 0;            // Cooldown da ulti
    }
    
    // Membro estático = Buff global para TODOS os campeões
    static VELOCIDADE_MOVIMENTO_BASE = 330;
    static MANA_REGENERACAO_BASE = 5;
    static DANO_MINION_BASE = 5;
}

// Validação antes do engagement no game loop
function podeAtacarAgora(campeao, alvo) {
    // Guard 1: Campeão existe?
    if (!campeao || typeof campeao !== "object") {
        return false;  // Campeão não foi criado
    }
    
    // Guard 2: Mana é suficiente?
    if (campeao.manaAtual <= 0) {
        return false;  // Broke de mana ? não pode cashar habilidade
    }
    
    // Guard 3: Alvo é válido e não tá morto?
    if (!alvo || alvo.healthAtual <= 0) {
        return false;  // Alvo morto ou null
    }
    
    // Guard 4: Está CC'd (stunado)?
    if (campeao.estaStunado) {
        return false;  // CC aplicado, sem permissão de agir
    }
    
    // Guard 5: Cooldown da ulti abaixou?
    if (campeao.level < 6 && campeao.ultimaProntaEm > Date.now()) {
        return false;  // Ulti ainda no cooldown
    }
    
    return true;  // Win condition ? pode atacar!
}

// Executar ataque (mutação controlada)
function executarAtaque(campeao, alvo) {
    if (!podeAtacarAgora(campeao, alvo)) {
        console.warn("Não pode atacar agora");
        return false;
    }
    
    // Seguro para alterar estado agora
    campeao.manaAtual -= 50;  // Consome mana
    alvo.healthAtual -= campeao.attackDamage;  // Aplica dano
    
    console.log(`${campeao.nome} atacou ${alvo.nome}!`);
    return true;
}
```

---

## Operadores Comparativos

| Operador | Significado | LoL | Exemplo |
|----------|------------|-----|---------|
| `==` | Igual (coerção) | Mesmo nível? | `if (level == 6)` ? cuidado com type coercion |
| `===` | Estritamente igual | Mesmo nível + mesmo tipo | `if (level === 6)` |
| `!=` | Diferente (coerção) | Campeões diferentes | `if (nome != "Garen")` |
| `!==` | Estritamente diferente | Campeões diferentes (tipo importa) | `if (nome !== "Garen")` |
| `>` | Maior que | Mais HP | `if (healthAtual > 100)` |
| `<` | Menor que | Menos mana | `if (manaAtual < 50)` |
| `>=` | Maior ou igual | HP >= necessário | `if (level >= 6)` ? ulti pronta |
| `<=` | Menor ou igual | Mana <= cap | `if (cooldown <= 0)` ? ready |

### Exemplo: `==` vs `===` (Trap comum)

```javascript
// ? Evite ==, JS faz coerção invisível
console.log(0 == "0");        // true  ? JavaScript converteu "0" pra 0!
console.log(false == 0);      // true  ? false virou 0!
console.log("" == null);      // false ? Confuso mesmo

// ? Use ===, sem coerção ? explícito
console.log(0 === "0");       // false ? Claro, tipos diferentes
console.log(false === 0);     // false ? Esperado, bool != number
console.log("" === null);     // false ? Óbvio

// Validação real em jogo
const levelAtual = "6";  // Vindo de API, é string

if (levelAtual == 6) {      // ?? Risco: "6" == 6 é TRUE
    console.log("Ulti pronta!");  // Pode quebrar se nivel for "5"
}

if (levelAtual === 6) {     // ? Seguro: "6" === 6 é FALSE
    console.log("Ulti pronta!");  // Só executa se realmente for número 6
}

// Comparação correta
const level = parseInt(levelAtual);
if (level === 6) {
    console.log("Agora é seguro!");  // ?
}
```

---

## Operadores Lógicos

| Operador | Significado | LoL | Quando Usar |
|----------|------------|-----|------------|
| `&&` | E (AND) | **Ambos** têm que estar certos | Validações em série |
| `\|\|` | OU (OR) | **Um ou outro** vale | Alternativas, valores padrão |
| `!` | NÃO (NOT) | Inverte condição | Negar estado |

### AND (`&&`) ? Decisão de Engajamento

```javascript
const podeEngajar = temMana && estaVivo && !estaStunado && temAlvoVisivel;
//                  ^^^^^^^^  ^^^^^^^^  ^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^
//                  Tudo tem que ser TRUE para atacar

if (podeEngajar) {
    executarCombo();  // Só executa se TUDO passar
}
```

**Analogia:** Você precisa estar vivo **E** ter mana **E** não estar CC'd **E** ver o inimigo. Se um falhar, sai do fight.

**Short-circuit em JavaScript (otimização):**
```javascript
const alvo = null;

// Para na primeira condição false
if (alvo && alvo.healthAtual > 0 && calcularDistancia(alvo) < 550) {
    console.log("Pode atacar...");  // Nunca executa (alvo é null)
}

// calcularDistancia() NUNCA é chamado! (short-circuit)
```

### OR (`||`) ? Valor Padrão (Fallback)

```javascript
// Tenta usar o primeiro; se não existir, usa o segundo
let nomeChampion = inputUsuario || "Champion Padrão";
//                 ^^^^^^^^^^^^   ^^^^^^^^^^^^^^^^^^^
//                 Usa este se existir, senão usa fallback

// Equivalente a:
let nomeChampion;
if (inputUsuario) {
    nomeChampion = inputUsuario;
} else {
    nomeChampion = "Champion Padrão";
}
```

**Analogia:** Vai pra lane de prioridade, se não tiver, vai pra outra.

```javascript
// Validação com múltiplas alternativas
const podeEntrarNoFight = 
    timeStatusGanho === "VENCENDO" ||
    timeStatusGanho === "EQUILIBRADO" ||
    temUltisProntas;

if (podeEntrarNoFight) {
    engajar();  // Executa se alguma condição for true
}
```

### NOT (`!`) ? Negação de Estado

```javascript
const podeRecuar = !estaEngajado;  // Inverte: true ? false, false ? true

if (!estaStunado) {
    console.log("Posso me mover!");  // Executa se estaStunado for FALSE
}

// ? Evite dupla negação (confunde)
if (!!temVisao) {  // ?? Converte para boolean (desnecessário)
    console.log("Tem visão");
}

// ? Melhor: use simples
if (temVisao) {  // Já é boolean
    console.log("Tem visão");
}
```

---

## Classes, Atributos e Estáticos em JavaScript

### Classes = Campeões Únicos

```javascript
class Champion {
    constructor(nome, healthAtual, manaAtual, attackDamage) {
        this.nome = nome;                   // "Garen", "Lux", "Ahri"
        this.healthAtual = healthAtual;     // HP desta instância
        this.manaAtual = manaAtual;         // Mana desta instância
        this.attackDamage = attackDamage;   // AD deste champ
    }
}

const garen = new Champion("Garen", 590, 0, 80);
const lux = new Champion("Lux", 520, 380, 50);

// garen e lux são DIFERENTES
// cada um tem seus próprios stats
console.log(garen.nome);  // "Garen"
console.log(lux.nome);    // "Lux"
```

### Atributos = Stats da Instância

```javascript
class Champion {
    constructor(nome, healthAtual, manaAtual, attackDamage) {
        this.nome = nome;              // Identificador único
        this.healthAtual = healthAtual;  // Health desta instância
        this.manaAtual = manaAtual;    // Mana desta instância
        this.attackDamage = attackDamage;  // AD (varia com itens)
        this.estaVivo = true;          // Flag: true/false
    }
}

// Cada Champion tem stats INDEPENDENTES
garen.healthAtual = 590;
lux.healthAtual = 520;  // Diferente de Garen!

console.log(garen.healthAtual);  // 590
console.log(lux.healthAtual);    // 520 (diferente!)
```

### Estáticos = Buff Global do Jogo

```javascript
class Champion {
    // Estático = Compartilhado por TODOS os campeões
    static VELOCIDADE_MOVIMENTO_BASE = 330;
    static MANA_REGENERACAO_BASE = 5;
    static DANO_MINION_BASE = 5;
    
    constructor(nome, healthAtual, manaAtual, attackDamage) {
        this.nome = nome;
        this.healthAtual = healthAtual;
        this.manaAtual = manaAtual;
        this.attackDamage = attackDamage;
    }
    
    calcularDanoFinal() {
        // Usa membro estático (igual para TODOS)
        return this.attackDamage + Champion.DANO_MINION_BASE;
    }
}

// Acesso estático (não precisa de instância)
console.log(Champion.VELOCIDADE_MOVIMENTO_BASE);  // 330 (para todos)

// Alterar estático = afeta TODO mundo
Champion.DANO_MINION_BASE = 7;  // Patch que afeta TODOS

const garen = new Champion("Garen", 590, 0, 80);
const lux = new Champion("Lux", 520, 380, 50);

// Ambos usam o novo dano base
console.log(garen.calcularDanoFinal());  // 80 + 7 = 87
console.log(lux.calcularDanoFinal());    // 50 + 7 = 57 (novo buff)
```

**Analogia:**
- **Atributo:** Garen tem 590 HP, Lux tem 520 HP ? stats únicos
- **Estático:** Todos têm velocidade 330 ? é regra do jogo
  - Mudar `DANO_MINION_BASE` de 5 pra 7 = patch que afeta TODO mundo simultaneamente

---

## Combinando Operadores ? Lógica Complexa em DOM

```javascript
function habilitarBotaoOS(formulario) {
    // Validar entrada
    const veiculoId = formulario.querySelector("#veiculoId").value;
    const kmRequisicao = parseInt(formulario.querySelector("#kmRequisicao").value);
    const statusSistema = formulario.dataset.status;
    
    // Validações em série (AND)
    const veiculoValido = veiculoId && veiculoId > 0;
    const kmValido = kmRequisicao > 0 && kmRequisicao <= 50_000;
    const sistemaOK = statusSistema === "ONLINE";
    
    // Resultado final
    const podeExecutar = veiculoValido && kmValido && sistemaOK;
    
    // Atualizar DOM
    const botao = formulario.querySelector("#btnExecutar");
    botao.disabled = !podeExecutar;  // Desativa se NÃO pode
    
    if (podeExecutar) {
        botao.classList.add("btn-success");
        botao.classList.remove("btn-disabled");
    } else {
        botao.classList.remove("btn-success");
        botao.classList.add("btn-disabled");
    }
}

// Executar ao mudar valores
document.getElementById("veiculoId").addEventListener("change", function() {
    habilitarBotaoOS(this.closest("form"));
});
```

---

## Operadores Especiais em JavaScript

### Nullish Coalescing (`??`) ? Valor Padrão Seguro

```javascript
// Diferente de ||, só usa padrão se null/undefined
let tempo = 0 ?? 30;
//          ^ NÃO é nulo, usa este valor
console.log(tempo);  // 0 (não 30)

let tempo2 = null ?? 30;
//           ^^^^ É nulo, usa padrão
console.log(tempo2);  // 30

// Vs || (evitar para valores "falsy")
let valor1 = 0 || 100;      // 100 (0 é falsy, problema!)
let valor2 = 0 ?? 100;      // 0 (0 não é null, seguro!)
```

### Optional Chaining (`?.`) ? Acesso Seguro

```javascript
const veiculo = { placa: "ABC1234", manutencao: { tipo: "Preventiva" } };

// ? Risco: erro se manutencao for undefined
const tipo1 = veiculo.manutencao.tipo;  // OK (existe)

// ? Seguro: retorna undefined se não existir
const tipo2 = veiculo.manutencao?.tipo;  // "Preventiva"
const tipo3 = veiculo.aro?.tamanho;      // undefined (sem erro)

// Em arrays
const osAberta = osArray?.[0]?.status;  // Seguro mesmo se array for vazio
```

### Ternário (`? :`) ? Quick Decision

```javascript
const statusBotao = podeExecutar ? "btn-success" : "btn-disabled";
//                  ^^^^^^^^^^^^   ^^^^^^^^^^^^   ^^^^^^^^^^^^^
//                  Condição       Se true        Se false

const mensagem = kmRequisicao > 50_000 ? "KM demais" : "KM OK";
```

---

## typeof ? Verifica Tipo

```javascript
typeof "string"      // "string"
typeof 123           // "number"
typeof true          // "boolean"
typeof undefined     // "undefined"
typeof {}            // "object"
typeof []            // "object" ? Cuidado! Array é object
typeof null          // "object" ? Bug histórico!

// Validação segura
function processarOS(os) {
    if (typeof os !== "object" || os === null) {
        return false;  // Não é um objeto válido
    }
    
    if (typeof os.kmRequisicao !== "number") {
        return false;  // KM deve ser número
    }
    
    return true;
}
```

---

## Boas Práticas ? Código Defensivo em JavaScript

```javascript
// ? Ruim: Assume que tudo existe
function obterKmVeiculo(veiculoId) {
    const veiculo = buscarVeiculo(veiculoId);
    return veiculo.kmMilhao * 1_000_000 + veiculo.kmAtual;  // NPE risco!
}

// ? Bom: Validação + early return
function obterKmVeiculo(veiculoId) {
    const veiculo = buscarVeiculo(veiculoId);
    
    if (!veiculo) {
        console.error(`Veículo ${veiculoId} não encontrado`);
        return 0;
    }
    
    const kmMilhao = veiculo.kmMilhao ?? 0;
    const kmAtual = veiculo.kmAtual ?? 0;
    
    return kmMilhao * 1_000_000 + kmAtual;
}
```

### Guard Clauses em Callback AJAX

```javascript
function salvarOS() {
    const osId = document.getElementById("osId").value;
    const veiculoId = document.getElementById("veiculoId").value;
    const kmRequisicao = parseInt(document.getElementById("kmRequisicao").value);
    
    // Guard: validações ANTES de chamar servidor
    if (!osId || !veiculoId || !kmRequisicao) {
        alert("Preencha todos os campos");
        return;  // Rejeita rápido
    }
    
    if (kmRequisicao <= 0 || kmRequisicao > 50_000) {
        alert("KM deve estar entre 1 e 50.000");
        return;
    }
    
    // Seguro para fazer a chamada
    $.ajax({
        type: "POST",
        url: "/gw/ordenServico/salvar",
        data: { osId, veiculoId, kmRequisicao },
        success: function(response) {
            if (response.sucesso) {
                alert("OS salva com sucesso!");
                location.reload();
            } else {
                alert("Erro: " + response.mensagem);
            }
        },
        error: function() {
            alert("Erro na conexão");
        }
    });
}
```

---

## Precedência de Operadores ? Cuidado!

```javascript
// Operador && tem precedência sobre ||
const resultado = true || false && false;
//                 ^^^^   ^^^^^ ^^ ^^^^^
//                         Isso avalia PRIMEIRO

// Equivalente a:
const resultado = true || (false && false);  // true

// Se quer outro comportamento, use parênteses:
const outro = (true || false) && false;  // false
```

---

## Resumo: When to Use What

| Situação | Operador | Exemplo |
|----------|----------|---------|
| Validação obrigatória | `&&` | `if (tem && valido && ativo)` |
| Valor padrão | `\|\|` ou `??` | `nome = input \|\| "Padrão"` |
| Inverter condição | `!` | `if (!emManutencao)` |
| Teste de igualdade | `===` | `if (status === "ABERTA")` |
| Teste de tipo | `typeof` | `if (typeof id === "number")` |
| Acesso seguro | `?.` | `obj?.campo?.subcampo` |
| Valor default seguro | `??` | `valor ?? 0` |