# Checklist de Leitura de Código ? LoL Edition

## 1. Antes de Ler

**Mapeie o terreno:**
- [ ] Qual é o objetivo? (validação, transformação, persistência, integração, efeito colateral)
- [ ] Em que "lane" está? (Controller ? BO ? DAO, JSP render, JavaScript event, SQL query, JasperReport)
- [ ] Quem chamou isso? (quem é a "lane ally" que depende desse código)
- [ ] Quais são as entradas esperadas? (tipos, nulls, listas vazias)
- [ ] Qual é a saída? (retorno, efeito colateral, estado alterado)

---

## 2. Durante a Leitura

**Siga a rotação passo a passo:**

### Fluxo e Execução
- [ ] Onde começa? (entry point ? Controller, listener, callback)
- [ ] Em que ordem as coisas acontecem?
- [ ] Tem bifurcação? (if/else, switch, validações)
- [ ] Loops criam overhead ou mudam lógica? (N+1 queries? Iterações desnecessárias)
- [ ] Tem async/promises/callbacks? (em qual ordem resolvem?)

### Estado e Dados
- [ ] Que variáveis existem antes do trecho?
- [ ] Quais são alteradas? (mutação local vs global)
- [ ] Vêm de onde? (parâmetro, banco, cache, estado externo)
- [ ] Depende de algo de fora? (timestamp, userId, config, estado da sessão)

### Funções e Dependências
- [ ] Que funções são chamadas aqui?
- [ ] Elas fazem o que prometem ou tem efeitos colaterais?
- [ ] Qual é o contrato delas? (o que entra, o que sai)

### Red Flags ? Procure Sempre Por:
- [ ] `null`/`undefined` sem guarda? ? NPE risk
- [ ] Array/Collection vazio não é tratado? ? IndexOutOfBoundsException ou loop que não executa
- [ ] Mutação direta do objeto recebido? ? side effect invisível
- [ ] String concatenação em SQL? ? SQL Injection
- [ ] Date/Time hardcoded ou assumindo timezone? ? Bug em produção
- [ ] Query sem índice ou JOIN pesado? ? Timeout
- [ ] Exception não é logada ou re-lançada? ? Debug hell
- [ ] Duplicação de lógica similar? ? Violação de DRY, SM vai reparar

---

## 3. Depois da Leitura

**Validação final:**

- [ ] Consigo explicar em uma frase?
- [ ] Consigo descrever a entrada, execução e saída sem olhar?
- [ ] O nome da função/classe combina com o que ela faz?
- [ ] Se algo der errado aqui, aonde procuro? (log, stack trace, database)
- [ ] Existe forma mais simples? (menos variáveis, menos bifurcações, menos dependências)
- [ ] Qual foi a decisão mais importante? (por que `for` e não `foreach`? Por que `Optional.orElse()` e não `if null`?)
- [ ] Se o SM perguntasse "por que assim?" eu tenho resposta sem balbuciar?

---

## Script Mental ? 7 Passos

### 1. **"Qual é a lane?"**
Descubra o papel do código no sistema.
- É Controller (recebe requisição, chama BO)?
- É BO (orquestra regras de negócio)?
- É DAO (persiste ou busca dados)?
- É helper/util (reutilizável)?
- É listener/interceptor (reage a algo)?

### 2. **"Quem é o inimigo?"**
Qual problema ele resolve?
- Validação de entrada ? protege de lixo
- Cálculo complexo ? evita erro manual
- Integração com terceiro ? mantém contrato
- Query otimizada ? evita timeout

### 3. **"Quem tem visão?"**
Que dados o código vê?
- Parâmetros de entrada
- Estado do objeto (this.campo)
- Banco de dados
- Config/properties
- Estado de sessão/thread-local

### 4. **"Qual é a rotação?"**
Trace passo a passo.
1. Entra requisição X
2. Valida Y
3. Busca Z do banco
4. Calcula W
5. Persiste ou retorna resultado

Se travar em qualquer passo ? volte para red flags.

### 5. **"Qual é a win condition?"**
O que significa sucesso aqui?
- Inserção no banco com status correto?
- Retorno com todos os campos obrigatórios?
- Cache atualizado?
- Nenhuma NPE?

### 6. **"Tem pickoff ou armadilha?"**
Procure por edge cases e bugs.
- E se a lista vier vazia?
- E se o timestamp for ano passado?
- E se houver divisão por zero?
- E se a conexão cair no meio?
- E se dois threads tentarem isso ao mesmo tempo?

### 7. **"Eu consigo shotcall isso?"**
Você consegue explicar para o time?
Tente uma frase assim:
> "Aqui a gente valida se o veículo tem preventivos pendentes **antes** de usar ele numa OS, porque se não validar, o usuário cria OS com pendência, que viola a regra da Demanda tal."

Se não conseguir falar isso sem tropeçar, releia.

---

## Analogias LoL ? Código

| LoL | Código |
|-----|--------|
| **Visão de mapa** | Rastrear dados + dependências + estado |
| **Recurso** (ouro, vida, mana) | Variáveis (inteiros, strings, objetos) |
| **Campeão/Habilidade** | Função/Classe (tem papel, cooldown, range, impacto) |
| **Cooldown** | Validação (se passou, pode executar; senão, erro) |
| **CC (crowd control)** | Guard/throw (bloqueia execução até condição ser verdadeira) |
| **Itemize** | Refatore/otimize (mude a estratégia se vencer menos) |
| **Replay review** | Debug (volta no exato momento, vê a stack trace, entende aonde quebrou) |
| **Enemy jungle na lane** | SM vendo PR (se você não justifica, você perde confiança) |
| **Base destruída** | Production down (validação falhou, rollback urgente) |

---

## Aplicação Prática ? Por Contexto

### Lendo DAO (Query/Persistência)
- [ ] SQL está correto? (JOINs são necessários? Há ambiguidade? Índices ajudam?)
- [ ] Parámetros estão escapados? (não tem concatenação?)
- [ ] Resultado é mapeado certo? (tipos, nulls, encoding?)
- [ ] Há N+1 queries? (loop chamando query dentro?)
- [ ] Está usando transaction? (quem faz commit?)

### Lendo BO (Lógica)
- [ ] Validações vêm antes ou depois da mutação?
- [ ] Chamadas a DAO estão na ordem certa?
- [ ] Se uma DAO falhar, o que acontece? (rollback, log, re-throw?)
- [ ] Estado compartilhado é mutável? (thread-safe?)
- [ ] Há duplicação entre métodos do BO? (merging opportunity?)

### Lendo Controller
- [ ] Entrada é validada antes de chamar BO?
- [ ] Response está com status HTTP certo? (200, 400, 404, 500)
- [ ] Exceção é capturada e retorna JSON sensato?
- [ ] CORS/auth está configurado?
- [ ] Qual BO é chamado e por quê?

### Lendo JavaScript (cadastrar_*.js)
- [ ] Evento dispara quando deveria? (onclick, onchange, onsubmit)
- [ ] AJAX espera resposta antes de fazer next step?
- [ ] Resposta é tratada? (sucesso, erro, timeout)
- [ ] DOM é manipulado correto? (ID existe, seletor não é genérico)
- [ ] Há validação client-side antes de enviar?
- [ ] Há atualização visual depois que servidor responde?

### Lendo JSP/HTML
- [ ] Template tem dados do servidor? (como chegam, de que objeto?)
- [ ] Loop itera corretamente? (forEach vs for, acesso a índice)
- [ ] Há XSS risk? (saída é escaped? `${fn:escapeXml(valor)}`)
- [ ] Form tem hidden fields obrigatórios?
- [ ] Button tem tipo certo? (submit vs button)

### Lendo JasperReport (.jrxml)
- [ ] Encoding está correto? (ISO-8859-1 for old systems)
- [ ] Query está dentro? (de onde vêm dados? Há parâmetros?)
- [ ] Acentuação usa `chr()`? (português funciona)
- [ ] Tem variável calculada? (e ela está em escopo certo?)
- [ ] Tipo de dado da coluna está certo? (numeric, string, date)

---

## Sinais de Alerta ? Red Flags Comuns

```
?? sem validação antes de usar
?? sem try/catch em integração externa
?? sem log de erro
?? query dinâmica com string concatenation
?? loop infinito (while sem break claro)
?? mutação de list dentro de stream/lambda
?? ignore exception com catch vazio
?? NPE porque assumiu non-null sem checar
?? duplicação exata de lógica em 2+ lugares
?? sem handling de case vazio (empty list, null object)
?? hardcode de ID, nome, path, URL
?? sem timezone em java.util.Date
?? sem índice em query pesada
?? sem ordenação explícita (SQL sem ORDER BY)
?? callback hell em JavaScript
?? promise não é await-ed
?? DOM query que não garante elemento existe
```

---

## Frases Prontas ? Use No Code Review

**Quando descobrir problema:**
> "Aqui pode vir null/vazio; precisamos validar com `if (list != null && !list.isEmpty())`."

**Quando encontrar duplicação:**
> "Essa lógica é idêntica à linha X; dá pra extrair num método helper?"

**Quando a ordem importar:**
> "O BO chama DAO antes de validar; se validação falhar, banco já foi alterado."

**Quando dúvida sobre decisão:**
> "Por que usou for em vez de forEach aqui? Precisa do índice ou break?"

**Quando coisa está simples demais:**
> "Se o usuário enviar lista vazia, o quê acontece?"

---

## Treino Rápido Diário ? 10 Minutos

Pegue um trecho pequeno (5?15 linhas) e responda:

1. **O quê?** ? Descreva em uma frase.
   - ? "Valida se veículo tem km acima do limite antes de criar OS"

2. **Como?** ? Passo a passo.
   - ? Busca veículo por ID ? Calcula km total ? Compara com limite ? Lança exception ou continua

3. **Onde quebra?** ? Aponte 1 risco.
   - ? "Se `kmMilhao` for null, `NPE` na multiplicação"

4. **Melhora?** ? Sugira refator.
   - ? "Extrair cálculo de km num método `calcularKmTotal()` pra reusar"

5. **Compare.** ? Releia, veja se acertou.
   - ? "Tá, eu vi certo? Mesmo que o banco retorne null, o null é checado aqui?"

---

## Quando Você Trava

**Se travar em qualquer ponto, faça isso:**

1. Volte ao contrato: entrada ? saída
2. Procure o null/empty case
3. Procure onde o estado é alterado
4. Procure chamadas a funções externas
5. Procure loops
6. Procure if/else

Um desses 6 está causando confusão. Garanto.

---

**Lembrete final:** Ler código é como mapear em LoL ? não olhe detalhes isolados, olhe fluxo, ameaça, visão e decisão. A win condition é entender por que aquele código existe e o que quebra se ele virar lixo.