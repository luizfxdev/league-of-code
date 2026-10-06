# JPA / Hibernate — Mapeamento Objeto-Relacional

> Baseado no material do Prof. Nelio Alves + exemplos do seu contexto.

---

## 1. O Problema

No seu dia a dia no GWEB, você lida com isso o tempo todo:

```java
// DAO puro — você escreve isso à mão
public Veiculo buscarPorId(int id) {
    Connection c = FabricaDeConexao.getConexao();
    PreparedStatement stmt = c.prepareStatement("SELECT * FROM veiculo WHERE id = ?");
    stmt.setInt(1, id);
    ResultSet rs = stmt.executeQuery();
    if (rs.next()) {
        Veiculo v = new Veiculo();
        v.setId(rs.getInt("id"));
        v.setPlaca(rs.getString("placa"));
        v.setTipo(rs.getString("tipo"));
        // ... campo por campo
    }
    rs.close(); c.close();
    return v;
}
```

Você tem um objeto `Veiculo` em Java e uma tabela `veiculo` no PostgreSQL. Toda hora precisa converter de um para o outro manualmente.

**Outros problemas que surgem junto:**
- Contexto de persistência (objeto está "atrelado" a uma conexão ou não?)
- Mapa de identidade (não carregar o mesmo objeto duas vezes)
- Lazy loading (não carregar dados que não vão ser usados)
- Gerenciamento de transações

Martin Fowler estimou que ~30% do esforço de um sistema enterprise fica nessa camada de persistência.

---

## 2. O Que é JPA

**JPA (Java Persistence API)** é uma **especificação** — ela define as regras de como o mapeamento objeto-relacional deve funcionar em Java EE. Pacote: `javax.persistence`.

**Hibernate** é uma **implementação** dessa especificação. É o que de fato executa o trabalho.

```
Sua aplicação
    └── JPA (especificação / contrato)
            └── Hibernate (implementação / motor)
                    └── Banco de Dados (PostgreSQL, MySQL, etc.)
```

Você programa contra a API do JPA. O Hibernate faz acontecer nos bastidores.

---

## 3. Principais Classes

### `EntityManager`

É a conexão com o banco. Você usa ele para fazer CRUD nas suas entidades.

```java
EntityManager em = factory.createEntityManager();

// INSERT
em.persist(pokemon);

// SELECT por ID
Pokemon p = em.find(Pokemon.class, 1);

// UPDATE (dentro de uma transação, só alterar o objeto já basta)
p.setNivel(p.getNivel() + 1);

// DELETE
em.remove(p);
```

**Escopo:** uma instância por thread / por requisição HTTP.

### `EntityManagerFactory`

Cria os `EntityManager`s. É pesado de inicializar (lê configurações, monta o schema).

**Escopo:** uma instância por aplicação inteira. Criada na inicialização, fechada no shutdown.

```java
EntityManagerFactory factory = Persistence.createEntityManagerFactory("meu-projeto");
// usa durante toda a vida da aplicação
factory.close(); // só no encerramento
```

---

## 4. Configuração

### `pom.xml` (Maven) — com PostgreSQL

```xml
<dependencies>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>5.6.15.Final</version>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.6.0</version>
    </dependency>
</dependencies>
```

### `persistence.xml`

Fica em `src/main/resources/META-INF/persistence.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
    <persistence-unit name="meu-projeto" transaction-type="RESOURCE_LOCAL">
        <properties>
            <property name="javax.persistence.jdbc.url"
                      value="jdbc:postgresql://localhost:5432/meubanco"/>
            <property name="javax.persistence.jdbc.driver"
                      value="org.postgresql.Driver"/>
            <property name="javax.persistence.jdbc.user" value="postgres"/>
            <property name="javax.persistence.jdbc.password" value="senha"/>

            <!-- create: recria o schema a cada run (dev) -->
            <!-- update: só aplica diferenças (cuidado em prod) -->
            <!-- validate: só verifica, não altera (prod) -->
            <!-- none: não faz nada (prod seguro) -->
            <property name="hibernate.hbm2ddl.auto" value="update"/>

            <property name="hibernate.dialect"
                      value="org.hibernate.dialect.PostgreSQL10Dialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```

---

## 5. Mapeando Entidades

Pense no exercício de logística que fizemos — `Veiculo`, `Caminhao`, `Motocicleta`, `Van`. Veja como ficaria com JPA:

### Entidade base

```java
@Entity
@Table(name = "veiculo")
@Inheritance(strategy = InheritanceType.JOINED) // cada subclasse tem sua própria tabela
public abstract class Veiculo implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // autoincrement do banco
    private Integer id;

    @Column(nullable = false, length = 8)
    private String placa;

    @Column(nullable = false)
    private Double capacidadeCarga;

    // getters e setters
}
```

### Subclasses

```java
@Entity
@Table(name = "caminhao")
public class Caminhao extends Veiculo {

    @Column(nullable = false)
    private Integer numeroEixos;

    @Override
    public Double calcularCusto(Double distancia) {
        return distancia * 3.5 + (getCapacidadeCarga() * 0.1);
    }
}
```

```java
@Entity
@Table(name = "motocicleta")
public class Motocicleta extends Veiculo {

    @Column(nullable = false)
    private Double cilindradas;

    @Override
    public Double calcularCusto(Double distancia) {
        return distancia * 1.2;
    }
}
```

O JPA/Hibernate vai criar as tabelas `veiculo`, `caminhao` e `motocicleta` automaticamente com os JOINs necessários.

### Exemplo com o sistema Pokémon

```java
@Entity
@Table(name = "pokemon")
public class Pokemon implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(nullable = false, length = 50)
    private String nome;

    @Column(nullable = false)
    private Integer nivel;

    @Enumerated(EnumType.STRING) // salva "FOGO" no banco, não 0, 1, 2...
    @Column(nullable = false)
    private TipoPokemon tipo;

    @ManyToOne
    @JoinColumn(name = "mochila_id")
    private MochilaPokemon mochila;

    // getters e setters
}
```

```java
@Entity
@Table(name = "mochila")
public class MochilaPokemon implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(nullable = false)
    private Integer capacidadeMaxima;

    @OneToMany(mappedBy = "mochila", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Pokemon> pokemons = new ArrayList<>();
}
```

---

## 6. Anotações Essenciais

| Anotação | O que faz |
|---|---|
| `@Entity` | Marca a classe como entidade JPA |
| `@Table(name = "x")` | Define o nome da tabela (opcional se for igual ao nome da classe) |
| `@Id` | Marca o atributo como chave primária |
| `@GeneratedValue` | Estratégia de geração do ID |
| `@Column` | Configura a coluna (nome, nullable, length, unique) |
| `@Enumerated(EnumType.STRING)` | Persiste enum como String |
| `@ManyToOne` | N registros apontam para 1 |
| `@OneToMany` | 1 registro tem N filhos |
| `@OneToOne` | 1 para 1 |
| `@ManyToMany` | N para N (gera tabela intermediária) |
| `@JoinColumn` | Define a FK na tabela |
| `@Transient` | Campo ignorado pelo JPA |

---

## 7. Estratégias de Herança

Você tem três opções para mapear herança (como no sistema de veículos):

### `SINGLE_TABLE` — tudo em uma tabela
```java
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo_veiculo")
```
- ✅ Performance (sem JOIN)
- ❌ Muitas colunas nulas

### `JOINED` — uma tabela por classe na hierarquia
```java
@Inheritance(strategy = InheritanceType.JOINED)
```
- ✅ Banco normalizado
- ❌ JOIN em toda consulta

### `TABLE_PER_CLASS` — uma tabela completa por subclasse
```java
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
```
- ✅ Sem JOIN
- ❌ Duplicação de colunas, difícil de consultar polimorficamente

**Recomendação prática:** use `JOINED` quando normalização importa, `SINGLE_TABLE` quando performance é prioridade.

---

## 8. Ciclo de Vida de uma Entidade

```
new Pokemon()          → TRANSIENT    (não existe no banco, não é monitorada)
em.persist(pokemon)    → MANAGED      (monitorada — qualquer mudança vai pro banco no commit)
em.detach(pokemon)     → DETACHED     (foi monitorada, mas não é mais)
em.remove(pokemon)     → REMOVED      (será deletada no commit)
```

Isso é o **contexto de persistência**. Enquanto uma entidade está `MANAGED`, você não precisa chamar `update` — só modificar o objeto já resolve.

```java
EntityManager em = factory.createEntityManager();
em.getTransaction().begin();

Pokemon p = em.find(Pokemon.class, 1); // MANAGED
p.setNivel(p.getNivel() + 1);          // sem em.update() — o Hibernate detecta a mudança

em.getTransaction().commit(); // UPDATE disparado automaticamente
em.close();
```

---

## 9. JPQL — Queries com Objetos

Em vez de SQL com nomes de tabela, você usa **nomes de classe e atributo**:

```java
// SQL normal
"SELECT * FROM veiculo WHERE capacidade_carga > 5000"

// JPQL equivalente
"SELECT v FROM Veiculo v WHERE v.capacidadeCarga > 5000"
```

```java
// Buscar todos os pokémons de nível >= 30
List<Pokemon> lista = em.createQuery(
    "SELECT p FROM Pokemon p WHERE p.nivel >= :nivel", Pokemon.class)
    .setParameter("nivel", 30)
    .getResultList();

// Buscar caminhões por número de eixos
List<Caminhao> caminhoes = em.createQuery(
    "SELECT c FROM Caminhao c WHERE c.numeroEixos = :eixos ORDER BY c.placa", Caminhao.class)
    .setParameter("eixos", 4)
    .getResultList();
```

---

## 10. Lazy vs Eager Loading

```java
// LAZY (padrão em @OneToMany) — a lista só é carregada quando você acessar
@OneToMany(mappedBy = "mochila", fetch = FetchType.LAZY)
private List<Pokemon> pokemons;

// EAGER — carrega junto com a entidade pai
@ManyToOne(fetch = FetchType.EAGER) // padrão em @ManyToOne
private MochilaPokemon mochila;
```

**Regra prática:**
- `@OneToMany` e `@ManyToMany` → LAZY por padrão (correto)
- `@ManyToOne` e `@OneToOne` → EAGER por padrão (geralmente ok)

Cuidado com o **N+1 problem**: buscar 100 pokémons e depois fazer 100 queries para carregar a mochila de cada um. Solução: `JOIN FETCH` no JPQL.

```java
"SELECT p FROM Pokemon p JOIN FETCH p.mochila WHERE p.nivel > 20"
```

---

## 11. Cascading

Controla o que acontece com as entidades filhas quando você opera na pai:

```java
@OneToMany(mappedBy = "mochila", cascade = CascadeType.ALL)
private List<Pokemon> pokemons;
```

| Tipo | O que propaga |
|---|---|
| `PERSIST` | `em.persist(mochila)` também persiste os pokémons |
| `REMOVE` | `em.remove(mochila)` também deleta os pokémons |
| `MERGE` | Merge propaga para os filhos |
| `REFRESH` | Recarrega os filhos do banco |
| `ALL` | Todos os acima |

---

## 12. Paralelo com o GWEB

No GWEB você tem:
```
Controller → BO → DAO → PostgreSQL (JDBC puro)
```

Com JPA seria:
```
Controller → Service → Repository (EntityManager) → PostgreSQL
```

O DAO deixa de ser JDBC manual e passa a ser operações no `EntityManager`. O trabalho de mapeamento `ResultSet → Objeto` desaparece.

```java
// DAO atual no GWEB (exemplo)
public Veiculo buscarPorPlaca(String placa) {
    // 20 linhas de PreparedStatement, ResultSet, try/catch...
}

// Com JPA
public Veiculo buscarPorPlaca(String placa) {
    return em.createQuery("SELECT v FROM Veiculo v WHERE v.placa = :placa", Veiculo.class)
             .setParameter("placa", placa)
             .getSingleResult();
}
```

---

## 13. Exemplo Completo — CRUD de Veículos

```java
public class Main {
    public static void main(String[] args) {
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("meu-projeto");
        EntityManager em = factory.createEntityManager();

        // INSERT
        em.getTransaction().begin();
        Caminhao c1 = new Caminhao();
        c1.setPlaca("ABC1234");
        c1.setCapacidadeCarga(15000.0);
        c1.setNumeroEixos(6);
        em.persist(c1);
        em.getTransaction().commit();

        // SELECT
        Caminhao encontrado = em.find(Caminhao.class, c1.getId());
        System.out.println(encontrado.getPlaca()); // ABC1234

        // UPDATE
        em.getTransaction().begin();
        encontrado.setCapacidadeCarga(18000.0);
        em.getTransaction().commit(); // Hibernate detecta a mudança e faz UPDATE

        // DELETE
        em.getTransaction().begin();
        em.remove(encontrado);
        em.getTransaction().commit();

        em.close();
        factory.close();
    }
}
```

---

## 14. Spring Boot simplifica tudo

Quando você usa Spring Boot (como no projeto Pokémon), o `EntityManager` e as transações são gerenciados pelo framework:

```java
// Sem Spring Boot — você gerencia tudo
EntityManager em = factory.createEntityManager();
em.getTransaction().begin();
em.persist(pokemon);
em.getTransaction().commit();
em.close();

// Com Spring Boot — Spring cuida do ciclo de vida
@Service
public class PokemonService {

    @Autowired
    private PokemonRepository repository; // Spring Data JPA

    @Transactional // Spring abre e fecha a transação pra você
    public Pokemon salvar(Pokemon pokemon) {
        return repository.save(pokemon);
    }
}
```

O `PokemonRepository` estende `JpaRepository<Pokemon, Integer>` e você ganha `save()`, `findById()`, `findAll()`, `delete()` de graça — sem escrever uma linha de JPQL.

---

## Referências

- [Especificação JPA (JSR 338)](https://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf)
- [EntityManager Javadoc](https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html)
- [Hibernate Docs](https://hibernate.org/orm/documentation/)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)