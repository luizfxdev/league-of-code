# JDBC em Java


---

## Sumário

1. [Modelo de domínio usado nos exemplos](#1-modelo-de-domínio-usado-nos-exemplos)
2. [O que é JDBC](#2-o-que-é-jdbc)
3. [Arquitetura JDBC: Driver, DriverManager e DataSource](#3-arquitetura-jdbc-driver-drivermanager-e-datasource)
4. [Obtendo uma conexão](#4-obtendo-uma-conexão)
5. [`Statement` vs `PreparedStatement`](#5-statement-vs-preparedstatement)
6. [SQL Injection: o motivo de nunca usar `Statement` com dados do usuário](#6-sql-injection-o-motivo-de-nunca-usar-statement-com-dados-do-usuário)
7. [`ResultSet`: lendo dados retornados](#7-resultset-lendo-dados-retornados)
8. [CRUD básico com JDBC](#8-crud-básico-com-jdbc)
9. [try-with-resources e fechamento de recursos](#9-try-with-resources-e-fechamento-de-recursos)
10. [Transações: `commit` e `rollback`](#10-transações-commit-e-rollback)
11. [Batch: múltiplas operações em um único round-trip](#11-batch-múltiplas-operações-em-um-único-round-trip)
12. [`CallableStatement`: chamando procedures](#12-callablestatement-chamando-procedures)
13. [Connection pooling (HikariCP)](#13-connection-pooling-hikaricp)
14. [Metadados: `DatabaseMetaData` e `ResultSetMetaData`](#14-metadados-databasemetadata-e-resultsetmetadata)
15. [Exceções JDBC (`SQLException`)](#15-exceções-jdbc-sqlexception)
16. [Exercício resolvido](#16-exercício-resolvido)
17. [Armadilhas e boas práticas](#17-armadilhas-e-boas-práticas)
18. [Cheat sheet](#18-cheat-sheet)

---

## 1. Modelo de domínio usado nos exemplos

```java
public class Entrega {
    private String codigo;
    private String destino;
    private double pesoKg;
    private double frete;
    private String status;

    public Entrega(String codigo, String destino, double pesoKg, double frete, String status) {
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
    public String getStatus() { return status; }

    @Override
    public String toString() {
        return codigo + " | " + destino + " | " + pesoKg + "kg | R$ " + String.format("%.2f", frete) + " | " + status;
    }
}
```

**Tabela usada nos exemplos (PostgreSQL/MySQL-compatível):**

```sql
CREATE TABLE entregas (
    codigo    VARCHAR(20) PRIMARY KEY,
    destino   VARCHAR(100) NOT NULL,
    peso_kg   DECIMAL(10,2) NOT NULL,
    frete     DECIMAL(10,2) NOT NULL,
    status    VARCHAR(20) NOT NULL DEFAULT 'PENDENTE'
);
```

**Cenário:** o sistema de gestão da transportadora persiste as entregas em um banco relacional. JDBC é a camada que conecta o código Java ao banco de dados.

---

## 2. O que é JDBC

**JDBC (Java Database Connectivity)** é a API padrão do Java para se conectar e executar comandos SQL em bancos de dados relacionais. Faz parte do próprio JDK (`java.sql`), sem necessidade de bibliotecas externas para o contrato — só o **driver específico** de cada banco (PostgreSQL, MySQL, Oracle etc.) é uma dependência externa.

**Analogia logística:** JDBC é como um **padrão universal de container de carga**. Não importa se o navio é de uma transportadora ou outra (MySQL, PostgreSQL, Oracle) — o container (JDBC API) encaixa do mesmo jeito em qualquer navio, porque todos seguem a mesma interface física. O "adaptador específico do navio" é o **driver JDBC** de cada banco.

```java
// O código Java fala apenas com a API JDBC (java.sql.*)...
Connection conexao = DriverManager.getConnection(url, usuario, senha);

// ...e o DRIVER (dependência específica do banco) traduz isso para o protocolo do banco real
// Exemplo de dependência Maven para PostgreSQL:
// <dependency>
//   <groupId>org.postgresql</groupId>
//   <artifactId>postgresql</artifactId>
//   <version>42.7.3</version>
// </dependency>
```

| Camada | Exemplo |
|---|---|
| API (contrato, parte do JDK) | `java.sql.Connection`, `java.sql.Statement`, `java.sql.ResultSet` |
| Driver (implementação específica do banco) | `org.postgresql.Driver`, `com.mysql.cj.jdbc.Driver` |
| Banco de dados real | PostgreSQL, MySQL, Oracle, SQL Server |

> Frameworks como **Spring Data JPA**, **Hibernate** e **MyBatis** são construídos **sobre** JDBC — entender JDBC ajuda a entender o que essas camadas fazem por baixo e a debugar problemas de performance/SQL quando a "mágica" do framework não é suficiente.

---

## 3. Arquitetura JDBC: Driver, DriverManager e DataSource

```java
// String de conexão (JDBC URL): jdbc:<banco>://<host>:<porta>/<database>
String url = "jdbc:postgresql://localhost:5432/transportadora";
```

| Componente | Papel |
|---|---|
| `Driver` | Implementação que sabe falar o protocolo de um banco específico |
| `DriverManager` | Fábrica simples de conexões — carrega o driver pela URL e abre uma `Connection` nova a cada chamada |
| `DataSource` | Fábrica de conexões mais robusta, geralmente com **pool de conexões** por trás (ver seção 13) |

```java
import java.sql.*;

// Via DriverManager: abre uma conexão nova a cada chamada (sem pool)
Connection con = DriverManager.getConnection(url, "usuario", "senha");
```

```java
import javax.sql.DataSource;
import com.zaxxer.hikari.HikariDataSource;

// Via DataSource com pool (abordagem recomendada em produção)
HikariDataSource dataSource = new HikariDataSource();
dataSource.setJdbcUrl(url);
dataSource.setUsername("usuario");
dataSource.setPassword("senha");

Connection con = dataSource.getConnection();   // reaproveita conexão do pool
```

> Em Java 6+ não é mais necessário chamar `Class.forName("org.postgresql.Driver")` manualmente — o driver é descoberto automaticamente via **Service Provider Interface (SPI)**, desde que esteja no classpath.

---

## 4. Obtendo uma conexão

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConexaoFactory {
    private static final String URL = "jdbc:postgresql://localhost:5432/transportadora";
    private static final String USUARIO = "app_user";
    private static final String SENHA = System.getenv("DB_PASSWORD");   // nunca hardcode senha no código

    public static Connection obterConexao() {
        try {
            return DriverManager.getConnection(URL, USUARIO, SENHA);
        } catch (SQLException e) {
            throw new RuntimeException("Falha ao conectar ao banco", e);
        }
    }
}
```

```java
try (Connection con = ConexaoFactory.obterConexao()) {
    System.out.println("Conectado: " + !con.isClosed());
} catch (SQLException e) {
    e.printStackTrace();
}
```

> **Credenciais nunca devem ficar hardcoded no código-fonte.** Use variáveis de ambiente, arquivos de configuração fora do controle de versão, ou um gerenciador de segredos (Vault, AWS Secrets Manager etc.).

---

## 5. `Statement` vs `PreparedStatement`

### `Statement` (SQL puro, sem parâmetros) — evitar com dados externos

```java
try (Connection con = ConexaoFactory.obterConexao();
     Statement stmt = con.createStatement()) {

    ResultSet rs = stmt.executeQuery("SELECT * FROM entregas WHERE status = 'PENDENTE'");
    while (rs.next()) {
        System.out.println(rs.getString("codigo"));
    }
}
```

### `PreparedStatement` (com parâmetros `?`) — sempre que houver entrada externa

```java
String sql = "SELECT * FROM entregas WHERE status = ? AND peso_kg > ?";

try (Connection con = ConexaoFactory.obterConexao();
     PreparedStatement stmt = con.prepareStatement(sql)) {

    stmt.setString(1, "PENDENTE");
    stmt.setDouble(2, 50.0);

    try (ResultSet rs = stmt.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getString("codigo"));
        }
    }
}
```

| Aspecto | `Statement` | `PreparedStatement` |
|---|---|---|
| Parâmetros | Concatenados manualmente na string SQL | Passados via `set*(índice, valor)` |
| Segurança contra SQL Injection | Nenhuma | Alta — valores nunca são interpretados como SQL |
| Performance em execuções repetidas | Reenvia o SQL completo sempre | Banco pode reutilizar o plano de execução já compilado |
| Uso recomendado | Apenas SQL fixo, sem nenhuma entrada externa (raro) | **Padrão para qualquer SQL com valor variável** |

---

## 6. SQL Injection: o motivo de nunca usar `Statement` com dados do usuário

### ❌ Vulnerável

```java
public List<Entrega> buscarPorDestino(String destinoDigitadoPeloUsuario) throws SQLException {
    String sql = "SELECT * FROM entregas WHERE destino = '" + destinoDigitadoPeloUsuario + "'";

    try (Connection con = ConexaoFactory.obterConexao();
         Statement stmt = con.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        // ...
    }
}
```

Se o usuário digitar `Lorena' OR '1'='1`, a query final se torna:

```sql
SELECT * FROM entregas WHERE destino = 'Lorena' OR '1'='1'
```

Isso retorna **todas as entregas**, não só as de Lorena — a condição `'1'='1'` é sempre verdadeira. Em casos piores, um valor como `'; DROP TABLE entregas; --` pode **apagar a tabela inteira**, dependendo das permissões do usuário do banco.

### ✅ Protegido

```java
public List<Entrega> buscarPorDestino(String destino) throws SQLException {
    String sql = "SELECT * FROM entregas WHERE destino = ?";

    try (Connection con = ConexaoFactory.obterConexao();
         PreparedStatement stmt = con.prepareStatement(sql)) {

        stmt.setString(1, destino);   // o driver trata o valor como DADO, nunca como comando SQL

        try (ResultSet rs = stmt.executeQuery()) {
            List<Entrega> resultado = new ArrayList<>();
            while (rs.next()) {
                resultado.add(mapearLinha(rs));
            }
            return resultado;
        }
    }
}
```

Com `PreparedStatement`, o valor `Lorena' OR '1'='1` é tratado **literalmente** como o texto de busca — não há forma de o valor "escapar" para se tornar comando SQL.

> Regra absoluta: **nunca concatene valor de entrada (usuário, API, arquivo) diretamente em uma string SQL.** Sempre use `?` + `set*()`.

---

## 7. `ResultSet`: lendo dados retornados

```java
String sql = "SELECT codigo, destino, peso_kg, frete, status FROM entregas WHERE codigo = ?";

try (Connection con = ConexaoFactory.obterConexao();
     PreparedStatement stmt = con.prepareStatement(sql)) {

    stmt.setString(1, "E001");

    try (ResultSet rs = stmt.executeQuery()) {
        if (rs.next()) {   // avança para a primeira linha; false se não houver resultado
            String codigo = rs.getString("codigo");
            String destino = rs.getString("destino");
            double pesoKg = rs.getDouble("peso_kg");
            double frete = rs.getDouble("frete");
            String status = rs.getString("status");

            Entrega entrega = new Entrega(codigo, destino, pesoKg, frete, status);
            System.out.println(entrega);
        }
    }
}
```

| Método | Tipo retornado |
|---|---|
| `getString(coluna)` | `String` |
| `getInt(coluna)` / `getLong(coluna)` | `int` / `long` |
| `getDouble(coluna)` / `getBigDecimal(coluna)` | `double` / `BigDecimal` (prefira `BigDecimal` para valores monetários) |
| `getBoolean(coluna)` | `boolean` |
| `getDate(coluna)` / `getTimestamp(coluna)` | `java.sql.Date` / `java.sql.Timestamp` |

```java
// Colunas também podem ser acessadas por índice (1-based), mas por nome é mais legível e resiliente a mudança de ordem
String destino = rs.getString(2);   // funciona, mas quebra se a ordem das colunas no SELECT mudar
```

```java
// Percorrendo múltiplas linhas
List<Entrega> todas = new ArrayList<>();
try (ResultSet rs = stmt.executeQuery()) {
    while (rs.next()) {
        todas.add(mapearLinha(rs));
    }
}
```

> `rs.next()` **precisa** ser chamado antes de ler qualquer coluna — ele posiciona o cursor na primeira linha (ou confirma que não há linhas). Esquecer isso é um erro comum para quem está começando com JDBC.

---

## 8. CRUD básico com JDBC

```java
public class EntregaDao {
    private final DataSource dataSource;

    public EntregaDao(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    // CREATE
    public void inserir(Entrega entrega) throws SQLException {
        String sql = "INSERT INTO entregas (codigo, destino, peso_kg, frete, status) VALUES (?, ?, ?, ?, ?)";
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql)) {
            stmt.setString(1, entrega.getCodigo());
            stmt.setString(2, entrega.getDestino());
            stmt.setDouble(3, entrega.getPesoKg());
            stmt.setDouble(4, entrega.getFrete());
            stmt.setString(5, entrega.getStatus());
            stmt.executeUpdate();
        }
    }

    // READ
    public Optional<Entrega> buscarPorCodigo(String codigo) throws SQLException {
        String sql = "SELECT * FROM entregas WHERE codigo = ?";
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql)) {
            stmt.setString(1, codigo);
            try (ResultSet rs = stmt.executeQuery()) {
                return rs.next() ? Optional.of(mapearLinha(rs)) : Optional.empty();
            }
        }
    }

    // UPDATE
    public void atualizarStatus(String codigo, String novoStatus) throws SQLException {
        String sql = "UPDATE entregas SET status = ? WHERE codigo = ?";
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql)) {
            stmt.setString(1, novoStatus);
            stmt.setString(2, codigo);
            int linhasAfetadas = stmt.executeUpdate();
            if (linhasAfetadas == 0) {
                throw new IllegalStateException("Entrega " + codigo + " não encontrada");
            }
        }
    }

    // DELETE
    public void remover(String codigo) throws SQLException {
        String sql = "DELETE FROM entregas WHERE codigo = ?";
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql)) {
            stmt.setString(1, codigo);
            stmt.executeUpdate();
        }
    }

    private Entrega mapearLinha(ResultSet rs) throws SQLException {
        return new Entrega(
            rs.getString("codigo"),
            rs.getString("destino"),
            rs.getDouble("peso_kg"),
            rs.getDouble("frete"),
            rs.getString("status")
        );
    }
}
```

| Método | Usado para | Retorna |
|---|---|---|
| `executeQuery()` | `SELECT` | `ResultSet` |
| `executeUpdate()` | `INSERT`, `UPDATE`, `DELETE` | `int` (número de linhas afetadas) |
| `execute()` | SQL genérico (quando não se sabe o tipo em tempo de compilação) | `boolean` (indica se o resultado é um `ResultSet`) |

> `executeUpdate()` retornando `0` é um sinal útil: significa que o `WHERE` não encontrou nenhuma linha — frequentemente indica um `UPDATE`/`DELETE` para um código inexistente.

---

## 9. try-with-resources e fechamento de recursos

`Connection`, `Statement`/`PreparedStatement` e `ResultSet` implementam `AutoCloseable` — **sempre** devem ser fechados, ou cada um deles mantém recursos de rede/memória no banco e na aplicação.

```java
// ERRADO: recursos nunca fechados se uma exceção ocorrer, ou mesmo em caminho normal se esquecido
Connection con = dataSource.getConnection();
PreparedStatement stmt = con.prepareStatement(sql);
ResultSet rs = stmt.executeQuery();
// ... se algo lançar exceção aqui, con/stmt/rs ficam abertos para sempre
```

```java
// CORRETO: try-with-resources fecha automaticamente, mesmo em caso de exceção,
// na ordem inversa de abertura (rs -> stmt -> con)
try (Connection con = dataSource.getConnection();
     PreparedStatement stmt = con.prepareStatement(sql);
     ResultSet rs = stmt.executeQuery()) {

    while (rs.next()) {
        processar(rs);
    }
} catch (SQLException e) {
    throw new RuntimeException("Erro ao consultar entregas", e);
}
```

> Vazamento de conexões (`Connection` não fechada) é uma das causas mais comuns de uma aplicação **esgotar o pool de conexões** em produção, travando todo o sistema sob carga.

---

## 10. Transações: `commit` e `rollback`

Por padrão, cada `Connection` JDBC está em **auto-commit**: cada comando é confirmado (`commit`) imediatamente. Para agrupar múltiplas operações como uma unidade atômica, desative o auto-commit.

```java
public void cancelarEDevolverEstoque(String codigoEntrega, String produto, int quantidade) throws SQLException {
    try (Connection con = dataSource.getConnection()) {
        con.setAutoCommit(false);

        try (PreparedStatement stmt1 = con.prepareStatement(
                 "UPDATE entregas SET status = 'CANCELADA' WHERE codigo = ?");
             PreparedStatement stmt2 = con.prepareStatement(
                 "UPDATE estoque SET quantidade = quantidade + ? WHERE produto = ?")) {

            stmt1.setString(1, codigoEntrega);
            stmt1.executeUpdate();

            stmt2.setInt(1, quantidade);
            stmt2.setString(2, produto);
            stmt2.executeUpdate();

            con.commit();   // confirma as duas alterações juntas
        } catch (SQLException e) {
            con.rollback();   // desfaz TUDO desde o início da transação
            throw e;
        } finally {
            con.setAutoCommit(true);   // devolve a conexão ao estado padrão antes de voltar ao pool
        }
    }
}
```

> Sem a transação, se a primeira `UPDATE` tiver sucesso e a segunda falhar, o sistema fica com a entrega cancelada **sem** o estoque devolvido — um estado inconsistente que a transação evita.

### Savepoints (reverter parcialmente)

```java
con.setAutoCommit(false);
Savepoint ponto = con.setSavepoint("antesDoEstoque");

try {
    atualizarStatusCancelada(con, codigo);
    atualizarEstoque(con, produto, quantidade);
    con.commit();
} catch (SQLException e) {
    con.rollback(ponto);   // desfaz só a partir do savepoint, não a transação inteira
}
```

---

## 11. Batch: múltiplas operações em um único round-trip

Enviar comandos um por um gera uma viagem de rede por comando. **Batch** agrupa várias operações, reduzindo drasticamente o overhead em cargas grandes.

```java
public void inserirVarias(List<Entrega> entregas) throws SQLException {
    String sql = "INSERT INTO entregas (codigo, destino, peso_kg, frete, status) VALUES (?, ?, ?, ?, ?)";

    try (Connection con = dataSource.getConnection();
         PreparedStatement stmt = con.prepareStatement(sql)) {

        con.setAutoCommit(false);

        for (Entrega e : entregas) {
            stmt.setString(1, e.getCodigo());
            stmt.setString(2, e.getDestino());
            stmt.setDouble(3, e.getPesoKg());
            stmt.setDouble(4, e.getFrete());
            stmt.setString(5, e.getStatus());
            stmt.addBatch();   // adiciona ao lote, ainda não executa

            if (entregas.indexOf(e) % 1000 == 0) {   // executa em lotes de 1000 para não sobrecarregar memória
                stmt.executeBatch();
            }
        }

        stmt.executeBatch();   // executa o restante
        con.commit();
    }
}
```

> Para inserir 100.000 registros, `addBatch()` + `executeBatch()` pode ser **ordens de magnitude mais rápido** do que 100.000 chamadas individuais de `executeUpdate()`.

---

## 12. `CallableStatement`: chamando procedures

```sql
-- Procedure (ilustrativa) que calcula e atualiza o frete com base no peso
CREATE PROCEDURE recalcular_frete(IN p_codigo VARCHAR(20), OUT p_novo_frete DECIMAL(10,2))
...
```

```java
String sql = "{call recalcular_frete(?, ?)}";

try (Connection con = dataSource.getConnection();
     CallableStatement stmt = con.prepareCall(sql)) {

    stmt.setString(1, "E001");
    stmt.registerOutParameter(2, Types.DECIMAL);

    stmt.execute();

    double novoFrete = stmt.getDouble(2);
    System.out.println("Novo frete calculado: " + novoFrete);
}
```

> `CallableStatement` é usado quando a lógica já existe como **stored procedure** no banco — comum em sistemas legados ou com regras fortemente ligadas ao banco de dados.

---

## 13. Connection pooling (HikariCP)

Abrir uma conexão TCP nova com o banco a cada operação é caro (handshake, autenticação). Um **pool de conexões** mantém conexões abertas e reutilizáveis.

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

public class DataSourceFactory {
    public static HikariDataSource criar() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/transportadora");
        config.setUsername("app_user");
        config.setPassword(System.getenv("DB_PASSWORD"));
        config.setMaximumPoolSize(10);       // número máximo de conexões simultâneas
        config.setMinimumIdle(2);            // conexões mantidas abertas mesmo sem uso
        config.setConnectionTimeout(5000);   // tempo máximo esperando uma conexão livre do pool (ms)
        config.setIdleTimeout(600000);       // tempo até fechar conexão ociosa

        return new HikariDataSource(config);
    }
}
```

```java
HikariDataSource dataSource = DataSourceFactory.criar();
EntregaDao dao = new EntregaDao(dataSource);

// Cada getConnection() pega uma conexão do pool (reaproveitada), não abre uma nova do zero
try (Connection con = dataSource.getConnection()) {
    // ...
}
// close() devolve a conexão ao pool, não encerra a conexão física com o banco
```

| Sem pool (`DriverManager`) | Com pool (`HikariDataSource`) |
|---|---|
| Nova conexão TCP a cada `getConnection()` | Conexão reaproveitada de um conjunto já aberto |
| Alto custo sob carga (muitas conexões simultâneas) | Custo amortizado, limite de conexões controlado |
| Adequado para scripts simples/didáticos | **Padrão em qualquer aplicação real** |

> **HikariCP** é o pool mais usado no ecossistema Java moderno (inclusive o padrão do Spring Boot) por sua performance e simplicidade de configuração.

---

## 14. Metadados: `DatabaseMetaData` e `ResultSetMetaData`

```java
try (Connection con = dataSource.getConnection()) {
    DatabaseMetaData meta = con.getMetaData();
    System.out.println("Banco: " + meta.getDatabaseProductName());
    System.out.println("Versão: " + meta.getDatabaseProductVersion());
    System.out.println("Driver: " + meta.getDriverName());
}
```

```java
try (Connection con = dataSource.getConnection();
     PreparedStatement stmt = con.prepareStatement("SELECT * FROM entregas");
     ResultSet rs = stmt.executeQuery()) {

    ResultSetMetaData rsMeta = rs.getMetaData();
    for (int i = 1; i <= rsMeta.getColumnCount(); i++) {
        System.out.println(rsMeta.getColumnName(i) + " : " + rsMeta.getColumnTypeName(i));
    }
}
```

> Útil para ferramentas genéricas (exportar qualquer tabela para CSV, por exemplo) que não conhecem o esquema em tempo de compilação.

---

## 15. Exceções JDBC (`SQLException`)

```java
try {
    dao.inserir(entrega);
} catch (SQLException e) {
    System.out.println("Código de erro do banco: " + e.getErrorCode());
    System.out.println("SQLState: " + e.getSQLState());   // código padronizado (ex.: "23505" = unique violation)
    System.out.println("Mensagem: " + e.getMessage());

    // SQLException pode encadear múltiplas causas (getNextException)
    SQLException proxima = e.getNextException();
    while (proxima != null) {
        System.out.println("Causa adicional: " + proxima.getMessage());
        proxima = proxima.getNextException();
    }
}
```

```java
// Tratando violação de chave única especificamente (ex.: código de entrega duplicado)
try {
    dao.inserir(entrega);
} catch (SQLException e) {
    if ("23505".equals(e.getSQLState())) {   // SQLState de unique_violation no PostgreSQL
        throw new RegraNegocioException("Código de entrega já existe: " + entrega.getCodigo());
    }
    throw new RuntimeException("Erro ao inserir entrega", e);
}
```

> `SQLException` é uma exceção **checada** — todo método que a declara (`throws SQLException`) obriga o chamador a tratar ou repropagar. Em camadas de serviço, é comum envolvê-la em uma exceção de runtime própria da aplicação (ex.: `PersistenciaException`), evitando que `java.sql.SQLException` "contamine" camadas que não deveriam conhecer detalhes de banco.

---

## 16. Exercício resolvido

**Problema:** implementar um DAO completo de entregas com conexão via pool, tratamento de violação de chave única no `insert`, e uma operação transacional de cancelamento com devolução de estoque.

```java
import java.sql.*;
import java.util.*;
import javax.sql.DataSource;

public class Program {
    public static void main(String[] args) throws SQLException {
        DataSource dataSource = DataSourceFactory.criar();
        EntregaDao dao = new EntregaDao(dataSource);

        Entrega nova = new Entrega("E100", "Guará", 45.0, 112.5, "PENDENTE");
        dao.inserir(nova);
        System.out.println("Inserida com sucesso: " + nova);

        try {
            dao.inserir(nova);   // tentativa de inserir o mesmo código de novo
        } catch (RegraNegocioException e) {
            System.out.println("Esperado: " + e.getMessage());
        }

        Optional<Entrega> consultada = dao.buscarPorCodigo("E100");
        consultada.ifPresent(e -> System.out.println("Encontrada: " + e));

        dao.cancelarEDevolverEstoque("E100", "PALLET-01", 3);
        System.out.println("Cancelamento e devolução de estoque confirmados em uma única transação.");
    }
}

class EntregaDao {
    private final DataSource dataSource;

    EntregaDao(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    void inserir(Entrega e) {
        String sql = "INSERT INTO entregas (codigo, destino, peso_kg, frete, status) VALUES (?, ?, ?, ?, ?)";
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql)) {
            stmt.setString(1, e.getCodigo());
            stmt.setString(2, e.getDestino());
            stmt.setDouble(3, e.getPesoKg());
            stmt.setDouble(4, e.getFrete());
            stmt.setString(5, e.getStatus());
            stmt.executeUpdate();
        } catch (SQLException ex) {
            if ("23505".equals(ex.getSQLState())) {
                throw new RegraNegocioException("Código já existe: " + e.getCodigo());
            }
            throw new RuntimeException("Erro ao inserir entrega", ex);
        }
    }

    Optional<Entrega> buscarPorCodigo(String codigo) {
        String sql = "SELECT * FROM entregas WHERE codigo = ?";
        try (Connection con = dataSource.getConnection();
             PreparedStatement stmt = con.prepareStatement(sql)) {
            stmt.setString(1, codigo);
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return Optional.of(new Entrega(
                        rs.getString("codigo"), rs.getString("destino"),
                        rs.getDouble("peso_kg"), rs.getDouble("frete"), rs.getString("status")
                    ));
                }
                return Optional.empty();
            }
        } catch (SQLException ex) {
            throw new RuntimeException("Erro ao buscar entrega", ex);
        }
    }

    void cancelarEDevolverEstoque(String codigoEntrega, String produto, int quantidade) {
        try (Connection con = dataSource.getConnection()) {
            con.setAutoCommit(false);
            try (PreparedStatement stmt1 = con.prepareStatement(
                     "UPDATE entregas SET status = 'CANCELADA' WHERE codigo = ?");
                 PreparedStatement stmt2 = con.prepareStatement(
                     "UPDATE estoque SET quantidade = quantidade + ? WHERE produto = ?")) {

                stmt1.setString(1, codigoEntrega);
                stmt1.executeUpdate();

                stmt2.setInt(1, quantidade);
                stmt2.setString(2, produto);
                stmt2.executeUpdate();

                con.commit();
            } catch (SQLException ex) {
                con.rollback();
                throw new RuntimeException("Falha no cancelamento, rollback aplicado", ex);
            } finally {
                con.setAutoCommit(true);
            }
        } catch (SQLException ex) {
            throw new RuntimeException("Erro de conexão", ex);
        }
    }
}

class RegraNegocioException extends RuntimeException {
    RegraNegocioException(String msg) { super(msg); }
}
```

**Saída esperada (ilustrativa, depende do banco configurado):**

```
Inserida com sucesso: E100 | Guará | 45.0kg | R$ 112.50 | PENDENTE
Esperado: Código já existe: E100
Encontrada: E100 | Guará | 45.0kg | R$ 112.50 | PENDENTE
Cancelamento e devolução de estoque confirmados em uma única transação.
```

---

## 17. Armadilhas e boas práticas

| Armadilha | Explicação | Solução |
|---|---|---|
| Concatenar valores em SQL | Vulnerabilidade de SQL Injection | Sempre `PreparedStatement` com `?` |
| Não fechar `Connection`/`Statement`/`ResultSet` | Vazamento de recursos, pool se esgota sob carga | `try-with-resources` sempre |
| Esquecer `rs.next()` antes de ler colunas | `SQLException` ou leitura de linha errada | Sempre checar/avançar o cursor antes de ler |
| Usar `DriverManager` direto em produção | Sem pool, alto custo de conexão sob carga | `DataSource` com pool (HikariCP) |
| Múltiplas escritas relacionadas sem transação | Estado inconsistente se uma escrita falhar | `setAutoCommit(false)` + `commit`/`rollback` |
| Hardcode de credenciais no código | Risco de segurança, segredo versionado no Git | Variáveis de ambiente ou gerenciador de segredos |
| Inserir milhares de linhas uma por uma | Lento por overhead de round-trip | `addBatch()` + `executeBatch()` |
| Ignorar `SQLState` ao tratar erros | Trata toda falha de banco genericamente, perde contexto útil (ex.: violação de unique) | Checar `getSQLState()`/`getErrorCode()` para tratamento específico |
| Deixar `SQLException` (checada) vazar para camadas de domínio | Acopla regra de negócio a detalhes de JDBC | Encapsular em exceção de aplicação (`PersistenciaException`) |

### Regras de ouro

1. **`PreparedStatement` é o padrão**, `Statement` só para SQL totalmente fixo (raro).
2. **Sempre `try-with-resources`** para `Connection`, `Statement`, `ResultSet`.
3. **Use `DataSource` com pool** em qualquer aplicação real, nunca `DriverManager` puro em produção.
4. **Agrupe escritas relacionadas em transação** — commit ou rollback, nunca estado parcial.
5. **Nunca deixe `SQLException` vazar** para camadas que não deveriam conhecer JDBC.

---

## 18. Cheat sheet

```java
// Conexão via pool
try (Connection con = dataSource.getConnection()) { ... }

// Consulta parametrizada
try (PreparedStatement stmt = con.prepareStatement("SELECT * FROM tabela WHERE id = ?")) {
    stmt.setString(1, valor);
    try (ResultSet rs = stmt.executeQuery()) {
        while (rs.next()) { /* ler colunas */ }
    }
}

// Insert/Update/Delete
try (PreparedStatement stmt = con.prepareStatement("UPDATE tabela SET campo = ? WHERE id = ?")) {
    stmt.setString(1, novoValor);
    stmt.setString(2, id);
    int linhas = stmt.executeUpdate();
}

// Transação
con.setAutoCommit(false);
try {
    // múltiplos executeUpdate()
    con.commit();
} catch (SQLException e) {
    con.rollback();
    throw e;
} finally {
    con.setAutoCommit(true);
}

// Batch
for (Item item : itens) {
    stmt.setString(1, item.getValor());
    stmt.addBatch();
}
stmt.executeBatch();
```

| Interface | Para quê |
|---|---|
| `DriverManager` | Conexão simples, sem pool (scripts, exemplos) |
| `DataSource` | Conexão com pool (produção) |
| `Statement` | SQL fixo, sem parâmetros |
| `PreparedStatement` | SQL parametrizado (padrão) |
| `CallableStatement` | Chamada de stored procedure |
| `ResultSet` | Leitura de resultado de `SELECT` |

### Referências

- Docs Java: [`java.sql`](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/java/sql/package-summary.html), [`PreparedStatement`](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/java/sql/PreparedStatement.html), [`DataSource`](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/javax/sql/DataSource.html)
- HikariCP: [github.com/brettwooldridge/HikariCP](https://github.com/brettwooldridge/HikariCP)
- Ver também: [[34-CRUD-JAVA]] (CRUD construído sobre JDBC), [[32-IDEMPOTENCIA-JAVA]] (constraint única como garantia distribuída)
