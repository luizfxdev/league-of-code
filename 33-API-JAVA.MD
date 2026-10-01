# APIs em Java


---

## Sumário

1. [Modelo de domínio usado nos exemplos](#1-modelo-de-domínio-usado-nos-exemplos)
2. [O que é uma API](#2-o-que-é-uma-api)
3. [API em camadas: Controller, Service, Repository](#3-api-em-camadas-controller-service-repository)
4. [Anatomia de uma requisição HTTP](#4-anatomia-de-uma-requisição-http)
5. [Verbos HTTP e seu significado](#5-verbos-http-e-seu-significado)
6. [Status codes: o que cada faixa comunica](#6-status-codes-o-que-cada-faixa-comunica)
7. [Servidor HTTP mínimo em Java puro (`com.sun.net.httpserver`)](#7-servidor-http-mínimo-em-java-puro-comsunnethttpserver)
8. [Serialização JSON (requisição e resposta)](#8-serialização-json-requisição-e-resposta)
9. [Validação de entrada](#9-validação-de-entrada)
10. [Tratamento de erros e corpo de erro padronizado](#10-tratamento-de-erros-e-corpo-de-erro-padronizado)
11. [Paginação, filtros e ordenação](#11-paginação-filtros-e-ordenação)
12. [Versionamento de API](#12-versionamento-de-api)
13. [Autenticação e autorização (visão geral)](#13-autenticação-e-autorização-visão-geral)
14. [Idempotência em APIs (ligação com tópico anterior)](#14-idempotência-em-apis-ligação-com-tópico-anterior)
15. [Exercício resolvido: API de rastreamento de entregas](#15-exercício-resolvido-api-de-rastreamento-de-entregas)
16. [Armadilhas e boas práticas](#16-armadilhas-e-boas-práticas)
17. [Cheat sheet](#17-cheat-sheet)

---

## 1. Modelo de domínio usado nos exemplos

```java
public class Entrega {
    private String codigo;
    private String destino;
    private double pesoKg;
    private double frete;
    private StatusEntrega status;

    public Entrega(String codigo, String destino, double pesoKg, double frete, StatusEntrega status) {
        this.codigo = codigo;
        this.destino = destino;
        this.pesoKg = pesoKg;
        this.frete = frete;
        this.status = status;
    }

    public String getCodigo() { return codigo; }
    public String getDestino() { return destino; }
    public double getPesoKg() { return pesoKg; }
    public double getFrete() { return frete; }
    public StatusEntrega getStatus() { return status; }
    public void setStatus(StatusEntrega status) { this.status = status; }
    public void setDestino(String destino) { this.destino = destino; }
    public void setFrete(double frete) { this.frete = frete; }

    @Override
    public String toString() {
        return codigo + " | " + destino + " | " + pesoKg + "kg | R$ " + String.format("%.2f", frete) + " | " + status;
    }
}

public enum StatusEntrega { PENDENTE, EM_ROTA, ENTREGUE, CANCELADA }
```

**Cenário:** uma transportadora expõe uma **API de rastreamento de entregas**, usada pelo app do motorista, pelo painel do cliente e por sistemas parceiros (e-commerce) que consultam status de pedidos.

---

## 2. O que é uma API

**API (Application Programming Interface)** é o **contrato** pelo qual um software expõe funcionalidades para outros softwares consumirem, sem que o consumidor precise conhecer a implementação interna.

**Analogia logística:** a API é o **balcão de atendimento** da transportadora. O cliente não entra no galpão para procurar o pacote sozinho — ele vai ao balcão, pede "status da entrega E001", e recebe uma resposta padronizada. O balcão (API) é estável mesmo que o galpão (implementação interna) mude de lugar as prateleiras.

| Tipo de API | Exemplo |
|---|---|
| **Web API (REST/HTTP)** | Endpoint `GET /entregas/E001` retornando JSON |
| **Biblioteca/SDK** | Uma classe pública com métodos (`java.util.List`, por exemplo, é a "API" da lista) |
| **RPC/gRPC** | Chamada de método remoto com contrato binário (protobuf) |
| **Mensageria** | API é o formato do evento publicado/consumido em uma fila |

Este tópico foca em **Web APIs REST sobre HTTP**, o tipo mais comum em sistemas de backend.

---

## 3. API em camadas: Controller, Service, Repository

Uma API bem estruturada separa responsabilidades em camadas, cada uma com um motivo próprio para mudar:

```
Requisição HTTP
     |
     v
[ Controller ]   -> traduz HTTP <-> objetos Java, não contém regra de negócio
     |
     v
[ Service ]      -> regra de negócio, validações, orquestração
     |
     v
[ Repository ]   -> acesso a dados (banco, memória, arquivo)
```

```java
// Camada de dados
public interface EntregaRepository {
    Optional<Entrega> buscarPorCodigo(String codigo);
    List<Entrega> listarTodas();
    Entrega salvar(Entrega entrega);
    void remover(String codigo);
}

// Camada de negócio
public class EntregaService {
    private final EntregaRepository repository;

    public EntregaService(EntregaRepository repository) {
        this.repository = repository;
    }

    public Entrega buscarOuFalhar(String codigo) {
        return repository.buscarPorCodigo(codigo)
            .orElseThrow(() -> new EntregaNaoEncontradaException(codigo));
    }

    public Entrega confirmarEntrega(String codigo) {
        Entrega entrega = buscarOuFalhar(codigo);
        if (entrega.getStatus() == StatusEntrega.CANCELADA) {
            throw new RegraNegocioException("Entrega cancelada não pode ser confirmada.");
        }
        entrega.setStatus(StatusEntrega.ENTREGUE);
        return repository.salvar(entrega);
    }
}

// Camada de apresentação (HTTP) — ver seção 7 para a implementação completa do handler
public class EntregaController {
    private final EntregaService service;

    public EntregaController(EntregaService service) {
        this.service = service;
    }

    public Entrega get(String codigo) {
        return service.buscarOuFalhar(codigo);
    }
}
```

> O **Controller não deve conter regra de negócio**. Isso mantém a regra testável sem precisar simular uma requisição HTTP inteira.

---

## 4. Anatomia de uma requisição HTTP

```
GET /entregas/E001?incluirHistorico=true HTTP/1.1
Host: api.transportadora.com
Authorization: Bearer <token>
Accept: application/json

(corpo vazio em GET)
```

```
POST /entregas HTTP/1.1
Content-Type: application/json

{
  "destino": "Lorena",
  "pesoKg": 15.5
}
```

| Parte | Onde fica | Exemplo |
|---|---|---|
| Método | Linha inicial | `GET`, `POST`, `PUT`, `DELETE` |
| Path | URL | `/entregas/E001` |
| Query string | URL, após `?` | `?incluirHistorico=true` |
| Headers | Linhas após a inicial | `Authorization`, `Content-Type` |
| Corpo (body) | Após linha em branco | JSON com os dados |

---

## 5. Verbos HTTP e seu significado

| Verbo | Uso típico | Idempotente? | Tem corpo de resposta? |
|---|---|---|---|
| `GET` | Buscar recurso(s) | Sim | Sim |
| `POST` | Criar recurso, ou ação que não mapeia para CRUD simples | Não (por padrão) | Geralmente sim |
| `PUT` | Substituir o recurso inteiro | Sim | Opcional |
| `PATCH` | Atualizar parcialmente | Depende | Opcional |
| `DELETE` | Remover recurso | Sim | Geralmente não |

```
GET    /entregas           -> lista todas as entregas
GET    /entregas/E001      -> busca uma entrega específica
POST   /entregas           -> cria uma nova entrega
PUT    /entregas/E001      -> substitui a entrega E001 por completo
PATCH  /entregas/E001      -> atualiza campos específicos (ex.: só o status)
DELETE /entregas/E001      -> remove a entrega E001
```

> Idempotência de cada verbo é discutida em profundidade em [[32-IDEMPOTENCIA-JAVA]] — aqui o foco é o papel de cada verbo no desenho da API.

---

## 6. Status codes: o que cada faixa comunica

| Faixa | Significado | Exemplos usados na API de entregas |
|---|---|---|
| `2xx` | Sucesso | `200 OK` (consulta), `201 Created` (nova entrega), `204 No Content` (remoção) |
| `3xx` | Redirecionamento | `301 Moved Permanently` (endpoint antigo movido) |
| `4xx` | Erro do cliente | `400 Bad Request` (payload inválido), `404 Not Found` (entrega não existe), `409 Conflict` (status inconsistente) |
| `5xx` | Erro do servidor | `500 Internal Server Error` (falha inesperada) |

```java
public enum HttpStatusApi {
    OK(200), CREATED(201), NO_CONTENT(204),
    BAD_REQUEST(400), NOT_FOUND(404), CONFLICT(409),
    INTERNAL_SERVER_ERROR(500);

    public final int codigo;
    HttpStatusApi(int codigo) { this.codigo = codigo; }
}
```

> Nunca devolva `200 OK` com um corpo `{ "erro": "não encontrado" }` — o status HTTP é parte do contrato; clientes (e bibliotecas HTTP) decidem comportamento (retry, log, alerta) com base nele.

---

## 7. Servidor HTTP mínimo em Java puro (`com.sun.net.httpserver`)

O JDK inclui um servidor HTTP embutido, suficiente para exemplos didáticos e serviços simples, sem depender de frameworks externos (Spring, Micronaut etc.).

```java
import com.sun.net.httpserver.HttpServer;
import com.sun.net.httpserver.HttpExchange;
import java.net.InetSocketAddress;
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class ApiServidor {
    private static final Map<String, Entrega> banco = new ConcurrentHashMap<>();

    public static void main(String[] args) throws IOException {
        banco.put("E001", new Entrega("E001", "Lorena", 15.0, 80.0, StatusEntrega.EM_ROTA));

        HttpServer servidor = HttpServer.create(new InetSocketAddress(8080), 0);

        servidor.createContext("/entregas", ApiServidor::tratarEntregas);
        servidor.setExecutor(java.util.concurrent.Executors.newFixedThreadPool(4));
        servidor.start();

        System.out.println("API rodando em http://localhost:8080/entregas");
    }

    private static void tratarEntregas(HttpExchange exchange) throws IOException {
        String metodo = exchange.getRequestMethod();
        String path = exchange.getRequestURI().getPath();   // ex.: /entregas/E001
        String codigo = extrairCodigo(path);

        switch (metodo) {
            case "GET" -> {
                if (codigo == null) {
                    responder(exchange, 200, toJsonLista(banco.values()));
                } else if (banco.containsKey(codigo)) {
                    responder(exchange, 200, toJson(banco.get(codigo)));
                } else {
                    responder(exchange, 404, "{\"erro\":\"Entrega " + codigo + " não encontrada\"}");
                }
            }
            case "POST" -> {
                String corpo = lerCorpo(exchange);
                Entrega nova = fromJson(corpo);
                banco.put(nova.getCodigo(), nova);
                responder(exchange, 201, toJson(nova));
            }
            case "DELETE" -> {
                banco.remove(codigo);
                responder(exchange, 204, "");
            }
            default -> responder(exchange, 405, "{\"erro\":\"Método não suportado\"}");
        }
    }

    private static String extrairCodigo(String path) {
        String[] partes = path.split("/");
        return partes.length > 2 ? partes[2] : null;
    }

    private static String lerCorpo(HttpExchange exchange) throws IOException {
        try (InputStream is = exchange.getRequestBody()) {
            return new String(is.readAllBytes(), StandardCharsets.UTF_8);
        }
    }

    private static void responder(HttpExchange exchange, int status, String corpoJson) throws IOException {
        byte[] bytes = corpoJson.getBytes(StandardCharsets.UTF_8);
        exchange.getResponseHeaders().add("Content-Type", "application/json; charset=utf-8");
        exchange.sendResponseHeaders(status, bytes.length == 0 ? -1 : bytes.length);
        if (bytes.length > 0) {
            try (OutputStream os = exchange.getResponseBody()) {
                os.write(bytes);
            }
        }
    }

    // toJson/fromJson/toJsonLista: ver seção 8
    private static String toJson(Entrega e) { return "{}"; }
    private static String toJsonLista(Collection<Entrega> lista) { return "[]"; }
    private static Entrega fromJson(String json) { return null; }
}
```

> Em produção, frameworks como **Spring Boot** ou **Micronaut** cuidam de roteamento, serialização e validação automaticamente — este exemplo expõe o que esses frameworks fazem "por baixo", o que ajuda a entender erros e comportamento real do protocolo.

---

## 8. Serialização JSON (requisição e resposta)

JSON é o formato de corpo mais comum em APIs REST. Pode-se serializar manualmente (didático) ou com uma biblioteca (`Jackson`, `Gson`) em projetos reais.

### Serialização manual (ilustrativa)

```java
public static String toJson(Entrega e) {
    return String.format(
        "{\"codigo\":\"%s\",\"destino\":\"%s\",\"pesoKg\":%.2f,\"frete\":%.2f,\"status\":\"%s\"}",
        e.getCodigo(), e.getDestino(), e.getPesoKg(), e.getFrete(), e.getStatus()
    );
}
```

### Com Jackson (abordagem real em produção)

```java
import com.fasterxml.jackson.databind.ObjectMapper;

ObjectMapper mapper = new ObjectMapper();

// Objeto -> JSON
String json = mapper.writeValueAsString(entrega);

// JSON -> Objeto
Entrega entrega = mapper.readValue(jsonRecebido, Entrega.class);

// JSON -> Lista de objetos
List<Entrega> lista = mapper.readValue(jsonLista, new TypeReference<List<Entrega>>() {});
```

### DTO (Data Transfer Object) em vez da entidade direta

```java
// Evita expor campos internos (ex.: custoOperacionalInterno) e evita acoplar o contrato da API à entidade de domínio
public record EntregaResponseDTO(String codigo, String destino, double pesoKg, double frete, String status) {
    public static EntregaResponseDTO deEntrega(Entrega e) {
        return new EntregaResponseDTO(e.getCodigo(), e.getDestino(), e.getPesoKg(), e.getFrete(), e.getStatus().name());
    }
}

public record NovaEntregaRequestDTO(String destino, double pesoKg) { }
```

> Usar `record` (Java 17) para DTOs é idiomático: imutável, `equals`/`hashCode`/`toString` gerados automaticamente, e sintaxe concisa.

---

## 9. Validação de entrada

```java
public record NovaEntregaRequestDTO(String destino, double pesoKg) {

    public NovaEntregaRequestDTO {
        if (destino == null || destino.isBlank()) {
            throw new ValidacaoException("destino é obrigatório");
        }
        if (pesoKg <= 0) {
            throw new ValidacaoException("pesoKg deve ser maior que zero");
        }
    }
}
```

> Validar dentro do **construtor compacto do record** garante que **nenhuma instância inválida consiga existir** — a validação acontece no único ponto de criação do objeto.

```java
public class ValidacaoException extends RuntimeException {
    public ValidacaoException(String mensagem) {
        super(mensagem);
    }
}
```

```java
// Na camada de controller, converter a exceção de validação em 400 Bad Request
try {
    NovaEntregaRequestDTO dto = mapper.readValue(corpo, NovaEntregaRequestDTO.class);
    Entrega criada = service.criar(dto);
    responder(exchange, 201, toJson(criada));
} catch (ValidacaoException e) {
    responder(exchange, 400, "{\"erro\":\"" + e.getMessage() + "\"}");
}
```

---

## 10. Tratamento de erros e corpo de erro padronizado

Nunca deixe uma exceção vazar como stack trace para o cliente. Padronize o formato de erro:

```java
public record ErroResponseDTO(String codigo, String mensagem, long timestamp) {
    public static ErroResponseDTO de(String codigo, String mensagem) {
        return new ErroResponseDTO(codigo, mensagem, System.currentTimeMillis());
    }
}
```

```java
public class EntregaNaoEncontradaException extends RuntimeException {
    public EntregaNaoEncontradaException(String codigo) {
        super("Entrega " + codigo + " não encontrada");
    }
}

public class RegraNegocioException extends RuntimeException {
    public RegraNegocioException(String mensagem) {
        super(mensagem);
    }
}
```

```java
// Mapeamento centralizado de exceção -> status HTTP (no controller/handler)
private static void tratarErro(HttpExchange exchange, Exception e) throws IOException {
    int status;
    String codigoErro;

    if (e instanceof EntregaNaoEncontradaException) {
        status = 404; codigoErro = "ENTREGA_NAO_ENCONTRADA";
    } else if (e instanceof ValidacaoException) {
        status = 400; codigoErro = "VALIDACAO_FALHOU";
    } else if (e instanceof RegraNegocioException) {
        status = 409; codigoErro = "REGRA_NEGOCIO_VIOLADA";
    } else {
        status = 500; codigoErro = "ERRO_INTERNO";
    }

    ErroResponseDTO erro = ErroResponseDTO.de(codigoErro, e.getMessage());
    responder(exchange, status, mapper.writeValueAsString(erro));
}
```

> Centralizar esse mapeamento (em frameworks reais: `@ExceptionHandler`/`@ControllerAdvice` no Spring) evita duplicar `try/catch` em cada endpoint.

---

## 11. Paginação, filtros e ordenação

```
GET /entregas?pagina=2&tamanho=20&status=EM_ROTA&ordenarPor=pesoKg&direcao=desc
```

```java
public record FiltroEntregaDTO(int pagina, int tamanho, StatusEntrega status, String ordenarPor, boolean decrescente) { }

public record PaginaResponseDTO<T>(List<T> itens, int paginaAtual, int totalPaginas, long totalItens) { }
```

```java
public PaginaResponseDTO<EntregaResponseDTO> listar(FiltroEntregaDTO filtro) {
    List<Entrega> todas = repository.listarTodas();

    List<Entrega> filtradas = todas.stream()
        .filter(e -> filtro.status() == null || e.getStatus() == filtro.status())
        .sorted(construirComparador(filtro))
        .toList();

    int inicio = filtro.pagina() * filtro.tamanho();
    int fim = Math.min(inicio + filtro.tamanho(), filtradas.size());
    List<Entrega> pagina = inicio < filtradas.size() ? filtradas.subList(inicio, fim) : List.of();

    List<EntregaResponseDTO> dtos = pagina.stream().map(EntregaResponseDTO::deEntrega).toList();
    int totalPaginas = (int) Math.ceil((double) filtradas.size() / filtro.tamanho());

    return new PaginaResponseDTO<>(dtos, filtro.pagina(), totalPaginas, filtradas.size());
}

private Comparator<Entrega> construirComparador(FiltroEntregaDTO filtro) {
    Comparator<Entrega> comparador = switch (filtro.ordenarPor()) {
        case "pesoKg" -> Comparator.comparingDouble(Entrega::getPesoKg);
        case "frete" -> Comparator.comparingDouble(Entrega::getFrete);
        default -> Comparator.comparing(Entrega::getCodigo);
    };
    return filtro.decrescente() ? comparador.reversed() : comparador;
}
```

> Nunca devolva **todas** as linhas de uma tabela grande sem paginação — isso é uma das causas mais comuns de degradação de performance em APIs que crescem com o tempo.

---

## 12. Versionamento de API

```
/v1/entregas/E001      <- versionamento na URL (mais comum e explícito)
Accept: application/vnd.transportadora.v2+json   <- versionamento por header (menos invasivo na URL)
```

```java
servidor.createContext("/v1/entregas", ApiServidor::tratarEntregasV1);
servidor.createContext("/v2/entregas", ApiServidor::tratarEntregasV2);
```

| Estratégia | Vantagem | Desvantagem |
|---|---|---|
| Versão na URL (`/v1/...`) | Simples, visível, fácil de testar no navegador | "Poluui" a URL, pode levar a duplicação de rotas |
| Versão no header | URL permanece limpa | Menos óbvio, exige documentação clara |

> Regra prática: **nunca** faça uma mudança que quebra contrato (remover campo, mudar tipo, mudar semântica) em uma versão existente — crie uma nova versão.

---

## 13. Autenticação e autorização (visão geral)

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

```java
// Middleware/filtro simplificado de autenticação
private static boolean autenticado(HttpExchange exchange) {
    String header = exchange.getRequestHeaders().getFirst("Authorization");
    if (header == null || !header.startsWith("Bearer ")) {
        return false;
    }
    String token = header.substring("Bearer ".length());
    return validarToken(token);   // ex.: validar assinatura JWT
}
```

```java
private static void tratarEntregas(HttpExchange exchange) throws IOException {
    if (!autenticado(exchange)) {
        responder(exchange, 401, "{\"erro\":\"Não autenticado\"}");
        return;
    }
    if (!autorizado(exchange, "ENTREGAS_LEITURA")) {
        responder(exchange, 403, "{\"erro\":\"Sem permissão\"}");
        return;
    }
    // ... segue o tratamento normal
}
```

| Código | Significado |
|---|---|
| `401 Unauthorized` | Cliente **não está autenticado** (token ausente/inválido) |
| `403 Forbidden` | Cliente **está autenticado**, mas não tem permissão para a ação |

> Autenticação (quem é você) e autorização (o que você pode fazer) são conceitos distintos — confundi-los leva a `403` onde deveria haver `401`, dificultando o diagnóstico pelo lado do cliente.

---

## 14. Idempotência em APIs (ligação com tópico anterior)

```java
// POST tornado idempotente via chave enviada pelo cliente no header
private static void criarEntrega(HttpExchange exchange) throws IOException {
    String chaveIdempotencia = exchange.getRequestHeaders().getFirst("Idempotency-Key");

    if (chaveIdempotencia != null && chavesProcessadas.contains(chaveIdempotencia)) {
        Entrega jaProcessada = resultadosPorChave.get(chaveIdempotencia);
        responder(exchange, 200, toJson(jaProcessada));   // devolve o mesmo resultado, não cria de novo
        return;
    }

    Entrega nova = criarNaBase(lerCorpo(exchange));
    if (chaveIdempotencia != null) {
        chavesProcessadas.add(chaveIdempotencia);
        resultadosPorChave.put(chaveIdempotencia, nova);
    }
    responder(exchange, 201, toJson(nova));
}
```

> Detalhamento completo do padrão de chave de idempotência, atomicidade sob concorrência e persistência distribuída está em [[32-IDEMPOTENCIA-JAVA]].

---

## 15. Exercício resolvido: API de rastreamento de entregas

**Problema:** implementar os endpoints `GET /entregas`, `GET /entregas/{codigo}` e `PATCH /entregas/{codigo}/status`, com validação, tratamento de erro padronizado e DTOs.

```java
import com.sun.net.httpserver.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.net.InetSocketAddress;
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.Executors;

public class RastreamentoApi {
    private static final Map<String, Entrega> banco = new ConcurrentHashMap<>();
    private static final ObjectMapper mapper = new ObjectMapper();

    public static void main(String[] args) throws IOException {
        banco.put("E001", new Entrega("E001", "Lorena", 15.0, 80.0, StatusEntrega.EM_ROTA));
        banco.put("E002", new Entrega("E002", "Taubaté", 120.0, 450.0, StatusEntrega.PENDENTE));

        HttpServer servidor = HttpServer.create(new InetSocketAddress(8080), 0);
        servidor.createContext("/entregas", RastreamentoApi::rotear);
        servidor.setExecutor(Executors.newFixedThreadPool(4));
        servidor.start();
        System.out.println("API no ar em http://localhost:8080/entregas");
    }

    private static void rotear(HttpExchange exchange) {
        try {
            String[] partes = exchange.getRequestURI().getPath().split("/");
            String metodo = exchange.getRequestMethod();

            if (partes.length == 2 && metodo.equals("GET")) {
                listar(exchange);
            } else if (partes.length == 3 && metodo.equals("GET")) {
                buscar(exchange, partes[2]);
            } else if (partes.length == 4 && partes[3].equals("status") && metodo.equals("PATCH")) {
                atualizarStatus(exchange, partes[2]);
            } else {
                responder(exchange, 404, Map.of("erro", "Rota não encontrada"));
            }
        } catch (ValidacaoException e) {
            responderSilenciosamente(exchange, 400, Map.of("erro", e.getMessage()));
        } catch (EntregaNaoEncontradaException e) {
            responderSilenciosamente(exchange, 404, Map.of("erro", e.getMessage()));
        } catch (Exception e) {
            responderSilenciosamente(exchange, 500, Map.of("erro", "Erro interno"));
        }
    }

    private static void listar(HttpExchange exchange) throws IOException {
        List<Map<String, Object>> resposta = banco.values().stream()
            .map(RastreamentoApi::paraDto)
            .toList();
        responder(exchange, 200, resposta);
    }

    private static void buscar(HttpExchange exchange, String codigo) throws IOException {
        Entrega entrega = banco.get(codigo);
        if (entrega == null) {
            throw new EntregaNaoEncontradaException(codigo);
        }
        responder(exchange, 200, paraDto(entrega));
    }

    private static void atualizarStatus(HttpExchange exchange, String codigo) throws IOException {
        Entrega entrega = banco.get(codigo);
        if (entrega == null) {
            throw new EntregaNaoEncontradaException(codigo);
        }

        String corpo = new String(exchange.getRequestBody().readAllBytes(), StandardCharsets.UTF_8);
        Map<String, String> payload = mapper.readValue(corpo, Map.class);
        String novoStatus = payload.get("status");

        if (novoStatus == null || novoStatus.isBlank()) {
            throw new ValidacaoException("Campo 'status' é obrigatório");
        }

        try {
            entrega.setStatus(StatusEntrega.valueOf(novoStatus));
        } catch (IllegalArgumentException e) {
            throw new ValidacaoException("Status inválido: " + novoStatus);
        }

        responder(exchange, 200, paraDto(entrega));
    }

    private static Map<String, Object> paraDto(Entrega e) {
        return Map.of(
            "codigo", e.getCodigo(),
            "destino", e.getDestino(),
            "pesoKg", e.getPesoKg(),
            "frete", e.getFrete(),
            "status", e.getStatus().name()
        );
    }

    private static void responder(HttpExchange exchange, int status, Object corpo) throws IOException {
        byte[] bytes = mapper.writeValueAsBytes(corpo);
        exchange.getResponseHeaders().add("Content-Type", "application/json; charset=utf-8");
        exchange.sendResponseHeaders(status, bytes.length);
        try (OutputStream os = exchange.getResponseBody()) {
            os.write(bytes);
        }
    }

    private static void responderSilenciosamente(HttpExchange exchange, int status, Object corpo) {
        try { responder(exchange, status, corpo); } catch (IOException ignored) { }
    }
}
```

**Testando com `curl`:**

```
curl http://localhost:8080/entregas
curl http://localhost:8080/entregas/E001
curl -X PATCH http://localhost:8080/entregas/E001/status -d "{\"status\":\"ENTREGUE\"}"
curl http://localhost:8080/entregas/E999   -> 404 {"erro":"Entrega E999 não encontrada"}
```

---

## 16. Armadilhas e boas práticas

| Armadilha | Explicação | Solução |
|---|---|---|
| Expor a entidade de domínio direto como resposta | Acopla contrato da API à estrutura interna; mudanças internas quebram clientes | Usar DTOs (`record`) específicos para entrada/saída |
| Retornar `200 OK` para erros | Cliente não consegue distinguir sucesso de falha de forma programática | Usar o status HTTP correto (`4xx`/`5xx`) |
| Vazar stack trace para o cliente | Expõe detalhes internos (possível risco de segurança) e é inútil para o consumidor | Corpo de erro padronizado, log interno separado |
| Não paginar listas grandes | Degrada performance conforme os dados crescem | Paginação desde o primeiro endpoint de listagem |
| Misturar regra de negócio no controller | Dificulta testar a regra isoladamente | Mover regra para a camada de serviço |
| Quebrar contrato em uma versão já publicada | Clientes existentes param de funcionar sem aviso | Versionar a API antes de mudanças incompatíveis |
| Confundir `401` com `403` | Dificulta ao cliente diagnosticar o problema real | `401` = não autenticado, `403` = autenticado mas sem permissão |
| `POST` sem suporte a idempotência em operações críticas (pagamento, criação) | Retry duplica efeito | Aceitar `Idempotency-Key` conforme [[32-IDEMPOTENCIA-JAVA]] |

### Regras de ouro

1. **O controller traduz HTTP; o service decide regra de negócio.**
2. **Status HTTP é parte do contrato** — use a faixa certa, sempre.
3. **DTOs de entrada e saída, nunca a entidade crua.**
4. **Erros em formato padronizado**, nunca stack trace exposta.
5. **Pagine, filtre e documente** qualquer endpoint que devolva coleções.

---

## 17. Cheat sheet

```java
// Servidor HTTP mínimo (JDK puro)
HttpServer servidor = HttpServer.create(new InetSocketAddress(8080), 0);
servidor.createContext("/recurso", handler);
servidor.setExecutor(Executors.newFixedThreadPool(4));
servidor.start();

// DTO com validação (Java 17 record)
public record MeuDTO(String campo) {
    public MeuDTO {
        if (campo == null || campo.isBlank()) throw new ValidacaoException("campo obrigatório");
    }
}

// Mapeamento de exceção -> status HTTP
EntregaNaoEncontradaException -> 404
ValidacaoException            -> 400
RegraNegocioException         -> 409
(qualquer outra)              -> 500
```

| Verbo | Endpoint | Ação |
|---|---|---|
| `GET` | `/entregas` | listar |
| `GET` | `/entregas/{codigo}` | buscar uma |
| `POST` | `/entregas` | criar |
| `PUT` | `/entregas/{codigo}` | substituir |
| `PATCH` | `/entregas/{codigo}` | atualizar parcialmente |
| `DELETE` | `/entregas/{codigo}` | remover |

### Referências

- Docs Java: [`com.sun.net.httpserver.HttpServer`](https://docs.oracle.com/en/java/javase/17/docs/api/jdk.httpserver/com/sun/net/httpserver/HttpServer.html)
- RFC 9110 (HTTP Semantics) — métodos, status codes
- Ver também: [[32-IDEMPOTENCIA-JAVA]], [[34-CRUD-JAVA]]
