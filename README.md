<div align="center">

# ⚔️ League of Code

### Meu caderno de estudos de programação — do `if` ao `Virtual Thread`

![Java](https://img.shields.io/badge/Java-17-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Go](https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Hibernate](https://img.shields.io/badge/JPA-Hibernate-59666C?style=for-the-badge&logo=hibernate&logoColor=white)
![Git](https://img.shields.io/badge/Git-GitHub-F05032?style=for-the-badge&logo=git&logoColor=white)

![Conteúdo](https://img.shields.io/badge/anota%C3%A7%C3%B5es-41%20arquivos-blue?style=flat-square)
![Idioma](https://img.shields.io/badge/idioma-PT--BR-green?style=flat-square)
![Formato](https://img.shields.io/badge/formato-Markdown-lightgrey?style=flat-square&logo=markdown)
![Status](https://img.shields.io/badge/status-em%20evolu%C3%A7%C3%A3o-orange?style=flat-square)

</div>

---

## 📖 Sobre

Repositório de **anotações e guias de estudo** feitos durante meus estudos de Java (curso Java Udemy e formação de logística/backend), com pitadas de JavaScript, Go e SQL.

O material usa analogias para facilitar o aprendizado:

- 🎮 **League of Legends / Pokémon** — fundamentos e POO, com campeões, runas, Pokédex e teamfights.
- 🚚 **Logística** — da metade do repositório em diante, com frota, fretes, entregas e armazéns como domínio dos exemplos.

---

## 🗺️ Mapa da Jornada

```mermaid
flowchart LR
    A[🧭 Fundamentos] --> B[🧱 POO]
    B --> C[🛡️ Exceções e Arquivos]
    C --> D[📦 Collections e Generics]
    D --> E[⚡ Funcional e Concorrência]
    E --> F[🌐 API, CRUD e Banco]
    F --> G[🚀 Java Moderno e Debug]
```

---

## 📚 Índice de Conteúdos

| Trilha | Arquivo | O que aborda |
|--------|---------|--------------|
| 🧭 **Fundamentos** | [01-INTRO](01-INTRO.MD) | 🔍 Leitura de código: checklist, script mental de 7 passos, red flags e frases para code review |
| 🧭 **Fundamentos** | [02-JAVA](02-JAVA.MD) | ☕ Operadores em Java: comparativos, lógicos, especiais, estáticos e guard clauses |
| 🧭 **Fundamentos** | [03-JAVASCRIPT](03-JAVASCRIPT.MD) | 🟨 Operadores em JS: game loop, `typeof`, precedência, código defensivo |
| 🧭 **Fundamentos** | [04-GO](04-GO.MD) | 🐹 Operadores em Go: structs, ponteiros, retorno de erro, bitwise, type assertion |
| 🧭 **Fundamentos** | [SQL](SQL.MD) | 🐘 Operadores em PostgreSQL: `JOIN`, agregações, `WHERE` vs `HAVING`, `CASE` |
| 🧭 **Fundamentos** | [09-ESTRUTURA CONDICIONAL](09-ESTRUTURA%20CONDICIONAL.MD) | 🔀 `if/else`, `switch`, ternário, escopo e exercício de runas |
| 🧭 **Fundamentos** | [10-ESTRUTURAS-CONTROLE-JAVA](10-ESTRUTURAS-CONTROLE-JAVA.MD) | 🔁 `if`, `switch`, `for`, `while`, `do-while`, padrões e erros comuns |
| 🧱 **POO** | [06-JAVA-POKEMON](06-JAVA-POKEMON.MD) | 🔴 Visão geral de POO com Pokémon: herança, polimorfismo, abstração, composição e SOLID |
| 🧱 **POO** | [07-CONSTRUTORES-JAVA](07-CONSTRUTORES-JAVA.MD) | 🏗️ Construtores, `this`, sobrecarga, modificadores de acesso, Conta Bancária e Pokédex |
| 🧱 **POO** | [08-ENCAPSULAMENTO-JAVA](08-ENCAPSULAMENTO-JAVA.MD) | 🔒 Getters/setters, imutabilidade, Builder, `record` e anti-padrões |
| 🧱 **POO** | [11-POO-OBJETOS-STRINGS-ARRAYS](11-POO-OBJETOS-STRINGS-ARRAYS.MD) | 🧩 Instanciação, referências, Strings e arrays |
| 🧱 **POO** | [12-MEMORIA-ARRAYS-LISTAS](12-MEMORIA-ARRAYS-LISTAS.MD) | 🧠 Stack/Heap, `null`, Garbage Collector, boxing, `ArrayList` e matrizes |
| 🧱 **POO** | [12-HERANCA-JAVA](12-HERANCA-JAVA.MD) | 🧬 `extends`, `super`, herança multinível e por que Java não tem herança múltipla |
| 🧱 **POO** | [12-POLIMORFISMO-JAVA](12-POLIMORFISMO-JAVA.MD) | 🎭 Override, referência genérica, downcasting e interfaces |
| 🧱 **POO** | [13-INTERFACES-JAVA](13-INTERFACES-JAVA.MD) | 📜 Contratos, métodos `default`, interface vs classe abstrata |
| 🧱 **POO** | [16-CASTING-ENUMS-JAVA](16-CASTING-ENUMS-JAVA.MD) | 🔄 Casting, pattern matching e `switch` com enum |
| 🧱 **POO** | [17-ENUMS-JAVA](17-ENUMS-JAVA.MD) | 🏷️ Enums com propriedades, composição e exercício de pedidos |
| 🧱 **POO** | [19-CLASSES-ABSTRATAS-JAVA](19-CLASSES-ABSTRATAS-JAVA.MD) | 🧪 Métodos abstratos, *Template Method*, abstract vs interface |
| 🧱 **POO** | [20-HERANCA-E-POLIMORFISMO-JAVA](20-HERANCA-E-POLIMORFISMO-JAVA.MD) | 🧬 Up/downcasting, `final`, classes abstratas e hierarquia de contas bancárias |
| 🧱 **POO** | [21-CLASSES-E-METODOS-JAVA](21-CLASSES-E-METODOS-JAVA.MD) | 🚛 Visibilidade, retornos, estáticos, overloading e `@Override` |
| 🧱 **POO** | [25-INTERFACES-JAVA](25-INTERFACES-JAVA.MD) | 🔌 Baixo acoplamento, DI, `Comparable`, problema do diamante, `default`/`static`/`private` |
| 🛡️ **Exceções, Datas e Arquivos** | [14-EXCECOES-JAVA](14-EXCECOES-JAVA.MD) | 💥 `try/catch/finally`, `throws`, tipos e exceções personalizadas |
| 🛡️ **Exceções, Datas e Arquivos** | [22-EXCEPTIONS-JAVA](22-EXCEPTIONS-JAVA.MD) | 🚨 Call stack, `throw` vs `throws`, exceção com dados extras |
| 🛡️ **Exceções, Datas e Arquivos** | [23-TRY-CATCH-JAVA](23-TRY-CATCH-JAVA.MD) | 🪤 Múltiplos `catch`, ordem, hierarquia, `catch` vazio, try-catch vs if/else |
| 🛡️ **Exceções, Datas e Arquivos** | [18-DATA-HORA-JAVA](18-DATA-HORA-JAVA.MD) | 🕒 `LocalDate`, `Instant`, timezone, ISO 8601, `Duration` e legado |
| 🛡️ **Exceções, Datas e Arquivos** | [24-TRATAMENTO-COM-ARQUIVOS-JAVA](24-TRATAMENTO-COM-ARQUIVOS-JAVA.MD) | 📁 `File`, `Scanner`, `BufferedReader/Writer`, try-with-resources, `java.nio.file` |
| 📦 **Collections e Generics** | [15-COLLECTIONS-JAVA](15-COLLECTIONS-JAVA.MD) | 🗃️ `List`, `Set`, `Map`, `Queue` e performance |
| 📦 **Collections e Generics** | [26-GENERICS-JAVA](26-GENERICS-JAVA.MD) | 🧬 Bounded types, wildcards, covariância, get/put, *type erasure* |
| 📦 **Collections e Generics** | [27-SET-JAVA](27-SET-JAVA.MD) | 🔵 `HashSet`, `TreeSet`, `LinkedHashSet`, `hashCode`/`equals`, operações de conjunto |
| 📦 **Collections e Generics** | [28-MAP-JAVA](28-MAP-JAVA.MD) | 🗺️ `HashMap`, `TreeMap`, `LinkedHashMap`, iteração e métodos Java 8+ |
| ⚡ **Funcional e Concorrência** | [29-PROGRAMACAO-FUNCIONAL-JAVA](29-PROGRAMACAO-FUNCIONAL-JAVA.MD) | λ Lambdas, interfaces funcionais, `Stream`, `Collectors`, `Optional` |
| ⚡ **Funcional e Concorrência** | [30-THREADS-CONCORRENCIA-JAVA](30-THREADS-CONCORRENCIA-JAVA.MD) | 🧵 Race condition, `synchronized`, `Lock`, deadlock, `ExecutorService`, `CompletableFuture`, virtual threads |
| ⚡ **Funcional e Concorrência** | [31-PARALELISMO-JAVA](31-PARALELISMO-JAVA.MD) | 🚀 `parallelStream`, `ForkJoinPool`, `RecursiveTask`, quando paralelizar |
| ⚡ **Funcional e Concorrência** | [32-IDEMPOTENCIA-JAVA](32-IDEMPOTENCIA-JAVA.MD) | ♻️ Chaves de idempotência, upsert, constraints únicas, filas *at-least-once* |
| 🌐 **APIs e Banco de Dados** | [33-API-JAVA](33-API-JAVA.MD) | 🔗 Camadas, HTTP, status codes, JSON, validação, paginação, versionamento |
| 🌐 **APIs e Banco de Dados** | [34-CRUD-JAVA](34-CRUD-JAVA.MD) | 🛠️ Repository, soft delete, transações, REST, *optimistic locking* |
| 🌐 **APIs e Banco de Dados** | [35-JDBC-JAVA](35-JDBC-JAVA.MD) | 🔌 `PreparedStatement`, SQL Injection, transações, batch, HikariCP, padrão DAO |
| 🌐 **APIs e Banco de Dados** | [37-JPA-HIBERNATE](37-JPA-HIBERNATE.MD) | 🗄️ Mapeamento ORM, anotações, herança, ciclo de vida, JPQL, lazy/eager, cascade |
| 🚀 **Ferramentas e Java Moderno** | [36-GIT-GITHUB](36-GIT-GITHUB.MD) | 🌿 SSH, `checkout`, `.gitignore`, cherry-pick, commits semânticos, fluxo até o PR |
| 🚀 **Ferramentas e Java Moderno** | [38-JAVA-17](38-JAVA-17.MD) | ✨ Java 8 → 17: records, text blocks, mudanças de API/JVM e refatoração de legado |
| 🚀 **Ferramentas e Java Moderno** | [39-DEBUGANDO-JAVA](39-DEBUGANDO-JAVA.md) | 🐞 Debug remoto, breakpoints, stack traces, logger vs `System.out`, streams, profiling |

---

## 💡 Como estudar

1. 🧭 Comece pelos **Fundamentos** para firmar lógica e operadores.
2. 🧱 Avance em **POO** — é a base de tudo que vem depois.
3. 🔁 Cada guia termina com **resumo, exercícios e/ou cheat sheet** — use-os para revisar.
4. 🧪 Reescreva os exemplos na sua IDE, não apenas leia.

---

<div align="center">

Desenvolvido por[**luizfxdev**](https://www.linkedin.com/in/luizfxdev/)

</div>
