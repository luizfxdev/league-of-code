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

O material usa **analogias para facilitar o aprendizado** e segue dois "universos":

| Universo | Onde aparece | Ideia |
|----------|--------------|-------|
| 🎮 **League of Legends / Pokémon** | Trilha 1 e 2 (fundamentos e POO) | Campeões, runas, Pokédex e teamfights para explicar conceitos |
| 🚚 **Logística** | Trilha 3 em diante | Frota, fretes, entregas e armazéns como domínio dos exemplos |

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

## 🧭 Trilha 1 — Fundamentos e Multilinguagem

| # | Arquivo | Tema | O que aborda |
|---|---------|------|--------------|
| 01 | [01-INTRO](01-INTRO.MD) | 🔍 Leitura de código | Checklist antes/durante/depois, script mental de 7 passos, red flags e frases prontas para code review |
| 02 | [02-JAVA](02-JAVA.MD) | ☕ Operadores em Java | Comparativos, lógicos, especiais, estáticos e guard clauses |
| 03 | [03-JAVASCRIPT](03-JAVASCRIPT.MD) | 🟨 Operadores em JS | Game loop, `typeof`, precedência, código defensivo |
| 04 | [04-GO](04-GO.MD) | 🐹 Operadores em Go | Structs, ponteiros, múltiplos retornos de erro, bitwise, type assertion |
| — | [SQL](SQL.MD) | 🐘 Operadores em PostgreSQL | Comparação, lógicos, `JOIN`, agregações, `WHERE` vs `HAVING`, `CASE` |
| 09 | [09-ESTRUTURA CONDICIONAL](09-ESTRUTURA%20CONDICIONAL.MD) | 🔀 Condicionais | `if/else`, `switch`, ternário, escopo e exercício de runas |
| 10 | [10-ESTRUTURAS-CONTROLE-JAVA](10-ESTRUTURAS-CONTROLE-JAVA.MD) | 🔁 Estruturas de controle | `if`, `switch`, `for`, `while`, `do-while`, padrões e erros comuns |

---

## 🧱 Trilha 2 — Programação Orientada a Objetos

| # | Arquivo | Tema | O que aborda |
|---|---------|------|--------------|
| 06 | [06-JAVA-POKEMON](06-JAVA-POKEMON.MD) | 🔴 POO com Pokémon | Visão geral: classes, herança, polimorfismo, abstração, composição e SOLID |
| 07 | [07-CONSTRUTORES-JAVA](07-CONSTRUTORES-JAVA.MD) | 🏗️ Construtores | `this`, sobrecarga, modificadores de acesso, Conta Bancária e Pokédex |
| 08 | [08-ENCAPSULAMENTO-JAVA](08-ENCAPSULAMENTO-JAVA.MD) | 🔒 Encapsulamento | Getters/setters, imutabilidade, Builder, `record` e anti-padrões |
| 11 | [11-POO-OBJETOS-STRINGS-ARRAYS](11-POO-OBJETOS-STRINGS-ARRAYS.MD) | 🧩 Objetos, Strings e Arrays | Instanciação, referências, concatenação e arrays |
| 12 | [12-MEMORIA-ARRAYS-LISTAS](12-MEMORIA-ARRAYS-LISTAS.MD) | 🧠 Memória | Stack/Heap, `null`, Garbage Collector, boxing, `ArrayList` e matrizes |
| 12 | [12-HERANCA-JAVA](12-HERANCA-JAVA.MD) | 🧬 Herança | `extends`, `super`, multinível, sem herança múltipla |
| 12 | [12-POLIMORFISMO-JAVA](12-POLIMORFISMO-JAVA.MD) | 🎭 Polimorfismo | Override, referência genérica, downcasting, interfaces |
| 13 | [13-INTERFACES-JAVA](13-INTERFACES-JAVA.MD) | 📜 Interfaces (LoL) | Contratos, métodos `default`, interface vs classe abstrata |
| 16 | [16-CASTING-ENUMS-JAVA](16-CASTING-ENUMS-JAVA.MD) | 🔄 Casting e Enums | Conversão de tipos, pattern matching, `switch` com enum |
| 17 | [17-ENUMS-JAVA](17-ENUMS-JAVA.MD) | 🏷️ Enumerações | Enums com propriedades, composição e exercício de pedidos |
| 19 | [19-CLASSES-ABSTRATAS-JAVA](19-CLASSES-ABSTRATAS-JAVA.MD) | 🧪 Classes abstratas | Métodos abstratos, *Template Method*, abstract vs interface |
| 20 | [20-HERANCA-E-POLIMORFISMO-JAVA](20-HERANCA-E-POLIMORFISMO-JAVA.MD) | 🧬🎭 Herança + Polimorfismo | Up/downcasting, `final`, classes abstratas, contas bancárias |
| 21 | [21-CLASSES-E-METODOS-JAVA](21-CLASSES-E-METODOS-JAVA.MD) | 🚛 Classes e Métodos | Visibilidade, retornos, estáticos, overloading, `@Override` (logística) |
| 25 | [25-INTERFACES-JAVA](25-INTERFACES-JAVA.MD) | 🔌 Interfaces (logística) | Baixo acoplamento, DI, `Comparable`, problema do diamante, `default`/`static`/`private` |

---

## 🛡️ Trilha 3 — Exceções, Datas e Arquivos

| # | Arquivo | Tema | O que aborda |
|---|---------|------|--------------|
| 14 | [14-EXCECOES-JAVA](14-EXCECOES-JAVA.MD) | 💥 Exceções (LoL) | `try/catch/finally`, `throws`, tipos e exceções personalizadas |
| 22 | [22-EXCEPTIONS-JAVA](22-EXCEPTIONS-JAVA.MD) | 🚨 Exceções (logística) | Call stack, `throw` vs `throws`, exceção com dados extras |
| 23 | [23-TRY-CATCH-JAVA](23-TRY-CATCH-JAVA.MD) | 🪤 `try-catch` | Múltiplos `catch`, ordem, hierarquia, `catch` vazio, try-catch vs if/else |
| 18 | [18-DATA-HORA-JAVA](18-DATA-HORA-JAVA.MD) | 🕒 Data e Hora | `LocalDate`, `Instant`, timezone, ISO 8601, `Duration` e legado |
| 24 | [24-TRATAMENTO-COM-ARQUIVOS-JAVA](24-TRATAMENTO-COM-ARQUIVOS-JAVA.MD) | 📁 Arquivos | `File`, `Scanner`, `BufferedReader/Writer`, try-with-resources, `java.nio.file` |

---

## 📦 Trilha 4 — Collections e Generics

| # | Arquivo | Tema | O que aborda |
|---|---------|------|--------------|
| 15 | [15-COLLECTIONS-JAVA](15-COLLECTIONS-JAVA.MD) | 🗃️ Collections | Visão geral de `List`, `Set`, `Map`, `Queue` e performance |
| 26 | [26-GENERICS-JAVA](26-GENERICS-JAVA.MD) | 🧬 Generics | Bounded types, wildcards, covariância, get/put, *type erasure* |
| 27 | [27-SET-JAVA](27-SET-JAVA.MD) | 🔵 Set | `HashSet`, `TreeSet`, `LinkedHashSet`, `hashCode`/`equals`, operações de conjunto |
| 28 | [28-MAP-JAVA](28-MAP-JAVA.MD) | 🗺️ Map | `HashMap`, `TreeMap`, `LinkedHashMap`, iteração e métodos Java 8+ |

---

## ⚡ Trilha 5 — Funcional, Concorrência e Resiliência

| # | Arquivo | Tema | O que aborda |
|---|---------|------|--------------|
| 29 | [29-PROGRAMACAO-FUNCIONAL-JAVA](29-PROGRAMACAO-FUNCIONAL-JAVA.MD) | λ Funcional | Lambdas, interfaces funcionais, `Stream`, `Collectors`, `Optional` |
| 30 | [30-THREADS-CONCORRENCIA-JAVA](30-THREADS-CONCORRENCIA-JAVA.MD) | 🧵 Threads | Race condition, `synchronized`, `Lock`, deadlock, `ExecutorService`, `CompletableFuture`, virtual threads |
| 31 | [31-PARALELISMO-JAVA](31-PARALELISMO-JAVA.MD) | 🚀 Paralelismo | `parallelStream`, `ForkJoinPool`, `RecursiveTask`, quando paralelizar |
| 32 | [32-IDEMPOTENCIA-JAVA](32-IDEMPOTENCIA-JAVA.MD) | ♻️ Idempotência | Chaves de idempotência, upsert, constraints únicas, filas *at-least-once* |

---

## 🌐 Trilha 6 — APIs, Banco de Dados e Persistência

| # | Arquivo | Tema | O que aborda |
|---|---------|------|--------------|
| 33 | [33-API-JAVA](33-API-JAVA.MD) | 🔗 APIs | Camadas, HTTP, status codes, JSON, validação, paginação, versionamento |
| 34 | [34-CRUD-JAVA](34-CRUD-JAVA.MD) | 🛠️ CRUD | Repository, soft delete, transações, REST, *optimistic locking* |
| 35 | [35-JDBC-JAVA](35-JDBC-JAVA.MD) | 🔌 JDBC | `PreparedStatement`, SQL Injection, transações, batch, HikariCP, padrão DAO |
| 37 | [37-JPA-HIBERNATE](37-JPA-HIBERNATE.MD) | 🗄️ JPA / Hibernate | Mapeamento ORM, anotações, herança, ciclo de vida, JPQL, lazy/eager, cascade |

---

## 🚀 Trilha 7 — Ferramentas e Java Moderno

| # | Arquivo | Tema | O que aborda |
|---|---------|------|--------------|
| 36 | [36-GIT-GITHUB](36-GIT-GITHUB.MD) | 🌿 Git e GitHub | SSH, ciclo de vida de arquivos, `checkout`, `.gitignore`, cherry-pick, commits semânticos, fluxo até o PR |
| 38 | [38-JAVA-17](38-JAVA-17.MD) | ✨ Java 8 → 17 | Records, text blocks, mudanças de API/JVM, refatoração de legado e checklist |
| 39 | [39-DEBUGANDO-JAVA](39-DEBUGANDO-JAVA.md) | 🐞 Debug | Debug remoto, breakpoints, stack traces, logger vs `System.out`, streams, profiling |

---

## 🎯 Resumo Rápido por Tecnologia

| Tecnologia | Arquivos |
|:----------:|----------|
| ☕ **Java (core)** | 02, 06–13, 15–17, 19–28 |
| 🟨 **JavaScript** | 03 |
| 🐹 **Go** | 04 |
| 🐘 **SQL / PostgreSQL** | SQL, 35 |
| 🗄️ **Persistência** | 34, 35, 37 |
| ⚡ **Concorrência e Funcional** | 29–32 |
| 🌐 **Backend / API** | 32, 33, 34 |
| 🛠️ **Ferramentas** | 36, 38, 39 |

---

## 💡 Como estudar

1. 🧭 Comece pela **Trilha 1** para firmar lógica e operadores.
2. 🧱 Avance em **POO** — é a base de tudo que vem depois.
3. 🔁 Cada guia termina com **resumo, exercícios e/ou cheat sheet** — use-os para revisar.
4. 🧪 Reescreva os exemplos na sua IDE, não apenas leia.

---

<div align="center">

Feito com ☕ e muitos teamfights por [**luizfxdev**](https://github.com/luizfxdev)

</div>
