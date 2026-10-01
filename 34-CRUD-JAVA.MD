# CRUD em Java


---

## Sumário

1. [Modelo de domínio usado nos exemplos](#1-modelo-de-domínio-usado-nos-exemplos)
2. [O que é CRUD](#2-o-que-é-crud)
3. [CRUD em memória: a base de tudo](#3-crud-em-memória-a-base-de-tudo)
4. [Interface `Repository`: contrato independente de implementação](#4-interface-repository-contrato-independente-de-implementação)
5. [Create: criando registros corretamente](#5-create-criando-registros-corretamente)
6. [Read: buscar um, buscar todos, buscar com filtro](#6-read-buscar-um-buscar-todos-buscar-com-filtro)
7. [Update: substituição total vs parcial](#7-update-substituição-total-vs-parcial)
8. [Delete: remoção física vs lógica (soft delete)](#8-delete-remoção-física-vs-lógica-soft-delete)
9. [CRUD com JDBC puro](#9-crud-com-jdbc-puro)
10. [CRUD com transações](#10-crud-com-transações)
11. [Camada de serviço sobre o CRUD](#11-camada-de-serviço-sobre-o-crud)
12. [CRUD exposto como API REST](#12-crud-exposto-como-api-rest)
13. [Concorrência em operações CRUD (optimistic locking)](#13-concorrência-em-operações-crud-optimistic-locking)
14. [Idempotência nas operações CRUD](#14-idempotência-nas-operações-crud)
15. [Exercício resolvido](#15-exercício-resolvido)
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
    private boolean removida;
    private long versao;   // usado na seção 13 (optimistic locking)

    public Entrega(String codigo, String destino, double pesoKg, double frete, StatusEntrega status) {
        this.codigo = codigo;
        this.destino = destino;
        this.pesoKg = pesoKg;
        this.frete = frete;
        this.status = status;
        this.removida = false;
        this.versao = 0;
    }

    public String getCodigo() { return codigo; }
    public String getDestino() { return destino; }
    public double getPesoKg() { return pesoKg; }
    public double getFrete() { return frete; }
    public StatusEntrega getStatus() { return status; }
    public boolean isRemovida() { return removida; }
    public long getVersao() { return versao; }

    public void setDestino(String destino) { this.destino = destino; }
    public void setPesoKg(double pesoKg) { this.pesoKg = pesoKg; }
    public void setFrete(double frete) { this.frete = frete; }
    public void setStatus(StatusEntrega status) { this.status = status; }
    public void marcarComoRemovida() { this.removida = true; }
    public void incrementarVersao() { this.versao++; }

    @Override
    public String toString() {
        return codigo + " | " + destino + " | " + pesoKg + "kg | R$ " + String.format("%.2f", frete)
             + " | " + status + (removida ? " [REMOVIDA]" : "");
    }
}

public enum StatusEntrega { PENDENTE, EM_ROTA, ENTREGUE, CANCELADA }
```

**Cenário:** o sistema de gestão da transportadora precisa **cadastrar, consultar, atualizar e remover** entregas — a base de qualquer tela administrativa ou API de backoffice.

---

## 2. O que é CRUD

**CRUD** é o acrônimo para as quatro operações fundamentais de persistência de dados:

| Letra | Operação | Verbo HTTP equivalente | SQL equivalente |
|---|---|---|---|
| **C**reate | Criar um novo registro | `POST` | `INSERT` |
| **R**ead | Ler (um ou vários registros) | `GET` | `SELECT` |
| **U**pdate | Atualizar um registro existente | `PUT`/`PATCH` | `UPDATE` |
| **D**elete | Remover um registro | `DELETE` | `DELETE` |

**Analogia logística:** é exatamente o ciclo de vida de uma ficha de entrega no sistema do galpão: **cadastrar** o pedido (Create), **consultar** onde ele está (Read), **corrigir** o destino ou peso informado errado (Update), e **arquivar/excluir** a ficha quando cancelada definitivamente (Delete).

Praticamente toda tela administrativa, painel ou API de gestão é, no fundo, CRUD com regras de negócio em volta.

---

## 3. CRUD em memória: a base de tudo

Antes de qualquer banco de dados, vale implementar o CRUD em memória — a lógica de negócio é a mesma, só a camada de armazenamento muda depois.

```java
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class EntregaRepositoryEmMemoria {
    private final Map<String, Entrega> dados = new ConcurrentHashMap<>();

    // CREATE
    public Entrega salvar(Entrega entrega) {
        dados.put(entrega.getCodigo(), entrega);
        return entrega;
    }

    // READ (um)
    public Optional<Entrega> buscarPorCodigo(String codigo) {
        return Optional.ofNullable(dados.get(codigo));
    }

    // READ (todos)
    public List<Entrega> listarTodas() {
        return new ArrayList<>(dados.values());
    }

    // UPDATE (via salvar: upsert idempotente, ver [[32-IDEMPOTENCIA-JAVA]])
    public Entrega atualizar(Entrega entrega) {
        if (!dados.containsKey(entrega.getCodigo())) {
            throw new EntregaNaoEncontradaException(entrega.getCodigo());
        }
        dados.put(entrega.getCodigo(), entrega);
        return entrega;
    }

    // DELETE
    public void remover(String codigo) {
        dados.remove(codigo);
    }
}
```

---

## 4. Interface `Repository`: contrato independente de implementação

Separar o **contrato** (interface) da **implementação** (memória, JDBC, arquivo) permite trocar a origem dos dados sem alterar a camada de serviço.

```java
public interface EntregaRepository {
    Entrega salvar(Entrega entrega);
    Optional<Entrega> buscarPorCodigo(String codigo);
    List<Entrega> listarTodas();
    Entrega atualizar(Entrega entrega);
    void remover(String codigo);
}
```

```java
public class EntregaRepositoryEmMemoria implements EntregaRepository { /* seção 3 */ }

public class EntregaRepositoryJdbc implements EntregaRepository { /* seção 9 */ }
```

```java
// O serviço depende da interface, não da implementação concreta
public class EntregaService {
    private final EntregaRepository repository;

    public EntregaService(EntregaRepository repository) {   // injeção de dependência
        this.repository = repository;
    }
}

// Em testes: usa-se a implementação em memória, rápida e sem infraestrutura
EntregaService servicoParaTeste = new EntregaService(new EntregaRepositoryEmMemoria());

// Em produção: usa-se a implementação real
EntregaService servicoReal = new EntregaService(new EntregaRepositoryJdbc(dataSource));
```

---

## 5. Create: criando registros corretamente

```java
public class EntregaService {
    private final EntregaRepository repository;

    public EntregaService(EntregaRepository repository) {
        this.repository = repository;
    }

    public Entrega criar(String destino, double pesoKg) {
        validarCriacao(destino, pesoKg);

        String codigo = gerarCodigo();
        if (repository.buscarPorCodigo(codigo).isPresent()) {
            throw new RegraNegocioException("Código já existe: " + codigo);
        }

        double freteCalculado = pesoKg * 2.5;
        Entrega entrega = new Entrega(codigo, destino, pesoKg, freteCalculado, StatusEntrega.PENDENTE);
        return repository.salvar(entrega);
    }

    private void validarCriacao(String destino, double pesoKg) {
        if (destino == null || destino.isBlank()) {
            throw new ValidacaoException("destino é obrigatório");
        }
        if (pesoKg <= 0) {
            throw new ValidacaoException("pesoKg deve ser maior que zero");
        }
    }

    private String gerarCodigo() {
        return "E" + UUID.randomUUID().toString().substring(0, 8).toUpperCase();
    }
}
```

> **Create não deve confiar em dados calculados pelo cliente** (ex.: o cliente não deveria enviar o `frete` — ele é calculado no servidor a partir do peso, evitando manipulação indevida).

---

## 6. Read: buscar um, buscar todos, buscar com filtro

```java
public class EntregaService {
    // ...

    public Entrega buscarOuFalhar(String codigo) {
        return repository.buscarPorCodigo(codigo)
            .filter(e -> !e.isRemovida())
            .orElseThrow(() -> new EntregaNaoEncontradaException(codigo));
    }

    public List<Entrega> listarTodas() {
        return repository.listarTodas().stream()
            .filter(e -> !e.isRemovida())
            .toList();
    }

    public List<Entrega> listarPorStatus(StatusEntrega status) {
        return repository.listarTodas().stream()
            .filter(e -> !e.isRemovida())
            .filter(e -> e.getStatus() == status)
            .toList();
    }

    public List<Entrega> listarComFiltroEOrdenacao(StatusEntrega status, Comparator<Entrega> ordenacao) {
        return repository.listarTodas().stream()
            .filter(e -> !e.isRemovida())
            .filter(e -> status == null || e.getStatus() == status)
            .sorted(ordenacao)
            .toList();
    }
}
```

> Note o `filter(e -> !e.isRemovida())` repetido — isso é reaproveitado consistentemente para que itens removidos logicamente (seção 8) nunca apareçam em consultas comuns.

---

## 7. Update: substituição total vs parcial

### Update total (substitui todos os campos — mapeia para `PUT`)

```java
public Entrega atualizarCompleto(String codigo, String novoDestino, double novoPeso, double novoFrete) {
    Entrega existente = buscarOuFalhar(codigo);

    existente.setDestino(novoDestino);
    existente.setPesoKg(novoPeso);
    existente.setFrete(novoFrete);

    return repository.atualizar(existente);
}
```

### Update parcial (só os campos enviados — mapeia para `PATCH`)

```java
public record AtualizacaoParcialDTO(String destino, Double pesoKg, StatusEntrega status) { }

public Entrega atualizarParcial(String codigo, AtualizacaoParcialDTO mudancas) {
    Entrega existente = buscarOuFalhar(codigo);

    if (mudancas.destino() != null) {
        existente.setDestino(mudancas.destino());
    }
    if (mudancas.pesoKg() != null) {
        if (mudancas.pesoKg() <= 0) throw new ValidacaoException("pesoKg deve ser maior que zero");
        existente.setPesoKg(mudancas.pesoKg());
    }
    if (mudancas.status() != null) {
        validarTransicaoStatus(existente.getStatus(), mudancas.status());
        existente.setStatus(mudancas.status());
    }

    return repository.atualizar(existente);
}

private void validarTransicaoStatus(StatusEntrega atual, StatusEntrega novo) {
    if (atual == StatusEntrega.ENTREGUE && novo != StatusEntrega.ENTREGUE) {
        throw new RegraNegocioException("Entrega já finalizada não pode mudar de status");
    }
    if (atual == StatusEntrega.CANCELADA) {
        throw new RegraNegocioException("Entrega cancelada não pode mudar de status");
    }
}
```

> Validar **transições de status** (não apenas o valor isolado) evita, por exemplo, voltar uma entrega `ENTREGUE` para `PENDENTE` por engano — uma regra de negócio, não um detalhe de persistência.

---

## 8. Delete: remoção física vs lógica (soft delete)

### Remoção física (hard delete) — remove de verdade

```java
public void removerFisicamente(String codigo) {
    buscarOuFalhar(codigo);   // garante que existe (lança 404 se não existir)
    repository.remover(codigo);
}
```

### Remoção lógica (soft delete) — mantém o histórico, só marca como removida

```java
public void removerLogicamente(String codigo) {
    Entrega entrega = buscarOuFalhar(codigo);
    entrega.marcarComoRemovida();
    repository.atualizar(entrega);
}
```

| Tipo | Vantagem | Desvantagem | Quando usar |
|---|---|---|---|
| **Física** | Simples, libera espaço | Perde histórico, pode violar auditoria | Dados sem valor de auditoria, fácil de recriar |
| **Lógica** | Mantém histórico, permite auditoria/relatório, reversível | Toda consulta precisa filtrar `removida = false` | Dados de negócio (entregas, pedidos, pagamentos) quase sempre devem usar soft delete |

> Em domínios como logística, **nunca apague fisicamente** uma entrega — ela pode ser necessária para auditoria fiscal, disputas com clientes ou relatórios históricos.

---

## 9. CRUD com JDBC puro

```java
import java.sql.*;
import java.util.*;
import javax.sql.DataSource;

public class EntregaRepositoryJdbc implements EntregaRepository {
    private final DataSource dataSource;

    public EntregaRepositoryJdbc(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Entrega salvar(Entrega entrega) {
        String sql = "INSERT INTO entregas (codigo, destino, peso_kg, frete, status, removida, versao) "
                   + "VALUES (?, ?, ?, ?, ?, ?, ?)";
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql)) {
            stmt.setString(1, entrega.getCodigo());
            stmt.setString(2, entrega.getDestino());
            stmt.setDouble(3, entrega.getPesoKg());
            stmt.setDouble(4, entrega.getFrete());
            stmt.setString(5, entrega.getStatus().name());
            stmt.setBoolean(6, entrega.isRemovida());
            stmt.setLong(7, entrega.getVersao());
            stmt.executeUpdate();
            return entrega;
        } catch (SQLException e) {
            throw new PersistenciaException("Erro ao salvar entrega", e);
        }
    }

    @Override
    public Optional<Entrega> buscarPorCodigo(String codigo) {
        String sql = "SELECT * FROM entregas WHERE codigo = ?";
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql)) {
            stmt.setString(1, codigo);
            try (ResultSet rs = stmt.executeQuery()) {
                return rs.next() ? Optional.of(mapearLinha(rs)) : Optional.empty();
            }
        } catch (SQLException e) {
            throw new PersistenciaException("Erro ao buscar entrega", e);
        }
    }

    @Override
    public List<Entrega> listarTodas() {
        String sql = "SELECT * FROM entregas";
        List<Entrega> resultado = new ArrayList<>();
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql);
             ResultSet rs = stmt.executeQuery()) {
            while (rs.next()) {
                resultado.add(mapearLinha(rs));
            }
            return resultado;
        } catch (SQLException e) {
            throw new PersistenciaException("Erro ao listar entregas", e);
        }
    }

    @Override
    public Entrega atualizar(Entrega entrega) {
        String sql = "UPDATE entregas SET destino = ?, peso_kg = ?, frete = ?, status = ?, removida = ?, versao = versao + 1 "
                   + "WHERE codigo = ? AND versao = ?";
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql)) {
            stmt.setString(1, entrega.getDestino());
            stmt.setDouble(2, entrega.getPesoKg());
            stmt.setDouble(3, entrega.getFrete());
            stmt.setString(4, entrega.getStatus().name());
            stmt.setBoolean(5, entrega.isRemovida());
            stmt.setString(6, entrega.getCodigo());
            stmt.setLong(7, entrega.getVersao());

            int linhasAfetadas = stmt.executeUpdate();
            if (linhasAfetadas == 0) {
                throw new ConflitoVersaoException(entrega.getCodigo());   // ver seção 13
            }
            return entrega;
        } catch (SQLException e) {
            throw new PersistenciaException("Erro ao atualizar entrega", e);
        }
    }

    @Override
    public void remover(String codigo) {
        String sql = "DELETE FROM entregas WHERE codigo = ?";
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql)) {
            stmt.setString(1, codigo);
            stmt.executeUpdate();
        } catch (SQLException e) {
            throw new PersistenciaException("Erro ao remover entrega", e);
        }
    }

    private Entrega mapearLinha(ResultSet rs) throws SQLException {
        Entrega e = new Entrega(
            rs.getString("codigo"),
            rs.getString("destino"),
            rs.getDouble("peso_kg"),
            rs.getDouble("frete"),
            StatusEntrega.valueOf(rs.getString("status"))
        );
        if (rs.getBoolean("removida")) e.marcarComoRemovida();
        return e;
    }
}
```

> **Sempre use `PreparedStatement`** com parâmetros (`?`), nunca concatene valores diretamente no SQL — concatenação abre a porta para **SQL Injection**.

```java
// NUNCA faça isso:
String sql = "SELECT * FROM entregas WHERE codigo = '" + codigoRecebidoDoUsuario + "'";
// Um valor como  ' OR '1'='1  quebra a query inteira
```

---

## 10. CRUD com transações

Quando uma operação de negócio envolve **mais de uma escrita**, todas precisam ser atômicas: ou todas acontecem, ou nenhuma.

```java
public class EntregaService {
    private final DataSource dataSource;

    public void cancelarEDevolverEstoque(String codigoEntrega, EstoqueRepository estoqueRepository) {
        try (Connection con = dataSource.getConnection()) {
            con.setAutoCommit(false);
            try {
                atualizarStatusParaCancelada(con, codigoEntrega);
                estoqueRepository.devolverItens(con, codigoEntrega);   // mesma conexão, mesma transação

                con.commit();
            } catch (Exception e) {
                con.rollback();
                throw new PersistenciaException("Falha ao cancelar entrega, rollback aplicado", e);
            }
        } catch (SQLException e) {
            throw new PersistenciaException("Erro de conexão", e);
        }
    }

    private void atualizarStatusParaCancelada(Connection con, String codigo) throws SQLException {
        try (PreparedStatement stmt = con.prepareStatement(
                "UPDATE entregas SET status = 'CANCELADA' WHERE codigo = ?")) {
            stmt.setString(1, codigo);
            stmt.executeUpdate();
        }
    }
}
```

> Sem a transação, se a atualização de status funcionar mas a devolução de estoque falhar, o sistema fica em um **estado inconsistente** (entrega cancelada, mas estoque não devolvido).

---

## 11. Camada de serviço sobre o CRUD

A camada de serviço é onde regras de negócio **envolvem** as operações de CRUD puras — o repository nunca deveria tomar decisões de negócio.

```java
public class EntregaService {
    private final EntregaRepository repository;

    public EntregaService(EntregaRepository repository) {
        this.repository = repository;
    }

    public Entrega criar(String destino, double pesoKg) { /* seção 5 */ return null; }

    public Entrega buscarOuFalhar(String codigo) { /* seção 6 */ return null; }

    public List<Entrega> listarTodas() { /* seção 6 */ return null; }

    public Entrega atualizarParcial(String codigo, AtualizacaoParcialDTO mudancas) { /* seção 7 */ return null; }

    public void cancelar(String codigo) {
        Entrega entrega = buscarOuFalhar(codigo);
        if (entrega.getStatus() == StatusEntrega.ENTREGUE) {
            throw new RegraNegocioException("Entrega já finalizada não pode ser cancelada");
        }
        entrega.setStatus(StatusEntrega.CANCELADA);
        repository.atualizar(entrega);
    }

    public void remover(String codigo) { /* seção 8: removerLogicamente */ }
}
```

---

## 12. CRUD exposto como API REST

A camada de serviço criada acima é exatamente o que um `Controller` (visto em [[33-API-JAVA]]) chama para atender cada verbo HTTP:

```java
// GET    /entregas           -> service.listarTodas()
// GET    /entregas/{codigo}  -> service.buscarOuFalhar(codigo)
// POST   /entregas           -> service.criar(dto.destino(), dto.pesoKg())
// PATCH  /entregas/{codigo}  -> service.atualizarParcial(codigo, dto)
// DELETE /entregas/{codigo}  -> service.remover(codigo)
```

```java
private static void tratarRequisicao(HttpExchange exchange) throws IOException {
    String metodo = exchange.getRequestMethod();
    String codigo = extrairCodigo(exchange.getRequestURI().getPath());

    switch (metodo) {
        case "GET" -> {
            if (codigo == null) responder(exchange, 200, service.listarTodas());
            else responder(exchange, 200, service.buscarOuFalhar(codigo));
        }
        case "POST" -> {
            NovaEntregaDTO dto = lerCorpo(exchange, NovaEntregaDTO.class);
            responder(exchange, 201, service.criar(dto.destino(), dto.pesoKg()));
        }
        case "PATCH" -> {
            AtualizacaoParcialDTO dto = lerCorpo(exchange, AtualizacaoParcialDTO.class);
            responder(exchange, 200, service.atualizarParcial(codigo, dto));
        }
        case "DELETE" -> {
            service.remover(codigo);
            responder(exchange, 204, null);
        }
    }
}
```

> O CRUD é o **conteúdo**; a API é a **casca HTTP** em volta dele. Separá-los (seção 4) é o que permite testar toda a lógica de CRUD sem precisar de um servidor HTTP rodando.

---

## 13. Concorrência em operações CRUD (optimistic locking)

Quando dois usuários editam o **mesmo registro** ao mesmo tempo, um `UPDATE` pode sobrescrever silenciosamente a alteração do outro ("last write wins"). **Optimistic locking** usa um campo de versão para detectar o conflito.

```java
public class ConflitoVersaoException extends RuntimeException {
    public ConflitoVersaoException(String codigo) {
        super("Entrega " + codigo + " foi modificada por outra operação. Recarregue e tente novamente.");
    }
}
```

O `UPDATE` da seção 9 já implementa isso: `WHERE codigo = ? AND versao = ?`. Se outra transação já alterou a linha (incrementando a versão), **zero linhas são afetadas**, e o código lança `ConflitoVersaoException` em vez de sobrescrever silenciosamente.

```java
// Fluxo do cliente diante de um 409 Conflict por versão desatualizada
Entrega atual = service.buscarOuFalhar("E001");       // versao = 3
// ... usuário edita em uma tela, demora alguns minutos ...
try {
    service.atualizarParcial("E001", mudancas);       // ainda pensa que a versao é 3
} catch (ConflitoVersaoException e) {
    // 409 Conflict: outro usuário já alterou a E001 nesse intervalo
    // cliente deve recarregar os dados atuais e pedir confirmação ao usuário
}
```

> Alternativa mais pessimista (**pessimistic locking**, `SELECT ... FOR UPDATE`) trava a linha durante toda a transação — mais segura contra conflitos, porém reduz a concorrência de leitura/escrita simultânea.

---

## 14. Idempotência nas operações CRUD

| Operação CRUD | Idempotente por natureza? | Observação |
|---|---|---|
| Create | Não | Cada chamada cria um novo registro — proteja com chave de idempotência se o cliente pode reenviar |
| Read | Sim | Leitura nunca altera estado |
| Update (total, `PUT`) | Sim | Define o estado final; repetir com o mesmo corpo não agrava nada |
| Update (parcial, `PATCH`) | Depende | Idempotente se o patch define valores absolutos; não idempotente se aplica deltas |
| Delete | Sim | Remover algo já removido continua "não existe" |

```java
// Create com proteção de idempotência (retomando o padrão de [[32-IDEMPOTENCIA-JAVA]])
public Entrega criarComIdempotencia(String chaveIdempotencia, String destino, double pesoKg) {
    Optional<Entrega> existente = buscarPorChaveIdempotencia(chaveIdempotencia);
    if (existente.isPresent()) {
        return existente.get();   // retry do cliente: devolve o mesmo resultado, não cria de novo
    }
    Entrega criada = criar(destino, pesoKg);
    registrarChaveIdempotencia(chaveIdempotencia, criada.getCodigo());
    return criada;
}
```

---

## 15. Exercício resolvido

**Problema:** implementar o CRUD completo de entregas em memória, com soft delete, validação de transição de status e optimistic locking simplificado.

```java
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class Program {
    public static void main(String[] args) {
        EntregaRepositoryEmMemoria repository = new EntregaRepositoryEmMemoria();
        EntregaService service = new EntregaService(repository);

        // CREATE
        Entrega e1 = service.criar("Lorena", 20.0);
        System.out.println("Criada: " + e1);

        // READ
        Entrega consultada = service.buscarOuFalhar(e1.getCodigo());
        System.out.println("Consultada: " + consultada);

        // UPDATE parcial
        var mudancas = new AtualizacaoParcialDTO(null, null, StatusEntrega.EM_ROTA);
        Entrega atualizada = service.atualizarParcial(e1.getCodigo(), mudancas);
        System.out.println("Atualizada: " + atualizada);

        // UPDATE com conflito de versão simulado
        try {
            atualizada.incrementarVersao();   // simula alteração concorrente de outra sessão
            service.atualizarComVersao(e1.getCodigo(), atualizada, 0);   // versão antiga
        } catch (ConflitoVersaoException ex) {
            System.out.println("Conflito detectado: " + ex.getMessage());
        }

        // DELETE lógico
        service.remover(e1.getCodigo());
        System.out.println("Após remoção lógica, aparece na listagem? "
            + service.listarTodas().stream().anyMatch(e -> e.getCodigo().equals(e1.getCodigo())));
    }
}

class EntregaRepositoryEmMemoria implements EntregaRepository {
    private final Map<String, Entrega> dados = new ConcurrentHashMap<>();

    public Entrega salvar(Entrega entrega) { dados.put(entrega.getCodigo(), entrega); return entrega; }
    public Optional<Entrega> buscarPorCodigo(String codigo) { return Optional.ofNullable(dados.get(codigo)); }
    public List<Entrega> listarTodas() { return new ArrayList<>(dados.values()); }
    public Entrega atualizar(Entrega entrega) { dados.put(entrega.getCodigo(), entrega); return entrega; }
    public void remover(String codigo) { dados.remove(codigo); }
}

class EntregaService {
    private final EntregaRepository repository;

    EntregaService(EntregaRepository repository) { this.repository = repository; }

    Entrega criar(String destino, double pesoKg) {
        if (destino == null || destino.isBlank()) throw new ValidacaoException("destino é obrigatório");
        if (pesoKg <= 0) throw new ValidacaoException("pesoKg deve ser maior que zero");

        String codigo = "E" + UUID.randomUUID().toString().substring(0, 8).toUpperCase();
        Entrega entrega = new Entrega(codigo, destino, pesoKg, pesoKg * 2.5, StatusEntrega.PENDENTE);
        return repository.salvar(entrega);
    }

    Entrega buscarOuFalhar(String codigo) {
        return repository.buscarPorCodigo(codigo)
            .filter(e -> !e.isRemovida())
            .orElseThrow(() -> new EntregaNaoEncontradaException(codigo));
    }

    List<Entrega> listarTodas() {
        return repository.listarTodas().stream().filter(e -> !e.isRemovida()).toList();
    }

    Entrega atualizarParcial(String codigo, AtualizacaoParcialDTO mudancas) {
        Entrega existente = buscarOuFalhar(codigo);
        if (mudancas.status() != null) {
            if (existente.getStatus() == StatusEntrega.ENTREGUE) {
                throw new RegraNegocioException("Entrega já finalizada não pode mudar de status");
            }
            existente.setStatus(mudancas.status());
        }
        existente.incrementarVersao();
        return repository.atualizar(existente);
    }

    Entrega atualizarComVersao(String codigo, Entrega novoEstado, long versaoEsperada) {
        Entrega existente = buscarOuFalhar(codigo);
        if (existente.getVersao() != versaoEsperada) {
            throw new ConflitoVersaoException(codigo);
        }
        return repository.atualizar(novoEstado);
    }

    void remover(String codigo) {
        Entrega entrega = buscarOuFalhar(codigo);
        entrega.marcarComoRemovida();
        repository.atualizar(entrega);
    }
}

record AtualizacaoParcialDTO(String destino, Double pesoKg, StatusEntrega status) { }

class ValidacaoException extends RuntimeException {
    ValidacaoException(String msg) { super(msg); }
}
class RegraNegocioException extends RuntimeException {
    RegraNegocioException(String msg) { super(msg); }
}
class EntregaNaoEncontradaException extends RuntimeException {
    EntregaNaoEncontradaException(String codigo) { super("Entrega " + codigo + " não encontrada"); }
}
class ConflitoVersaoException extends RuntimeException {
    ConflitoVersaoException(String codigo) { super("Entrega " + codigo + " foi modificada por outra operação"); }
}

interface EntregaRepository {
    Entrega salvar(Entrega entrega);
    Optional<Entrega> buscarPorCodigo(String codigo);
    List<Entrega> listarTodas();
    Entrega atualizar(Entrega entrega);
    void remover(String codigo);
}
```

**Saída esperada (código gerado aleatoriamente, formato ilustrado):**

```
Criada: E3F7A9B2 | Lorena | 20.0kg | R$ 50.00 | PENDENTE
Consultada: E3F7A9B2 | Lorena | 20.0kg | R$ 50.00 | PENDENTE
Atualizada: E3F7A9B2 | Lorena | 20.0kg | R$ 50.00 | EM_ROTA
Conflito detectado: Entrega E3F7A9B2 foi modificada por outra operação
Após remoção lógica, aparece na listagem? false
```

---

## 16. Armadilhas e boas práticas

| Armadilha | Explicação | Solução |
|---|---|---|
| Deixar o cliente enviar campos calculados (ex.: frete) no Create | Permite manipulação indevida de valores | Calcular no servidor, ignorar valor enviado pelo cliente |
| Concatenar valores em SQL | Abre brecha para SQL Injection | Sempre `PreparedStatement` com parâmetros |
| `DELETE` físico em dados de negócio relevantes para auditoria | Perda permanente de histórico | Soft delete (campo `removida`/`deletedAt`) |
| Esquecer de filtrar registros removidos logicamente nas consultas | Itens "excluídos" continuam aparecendo | Sempre filtrar `removida = false` nas queries/streams de leitura |
| Update sem checar conflito de versão | "Last write wins" silencioso, perde alterações concorrentes | Optimistic locking (`WHERE ... AND versao = ?`) |
| Múltiplas escritas relacionadas sem transação | Estado inconsistente se uma escrita falhar e outra não | Envolver em transação (`commit`/`rollback`) |
| Regra de negócio dentro do `Repository` | Mistura persistência com regra, dificulta testes e reuso | Regra de negócio na camada de `Service` |
| Create sem proteção de idempotência em fluxos sujeitos a retry | Registros duplicados | Chave de idempotência (ver [[32-IDEMPOTENCIA-JAVA]]) |

### Regras de ouro

1. **Separe contrato (`Repository` interface) de implementação** — memória, JDBC, ou qualquer outro armazenamento.
2. **Regra de negócio vive no `Service`**, nunca no `Repository` nem no `Controller`.
3. **Prefira soft delete** para qualquer dado com valor de auditoria ou histórico.
4. **Proteja updates concorrentes** com controle de versão.
5. **Nunca confie em valores calculados enviados pelo cliente** — recalcule no servidor.

---

## 17. Cheat sheet

```java
public interface Repository<T, ID> {
    T salvar(T entidade);
    Optional<T> buscarPorId(ID id);
    List<T> listarTodos();
    T atualizar(T entidade);
    void remover(ID id);
}
```

| Operação | Memória | JDBC | HTTP |
|---|---|---|---|
| Create | `map.put(id, obj)` | `INSERT ... VALUES (...)` | `POST` |
| Read | `map.get(id)` / `map.values()` | `SELECT ... WHERE id = ?` | `GET` |
| Update | `map.put(id, novoObj)` | `UPDATE ... WHERE id = ? [AND versao = ?]` | `PUT`/`PATCH` |
| Delete | `map.remove(id)` ou flag `removida=true` | `DELETE ...` ou `UPDATE ... SET removida=true` | `DELETE` |

```java
// Transação JDBC
con.setAutoCommit(false);
try {
    // múltiplas escritas
    con.commit();
} catch (Exception e) {
    con.rollback();
    throw e;
}
```

### Referências

- Docs Java: [`java.sql.PreparedStatement`](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/java/sql/PreparedStatement.html), [`java.sql.Connection`](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/java/sql/Connection.html)
- Ver também: [[33-API-JAVA]] (como expor este CRUD via HTTP), [[32-IDEMPOTENCIA-JAVA]] (Create seguro sob retry), [[30-THREADS-CONCORRENCIA-JAVA]] (concorrência em updates)
