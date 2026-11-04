# AJAX com Java (Servlets, JSON e JavaScript)

---

## Sumário

1. [Introdução: o que é AJAX e por que existe](#1-introdução-o-que-é-ajax-e-por-que-existe)
2. [Fluxo clássico vs fluxo AJAX](#2-fluxo-clássico-vs-fluxo-ajax)
3. [Anatomia de uma chamada AJAX](#3-anatomia-de-uma-chamada-ajax)
4. [Lado servidor: Servlet respondendo texto e JSON](#4-lado-servidor-servlet-respondendo-texto-e-json)
5. [`XMLHttpRequest`: a base de tudo](#5-xmlhttprequest-a-base-de-tudo)
6. [`fetch` e `async/await`](#6-fetch-e-asyncawait)
7. [jQuery: `$.ajax`, `$.get` e `$.post`](#7-jquery-ajax-get-e-post)
8. [Enviando dados: query string, form e JSON](#8-enviando-dados-query-string-form-e-json)
9. [Recebendo e manipulando JSON (Gson)](#9-recebendo-e-manipulando-json-gson)
10. [Tratamento de erros e status HTTP](#10-tratamento-de-erros-e-status-http)
11. [Padrões comuns em sistemas legados](#11-padrões-comuns-em-sistemas-legados)
12. [Sessão expirada, loading e debounce](#12-sessão-expirada-loading-e-debounce)
13. [Segurança: CSRF, XSS e validação](#13-segurança-csrf-xss-e-validação)
14. [Debugando AJAX](#14-debugando-ajax)
15. [Exercício resolvido: rastreamento de entregas sem recarregar a página](#15-exercício-resolvido-rastreamento-de-entregas-sem-recarregar-a-página)
16. [Armadilhas e boas práticas](#16-armadilhas-e-boas-práticas)
17. [Cheat sheet](#17-cheat-sheet)

---

## 1. Introdução: o que é AJAX e por que existe

**AJAX** = *Asynchronous JavaScript And XML*. Apesar do nome, hoje quase ninguém trafega XML: o formato dominante é **JSON** (ver [[33-API-JAVA]]).

A ideia central: **o navegador conversa com o servidor em segundo plano, sem recarregar a página inteira**.

> 🚚 Analogia logística: sem AJAX, toda vez que você quer saber o status de **uma** entrega, o sistema reimprime o **manifesto inteiro** da frota. Com AJAX, você faz uma ligação rápida para o despachante, pergunta só daquela entrega, e atualiza **apenas aquela linha** no painel.

### O que AJAX não é

- Não é uma linguagem nem uma biblioteca: é uma **técnica**.
- Não é "o Java fazendo algo diferente": o Java (Servlet) continua recebendo uma requisição HTTP normal e devolvendo uma resposta. Quem muda é o **cliente**, que decide o que fazer com a resposta.

### Por que importa para quem faz backend Java

O Servlet que atende AJAX é o mesmo que você já conhece. A diferença está no **contrato de resposta**: em vez de redirecionar para uma JSP, ele devolve **dados** (texto ou JSON) e o JavaScript decide como mostrá-los.

---

## 2. Fluxo clássico vs fluxo AJAX

```
FLUXO CLÁSSICO (página inteira)
Browser ──GET /EntregaControlador?acao=listar──► Servlet
Browser ◄── HTML completo (cabeçalho, menu, tabela, rodapé) ── Servlet
         └─ a tela inteira "pisca" e recarrega

FLUXO AJAX (só os dados)
Browser ──fetch('/EntregaControlador?acao=carregar&id=7')──► Servlet
Browser ◄── {"codigo":"LOG-007","status":"EM_TRANSITO"} ──── Servlet
         └─ JavaScript atualiza apenas o <span> do status
```

| Aspecto | Clássico | AJAX |
|---|---|---|
| O que trafega | HTML completo | Dados (JSON/texto) ou fragmento HTML |
| Estado da tela | Perdido a cada requisição | Preservado (scroll, campos preenchidos) |
| Quem monta a interface | Servidor (JSP) | Cliente (JavaScript) |
| Experiência | Pisca/recarrega | Fluida |
| Complexidade | Menor | Maior (erros, loading, sessão) |

---

## 3. Anatomia de uma chamada AJAX

Toda chamada AJAX, em qualquer biblioteca, tem os mesmos elementos:

| Elemento | Pergunta que responde | Exemplo |
|---|---|---|
| **URL** | Para onde vou? | `EntregaControlador` |
| **Método** | O que quero fazer? | `GET`, `POST` |
| **Parâmetros/corpo** | O que envio? | `acao=carregar&id=7` |
| **Tipo de resposta** | Como interpreto o retorno? | `json`, `text` |
| **Callback de sucesso** | O que faço se deu certo? | preencher campos |
| **Callback de erro** | O que faço se falhou? | avisar o usuário |

### Ciclo de vida

```
1. Evento (clique, ready, timer)
2. JavaScript monta a requisição
3. Navegador envia (sem travar a tela — é assíncrono)
4. Servlet processa e responde
5. Callback / Promise é executado com a resposta
6. JavaScript atualiza o DOM
```

> ⚠️ **Assíncrono** significa que a linha seguinte do JavaScript executa **antes** da resposta chegar. Quem usa o resultado precisa estar **dentro** do callback (ou após o `await`).

```javascript
let resultado;
fetch('EntregaControlador?acao=carregar&id=7')
    .then(r => r.json())
    .then(dados => { resultado = dados; });

console.log(resultado); // undefined! a resposta ainda não chegou
```

---

## 4. Lado servidor: Servlet respondendo texto e JSON

Muitos sistemas legados usam o padrão **Controlador com parâmetro `acao`**: um único Servlet atende várias operações, e o `acao` seleciona qual.

### Servlet de exemplo (didático, domínio de logística)

```java
package com.logistica.web;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.time.LocalDate;

public class EntregaControlador extends HttpServlet {

    private static final String CARREGAR = "carregar";
    private static final String EXCLUIR  = "excluir";

    private final EntregaBO bo = new EntregaBO();

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        processar(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        processar(req, resp);
    }

    private void processar(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        String acao = req.getParameter("acao");

        if (CARREGAR.equalsIgnoreCase(acao)) {
            carregar(req, resp);
        } else if (EXCLUIR.equalsIgnoreCase(acao)) {
            excluir(req, resp);
        } else {
            resp.sendError(HttpServletResponse.SC_BAD_REQUEST, "acao inválida");
        }
    }

    // Responde JSON
    private void carregar(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        int id = Integer.parseInt(req.getParameter("id"));
        Entrega entrega = bo.carregar(id);

        resp.setContentType("application/json;charset=UTF-8");
        Gson gson = new GsonBuilder().setDateFormat("dd/MM/yyyy").create();
        resp.getWriter().print(gson.toJson(entrega));
    }

    // Responde texto simples (mensagem para o usuário)
    private void excluir(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        bo.excluir(Integer.parseInt(req.getParameter("id")));

        resp.setContentType("text/plain;charset=UTF-8");
        try (PrintWriter out = resp.getWriter()) {
            out.print("Entrega excluída com sucesso");
        }
    }
}
```

### Regras de ouro do lado servidor

1. **Não fazer `forward`/`redirect` para JSP** em ação AJAX: o `fetch` receberia o HTML da página, não os dados.
2. **Definir o `Content-Type`** correto: `application/json` para JSON, `text/plain` para mensagens.
3. **Definir o `charset`** explicitamente. Atenção: sistemas legados costumam usar **ISO-8859-1**; um erro de charset gera "acentuação quebrada" (`Ã§Ã£o`).
4. **Usar status HTTP corretos** para erro (ver [seção 10](#10-tratamento-de-erros-e-status-http)).
5. **Camadas preservadas**: Controlador → BO → DAO. O Controlador só traduz HTTP.

> 📌 `javax.servlet` é usado em projetos que rodam até o Tomcat 9. A partir do Tomcat 10 / Jakarta EE 9, o pacote passa a ser `jakarta.servlet`.

---

## 5. `XMLHttpRequest`: a base de tudo

É a API original (anos 2000). Ainda aparece em código legado e em bibliotecas de terceiros.

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'EntregaControlador?acao=carregar&id=7', true); // true = assíncrono

xhr.onload = function () {
    if (xhr.status === 200) {
        const entrega = JSON.parse(xhr.responseText);
        console.log(entrega.status);
    } else {
        console.error('Falha HTTP', xhr.status);
    }
};

xhr.onerror = function () {
    console.error('Falha de rede');
};

xhr.send();
```

### `readyState`

| Valor | Constante | Significado |
|---|---|---|
| 0 | `UNSENT` | Objeto criado, `open()` ainda não chamado |
| 1 | `OPENED` | `open()` chamado |
| 2 | `HEADERS_RECEIVED` | Cabeçalhos recebidos |
| 3 | `LOADING` | Corpo sendo recebido |
| 4 | `DONE` | Operação concluída |

### Exemplo: download como `blob`

O `XMLHttpRequest` também baixa um arquivo binário:

```javascript
var ajax = new XMLHttpRequest();
ajax.open("GET", url, true);
ajax.responseType = 'blob';          // resposta binária, não texto
ajax.onload = function (e) {
    download(e.target.response, fileName, defaultMime);
};
ajax.send();
```

> `responseType = 'blob'` é o que permite baixar PDF, imagem ou planilha gerada pelo servidor.

---

## 6. `fetch` e `async/await`

`fetch` é a API moderna, baseada em **Promises**.

```javascript
fetch('EntregaControlador?acao=carregar&id=7')
    .then(response => {
        if (!response.ok) {
            throw new Error('HTTP ' + response.status);
        }
        return response.json();
    })
    .then(entrega => console.log(entrega.codigo))
    .catch(erro => console.error('Falhou:', erro));
```

### Com `async/await` (mais legível)

```javascript
async function carregarEntrega(id) {
    try {
        const response = await fetch('EntregaControlador?acao=carregar&id=' + id);
        if (!response.ok) {
            throw new Error('HTTP ' + response.status);
        }
        return await response.json();
    } catch (erro) {
        console.error('Falhou:', erro);
        return null;
    }
}
```

### Pegadinha fundamental

`fetch` **só rejeita a Promise em falha de rede**. Um `404` ou `500` **não cai no `catch`**: você precisa checar `response.ok` (ou `response.status`).

| Situação | `xhr.onerror` / `fetch catch` | Precisa checar status |
|---|---|---|
| Servidor fora do ar | ✅ dispara | — |
| Servidor respondeu `500` | ❌ não dispara | ✅ `response.ok === false` |
| Servidor respondeu `200` com mensagem de erro no corpo | ❌ não dispara | ✅ depende do seu contrato |

### Exemplo: `validaSession` como *guard*

```javascript
async function validaSession(metodo) {
    const responseAjax = await fetch('./UsuarioControlador?acao=validaSession');
    const response = await responseAjax.text();

    if (response === '0') {
        window.open('./UsuarioControlador?acao=iniciarLogarNovamente',
                    'loginNovamente', 'height=220, width=250, scrollbars=no,resizable=no');
    } else {
        metodo();
    }
}
```

**Como ler (script mental):**

1. `validaSession` recebe uma função (`metodo`) que só deve rodar se a sessão estiver válida (*callback como parâmetro*).
2. Faz `fetch` para o Controlador com `acao=validaSession`.
3. Lê a resposta como **texto** (`.text()`), não JSON.
4. Se vier `"0"` → sessão expirada → abre popup de login.
5. Caso contrário → executa `metodo()`.

E o lado Java correspondente:

```java
private void validaSession(HttpServletRequest request, HttpServletResponse response, PrintWriter out) throws IOException {
    response.setHeader("Cache-Control", "no-cache");

    if (request.getSession().getAttribute("autenticado") == null) {
        out.println("0");
    } else {
        out.println("1");
    }
}
```

> Repare no `Cache-Control: no-cache`: sem isso o navegador poderia servir uma resposta antiga em cache e dizer que a sessão está ativa quando já expirou.

---

## 7. jQuery: `$.ajax`, `$.get` e `$.post`

Muito presente em sistemas legados (onde é comum encontrar versões 1.x e 3.x convivendo). Abstrai diferenças entre navegadores.

### `$.get` / `$.post`

```javascript
$.get('EntregaControlador', { acao: 'carregar', id: 7 }, function (entrega) {
    console.log(entrega.status);
}, 'json');

$.post('EntregaControlador', { acao: 'excluir', id: 7 }, function (mensagem) {
    alert(mensagem);
}, 'text');
```

### `$.ajax` (forma completa)

```javascript
$.ajax({
    url: 'EntregaControlador',
    type: 'POST',               // ou method
    data: { acao: 'carregar', id: 7 },
    dataType: 'json',           // como interpretar a resposta
    beforeSend: function () { /* mostrar loading */ },
    success: function (dados) { /* sucesso HTTP 2xx */ },
    error: function (jqXHR, textStatus, errorThrown) { /* falha */ },
    complete: function () { /* roda SEMPRE: sucesso ou erro */ }
});
```

### `success` vs `complete`: a diferença que gera bugs

| Callback | Quando executa | Recebe |
|---|---|---|
| `success` | Apenas se HTTP 2xx **e** o parse do `dataType` deu certo | Dados já convertidos |
| `error` | Falha de rede, HTTP 4xx/5xx ou parse inválido | `jqXHR`, `textStatus`, `errorThrown` |
| `complete` | **Sempre**, depois de `success` ou `error` | `jqXHR`, `textStatus` |

### Equivalência `jQuery` ↔ `fetch`

| Conceito | jQuery | fetch |
|---|---|---|
| GET simples | `$.get(url, cb)` | `fetch(url).then(r => r.json())` |
| POST com dados | `$.post(url, obj, cb)` | `fetch(url, {method:'POST', body: new URLSearchParams(obj)})` |
| Erro HTTP | `error:` | `if (!response.ok)` |
| Sempre executar | `complete:` | `.finally()` |
| Resposta JSON | `dataType:'json'` | `response.json()` |

> ⚠️ Em sistemas legados, o código usa `$` e `jQuery` de forma intercambiável. Quando há outras bibliotecas (como `prototype.js`) na mesma página, `$` pode ser sobrescrito — por isso muitos arquivos usam `jQuery.ajax` explicitamente (modo `noConflict`).

---

## 8. Enviando dados: query string, form e JSON

### 8.1 Query string (GET)

```javascript
fetch('EntregaControlador?acao=carregar&id=' + encodeURIComponent(id));
```

Lado Java: `request.getParameter("id")`.

> Sempre use `encodeURIComponent` em valores digitados pelo usuário (espaços, `&`, `#`, acentos).

### 8.2 Formulário (`application/x-www-form-urlencoded`)

É o padrão do `$.post` com objeto:

```javascript
$.post('EntregaControlador', { acao: 'cadastrar', destino: 'Curitiba', pesoKg: 120 });
```

Lado Java: continua `request.getParameter("destino")`.

### 8.3 JSON dentro de um parâmetro (padrão legado)

Em sistemas legados é comum serializar o formulário inteiro em JSON e enviá-lo **em um único parâmetro `form`**:

```javascript
var formulario = formToJson('#formEntrega');   // função utilitária que converte o form em JSON

$.post('EntregaControlador', {
    'acao': 'cadastrar',
    'form': formulario
}, function (data) { /* ... */ }, 'json');
```

Lado Java:

```java
String form = request.getParameter("form");
JsonObject jsonObject = JsonParser.parseString(form).getAsJsonObject();

entrega.setCodigo(jsonObject.getAsJsonPrimitive("codigo").getAsString());
entrega.setDestino(jsonObject.getAsJsonPrimitive("destino").getAsString());
```

### 8.4 Corpo JSON puro (`application/json`)

```javascript
fetch('EntregaControlador', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ destino: 'Curitiba', pesoKg: 120 })
});
```

Lado Java: **`getParameter` NÃO funciona** aqui. É preciso ler o corpo:

```java
String corpo = req.getReader().lines().collect(Collectors.joining());
Entrega entrega = new Gson().fromJson(corpo, Entrega.class);
```

### Comparativo

| Formato | `Content-Type` | Java lê com | Quando usar |
|---|---|---|---|
| Query string | — | `getParameter` | Filtros, IDs, `GET` |
| Form urlencoded | `application/x-www-form-urlencoded` | `getParameter` | Formulários simples |
| JSON em parâmetro | `application/x-www-form-urlencoded` | `getParameter` + `JsonParser` | Padrão legado comum |
| JSON no corpo | `application/json` | `req.getReader()` + Gson | APIs REST modernas |

---

## 9. Recebendo e manipulando JSON (Gson)

### Serialização no servidor

```java
Gson gson = new GsonBuilder().setDateFormat("dd/MM/yyyy").create();
response.getWriter().print(gson.toJson(entrega));
```

O `setDateFormat` evita que o `Date` vire um texto ambíguo no JSON — o front recebe a data já no formato brasileiro.

### Consumo no cliente

```javascript
const obj = JSON.parse(texto);     // texto -> objeto
const texto2 = JSON.stringify(obj); // objeto -> texto
```

### Padrão de preenchimento de formulário

O JSON devolvido tem chaves iguais aos `id`s dos campos, então um laço preenche tudo:

```javascript
$.ajax({
    url: 'EntregaControlador',
    async: true,
    dataType: 'text',
    data: {
        'acao': 'carregar',
        'id': qs['id']
    },
    complete: function (jqXHR, textStatus) {
        var obj = JSON.parse(jqXHR.responseText);
        Object.keys(obj).forEach(function (chave) {
            var valor = obj[chave];
            $("#" + chave).val(valor);       // campo #destino recebe obj.destino
        });
        // ...
    }
});
```

**Leitura:**

1. Pede a entrega pelo `id`.
2. `dataType: 'text'` → o jQuery **não** converte; a conversão é manual (`JSON.parse`).
3. Para cada chave do JSON, procura `#chave` na tela e preenche o valor.

**Contrato implícito:** o nome do campo no HTML = nome da propriedade na entidade Java. Funciona, mas é frágil (ver [armadilhas](#16-armadilhas-e-boas-práticas)).

### Objetos aninhados

```javascript
obj['criadoPor']['nome']    // Entrega.criadoPor.nome (objeto Usuario)
```

Se `criadoPor` vier `null` do servidor, `obj['criadoPor']['nome']` lança `TypeError`. Prefira:

```javascript
const nome = obj.criadoPor?.nome ?? '—';   // optional chaining + nullish coalescing
```

---

## 10. Tratamento de erros e status HTTP

### Famílias de status relevantes para AJAX

| Faixa | Significado | O que o JavaScript deve fazer |
|---|---|---|
| `2xx` | Sucesso | Seguir o fluxo normal |
| `3xx` | Redirecionamento | Cuidado: `fetch` segue sozinho e pode devolver o HTML de login |
| `400` | Requisição inválida | Mostrar mensagem de validação |
| `401` | Não autenticado | Pedir login |
| `403` | Sem permissão | Avisar permissão negada |
| `404` | Não encontrado | Avisar recurso inexistente |
| `500` | Erro no servidor | Mensagem genérica + log |

### Lado servidor: erro com status correto

```java
try {
    bo.excluir(id);
    resp.setContentType("text/plain;charset=UTF-8");
    resp.getWriter().print("Entrega excluída com sucesso");
} catch (EntregaNaoEncontradaException e) {
    resp.setStatus(HttpServletResponse.SC_NOT_FOUND);       // 404
    resp.getWriter().print(e.getMessage());
} catch (RegraNegocioException e) {
    resp.setStatus(HttpServletResponse.SC_CONFLICT);        // 409
    resp.getWriter().print(e.getMessage());
}
```

### Lado cliente: tratar tudo

```javascript
async function excluirEntrega(id) {
    try {
        const response = await fetch('EntregaControlador', {
            method: 'POST',
            body: new URLSearchParams({ acao: 'excluir', id })
        });
        const mensagem = await response.text();

        if (!response.ok) {
            exibirErro(mensagem);          // 4xx / 5xx com mensagem do servidor
            return;
        }
        exibirSucesso(mensagem);
    } catch (e) {
        exibirErro('Sem conexão com o servidor');   // falha de rede
    }
}
```

### Padrão legado para erro de negócio

Muitos sistemas legados respondem **`200 OK` com um campo `erro` no JSON**:

```javascript
$.post('UsuarioControlador', { 'acao': 'renovarSenha', 'form': formulario },
    function (data) {
        if (data) {
            if (data['erro']) {
                chamarAlert(data['erro']);         // erro de negócio
            } else {
                chamarAlert('Senha atualizada com sucesso! Por favor, logue novamente', function () {
                    window.location = homePath;
                });
            }
        }
    }, 'json');
```

| Abordagem | Vantagem | Desvantagem |
|---|---|---|
| `200` + `{erro: "..."}` (legado) | Simples, o `success` sempre roda | Esconde falhas de monitoramento; status não reflete o resultado |
| Status `4xx/5xx` + mensagem | Semântico, ferramentas de monitoramento enxergam | Exige tratar `error`/`!ok` no cliente |

> Em código **novo**, prefira status HTTP corretos ([[33-API-JAVA]], seção 6). Em código **legado**, siga o padrão da tela para não quebrar o contrato existente.

---

## 11. Padrões comuns em sistemas legados

Resumo dos padrões que aparecem com frequência em sistemas Java web legados, do mais antigo ao mais novo:

| # | Padrão | Tecnologia |
|---|---|---|
| 1 | Carregar registro e preencher o form | `$.ajax` + `dataType:'text'` + `JSON.parse` |
| 2 | Excluir e recarregar | `jQuery.ajax` com `success` |
| 3 | Enviar formulário como JSON em parâmetro | `$.post` + `'json'` |
| 4 | Validar sessão antes de executar | `fetch` + `async/await` |
| 5 | Carregar componente sob demanda | `fetch` + loader de componentes |
| 6 | Download de binário | `XMLHttpRequest` + `blob` |

### Padrão 2: excluir com confirmação

```javascript
function aceitouExcluirEntrega(ids) {
    jQuery.ajax({
        url: 'EntregaControlador?acao=excluirAjax&ids=' + ids,
        success: function (data, textStatus, jqXHR) {
            chamarAlert(data, reload);
        }
    });
}

function reload() {
    window.location.reload();
}
```

**Fluxo:** usuário confirma (`chamarConfirm`) → AJAX exclui → servidor devolve **texto** com a mensagem → `chamarAlert` exibe e, ao fechar, `reload()` recarrega a lista.

> 🔎 Observação: `excluirAjax` usa `GET` para uma operação que **altera dados**. Isso viola a semântica HTTP (ver [[33-API-JAVA]], seção 5): `GET` deve ser seguro e idempotente. Em código novo, use `POST` ou `DELETE`.

### Padrão 5: cache-busting no carregamento de componentes

```javascript
getFile(url) {
    return fetch(url + '?v=' + Date.now())
        .then(response => response.ok ? response.text() : Promise.reject(response));
},
```

- `?v=` + `Date.now()` gera uma URL diferente a cada chamada → o navegador **não usa cache** do arquivo.
- `response.ok ? ... : Promise.reject(response)` é o jeito correto de transformar erro HTTP em rejeição da Promise.

### Convenção de Controlador

Os exemplos seguem o mesmo contrato:

```
URL:        <Nome>Controlador
Parâmetro:  acao=<operação>
Resposta:   texto simples (mensagens) ou JSON (dados)
```

Isso facilita localizar o código: ao ver `acao: 'carregar'` no JavaScript, procure a constante `CARREGAR` no `*Controlador.java` correspondente.

---

## 12. Sessão expirada, loading e debounce

### 12.1 Sessão expirada em chamada AJAX

Problema clássico: a sessão expira, o usuário clica em um botão que dispara AJAX e o servidor redireciona para o **HTML de login**. O `fetch` segue o redirect e o `response.json()` explode (`Unexpected token <`).

Soluções:

- Validar a sessão antes (como o `validaSession` da seção 6).
- Responder `401` em chamadas AJAX e tratar no cliente:

```javascript
if (response.status === 401) {
    abrirLoginNovamente();
    return;
}
```

Uso do `validaSession` como *guard*:

```javascript
validaSession(function () {
    salvarEntrega();       // só roda se a sessão estiver válida
});
```

### 12.2 Indicador de carregamento (loading)

Telas de cadastro costumam mostrar um bloqueio durante a requisição:

```javascript
$('.bloqueio-tela').show();
$('.gif-bloq-tela').show();

$.ajax({
    url: 'EntregaControlador',
    data: { acao: 'carregar', id: qs['id'] },
    dataType: 'text',
    complete: function (jqXHR) {
        // ... preenche a tela ...
        $('.bloqueio-tela').hide();
        $('.gif-bloq-tela').hide();
    }
});
```

Padrão ideal: **mostrar antes, esconder no `complete`/`finally`**, para que a tela nunca fique travada se ocorrer erro.

```javascript
mostrarLoading();
try {
    await fetch(url);
} finally {
    esconderLoading();   // roda mesmo com erro
}
```

### 12.3 Debounce: não disparar AJAX a cada tecla

Um exemplo clássico é salvar preferências do usuário só depois de **4 segundos sem alteração**:

```javascript
var saveTimeOut;
var tempoEspera = 4000;

function monitorarPreferenciaUsuario() {
    if (saveTimeOut !== null && saveTimeOut !== undefined) {
        clearTimeout(saveTimeOut);          // cancela o timer anterior
    }
    saveTimeOut = setTimeout(function () {
        salvarPreferencias();               // AJAX só dispara aqui
    }, tempoEspera);
}
```

```
usuário altera filtro ──► reinicia timer (4s)
usuário altera de novo ──► reinicia timer (4s)
usuário para            ──► 4s depois: UMA chamada AJAX
```

> 🚚 Analogia: o despachante só fecha o manifesto **depois** que o motorista parou de pedir mudanças. Sem debounce, cada mudança geraria uma ligação.

Usos típicos: autocomplete, pesquisa enquanto digita, autosave.

---

## 13. Segurança: CSRF, XSS e validação

AJAX não elimina riscos de uma requisição HTTP comum; só muda por onde eles passam.

| Risco | O que é | Defesa |
|---|---|---|
| **XSS** | Dado do servidor inserido como HTML executa script malicioso | Usar `.textContent`/`.text()`; nunca `.innerHTML`/`.html()` com dado não confiável |
| **CSRF** | Site malicioso dispara uma requisição usando a sessão do usuário logado | Token anti-CSRF; `SameSite` em cookies; não alterar dados via `GET` |
| **Validação só no front** | Usuário contorna o JavaScript e chama o Servlet direto | **Sempre** validar no servidor (BO) |
| **Falta de autorização** | `acao=excluir` aceita qualquer usuário logado | Checar permissão no BO a cada ação |
| **Vazamento de erro** | Stack trace devolvido ao cliente | Logar internamente, devolver mensagem genérica |

### XSS na prática

```javascript
// ❌ Perigoso: se entrega.destino contiver <script>, executa
$('#destino').html(entrega.destino);

// ✅ Seguro: trata como texto
$('#destino').text(entrega.destino);
document.getElementById('destino').textContent = entrega.destino;
```

### Regra central

> **O JavaScript é território do usuário.** Qualquer validação, botão escondido ou campo `disabled` pode ser burlado. A autoridade final está sempre no servidor.

---

## 14. Debugando AJAX

Complementa [[39-DEBUGANDO-JAVA]]: aqui o bug pode estar no **cliente**, no **servidor** ou no **contrato** entre eles.

### Aba *Network* do navegador (F12)

Para cada chamada, confira:

| Aba | O que verificar |
|---|---|
| **Headers** | URL final, método, `Content-Type` da requisição e da resposta, status |
| **Payload / Params** | O que foi realmente enviado (`acao`, `id`, `form`) |
| **Response** | O corpo cru devolvido (HTML de erro? JSON? texto?) |
| **Timing** | Tempo de espera do servidor (TTFB) — lentidão no backend |
| **Console** | Erros de `JSON.parse`, `TypeError`, CORS |

### Diagnóstico por sintoma

| Sintoma | Causa provável | Onde olhar |
|---|---|---|
| `Unexpected token < in JSON` | Servidor devolveu HTML (login, erro, JSP) | Aba *Response* |
| `acao` chega `null` no Servlet | Parâmetro não enviado ou corpo JSON lido com `getParameter` | Aba *Payload* |
| Acentos quebrados (`Ã§Ã£o`) | Charset divergente (UTF-8 vs ISO-8859-1) | `Content-Type` da resposta |
| Callback `success` nunca roda | `dataType:'json'` com resposta inválida → cai no `error` | Adicionar `error:` |
| Dado antigo aparece | Cache do navegador | Header `Cache-Control`, cache-busting `?v=` |
| `TypeError: Cannot read properties of null` | Objeto aninhado nulo (`criadoPor`) | Console + optional chaining |
| Nada acontece ao clicar | Erro JS anterior interrompeu o script | Console |
| `404` no Controlador | URL relativa errada (`./` vs `homePath`) | Aba *Headers* |

### Lado servidor

- Breakpoint no método do Controlador (`carregar`) e inspecionar `request.getParameterMap()`.
- Logar a ação recebida:

```java
LOG.debug("acao={} id={}", req.getParameter("acao"), req.getParameter("id"));
```

### Log temporário no cliente

```javascript
$.ajax({
    url: 'EntregaControlador',
    data: { acao: 'carregar', id: 7 },
    dataType: 'text',
    complete: function (jqXHR, textStatus) {
        console.log('status:', jqXHR.status, textStatus);
        console.log('corpo:', jqXHR.responseText);
    }
});
```

---

## 15. Exercício resolvido: rastreamento de entregas sem recarregar a página

**Enunciado:** uma tela de painel mostra o status de uma entrega. O usuário digita o código, clica em "Consultar" e o status aparece **sem recarregar a página**. Há também um botão "Cancelar" que usa `POST`, confirma e atualiza só a linha afetada.

### 15.1 Domínio

```java
package com.logistica.web;

import java.time.LocalDate;

public record Entrega(String codigo, String destino, double pesoKg, String status, LocalDate prazo) {}
```

### 15.2 BO simulado

```java
package com.logistica.web;

import java.time.LocalDate;
import java.util.HashMap;
import java.util.Map;

public class EntregaBO {

    private static final Map<String, Entrega> BASE = new HashMap<>();

    static {
        BASE.put("LOG-001", new Entrega("LOG-001", "São Paulo", 120.0, "EM_TRANSITO", LocalDate.now().plusDays(2)));
        BASE.put("LOG-002", new Entrega("LOG-002", "Curitiba", 75.5, "PENDENTE", LocalDate.now().plusDays(5)));
        BASE.put("LOG-003", new Entrega("LOG-003", "Recife", 200.0, "ENTREGUE", LocalDate.now().minusDays(1)));
    }

    public Entrega buscar(String codigo) {
        Entrega e = BASE.get(codigo);
        if (e == null) {
            throw new EntregaNaoEncontradaException("Entrega " + codigo + " não encontrada");
        }
        return e;
    }

    public Entrega cancelar(String codigo) {
        Entrega atual = buscar(codigo);
        if ("ENTREGUE".equals(atual.status())) {
            throw new RegraNegocioException("Entrega já concluída não pode ser cancelada");
        }
        Entrega cancelada = new Entrega(atual.codigo(), atual.destino(), atual.pesoKg(), "CANCELADA", atual.prazo());
        BASE.put(codigo, cancelada);
        return cancelada;
    }
}

class EntregaNaoEncontradaException extends RuntimeException {
    EntregaNaoEncontradaException(String m) { super(m); }
}

class RegraNegocioException extends RuntimeException {
    RegraNegocioException(String m) { super(m); }
}
```

### 15.3 Controlador (padrão `acao`, status HTTP corretos)

```java
package com.logistica.web;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonObject;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.time.LocalDate;

@WebServlet("/EntregaControlador")
public class EntregaControlador extends HttpServlet {

    private final EntregaBO bo = new EntregaBO();
    private final Gson gson = new GsonBuilder()
            .registerTypeAdapter(LocalDate.class,
                    (com.google.gson.JsonSerializer<LocalDate>) (d, t, c) ->
                            new com.google.gson.JsonPrimitive(d.toString()))
            .create();

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        if (!"carregar".equalsIgnoreCase(req.getParameter("acao"))) {
            responderErro(resp, HttpServletResponse.SC_BAD_REQUEST, "acao inválida");
            return;
        }
        try {
            Entrega e = bo.buscar(req.getParameter("codigo"));
            resp.setContentType("application/json;charset=UTF-8");
            resp.getWriter().print(gson.toJson(e));
        } catch (EntregaNaoEncontradaException ex) {
            responderErro(resp, HttpServletResponse.SC_NOT_FOUND, ex.getMessage());
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        if (!"cancelar".equalsIgnoreCase(req.getParameter("acao"))) {
            responderErro(resp, HttpServletResponse.SC_BAD_REQUEST, "acao inválida");
            return;
        }
        try {
            Entrega e = bo.cancelar(req.getParameter("codigo"));
            resp.setContentType("application/json;charset=UTF-8");
            resp.getWriter().print(gson.toJson(e));
        } catch (EntregaNaoEncontradaException ex) {
            responderErro(resp, HttpServletResponse.SC_NOT_FOUND, ex.getMessage());
        } catch (RegraNegocioException ex) {
            responderErro(resp, HttpServletResponse.SC_CONFLICT, ex.getMessage());
        }
    }

    private void responderErro(HttpServletResponse resp, int status, String mensagem) throws IOException {
        JsonObject corpo = new JsonObject();
        corpo.addProperty("erro", mensagem);
        resp.setStatus(status);
        resp.setContentType("application/json;charset=UTF-8");
        resp.getWriter().print(corpo);
    }
}
```

### 15.4 Página (HTML + JavaScript com `fetch`)

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Rastreamento</title>
</head>
<body>
    <h1>Rastrear entrega</h1>

    <input id="codigo" placeholder="LOG-001">
    <button id="btnConsultar">Consultar</button>

    <div id="loading" hidden>Carregando...</div>
    <p id="mensagem"></p>

    <table id="resultado" hidden>
        <tr><th>Código</th><td id="rCodigo"></td></tr>
        <tr><th>Destino</th><td id="rDestino"></td></tr>
        <tr><th>Peso (kg)</th><td id="rPeso"></td></tr>
        <tr><th>Status</th><td id="rStatus"></td></tr>
        <tr><th>Prazo</th><td id="rPrazo"></td></tr>
    </table>
    <button id="btnCancelar" hidden>Cancelar entrega</button>

    <script>
        const el = id => document.getElementById(id);
        let codigoAtual = null;

        async function chamar(url, opcoes) {
            el('loading').hidden = false;
            try {
                const response = await fetch(url, opcoes);
                const dados = await response.json();
                if (!response.ok) {
                    throw new Error(dados.erro || ('HTTP ' + response.status));
                }
                return dados;
            } finally {
                el('loading').hidden = true;        // sempre esconde o loading
            }
        }

        function mostrar(entrega) {
            codigoAtual = entrega.codigo;
            el('rCodigo').textContent  = entrega.codigo;      // textContent evita XSS
            el('rDestino').textContent = entrega.destino;
            el('rPeso').textContent    = entrega.pesoKg;
            el('rStatus').textContent  = entrega.status;
            el('rPrazo').textContent   = entrega.prazo;
            el('resultado').hidden  = false;
            el('btnCancelar').hidden = ['CANCELADA', 'ENTREGUE'].includes(entrega.status);
        }

        el('btnConsultar').addEventListener('click', async () => {
            el('mensagem').textContent = '';
            const codigo = el('codigo').value.trim();
            if (!codigo) {
                el('mensagem').textContent = 'Informe o código';
                return;
            }
            try {
                const entrega = await chamar(
                    'EntregaControlador?acao=carregar&codigo=' + encodeURIComponent(codigo));
                mostrar(entrega);
            } catch (e) {
                el('resultado').hidden = true;
                el('btnCancelar').hidden = true;
                el('mensagem').textContent = e.message;
            }
        });

        el('btnCancelar').addEventListener('click', async () => {
            if (!confirm('Cancelar a entrega ' + codigoAtual + '?')) return;
            try {
                const entrega = await chamar('EntregaControlador', {
                    method: 'POST',
                    body: new URLSearchParams({ acao: 'cancelar', codigo: codigoAtual })
                });
                mostrar(entrega);                  // atualiza só a tabela
                el('mensagem').textContent = 'Entrega cancelada';
            } catch (e) {
                el('mensagem').textContent = e.message;   // ex.: 409 regra de negócio
            }
        });
    </script>
</body>
</html>
```

### 15.5 O que acontece em cada cenário

| Ação do usuário | Requisição | Resposta | Efeito na tela |
|---|---|---|---|
| Consulta `LOG-001` | `GET ...?acao=carregar&codigo=LOG-001` | `200` + JSON | Tabela preenchida |
| Consulta `XYZ` | `GET ...?acao=carregar&codigo=XYZ` | `404` + `{"erro":"..."}` | Mensagem de erro |
| Cancela `LOG-001` | `POST acao=cancelar` | `200` + JSON (`CANCELADA`) | Status atualizado, botão some |
| Cancela `LOG-003` (entregue) | `POST acao=cancelar` | `409` + `{"erro":"..."}` | Mensagem de regra de negócio |

### 15.6 Pontos de atenção do exercício

- O código no JavaScript **não recarrega** a página em nenhum momento.
- `finally` garante que o loading some em qualquer cenário.
- Erro de negócio usa **status HTTP** (`404`, `409`), diferente do padrão legado `200 + erro`: ambos funcionam, mas este é mais semântico.
- Dados do servidor entram na tela via `textContent`.
- A operação que altera dados usa `POST`.

### 15.7 Para praticar

1. Troque `fetch` por `$.ajax` mantendo o mesmo comportamento.
2. Adicione um *debounce* de 500 ms para consultar automaticamente enquanto o usuário digita.
3. Implemente o endpoint `listar` e monte a tabela de todas as entregas dinamicamente.
4. Faça o servidor responder `401` quando não houver sessão e trate no cliente.

---

## 16. Armadilhas e boas práticas

| Armadilha | Explicação | Solução |
|---|---|---|
| Usar o resultado fora do callback | Código assíncrono: a resposta ainda não chegou | Colocar a lógica no `success`/`.then`/após `await` |
| Esperar que `fetch` rejeite em `404/500` | Só falhas de rede rejeitam | Checar `response.ok` |
| Só tratar `success`/`complete`, sem `error` | Falha silenciosa: tela fica travada ou vazia | Sempre tratar o erro e esconder o loading |
| `JSON.parse` em resposta que pode ser HTML | Sessão expirada devolve a página de login | Validar sessão/status antes; `try/catch` no parse |
| Alterar dados via `GET` (ex.: `acao=excluirAjax`) | Quebra semântica HTTP; risco de CSRF e de prefetch acidental | Usar `POST`/`DELETE` |
| Chave do JSON = `id` do campo (acoplamento implícito) | Renomear campo no Java ou HTML quebra a tela sem aviso | Documentar o contrato ou mapear explicitamente |
| Acessar objeto aninhado sem checar `null` | `Cannot read properties of null` | Optional chaining `?.` |
| `.html()`/`innerHTML` com dado do servidor | XSS | `.text()`/`textContent` |
| Dispara AJAX a cada tecla | Sobrecarga no servidor | Debounce |
| Cache do navegador servindo resposta antiga | Dados desatualizados (ex.: sessão) | `Cache-Control: no-cache` ou `?v=Date.now()` |
| Charset divergente | Acentuação quebrada | Declarar `charset` no `Content-Type` e manter consistência |
| `async: false` no jQuery | Trava a interface inteira (deprecado) | Sempre assíncrono |
| Validar só no front | Burlável | Validar no BO |
| Misturar `$` com outras libs | `$` pode ser sobrescrito | Usar `jQuery` ou `noConflict` |
| Disparar várias vezes o mesmo botão | Requisições duplicadas | Desabilitar o botão durante a chamada; idempotência ([[32-IDEMPOTENCIA-JAVA]]) |

### Regras de ouro

1. **Todo AJAX tem três destinos: sucesso, erro e fim** (`finally`/`complete`). Trate os três.
2. **O servidor é a autoridade**: valide, autorize e nunca confie no cliente.
3. **Defina o contrato**: URL, `acao`, parâmetros, formato e status de resposta, e mantenha estável.
4. **Dado do servidor é texto, não HTML.**
5. **Operação que altera estado não usa `GET`.**

---

## 17. Cheat sheet

```javascript
// ---------- fetch (moderno) ----------
const r = await fetch(url);                 // GET
if (!r.ok) throw new Error('HTTP ' + r.status);
const json  = await r.json();               // ou r.text(), r.blob()

await fetch(url, {                          // POST form
    method: 'POST',
    body: new URLSearchParams({ acao: 'x', id: 1 })
});

await fetch(url, {                          // POST JSON
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(obj)
});

// ---------- jQuery ----------
$.get(url, params, cb, 'json');
$.post(url, params, cb, 'json');
$.ajax({ url, type, data, dataType, success, error, complete });

// ---------- XMLHttpRequest ----------
const x = new XMLHttpRequest();
x.open('GET', url, true);
x.onload = () => console.log(x.status, x.responseText);
x.send();
```

```java
// ---------- Servlet ----------
resp.setContentType("application/json;charset=UTF-8");
resp.getWriter().print(new Gson().toJson(objeto));

resp.setStatus(HttpServletResponse.SC_NOT_FOUND);       // 404
resp.setHeader("Cache-Control", "no-cache");

String acao = req.getParameter("acao");                  // form / query
String corpo = req.getReader().lines()                   // corpo JSON
                  .collect(Collectors.joining());
```

| Quero... | Use |
|---|---|
| Ler JSON da resposta | `response.json()` / `dataType:'json'` |
| Ler mensagem simples | `response.text()` / `dataType:'text'` |
| Baixar arquivo | `responseType='blob'` / `response.blob()` |
| Rodar algo sempre ao final | `finally` / `complete` |
| Evitar excesso de chamadas | Debounce (`setTimeout` + `clearTimeout`) |
| Evitar cache | `Cache-Control: no-cache` / `?v=Date.now()` |
| Evitar XSS | `textContent` / `.text()` |

### Referências

- MDN: [Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- MDN: [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- jQuery: [`jQuery.ajax()`](https://api.jquery.com/jQuery.ajax/)
- Gson: [User Guide](https://github.com/google/gson/blob/main/UserGuide.md)
- Ver também: [[33-API-JAVA]], [[34-CRUD-JAVA]], [[32-IDEMPOTENCIA-JAVA]], [[39-DEBUGANDO-JAVA]], [[30-THREADS-CONCORRENCIA-JAVA]] (conceito de assincronia no Java)

---

## Conclusão

AJAX é, no fundo, **uma conversa HTTP comum com um cliente que decide o que fazer com a resposta**. Para o desenvolvedor Java, o Servlet continua sendo um Servlet: muda o contrato (JSON/texto em vez de JSP) e a responsabilidade de tratar os três destinos de toda chamada: **sucesso, erro e fim**.

O código legado de sistemas Java web mostra a evolução completa: `XMLHttpRequest` → `jQuery.ajax/$.post` → `fetch` + `async/await` → componentes carregados sob demanda. Saber ler os três estilos é o que permite manter um sistema atual e escrever o novo com segurança.

**Regra de ouro:** se a tela não reagiu como esperado, abra a aba **Network**: ela diz, em segundos, se o problema está no cliente, no servidor ou no contrato entre eles.
