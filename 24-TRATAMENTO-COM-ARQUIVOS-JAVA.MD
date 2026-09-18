# Trabalhando com Arquivos em Java — Guia com Analogias de Logística

---

## Sumário

1. [O mapa mental: arquivos como um centro de distribuição](#1-o-mapa-mental-arquivos-como-um-centro-de-distribuição)
2. [A classe `File` — a etiqueta de endereço](#2-a-classe-file--a-etiqueta-de-endereço)
3. [Lendo com `File` + `Scanner` — o conferente](#3-lendo-com-file--scanner--o-conferente)
4. [`FileReader` + `BufferedReader` — a esteira e o palete](#4-filereader--bufferedreader--a-esteira-e-o-palete)
5. [try-with-resources — a doca que fecha sozinha](#5-try-with-resources--a-doca-que-fecha-sozinha)
6. [`FileWriter` + `BufferedWriter` — expedição de dados](#6-filewriter--bufferedwriter--expedição-de-dados)
7. [Manipulando pastas — organizando o armazém](#7-manipulando-pastas--organizando-o-armazém)
8. [Informações do caminho — lendo o endereço](#8-informações-do-caminho--lendo-o-endereço)
9. [Exercício resolvido — resumo de carga (`summary.csv`)](#9-exercício-resolvido--resumo-de-carga-summarycsv)
10. [Bônus: API moderna `java.nio.file` (`Path` e `Files`)](#10-bônus-api-moderna-javaniofile-path-e-files)
11. [Armadilhas comuns](#11-armadilhas-comuns)
12. [Resumo rápido (cola)](#12-resumo-rápido-cola)

---

## 1. O mapa mental: arquivos como um centro de distribuição

Antes do código, a analogia que vai guiar todo o material:

| Conceito Java | Analogia logística |
|---|---|
| Arquivo no disco | Um **depósito** com mercadorias (dados) |
| Caminho (`C:\temp\in.txt`) | O **endereço** do depósito |
| `File` | A **etiqueta de endereço** — diz *onde* está, mas não carrega nada |
| Stream (`FileReader`, `FileWriter`) | A **esteira** que movimenta as mercadorias uma a uma |
| Buffer (`BufferedReader`, `BufferedWriter`) | O **pallet**: junta várias caixas e move tudo de uma vez |
| `Scanner` | O **conferente**: lê e interpreta o que chega |
| `close()` | **Fechar a doca** ao fim do turno |
| `IOException` | **Ocorrência de transporte**: depósito não existe, acesso negado, disco cheio… |

A regra de ouro de qualquer operação de doca: **abriu, tem que fechar**. Recurso aberto e não fechado é doca aberta a noite inteira — consome recurso do sistema operacional e pode travar o arquivo para outros processos.

---

## 2. A classe `File` — a etiqueta de endereço

`java.io.File` é uma **representação abstrata de um caminho**. Criar um `File` **não abre, não lê e não cria** nada no disco.

```java
File manifesto = new File("C:\\temp\\manifesto.txt");
```

É como escrever o endereço de um depósito numa etiqueta: a etiqueta existe mesmo que o depósito não exista.

```java
File manifesto = new File("C:\\temp\\manifesto.txt");

System.out.println(manifesto.exists());
System.out.println(manifesto.isFile());
System.out.println(manifesto.isDirectory());
System.out.println(manifesto.length());
```

| Método | O que responde |
|---|---|
| `exists()` | O depósito existe nesse endereço? |
| `isFile()` / `isDirectory()` | É um depósito (arquivo) ou um galpão com vários depósitos (pasta)? |
| `length()` | Tamanho em bytes (a "tonelagem" do arquivo) |
| `canRead()` / `canWrite()` | Tenho autorização para entrar/carregar? |

### Sobre o `\\`

Em Java, `\` inicia um caractere de escape (`\n`, `\t`). Para escrever uma barra invertida literal, usa-se `\\`. Em Linux/WSL o separador é `/`:

```java
new File("C:\\temp\\manifesto.txt");
new File("/home/luiz/temp/manifesto.txt");
```

> Dica: `/` também funciona no Windows em Java. Para código portável, prefira `File.separator` ou a API `Path` (seção 10).

---

## 3. Lendo com `File` + `Scanner` — o conferente

O `Scanner` é o conferente da doca de recebimento: pega cada linha do manifesto e entrega pra você.

Arquivo `C:\temp\manifesto.txt`:

```
NF-1001;São Paulo;120.5
NF-1002;Lorena;80.0
NF-1003;Taubaté;215.3
```

```java
package application;

import java.io.File;
import java.io.IOException;
import java.util.Scanner;

public class Program {

    public static void main(String[] args) {
        File file = new File("C:\\temp\\manifesto.txt");
        Scanner sc = null;
        try {
            sc = new Scanner(file);
            while (sc.hasNextLine()) {
                System.out.println(sc.nextLine());
            }
        } catch (IOException e) {
            System.out.println("Erro: " + e.getMessage());
        } finally {
            if (sc != null) {
                sc.close();
            }
        }
    }
}
```

### Passo a passo

1. `new Scanner(file)` → o conferente se posiciona na doca. Se o arquivo não existe, lança `FileNotFoundException` (filha de `IOException`).
2. `hasNextLine()` → "ainda tem caixa na esteira?"
3. `nextLine()` → pega a próxima linha.
4. `finally` → **sempre** executa, dando erro ou não. É ali que a doca é fechada.
5. `if (sc != null)` → se o `Scanner` nem chegou a ser criado (arquivo inexistente), não há o que fechar. Sem essa checagem: `NullPointerException`.

### Quando usar `Scanner`

Bom para leitura **simples e com parsing** (`nextInt()`, `nextDouble()`, delimitadores). É mais lento que `BufferedReader` para arquivos grandes — para um manifesto com 500 mil linhas, prefira a próxima seção.

---

## 4. `FileReader` + `BufferedReader` — a esteira e o palete

- **`FileReader`**: stream de leitura de **caracteres**. É a esteira: traz os dados do disco caractere a caractere.
- **`BufferedReader`**: envolve o `FileReader` e lê **em blocos** para a memória (o buffer). É o palete: em vez de 1.000 viagens com uma caixa cada, faz poucas viagens com o palete cheio.

Acesso a disco é caro. Menos viagens = mais rápido.

```java
package application;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class Program {

    public static void main(String[] args) {
        String path = "C:\\temp\\manifesto.txt";
        BufferedReader br = null;
        FileReader fr = null;

        try {
            fr = new FileReader(path);
            br = new BufferedReader(fr);

            String line = br.readLine();
            while (line != null) {
                System.out.println(line);
                line = br.readLine();
            }
        } catch (IOException e) {
            System.out.println("Erro: " + e.getMessage());
        } finally {
            try {
                if (br != null) br.close();
                if (fr != null) fr.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### Pontos-chave

- `readLine()` retorna `null` quando o arquivo acaba — é o sinal de "esteira vazia".
- Padrão de leitura: **lê antes do loop, processa, lê de novo no fim do loop**.
- O `finally` ficou feio: `close()` também lança `IOException`, então precisa de outro `try/catch` dentro. Esse incômodo é exatamente o que a próxima seção resolve.
- Fechar o `BufferedReader` já fecha o `FileReader` interno — fechar os dois é redundante, mas inofensivo.

### Processando o manifesto de verdade

Ler e imprimir é pouco útil. Na prática, você quebra a linha em campos:

```java
String line = br.readLine();
while (line != null) {
    String[] campos = line.split(";");
    String notaFiscal = campos[0];
    String destino = campos[1];
    double pesoKg = Double.parseDouble(campos[2]);

    System.out.printf("%s -> %s (%.1f kg)%n", notaFiscal, destino, pesoKg);
    line = br.readLine();
}
```

---

## 5. try-with-resources — a doca que fecha sozinha

Disponível desde o **Java 7**. Você declara os recursos entre parênteses no `try`, e o Java **garante o fechamento** ao sair do bloco — com ou sem exceção.

É como uma doca com porta automática: o caminhão saiu, a porta fecha. Ninguém precisa lembrar.

```java
package application;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class Program {

    public static void main(String[] args) {
        String path = "C:\\temp\\manifesto.txt";

        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            String line = br.readLine();
            while (line != null) {
                System.out.println(line);
                line = br.readLine();
            }
        } catch (IOException e) {
            System.out.println("Erro: " + e.getMessage());
        }
    }
}
```

### Comparação direta

| | `finally` manual | try-with-resources |
|---|---|---|
| Linhas de código | Muitas | Poucas |
| Risco de esquecer `close()` | Alto | Zero |
| Checagem de `null` | Obrigatória | Desnecessária |
| Exceção no `close()` | Precisa de `try` aninhado | Tratada automaticamente (vira *suppressed exception*) |

### Múltiplos recursos

Separados por `;`. São fechados na **ordem inversa** da abertura (último a abrir, primeiro a fechar — como descarregar um caminhão):

```java
try (BufferedReader br = new BufferedReader(new FileReader("C:\\temp\\entrada.csv"));
     BufferedWriter bw = new BufferedWriter(new FileWriter("C:\\temp\\saida.csv"))) {
    ...
}
```

### Requisito

Só funciona com classes que implementam `AutoCloseable` (ou `Closeable`). Todas as classes de I/O do Java já implementam. Suas próprias classes também podem:

```java
public class ConexaoRastreador implements AutoCloseable {

    @Override
    public void close() {
        System.out.println("Conexão com rastreador encerrada");
    }
}
```

> **Regra prática:** em código novo, use sempre try-with-resources. O padrão com `finally` aparece aqui para você reconhecer em código legado.

---

## 6. `FileWriter` + `BufferedWriter` — expedição de dados

Agora o fluxo inverso: em vez de receber, você **expede** dados para um arquivo.

- **`FileWriter`**: stream de escrita de caracteres.
- **`BufferedWriter`**: acumula no buffer e grava em blocos (o palete de novo).

### Dois modos de abertura — muito importante

| Construtor | Comportamento | Analogia |
|---|---|---|
| `new FileWriter(path)` | **Cria ou recria** o arquivo (apaga o conteúdo anterior) | Esvazia o depósito e recebe carga nova |
| `new FileWriter(path, true)` | **Acrescenta** ao final (append) | Empilha nova carga sobre a existente |

### Exemplo: gerando o romaneio do dia (recria)

```java
package application;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;

public class Program {

    public static void main(String[] args) {
        String[] entregas = {
            "NF-1001;São Paulo;ENTREGUE",
            "NF-1002;Lorena;EM ROTA",
            "NF-1003;Taubaté;AGUARDANDO"
        };
        String path = "C:\\temp\\romaneio.txt";

        try (BufferedWriter bw = new BufferedWriter(new FileWriter(path))) {
            for (String entrega : entregas) {
                bw.write(entrega);
                bw.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### Exemplo: log de rastreamento (append)

Cada evento de rastreio é **acrescentado** — nunca se apaga o histórico:

```java
public static void registrarEvento(String evento) {
    String path = "C:\\temp\\rastreamento.log";
    try (BufferedWriter bw = new BufferedWriter(new FileWriter(path, true))) {
        bw.write(LocalDateTime.now() + " - " + evento);
        bw.newLine();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

```java
registrarEvento("NF-1002 saiu do CD Lorena");
registrarEvento("NF-1002 chegou ao destino");
```

### Detalhes

- `newLine()` usa a quebra de linha do sistema (`\r\n` no Windows, `\n` no Linux). Mais portável que escrever `"\n"` na mão.
- Os dados ficam no buffer até ele encher ou até o `close()`. Se o programa morrer antes do `close()`, **dados no buffer se perdem**. Por isso try-with-resources é essencial na escrita. Para forçar gravação no meio do processo: `bw.flush()`.
- `FileWriter` **não cria pastas**. Se `C:\temp` não existir, lança `IOException`. Crie a pasta antes (próxima seção).

---

## 7. Manipulando pastas — organizando o armazém

Pastas são **galpões**: contêm depósitos (arquivos) e outros galpões (subpastas). A classe `File` também representa pastas.

```java
package application;

import java.io.File;
import java.util.Scanner;

public class Program {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.println("Informe o caminho do galpão (pasta): ");
        String strPath = sc.nextLine();

        File path = new File(strPath);

        File[] folders = path.listFiles(File::isDirectory);
        System.out.println("SUBPASTAS:");
        for (File folder : folders) {
            System.out.println(folder);
        }

        File[] files = path.listFiles(File::isFile);
        System.out.println("ARQUIVOS:");
        for (File file : files) {
            System.out.println(file);
        }

        boolean success = new File(strPath + "\\expedicao").mkdir();
        System.out.println("Pasta criada: " + success);

        sc.close();
    }
}
```

### Entendendo `listFiles(File::isDirectory)`

`listFiles` aceita um **filtro**. `File::isDirectory` é uma *method reference* — equivale a:

```java
path.listFiles(f -> f.isDirectory());
```

Ou seja: "de tudo que tem no galpão, me traga só o que for subgalpão". O mesmo vale para `File::isFile`.

Filtrando só os CSVs de notas fiscais:

```java
File[] csvs = path.listFiles(f -> f.isFile() && f.getName().endsWith(".csv"));
```

### `mkdir()` vs `mkdirs()`

| Método | Comportamento |
|---|---|
| `mkdir()` | Cria **uma** pasta. Falha (`false`) se a pasta-pai não existe ou se ela já existe |
| `mkdirs()` | Cria a pasta **e todas as pastas-pai** que faltarem |

```java
new File("C:\\temp\\cd-lorena\\2026\\setembro").mkdirs();
```

Com `mkdir()`, isso falharia se `cd-lorena` ou `2026` não existissem.

> Atenção: `mkdir()`/`mkdirs()` retornam `boolean`, **não lançam exceção**. Se não checar o retorno, a falha passa em silêncio.

### Cuidado: `listFiles()` pode retornar `null`

Se o caminho não existe ou não é pasta, `listFiles()` retorna `null` — e o `for` explode com `NullPointerException`. Em código real:

```java
File[] files = path.listFiles(File::isFile);
if (files == null) {
    System.out.println("Caminho inválido ou não é uma pasta");
    return;
}
```

---

## 8. Informações do caminho — lendo o endereço

Dado um endereço completo, você pode extrair partes dele — como ler rua, cidade e CEP de uma etiqueta.

```java
File path = new File("C:\\temp\\cd-lorena\\manifesto.csv");

System.out.println("getPath: " + path.getPath());
System.out.println("getParent: " + path.getParent());
System.out.println("getName: " + path.getName());
System.out.println("getAbsolutePath: " + path.getAbsolutePath());
```

Saída:

```
getPath: C:\temp\cd-lorena\manifesto.csv
getParent: C:\temp\cd-lorena
getName: manifesto.csv
getAbsolutePath: C:\temp\cd-lorena\manifesto.csv
```

| Método | Retorna | Analogia |
|---|---|---|
| `getPath()` | O caminho como foi informado | O endereço escrito na etiqueta |
| `getParent()` | A pasta que contém o arquivo | O galpão onde fica o depósito |
| `getName()` | Só o nome do arquivo/pasta | O número do depósito |
| `getAbsolutePath()` | Caminho completo a partir da raiz | Endereço completo com CEP |

### Por que `getParent()` importa

É a chave do exercício: a partir do arquivo de origem, você descobre a pasta dele e cria uma subpasta **ao lado**:

```java
String pastaOrigem = new File(caminhoArquivo).getParent();
new File(pastaOrigem + "\\out").mkdir();
```

### Caminho relativo vs absoluto

- **Absoluto**: `C:\temp\in.txt` — funciona de qualquer lugar.
- **Relativo**: `dados\in.txt` — relativo à pasta onde o programa foi **executado** (no Eclipse/IntelliJ, geralmente a raiz do projeto). Com caminho relativo, `getParent()` pode retornar `null` se não houver pasta no caminho informado.

---

## 9. Exercício resolvido — resumo de carga (`summary.csv`)

### Enunciado (adaptado para logística)

Um CD recebe um `.csv` com os itens de uma carga: **nome, preço unitário e quantidade**, separados por vírgula. O programa deve ler o caminho desse arquivo e gerar `summary.csv` em uma subpasta `out` **dentro da pasta do arquivo de origem**, contendo apenas **nome e valor total** (preço × quantidade).

Arquivo de origem (`C:\temp\carga.csv`):

```
Pallet Madeira PBR,89.90,40
Filme Stretch 500m,45.50,12
Etiqueta Térmica 100x150,0.35,5000
Leitor Código de Barras,650.00,2
```

Saída esperada (`C:\temp\out\summary.csv`):

```
Pallet Madeira PBR,3596.00
Filme Stretch 500m,546.00
Etiqueta Térmica 100x150,1750.00
Leitor Código de Barras,1300.00
```

### Fluxo da solução

```
[ler caminho] → [ler CSV linha a linha] → [split(",")] → [criar ItemCarga]
      → [lista em memória] → [criar pasta out] → [gravar summary.csv]
```

Analogia: **recebimento → conferência → armazenagem temporária → expedição**. Primeiro você recebe e confere tudo; só depois expede. Separar leitura e escrita deixa o código mais fácil de testar e evita arquivo de saída pela metade se a leitura falhar no meio.

### `entities/ItemCarga.java`

```java
package entities;

public class ItemCarga {

    private String nome;
    private double precoUnitario;
    private int quantidade;

    public ItemCarga(String nome, double precoUnitario, int quantidade) {
        this.nome = nome;
        this.precoUnitario = precoUnitario;
        this.quantidade = quantidade;
    }

    public String getNome() {
        return nome;
    }

    public double getPrecoUnitario() {
        return precoUnitario;
    }

    public int getQuantidade() {
        return quantidade;
    }

    public double total() {
        return precoUnitario * quantidade;
    }
}
```

### `application/Program.java`

```java
package application;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Locale;
import java.util.Scanner;

import entities.ItemCarga;

public class Program {

    public static void main(String[] args) {
        Locale.setDefault(Locale.US);
        Scanner sc = new Scanner(System.in);

        System.out.println("Caminho do arquivo de carga: ");
        String sourceFileStr = sc.nextLine();

        File sourceFile = new File(sourceFileStr);
        String sourceFolderStr = sourceFile.getParent();

        File outFolder = new File(sourceFolderStr + File.separator + "out");
        outFolder.mkdir();

        String targetFileStr = outFolder.getPath() + File.separator + "summary.csv";

        List<ItemCarga> itens = new ArrayList<>();

        try (BufferedReader br = new BufferedReader(new FileReader(sourceFileStr))) {
            String line = br.readLine();
            while (line != null) {
                String[] fields = line.split(",");
                String nome = fields[0];
                double preco = Double.parseDouble(fields[1]);
                int quantidade = Integer.parseInt(fields[2]);

                itens.add(new ItemCarga(nome, preco, quantidade));
                line = br.readLine();
            }

            try (BufferedWriter bw = new BufferedWriter(new FileWriter(targetFileStr))) {
                for (ItemCarga item : itens) {
                    bw.write(item.getNome() + "," + String.format("%.2f", item.total()));
                    bw.newLine();
                }
                System.out.println(targetFileStr + " CRIADO!");
            }
        } catch (IOException e) {
            System.out.println("Erro de I/O: " + e.getMessage());
        } catch (NumberFormatException | ArrayIndexOutOfBoundsException e) {
            System.out.println("Linha mal formatada no CSV: " + e.getMessage());
        }

        sc.close();
    }
}
```

### Decisões importantes

**1. `Locale.setDefault(Locale.US)` — pegadinha brasileira.**
Com o Windows em pt-BR, `String.format("%.2f", 3596.0)` produz `3596,00` (vírgula). Num CSV separado por vírgula, isso **quebra a coluna** em duas: `Pallet Madeira PBR,3596,00`. Forçando `Locale.US`, sai `3596.00`. Alternativa mais cirúrgica, sem mudar o locale global:

```java
String.format(Locale.US, "%.2f", item.total())
```

**2. `File.separator` em vez de `"\\"`.**
Funciona no Windows e no Linux/WSL sem mudar código.

**3. `getParent()` define onde fica o `out`.**
Se o usuário informar `C:\temp\carga.csv`, a pasta é `C:\temp`, e o resumo vai para `C:\temp\out\summary.csv`.

**4. Tratamento de linha mal formatada.**
Uma linha com campo faltando (`Pallet PBR,89.90`) gera `ArrayIndexOutOfBoundsException`; um preço inválido (`abc`) gera `NumberFormatException`. Capturando os dois, o programa não termina com stack trace na cara do usuário.

### Variante: ignorar linhas ruins e seguir

Em logística real, uma NF com problema não pode travar o processamento das outras 9.999. Versão que registra e continua:

```java
int numeroLinha = 0;
String line = br.readLine();
while (line != null) {
    numeroLinha++;
    try {
        String[] fields = line.split(",");
        itens.add(new ItemCarga(
                fields[0].trim(),
                Double.parseDouble(fields[1].trim()),
                Integer.parseInt(fields[2].trim())));
    } catch (NumberFormatException | ArrayIndexOutOfBoundsException e) {
        System.out.println("Linha " + numeroLinha + " ignorada: " + line);
    }
    line = br.readLine();
}
```

---

## 10. Bônus: API moderna `java.nio.file` (`Path` e `Files`)

O curso usa `java.io` (a API clássica). Desde o **Java 7** existe `java.nio.file`, que é a recomendada em código novo e em produção. Vale conhecer as duas: `java.io` domina código legado; `nio` é mais concisa e tem erros mais descritivos.

| Clássico (`java.io`) | Moderno (`java.nio.file`) |
|---|---|
| `new File("a/b.txt")` | `Path.of("a", "b.txt")` |
| `file.exists()` | `Files.exists(path)` |
| `file.getParent()` | `path.getParent()` |
| `new File(...).mkdirs()` → `boolean` | `Files.createDirectories(path)` → lança exceção se falhar |
| `BufferedReader` + loop | `Files.readAllLines(path)` ou `Files.lines(path)` |
| `BufferedWriter` + loop | `Files.write(path, linhas)` |

### O exercício da seção 9 em versão moderna

```java
package application;

import java.io.IOException;
import java.io.UncheckedIOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;
import java.util.Locale;
import java.util.Scanner;
import java.util.stream.Stream;

import entities.ItemCarga;

public class Program {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.println("Caminho do arquivo de carga: ");
        Path source = Path.of(sc.nextLine());
        sc.close();

        Path outDir = source.toAbsolutePath().getParent().resolve("out");
        Path target = outDir.resolve("summary.csv");

        try (Stream<String> lines = Files.lines(source, StandardCharsets.UTF_8)) {
            List<String> resumo = lines
                    .map(line -> line.split(","))
                    .map(f -> new ItemCarga(f[0], Double.parseDouble(f[1]), Integer.parseInt(f[2])))
                    .map(i -> i.getNome() + "," + String.format(Locale.US, "%.2f", i.total()))
                    .toList();

            Files.createDirectories(outDir);
            Files.write(target, resumo, StandardCharsets.UTF_8);
            System.out.println(target + " CRIADO!");
        } catch (IOException | UncheckedIOException e) {
            System.out.println("Erro de I/O: " + e.getMessage());
        }
    }
}
```

Destaques:

- `resolve("out")` monta o caminho com o separador correto do SO — sem `\\` nem `File.separator`.
- `toAbsolutePath()` antes de `getParent()` evita o `null` com caminho relativo.
- `Files.lines` é **lazy** (lê sob demanda) e precisa de try-with-resources. Bom para arquivos grandes.
- `Files.readAllLines` carrega tudo em memória — ok para arquivos pequenos, perigoso para um log de rastreamento de 2 GB.
- `.toList()` é Java 16+. Em Java 8–15: `.collect(Collectors.toList())`.

### Charset: sempre explícito

`FileReader`/`FileWriter` sem charset usam o padrão do SO (no Windows antigo, `windows-1252`; no Linux, `UTF-8`; no Java 18+, UTF-8 por padrão). Resultado clássico: `Taubaté` vira `TaubatÃ©`. Especifique sempre:

```java
new FileReader(path, StandardCharsets.UTF_8);
new FileWriter(path, StandardCharsets.UTF_8, true);
Files.newBufferedReader(path, StandardCharsets.UTF_8);
```

Se o arquivo vem de sistema legado em ISO-8859-1, use `StandardCharsets.ISO_8859_1` — o charset tem que bater com o de quem **gerou** o arquivo.

---

## 11. Armadilhas comuns

| Sintoma | Causa | Correção |
|---|---|---|
| `FileNotFoundException` na leitura | Caminho errado ou arquivo inexistente | Conferir caminho; checar `exists()` antes |
| `FileNotFoundException` na **escrita** | Pasta de destino não existe | `mkdirs()` / `Files.createDirectories()` antes |
| `NullPointerException` no `for` de `listFiles()` | Caminho não é pasta ou não existe | Checar `null` antes de iterar |
| Arquivo de saída vazio ou cortado | Buffer não foi descarregado (sem `close()`/`flush()`) | try-with-resources |
| Dados antigos sumiram | `new FileWriter(path)` recria o arquivo | `new FileWriter(path, true)` para append |
| CSV com colunas a mais (`3596,00`) | `String.format` usando locale pt-BR | `String.format(Locale.US, ...)` |
| `Ã©`, `Ã§` no lugar de acentos | Charset de leitura ≠ charset do arquivo | Charset explícito e correto |
| `mkdir()` "não fez nada" | Pasta-pai não existe ou pasta já existe (retorna `false` sem exceção) | `mkdirs()` e checar retorno |
| `getParent()` retorna `null` | Caminho relativo sem pasta (`carga.csv`) | `getAbsoluteFile().getParent()` |
| Arquivo travado no Windows ("em uso por outro processo") | Stream não foi fechado | try-with-resources |

---

## 12. Resumo rápido (cola)

```java
// Ler linha a linha
try (BufferedReader br = new BufferedReader(new FileReader(path, StandardCharsets.UTF_8))) {
    String line;
    while ((line = br.readLine()) != null) { ... }
}

// Escrever (recria)
try (BufferedWriter bw = new BufferedWriter(new FileWriter(path, StandardCharsets.UTF_8))) {
    bw.write("..."); bw.newLine();
}

// Escrever (append)
new FileWriter(path, StandardCharsets.UTF_8, true)

// Pastas
new File(dir).mkdirs();
new File(dir).listFiles(File::isFile);

// Partes do caminho
file.getParent(); file.getName(); file.getAbsolutePath();

// NIO
Path p = Path.of("C:", "temp", "carga.csv");
List<String> linhas = Files.readAllLines(p, StandardCharsets.UTF_8);
Files.createDirectories(p.getParent().resolve("out"));
Files.write(p.getParent().resolve("out").resolve("summary.csv"), linhas, StandardCharsets.UTF_8);
```

| Preciso de… | Use |
|---|---|
| Representar um caminho | `File` (legado) / `Path` (moderno) |
| Ler com parsing de tipos | `Scanner` |
| Ler texto rápido, linha a linha | `BufferedReader` |
| Escrever texto | `BufferedWriter` |
| Acrescentar ao final | `FileWriter(path, true)` |
| Garantir fechamento | try-with-resources |
| Criar pastas aninhadas | `mkdirs()` / `Files.createDirectories()` |
| Código novo em produção | `java.nio.file.Files` + charset explícito |

---

**Referências**

- [Oracle — `java.io.File`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/File.html)
- [Oracle — try-with-resources](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)
- [Oracle — `java.nio.file.Files`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/nio/file/Files.html)
- [Repositório do exercício original (acenelio/files1-java)](https://github.com/acenelio/files1-java)