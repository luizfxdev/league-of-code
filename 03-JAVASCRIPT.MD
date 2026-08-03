# Operadores em JavaScript ? LoL Edition

## Exemplo Prático: Validação e Atualização de OS no cadastrar_OrdemServico.js

```javascript
// Classe = Campeão (ES6)
class OrdemServico {
    // Atributos de instância = Stats do campeão
    constructor(osId, veiculoId, kmRequisicao, dataCriacao) {
        this.osId = osId;               // ID única (SummName)
        this.veiculoId = veiculoId;     // Referência (team mate)
        this.kmRequisicao = kmRequisicao;  // HP necessário
        this.dataCriacao = dataCriacao;    // Timestamp do spawn
        this.status = "ABERTA";         // Estado: Alive/Dead
    }
    
    // Membro estático = Buff global
    static KM_MAXIMO_POR_OS = 50_000;
    static TAXA_PROCESSAMENTO = 0.12;
    static STATUS_VALIDOS = ["ABERTA", "PROCESSANDO", "CONCLUIDA"];
}

// Validação antes do teamfight
function podeProcessarOS(os, kmVeiculoAtual) {
    // Guard 1: OS existe?
    if (!os || typeof os !== "object") {
        return false;  // Personagem não foi spawnado
    }
    
    // Guard 2: KM é válido?
    if (!kmRequisicao || kmRequisicao <= 0 || kmRequisicao > OrdemServico.KM_MAXIMO_POR_OS) {
        return false;  // Parâmetro fora do range
    }
    
    // Guard 3: Status está OK?
    if (!OrdemServico.STATUS_VALIDOS.includes(os.status)) {
        return false;  // CC aplicado (status inválido)
    }
    
    // Guard 4: Não vai exceder km total?
    if ((kmVeiculoAtual + kmRequisicao) > 500_000_000) {
        return false;  // Vai ficar oom
    }
    
    return true;  // Win condition
}

// Modificar estado (mutação controlada)
function executarOS(os, kmRequisicao) {
    if (!podeProcessarOS(os, kmRequisicao)) {
        alert("Não pode executar esta OS");
        return false;
    }
    
    // Seguro para mutar agora
    os.status = "PROCESSANDO";
    os.kmExecutado = kmRequisicao;
    os.dataExecucao = new Date();
    
    return true;
}
```

---

## Operadores Comparativos

| Operador | Significado | LoL | Exemplo |
|----------|------------|-----|---------|
| `==` | Igual (coerção) | Mesmo nível | `if (status == "ABERTA")` |
| `===` | Estritamente igual | Mesmo nível + mesmo tipo | `if (status === "ABERTA")` |
| `!=` | Diferente (coerção) | Inimigos | `if (status != "")` |
| `!==` | Estritamente diferente | Inimigos (tipo importa) | `if (status !== "")` |
| `>` | Maior que | Mais HP | `if (kmRequisicao > 100)` |
| `<` | Menor que | Menos ouro | `if (kmRequisicao < 50_000)` |
| `>=` | Maior ou igual | HP >= necessário | `if (idade >= 18)` |
| `<=` | Menor ou igual | Mana <= cap | `if (nivel <= 30)` |

### Exemplo Comparativo: `==` vs `===`

```javascript
// ? Evite ==, causa problemas
console.log(0 == "0");        // true  ? Coerção tipo!
console.log(false == 0);      // true  ? Inesperado
console.log("" == null);      // false ? Confuso

// ? Use ===, sem coerção
console.log(0 === "0");       // false ? Claro
console.log(false === 0);     // false ? Esperado
console.log("" === null);     // false ? Óbvio

// Validação real
const status = "ATIVO";

if (status == "ATIVO") {  // ?? Risco: coerção invisível
    console.log("OK");
}

if (status === "ATIVO") {  // ? Seguro: sem surpresas
    console.log("OK");
}
```

---

## Operadores Lógicos

| Operador | Significado | LoL | Quando Usar |
|----------|------------|-----|------------|
| `&&` | E (AND) | **Ambos** têm que estar certos | Validações em série |
| `\|\|` | OU (OR) | **Um ou outro** vale | Alternativas |
| `!` | NÃO (NOT) | Inverte condição | Negar estado |

### AND (`&&`) ? Guard Clauses

```javascript
const podeAtacar = temMana && temVida && naoEmStun && temAlvo;
//                 ^^^^^^^^  ^^^^^^^^  ^^^^^^^^^^^  ^^^^^^^^^
//                 Tudo tem que ser TRUE

if (podeAtacar) {
    soltarUlt();  // Só executa se TUDO passar
}
```

**Analogia:** Você precisa estar vivo **E** ter mana **E** não estar stun. Se um for false, não ataca.

**Short-circuit em JavaScript:**
```javascript
let veiculoId = null;

// Para na primeira condição false
if (veiculoId && veiculoId > 0 && validarVeiculo(veiculoId)) {
    console.log("Processando...");  // Nunca executa (veiculoId é null)
}

// validarVeiculo() nem é chamado!
```

### OR (`||`) ? Valores Padrão

```javascript
let nome = usuarioLogado || "Visitante";
//          ^^^^^^^^^^^^^^   ^^^^^^^^^^^
//          Usa este se existir, senão usa padrão

// Equivalente a:
let nome;
if (usuarioLogado) {
    nome = usuarioLogado;
} else {
    nome = "Visitante";
}
```

**Analogia:** Usa o primeiro recurso disponível, se não houver, usa a reserva.

```javascript
// Validação com alternativas
const statusPermitido = 
    statusOS === "ATIVA" ||
    statusOS === "PENDENTE" ||
    statusOS === "EMERGENCIA";

if (statusPermitido) {
    processarOS();  // Executa se alguma condição for true
}
```

### NOT (`!`) ? Negação

```javascript
const podeUsar = !emManutencao;  // Inverte: true ? false, false ? true

if (!temProblema) {
    console.log("Tudo certo");  // Executa se temProblema for FALSE
}

// Dupla negação (evitar)
if (!!valor) {  // ?? Converte para boolean (não faça isso)
    console.log("Tem valor");
}
```

---

## Classes, Atributos e Estáticos em JavaScript

### Classes = Campeões Únicos

```javascript
class Veiculo {
    constructor(placa, kmAtual, valor) {
        this.placa = placa;          // Nome único do champ
        this.kmAtual = kmAtual;      // HP desta instância
        this.valorCustodia = valor;  // Gold deste champ
    }
}

const v1 = new Veiculo("ABC1234", 100_000, 50_000);
const v2 = new Veiculo("XYZ9876", 250_000, 75_000);

// v1 e v2 são instâncias DIFERENTES
// cada uma tem seus próprios stats
console.log(v1.placa);  // "ABC1234"
console.log(v2.placa);  // "XYZ9876"
```

### Atributos = Stats do Campeão

```javascript
class Veiculo {
    constructor(placa, kmAtual, valor) {
        this.placa = placa;              // Identificador
        this.kmAtual = kmAtual;          // Health
        this.valorCustodia = valor;      // Armor
        this.emManutencao = false;       // Flag: true/false
    }
}

// Cada instância tem stats INDEPENDENTES
v1.kmAtual = 150_000;
v2.kmAtual = 200_000;

console.log(v1.kmAtual);  // 150_000
console.log(v2.kmAtual);  // 200_000 (diferente!)
```

### Estáticos = Buff Global

```javascript
class Veiculo {
    // Estático = Compartilhado por TODAS as instâncias
    static KM_LIMITE_MANUTENCAO = 50_000;
    static TAXA_OPERACAO = 0.15;
    
    constructor(placa, kmAtual) {
        this.placa = placa;
        this.kmAtual = kmAtual;
    }
    
    calcularCusto() {
        // Usa membro estático (igual para todos)
        return this.valorCustodia * Veiculo.TAXA_OPERACAO;
    }
}

// Acesso estático (não precisa de instância)
console.log(Veiculo.KM_LIMITE_MANUTENCAO);  // 50_000 (para todos)

// Alterar estático = afeta TODA classe
Veiculo.TAXA_OPERACAO = 0.20;  // Alterou globalmente

const v1 = new Veiculo("ABC", 100_000);
const v2 = new Veiculo("XYZ", 200_000);

// Ambos usam a nova taxa
console.log(v1.calcularCusto());  // Usa TAXA_OPERACAO 0.20
console.log(v2.calcularCusto());  // Usa TAXA_OPERACAO 0.20
```

**Analogia:**
- **Atributo:** Garen tem 100 HP, Darius tem 120 HP
- **Estático:** Ambos têm o mesmo cd de Flash ? é um buff do servidor, não do champ

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