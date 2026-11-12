# Construtores, This, Sobrecarga e Encapsulamento em Java

Aprofundamento em conceitos fundamentais de POO aplicados a um sistema de logística, usando **Java 17** como referência.

---

## 1. Construtores

### O que é um Construtor?

Método especial que **executa automaticamente** quando você cria um novo objeto com `new`. Responsável por **inicializar o estado** do objeto.

**Analogia logística:**
- Quando uma encomenda entra no sistema, ela não nasce vazia
- Já tem código de rastreio, destino, peso e status definidos no momento do registro
- O construtor é esse processo de registro inicial

### Problema: Objetos Vazios

```java
// ❌ Sem construtor customizado
public class Encomenda {
    public String codigoRastreio;
    public String destino;
    public double pesoKg;
}

Encomenda enc = new Encomenda();          // Objeto vazio
System.out.println(enc.codigoRastreio);   // null
System.out.println(enc.destino);          // null
System.out.println(enc.pesoKg);           // 0.0

// Problema: faz sentido uma encomenda sem destino?
// Um objeto inconsistente vai quebrar o cálculo de frete mais adiante
```

### Solução: Construtor Customizado

```java
// ✅ Com construtor
public class Encomenda {
    private String codigoRastreio;
    private String destino;
    private double pesoKg;

    // Construtor: força inicializar código, destino e peso
    public Encomenda(String codigoRastreio, String destino, double pesoKg) {
        this.codigoRastreio = codigoRastreio;
        this.destino = destino;
        this.pesoKg = pesoKg;
    }
}

Encomenda enc = new Encomenda("BR123456789", "Recife-PE", 2.5);
// Agora o objeto SEMPRE nasce consistente
```

### Construtor Padrão (Default)

Se você não cria um construtor customizado, Java fornece um vazio automaticamente:

```java
public class Encomenda {
    private String codigoRastreio;
}

// Java fornece implicitamente:
// public Encomenda() { }

Encomenda e = new Encomenda();  // Funciona
```

**Mas:** Se você criar um construtor customizado, o padrão desaparece:

```java
public class Encomenda {
    private String codigoRastreio;

    public Encomenda(String codigoRastreio) {
        this.codigoRastreio = codigoRastreio;
    }

    // O construtor padrão NÃO existe mais!
}

Encomenda e1 = new Encomenda("BR123456789");  // ✅ Funciona
Encomenda e2 = new Encomenda();               // ❌ ERRO de compilação!
```

**Solução:** Forneça o construtor padrão explicitamente:

```java
public class Encomenda {
    private String codigoRastreio;

    // Construtor padrão
    public Encomenda() {
        this.codigoRastreio = gerarCodigo();
    }

    // Construtor customizado
    public Encomenda(String codigoRastreio) {
        this.codigoRastreio = codigoRastreio;
    }

    private static String gerarCodigo() {
        return "BR" + System.currentTimeMillis();
    }
}

Encomenda e1 = new Encomenda();
Encomenda e2 = new Encomenda("BR123456789");
```

### Fluxo de Construção

```
Encomenda enc = new Encomenda("BR123456789", "Recife-PE", 2.5);

1. new Encomenda(...)  → Aloca memória na heap
2. Executa construtor  → Inicializa atributos
3. Retorna referência  → enc aponta para o objeto
```

### Validação no Construtor (Java 17)

Um construtor deve **rejeitar dados inválidos** para que o objeto nunca exista num estado quebrado. Java 17 oferece `Objects.requireNonNull` para isso.

```java
import java.util.Objects;

public class Encomenda {
    private final String codigoRastreio;
    private final String destino;
    private final double pesoKg;

    public Encomenda(String codigoRastreio, String destino, double pesoKg) {
        // requireNonNull lança NullPointerException com mensagem clara
        this.codigoRastreio = Objects.requireNonNull(codigoRastreio, "Código de rastreio obrigatório");
        this.destino = Objects.requireNonNull(destino, "Destino obrigatório");

        if (destino.isBlank()) {
            throw new IllegalArgumentException("Destino não pode ser vazio");
        }
        if (pesoKg <= 0) {
            throw new IllegalArgumentException("Peso deve ser positivo, recebido: " + pesoKg);
        }

        this.pesoKg = pesoKg;
    }
}

Encomenda e1 = new Encomenda("BR123456789", "Recife-PE", 2.5);  // ✅ OK
Encomenda e2 = new Encomenda("BR987654321", "", 1.0);           // ❌ IllegalArgumentException
Encomenda e3 = new Encomenda(null, "Olinda-PE", 1.0);           // ❌ NullPointerException
```

> **Regra:** falhe cedo (*fail fast*). É muito mais barato detectar o erro na criação do objeto do que três camadas adiante, quando o frete já foi cobrado errado.

---

## 2. A Palavra `this`

### O que é?

Referência ao **próprio objeto**. Torna explícita a diferença entre atributos e variáveis locais.

**Analogia logística:**
- `this` = a própria encomenda falando sobre si mesma
- "Meu destino" = `this.destino`
- "Meu peso" = `this.pesoKg`

### Uso 1: Diferenciar Atributos de Parâmetros

```java
public class Encomenda {
    private String destino;
    private double pesoKg;

    // ❌ Confuso: qual é qual?
    public void inicializar1(String destino, double pesoKg) {
        destino = destino;   // Atribui o parâmetro a ele mesmo — não faz nada!
        pesoKg = pesoKg;     // Bug silencioso
    }

    // ✅ Claro: this diferencia
    public void inicializar2(String destino, double pesoKg) {
        this.destino = destino;   // this.destino (atributo) = destino (parâmetro)
        this.pesoKg = pesoKg;
    }
}
```

### Uso 2: Em Construtores

```java
public class Encomenda {
    private String codigoRastreio;
    private String destino;
    private double pesoKg;

    public Encomenda(String codigoRastreio, String destino, double pesoKg) {
        this.codigoRastreio = codigoRastreio;
        this.destino = destino;
        this.pesoKg = pesoKg;
    }
}

// Stack (parâmetros)           Heap (objeto)
// codigoRastreio ("BR123") →   this.codigoRastreio = "BR123"
// destino ("Recife-PE")    →   this.destino = "Recife-PE"
// pesoKg (2.5)             →   this.pesoKg = 2.5
```

### Uso 3: Chamar Outro Construtor (Constructor Chaining)

```java
public class Encomenda {
    private String codigoRastreio;
    private String destino;
    private double pesoKg;
    private String status;

    // Construtor completo — faz o trabalho real e a validação
    public Encomenda(String codigoRastreio, String destino, double pesoKg, String status) {
        this.codigoRastreio = codigoRastreio;
        this.destino = destino;
        this.pesoKg = pesoKg;
        this.status = status;
    }

    // Construtor simplificado chama o completo com this()
    public Encomenda(String codigoRastreio, String destino) {
        this(codigoRastreio, destino, 1.0, "PENDENTE");
    }

    // Construtor padrão
    public Encomenda() {
        this("SEM-CODIGO", "NÃO INFORMADO");
    }
}

Encomenda e1 = new Encomenda();                                              // padrão
Encomenda e2 = new Encomenda("BR123456789", "Recife-PE");                    // simplificado
Encomenda e3 = new Encomenda("BR123456789", "Recife-PE", 2.5, "EM_TRANSITO"); // completo
```

**Vantagem:** Evita repetição de código

```java
// ❌ Repetição (errado)
public Encomenda(String codigoRastreio, String destino) {
    this.codigoRastreio = codigoRastreio;
    this.destino = destino;
    this.pesoKg = 1.0;
    this.status = "PENDENTE";
}

public Encomenda() {
    this.codigoRastreio = "SEM-CODIGO";
    this.destino = "NÃO INFORMADO";
    this.pesoKg = 1.0;
    this.status = "PENDENTE";
}

// ✅ Sem repetição (certo)
public Encomenda(String codigoRastreio, String destino, double pesoKg, String status) {
    this.codigoRastreio = codigoRastreio;
    this.destino = destino;
    this.pesoKg = pesoKg;
    this.status = status;
}

public Encomenda(String codigoRastreio, String destino) {
    this(codigoRastreio, destino, 1.0, "PENDENTE");
}

public Encomenda() {
    this("SEM-CODIGO", "NÃO INFORMADO");
}
```

> **Regra do `this()`:** a chamada precisa ser a **primeira instrução** do construtor. Centralize a validação no construtor mais completo — todos os outros passam por ele.

### Uso 4: Passar o Objeto como Argumento

```java
public class Rota {
    private Encomenda origem;
    private Encomenda destino;

    public Rota(Encomenda origem, Encomenda destino) {
        this.origem = origem;    // Guarda a referência
        this.destino = destino;
    }
}

public class Encomenda {
    // A encomenda passa uma referência a si mesma
    public Rota criarRotaPara(Encomenda outra) {
        return new Rota(this, outra);
    }
}

Encomenda e1 = new Encomenda("BR111", "Recife-PE", 2.5, "PENDENTE");
Encomenda e2 = new Encomenda("BR222", "Olinda-PE", 1.2, "PENDENTE");
Rota rota = e1.criarRotaPara(e2);
```

### Regra: Quando usar `this`?

**Use `this` quando:**
- Há ambiguidade entre atributo e parâmetro (OBRIGATÓRIO em construtores)
- Quer passar o objeto para outro método
- Quer chamar outro construtor da classe

```java
public class Veiculo {
    private double cargaAtualKg;
    private double capacidadeMaximaKg;

    public Veiculo(double capacidadeMaximaKg) {
        this.capacidadeMaximaKg = capacidadeMaximaKg;  // NECESSÁRIO (sem this, ambíguo)
        this.cargaAtualKg = 0;                         // BOM (deixa claro que é atributo)
    }

    public void carregar(double pesoKg) {
        this.cargaAtualKg += pesoKg;   // BOM (claro)
    }

    public double espacoLivre() {
        return this.capacidadeMaximaKg - this.cargaAtualKg;
    }
}
```

---

## 3. Sobrecarga (Overloading)

### O que é?

Múltiplos métodos/construtores com o **mesmo nome**, mas **listas de parâmetros diferentes**.

**Analogia logística:**
- Uma encomenda pode ser despachada de formas diferentes
- `despachar()` — despacho padrão, modalidade econômica
- `despachar(modalidade)` — especifica a modalidade
- `despachar(modalidade, transportadora)` — especifica modalidade e transportadora

### Sobrecarga de Construtores

```java
public class Encomenda {
    private String codigoRastreio;
    private String destino;
    private double pesoKg;
    private String status;

    // Construtor 1: sem parâmetros
    public Encomenda() {
        this.codigoRastreio = "SEM-CODIGO";
        this.destino = "NÃO INFORMADO";
        this.pesoKg = 1.0;
        this.status = "PENDENTE";
    }

    // Construtor 2: só código
    public Encomenda(String codigoRastreio) {
        this.codigoRastreio = codigoRastreio;
        this.destino = "NÃO INFORMADO";
        this.pesoKg = 1.0;
        this.status = "PENDENTE";
    }

    // Construtor 3: código e destino
    public Encomenda(String codigoRastreio, String destino) {
        this.codigoRastreio = codigoRastreio;
        this.destino = destino;
        this.pesoKg = 1.0;
        this.status = "PENDENTE";
    }

    // Construtor 4: completo
    public Encomenda(String codigoRastreio, String destino, double pesoKg, String status) {
        this.codigoRastreio = codigoRastreio;
        this.destino = destino;
        this.pesoKg = pesoKg;
        this.status = status;
    }
}

Encomenda e1 = new Encomenda();
Encomenda e2 = new Encomenda("BR123456789");
Encomenda e3 = new Encomenda("BR123456789", "Recife-PE");
Encomenda e4 = new Encomenda("BR123456789", "Recife-PE", 2.5, "EM_TRANSITO");
```

**Problema:** Repetição de código — os valores padrão `1.0` e `"PENDENTE"` aparecem três vezes. Se o padrão mudar, você precisa alterar em três lugares.

**Solução:** Constructor Chaining com `this()`

```java
// ✅ Bom: chamadas encadeadas, padrão definido em um lugar só
public Encomenda() {
    this("SEM-CODIGO", "NÃO INFORMADO", 1.0, "PENDENTE");
}

public Encomenda(String codigoRastreio) {
    this(codigoRastreio, "NÃO INFORMADO", 1.0, "PENDENTE");
}

public Encomenda(String codigoRastreio, String destino) {
    this(codigoRastreio, destino, 1.0, "PENDENTE");
}

public Encomenda(String codigoRastreio, String destino, double pesoKg, String status) {
    this.codigoRastreio = codigoRastreio;
    this.destino = destino;
    this.pesoKg = pesoKg;
    this.status = status;
}
```

### Sobrecarga de Métodos

```java
public class Encomenda {
    private double pesoKg;
    private String status;

    // Método 1: despacho padrão (modalidade econômica)
    public double despachar() {
        return calcularFrete(1.0);
    }

    // Método 2: despacho com multiplicador de modalidade
    public double despachar(double multiplicadorModalidade) {
        return calcularFrete(multiplicadorModalidade);
    }

    // Método 3: despacho por nome de modalidade
    public double despachar(String modalidade) {
        double multiplicador = switch (modalidade) {   // switch expression (Java 14+)
            case "ECONOMICO" -> 1.0;
            case "EXPRESSO"  -> 1.8;
            case "SEDEX_10"  -> 2.5;
            default -> throw new IllegalArgumentException("Modalidade inválida: " + modalidade);
        };
        return calcularFrete(multiplicador);
    }

    // Método 4: despacho com modalidade e transportadora
    public double despachar(String modalidade, String transportadora) {
        double frete = despachar(modalidade);
        double taxaTransportadora = transportadora.equals("CORREIOS") ? 0 : 8.50;
        return frete + taxaTransportadora;
    }

    private double calcularFrete(double multiplicador) {
        this.status = "EM_TRANSITO";
        return (10.0 + this.pesoKg * 2.5) * multiplicador;
    }
}

Encomenda enc = new Encomenda("BR123456789", "Recife-PE", 4.0, "PENDENTE");

enc.despachar();                          // Tipo 1 — R$ 20,00
enc.despachar(1.8);                       // Tipo 2 — R$ 36,00
enc.despachar("EXPRESSO");                // Tipo 3 — R$ 36,00
enc.despachar("EXPRESSO", "JADLOG");      // Tipo 4 — R$ 44,50
```

### Regras da Sobrecarga

**Deve variar:**
- ✅ Número de parâmetros
- ✅ Tipo de parâmetros
- ✅ Ordem de parâmetros

**Não basta:**
- ❌ Tipo de retorno (não diferencia a assinatura)

```java
// ❌ ERRO: mesmo número e tipo de parâmetros
public void registrarEntrega(double valor) { }
public double registrarEntrega(double valor) { }  // Ambíguo!

// ✅ OK: número diferente
public void registrarEntrega(double valor) { }
public void registrarEntrega(double valor, String observacao) { }

// ✅ OK: tipo diferente
public void registrarEntrega(double valor) { }
public void registrarEntrega(int valor) { }

// ✅ OK: ordem diferente
public void agendar(String motorista, int prioridade) { }
public void agendar(int prioridade, String motorista) { }
```

> **Cuidado com a ambiguidade:** `despachar(1.0)` e `despachar("EXPRESSO")` convivem bem porque os tipos são distintos. Mas evite sobrecargas que só diferem entre `int` e `long` ou `float` e `double` — o compilador resolve por promoção numérica e o resultado pode surpreender.

---

## 4. Encapsulamento

### O Princípio

**Esconder detalhes de implementação**, expondo apenas operações seguras que mantêm o objeto **consistente**.

**Analogia logística:**
- Um veículo da frota é um encapsulamento
- Você não pode simplesmente escrever um valor qualquer no hodômetro
- Você só pode fazer operações seguras: carregar, descarregar, registrar viagem
- O veículo internamente controla sua consistência (não aceita carga acima da capacidade)

### O Problema do Acesso Público

```java
// ❌ Atributos públicos (INSEGURO)
public class Veiculo {
    public String placa;
    public double capacidadeMaximaKg;
    public double cargaAtualKg;
}

Veiculo v = new Veiculo();
v.placa = "ABC1D23";
v.capacidadeMaximaKg = 3500;
v.cargaAtualKg = 1000;

// Qualquer um pode fazer isso:
v.cargaAtualKg = 99999;   // ❌ Excede a capacidade — caminhão sobrecarregado!
v.cargaAtualKg = -500;    // ❌ Carga negativa não existe
v.placa = "XXX0X00";      // ❌ Trocar a placa de um veículo já cadastrado?!

// Objeto em estado inválido — e o sistema de rotas vai confiar nesses dados
```

### A Solução: Getters e Setters

```java
// ✅ Atributos privados com controle
public class Veiculo {
    private final String placa;          // final: definido no construtor, nunca muda
    private final double capacidadeMaximaKg;
    private double cargaAtualKg;
    private double hodometroKm;

    public Veiculo(String placa, double capacidadeMaximaKg) {
        this.placa = Objects.requireNonNull(placa, "Placa obrigatória");
        if (capacidadeMaximaKg <= 0) {
            throw new IllegalArgumentException("Capacidade deve ser positiva");
        }
        this.capacidadeMaximaKg = capacidadeMaximaKg;
        this.cargaAtualKg = 0;
        this.hodometroKm = 0;
    }

    // Getters: leitura segura
    public String getPlaca() {
        return this.placa;
    }

    public double getCapacidadeMaximaKg() {
        return this.capacidadeMaximaKg;
    }

    public double getCargaAtualKg() {
        return this.cargaAtualKg;
    }

    public double getHodometroKm() {
        return this.hodometroKm;
    }

    // Operações seguras para modificar a carga
    public void carregar(double pesoKg) {
        if (pesoKg <= 0) {
            throw new IllegalArgumentException("Peso deve ser positivo");
        }
        if (this.cargaAtualKg + pesoKg > this.capacidadeMaximaKg) {
            throw new IllegalStateException(
                "Capacidade excedida: tentou carregar %.1fkg, espaço livre %.1fkg"
                    .formatted(pesoKg, espacoLivreKg())
            );
        }
        this.cargaAtualKg += pesoKg;
    }

    public void descarregar(double pesoKg) {
        if (pesoKg <= 0 || pesoKg > this.cargaAtualKg) {
            throw new IllegalArgumentException("Peso inválido para descarga");
        }
        this.cargaAtualKg -= pesoKg;
    }

    // Hodômetro só anda para frente — nunca tem setter
    public void registrarViagem(double distanciaKm) {
        if (distanciaKm <= 0) {
            throw new IllegalArgumentException("Distância deve ser positiva");
        }
        this.hodometroKm += distanciaKm;
    }

    // Getter com lógica (valor calculado, não armazenado)
    public double espacoLivreKg() {
        return this.capacidadeMaximaKg - this.cargaAtualKg;
    }

    public boolean estaCheio() {
        return this.cargaAtualKg >= this.capacidadeMaximaKg;
    }
}

Veiculo v = new Veiculo("ABC1D23", 3500);
v.carregar(1000);
v.carregar(2000);
System.out.println(v.espacoLivreKg());   // 500.0

v.carregar(1000);   // ❌ IllegalStateException: Capacidade excedida
// A regra de negócio mora dentro do objeto — ninguém consegue burlar
```

### Padrão JavaBeans

Convenção de nomenclatura em Java:

```
Atributo          Getter              Setter
-----------       -----------------   -----------------
placa             getPlaca()          setPlaca()
pesoKg            getPesoKg()         setPesoKg()
disponivel        isDisponivel()      setDisponivel()
```

```java
public class Motorista {
    private String cpf;
    private String nome;
    private boolean disponivel;

    public String getCpf() {
        return this.cpf;
    }

    public String getNome() {
        return this.nome;
    }

    public void setNome(String nome) {
        if (nome != null && !nome.isBlank()) {
            this.nome = nome;
        }
    }

    // Booleano usa "is", não "get"
    public boolean isDisponivel() {
        return this.disponivel;
    }

    public void setDisponivel(boolean disponivel) {
        this.disponivel = disponivel;
    }
}
```

### Quando Não Permitir Setter

```java
public class Encomenda {
    private final String codigoRastreio;   // Identidade — nunca muda
    private final LocalDateTime dataRegistro;
    private String destino;                // Pode ser corrigido antes do despacho
    private String status;

    public Encomenda(String codigoRastreio, String destino) {
        this.codigoRastreio = codigoRastreio;
        this.destino = destino;
        this.dataRegistro = LocalDateTime.now();
        this.status = "PENDENTE";
    }

    public String getCodigoRastreio() {
        return this.codigoRastreio;
    }
    // ❌ codigoRastreio não tem setter (imutável — é a identidade da encomenda)

    public LocalDateTime getDataRegistro() {
        return this.dataRegistro;
    }
    // ❌ dataRegistro não tem setter (fato histórico, não se reescreve)

    public String getDestino() {
        return this.destino;
    }

    // ✅ Destino tem setter, mas com regra de negócio
    public void setDestino(String destino) {
        if (!this.status.equals("PENDENTE")) {
            throw new IllegalStateException("Destino não pode mudar após o despacho");
        }
        this.destino = Objects.requireNonNull(destino);
    }

    public String getStatus() {
        return this.status;
    }
    // ❌ status não tem setter — muda só através de métodos de transição
    public void despachar() {
        if (!this.status.equals("PENDENTE")) {
            throw new IllegalStateException("Só encomendas pendentes podem ser despachadas");
        }
        this.status = "EM_TRANSITO";
    }
}
```

> **Princípio:** um setter público para cada atributo é encapsulamento de fachada. O ganho real vem de expor **operações do domínio** (`despachar()`, `carregar()`) em vez de setters genéricos.

### Cuidado: Vazamento de Referência

Devolver uma coleção interna diretamente quebra o encapsulamento — quem recebe pode modificá-la.

```java
public class Rota {
    private final List<Encomenda> encomendas = new ArrayList<>();

    // ❌ Ruim: expõe a lista interna
    public List<Encomenda> getEncomendasInseguro() {
        return this.encomendas;
    }

    // ✅ Bom: devolve uma visão não modificável
    public List<Encomenda> getEncomendas() {
        return Collections.unmodifiableList(this.encomendas);
    }

    // Alterações passam por um método que valida
    public void adicionar(Encomenda e) {
        if (this.encomendas.size() >= 50) {
            throw new IllegalStateException("Rota lotada");
        }
        this.encomendas.add(e);
    }
}

Rota rota = new Rota();
rota.getEncomendasInseguro().clear();   // ❌ Esvazia a rota por fora!
rota.getEncomendas().clear();           // ✅ UnsupportedOperationException
```

---

## 5. Records: Construtores em Java 17

**Record** (Java 16+) é uma forma concisa de declarar classes **imutáveis** que existem para carregar dados. O compilador gera construtor, getters, `equals`, `hashCode` e `toString` automaticamente.

### Classe Tradicional vs Record

```java
// Forma tradicional — ~40 linhas de código repetitivo
public final class Endereco {
    private final String logradouro;
    private final String cidade;
    private final String uf;
    private final String cep;

    public Endereco(String logradouro, String cidade, String uf, String cep) {
        this.logradouro = logradouro;
        this.cidade = cidade;
        this.uf = uf;
        this.cep = cep;
    }

    public String getLogradouro() { return logradouro; }
    public String getCidade() { return cidade; }
    public String getUf() { return uf; }
    public String getCep() { return cep; }

    @Override
    public boolean equals(Object o) { /* ... 8 linhas ... */ }

    @Override
    public int hashCode() { /* ... */ }

    @Override
    public String toString() { /* ... */ }
}
```

```java
// Record — uma linha, mesmo comportamento
public record Endereco(String logradouro, String cidade, String uf, String cep) { }

Endereco end = new Endereco("Av. Boa Viagem, 100", "Recife", "PE", "51011-000");
System.out.println(end.cidade());   // Recife  (acessor sem "get")
System.out.println(end);            // Endereco[logradouro=Av. Boa Viagem, 100, cidade=Recife, ...]
```

### Construtor Compacto: Validação em Records

O **construtor compacto** valida e normaliza os parâmetros antes de a atribuição acontecer.

```java
public record Encomenda(String codigoRastreio, Endereco destino, double pesoKg) {

    // Construtor compacto — sem lista de parâmetros, sem atribuições
    public Encomenda {
        Objects.requireNonNull(codigoRastreio, "Código de rastreio obrigatório");
        Objects.requireNonNull(destino, "Destino obrigatório");

        if (!codigoRastreio.matches("BR\\d{9}")) {
            throw new IllegalArgumentException("Código inválido: " + codigoRastreio);
        }
        if (pesoKg <= 0 || pesoKg > 30) {
            throw new IllegalArgumentException("Peso deve estar entre 0 e 30kg, recebido: " + pesoKg);
        }

        // Normalização: o valor ajustado é o que será atribuído ao campo
        codigoRastreio = codigoRastreio.toUpperCase();
    }

    // Records podem ter métodos de negócio
    public double calcularFrete() {
        return 10.0 + this.pesoKg * 2.5;
    }

    public boolean ehPesada() {
        return this.pesoKg > 10;
    }
}

var destino = new Endereco("Av. Boa Viagem, 100", "Recife", "PE", "51011-000");
var enc = new Encomenda("br123456789", destino, 4.0);

System.out.println(enc.codigoRastreio());  // BR123456789 (normalizado)
System.out.println(enc.calcularFrete());   // 20.0

new Encomenda("INVALIDO", destino, 4.0);   // ❌ IllegalArgumentException
new Encomenda("BR123456789", destino, 50); // ❌ IllegalArgumentException
```

### Construtores Sobrecarregados em Records

```java
public record Encomenda(String codigoRastreio, Endereco destino, double pesoKg, String status) {

    // Construtor compacto (canônico) — validação central
    public Encomenda {
        Objects.requireNonNull(codigoRastreio);
        Objects.requireNonNull(destino);
        if (pesoKg <= 0) {
            throw new IllegalArgumentException("Peso deve ser positivo");
        }
    }

    // Sobrecarga: status padrão
    public Encomenda(String codigoRastreio, Endereco destino, double pesoKg) {
        this(codigoRastreio, destino, pesoKg, "PENDENTE");
    }

    // Sobrecarga: peso e status padrão
    public Encomenda(String codigoRastreio, Endereco destino) {
        this(codigoRastreio, destino, 1.0, "PENDENTE");
    }
}
```

### Quando Usar Record vs Classe

| Use **record** quando | Use **classe** quando |
|---|---|
| O objeto é imutável | O estado precisa mudar ao longo do tempo |
| Existe para transportar dados (DTO, value object) | Tem ciclo de vida e comportamento rico |
| Identidade = valor dos campos | Identidade = referência ou um ID |
| Ex.: `Endereco`, `Coordenada`, `FaixaCep` | Ex.: `Veiculo`, `Rota`, `Motorista` |

```java
// ✅ Record: valor imutável
public record Coordenada(double latitude, double longitude) {
    public Coordenada {
        if (latitude < -90 || latitude > 90) {
            throw new IllegalArgumentException("Latitude fora do intervalo");
        }
        if (longitude < -180 || longitude > 180) {
            throw new IllegalArgumentException("Longitude fora do intervalo");
        }
    }
}

// ✅ Classe: estado muda, tem comportamento
public class Veiculo {
    private final String placa;
    private Coordenada posicaoAtual;   // muda a cada atualização de GPS
    private double cargaAtualKg;       // muda a cada carga/descarga
    // ...
}
```

---

## 6. Modificadores de Acesso

### Hierarquia de Visibilidade

```
private  <  (padrão)  <  protected  <  public
Mais restritivo ......................... Menos restritivo
```

### Tabela Completa

| Modificador | Mesma Classe | Mesmo Pacote | Subclasse (outro pacote) | Outro Pacote |
|-----------|--------------|-------------|------------------------|------------|
| `private` | ✅ | ❌ | ❌ | ❌ |
| (padrão / package-private) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

### Private — Máxima Proteção

```java
// entities/Encomenda.java
package entities;

public class Encomenda {
    private String codigoRastreio;   // Só Encomenda acessa
    private double pesoKg;

    public Encomenda(String codigoRastreio, double pesoKg) {
        this.codigoRastreio = codigoRastreio;
        this.pesoKg = pesoKg;
    }

    public String getCodigoRastreio() {
        return this.codigoRastreio;
    }
}

// application/Program.java
package application;
import entities.Encomenda;

public class Program {
    public static void main(String[] args) {
        Encomenda e = new Encomenda("BR123456789", 2.5);
        System.out.println(e.getCodigoRastreio());  // ✅ OK (getter público)
        System.out.println(e.codigoRastreio);       // ❌ ERRO (private)
        e.pesoKg = 50;                              // ❌ ERRO (private)
    }
}
```

### Protected — Herança

```java
// entities/ItemTransportavel.java
package entities;

public abstract class ItemTransportavel {
    protected String codigo;      // Subclasses acessam
    protected double pesoKg;

    protected ItemTransportavel(String codigo, double pesoKg) {
        this.codigo = codigo;
        this.pesoKg = pesoKg;
    }

    public abstract double calcularFrete();
}

// entities/Encomenda.java
package entities;

public class Encomenda extends ItemTransportavel {

    public Encomenda(String codigo, double pesoKg) {
        super(codigo, pesoKg);   // super() chama o construtor da classe pai
    }

    @Override
    public double calcularFrete() {
        return 10.0 + this.pesoKg * 2.5;   // ✅ Acessa protected da superclasse
    }
}
```

### `super()` vs `this()` em Construtores

```java
public class Encomenda extends ItemTransportavel {
    private String destino;

    // super() chama o construtor da CLASSE PAI
    public Encomenda(String codigo, double pesoKg, String destino) {
        super(codigo, pesoKg);     // Primeiro: inicializa a parte herdada
        this.destino = destino;    // Depois: inicializa a parte própria
    }

    // this() chama OUTRO CONSTRUTOR DA MESMA CLASSE
    public Encomenda(String codigo, double pesoKg) {
        this(codigo, pesoKg, "NÃO INFORMADO");
    }
}
```

| | `this(...)` | `super(...)` |
|---|---|---|
| Chama | Outro construtor da mesma classe | Construtor da classe pai |
| Posição | Primeira instrução | Primeira instrução |
| Juntos? | ❌ Não podem coexistir no mesmo construtor | |
| Implícito | Não | Sim — se você não escrever, Java insere `super()` |

### Public — Interface da Classe

```java
public class CalculadoraFrete {
    // Constante pública: faz parte do contrato
    public static final double TAXA_BASE = 10.0;

    // Método público: qualquer um chama
    public double calcular(double pesoKg, double distanciaKm) {
        return TAXA_BASE + custoPorPeso(pesoKg) + custoPorDistancia(distanciaKm);
    }

    // Detalhe de implementação: fica privado
    private double custoPorPeso(double pesoKg) {
        return pesoKg * 2.5;
    }

    private double custoPorDistancia(double km) {
        return km * 0.8;
    }
}
```

> **Regra prática:** comece tudo `private`. Promova para `public` apenas o que outra classe realmente precisa chamar. É fácil tornar algo público depois; é muito difícil tornar privado algo que meio mundo já usa.

---

## 7. Exemplo Prático Completo: Veículo da Frota

```java
package entities;

import java.util.Objects;

public class Veiculo {
    private final String placa;
    private final String modelo;
    private final double capacidadeMaximaKg;
    private double cargaAtualKg;
    private double hodometroKm;
    private boolean disponivel;

    private static final double CONSUMO_MEDIO_KM_POR_LITRO = 8.5;
    private static final double PRECO_LITRO = 6.20;

    // Construtor: sem modelo informado
    public Veiculo(String placa, double capacidadeMaximaKg) {
        this(placa, "NÃO INFORMADO", capacidadeMaximaKg);
    }

    // Construtor canônico — toda validação mora aqui
    public Veiculo(String placa, String modelo, double capacidadeMaximaKg) {
        this.placa = Objects.requireNonNull(placa, "Placa obrigatória");
        this.modelo = Objects.requireNonNull(modelo, "Modelo obrigatório");

        if (!placa.matches("[A-Z]{3}[0-9][A-Z0-9][0-9]{2}")) {
            throw new IllegalArgumentException("Placa fora do padrão Mercosul: " + placa);
        }
        if (capacidadeMaximaKg <= 0) {
            throw new IllegalArgumentException("Capacidade deve ser positiva");
        }

        this.capacidadeMaximaKg = capacidadeMaximaKg;
        this.cargaAtualKg = 0;
        this.hodometroKm = 0;
        this.disponivel = true;
    }

    // Getters
    public String getPlaca() { return this.placa; }
    public String getModelo() { return this.modelo; }
    public double getCapacidadeMaximaKg() { return this.capacidadeMaximaKg; }
    public double getCargaAtualKg() { return this.cargaAtualKg; }
    public double getHodometroKm() { return this.hodometroKm; }
    public boolean isDisponivel() { return this.disponivel; }

    // Operações do domínio
    public void carregar(double pesoKg) {
        if (pesoKg <= 0) {
            throw new IllegalArgumentException("Peso deve ser positivo");
        }
        if (this.cargaAtualKg + pesoKg > this.capacidadeMaximaKg) {
            throw new IllegalStateException(
                "Capacidade excedida: carga %.1fkg + %.1fkg > %.1fkg"
                    .formatted(this.cargaAtualKg, pesoKg, this.capacidadeMaximaKg)
            );
        }
        this.cargaAtualKg += pesoKg;
    }

    public void descarregar() {
        this.cargaAtualKg = 0;
    }

    public void registrarViagem(double distanciaKm) {
        if (distanciaKm <= 0) {
            throw new IllegalArgumentException("Distância deve ser positiva");
        }
        this.hodometroKm += distanciaKm;
    }

    // Valores calculados — não ocupam campo
    public double espacoLivreKg() {
        return this.capacidadeMaximaKg - this.cargaAtualKg;
    }

    public double percentualOcupacao() {
        return (this.cargaAtualKg / this.capacidadeMaximaKg) * 100;
    }

    public double custoCombustivel(double distanciaKm) {
        return (distanciaKm / CONSUMO_MEDIO_KM_POR_LITRO) * PRECO_LITRO;
    }

    @Override
    public String toString() {
        return "Veículo %s (%s) | Carga: %.1f/%.1fkg (%.0f%%) | Hodômetro: %.0fkm"
            .formatted(this.placa, this.modelo, this.cargaAtualKg,
                       this.capacidadeMaximaKg, percentualOcupacao(), this.hodometroKm);
    }
}
```

```java
// ===== USO =====
package application;

import java.util.Locale;
import java.util.Scanner;
import entities.Veiculo;

public class Program {
    public static void main(String[] args) {
        Locale.setDefault(Locale.US);
        var sc = new Scanner(System.in);

        System.out.print("Placa do veículo: ");
        String placa = sc.nextLine().toUpperCase();

        System.out.print("Modelo: ");
        String modelo = sc.nextLine();

        System.out.print("Capacidade máxima (kg): ");
        double capacidade = sc.nextDouble();

        var veiculo = new Veiculo(placa, modelo, capacidade);
        System.out.println("\n" + veiculo);

        System.out.print("\nPeso da carga (kg): ");
        double carga = sc.nextDouble();

        try {
            veiculo.carregar(carga);
            System.out.println(veiculo);
        } catch (IllegalStateException e) {
            System.out.println("Falha ao carregar: " + e.getMessage());
        }

        System.out.print("\nDistância da rota (km): ");
        double distancia = sc.nextDouble();

        veiculo.registrarViagem(distancia);
        System.out.println(veiculo);
        System.out.printf("Custo estimado de combustível: R$ %.2f%n",
            veiculo.custoCombustivel(distancia));

        sc.close();
    }
}
```

**Saída:**

```
Placa do veículo: ABC1D23
Modelo: Mercedes Sprinter
Capacidade máxima (kg): 1500

Veículo ABC1D23 (Mercedes Sprinter) | Carga: 0.0/1500.0kg (0%) | Hodômetro: 0km

Peso da carga (kg): 1200
Veículo ABC1D23 (Mercedes Sprinter) | Carga: 1200.0/1500.0kg (80%) | Hodômetro: 0km

Distância da rota (km): 340
Veículo ABC1D23 (Mercedes Sprinter) | Carga: 1200.0/1500.0kg (80%) | Hodômetro: 340km
Custo estimado de combustível: R$ 248.00
```

---

## 8. Exemplo Completo: Sistema de Rastreamento

Reúne `record` com construtor compacto, `enum` com construtor próprio, validação, encapsulamento e `switch` expression (Java 17).

```java
// ===== entities/Endereco.java =====
package entities;

import java.util.Objects;

// Record: valor imutável
public record Endereco(String logradouro, String cidade, String uf, String cep) {
    public Endereco {
        Objects.requireNonNull(logradouro);
        Objects.requireNonNull(cidade);
        if (!uf.matches("[A-Z]{2}")) {
            throw new IllegalArgumentException("UF inválida: " + uf);
        }
        if (!cep.matches("\\d{5}-?\\d{3}")) {
            throw new IllegalArgumentException("CEP inválido: " + cep);
        }
        cep = cep.replace("-", "");   // normaliza
    }

    public String cepFormatado() {
        return cep.substring(0, 5) + "-" + cep.substring(5);
    }
}

// ===== entities/StatusEncomenda.java =====
package entities;

// Enum com construtor próprio
public enum StatusEncomenda {
    PENDENTE("Aguardando coleta", false),
    EM_TRANSITO("Em rota de entrega", false),
    ENTREGUE("Entregue ao destinatário", true),
    EXTRAVIADA("Extraviada", true);

    private final String descricao;
    private final boolean finaliza;

    // Construtor de enum é sempre private
    StatusEncomenda(String descricao, boolean finaliza) {
        this.descricao = descricao;
        this.finaliza = finaliza;
    }

    public String getDescricao() { return this.descricao; }
    public boolean isFinal() { return this.finaliza; }
}

// ===== entities/Encomenda.java =====
package entities;

import java.time.LocalDateTime;
import java.util.*;

// Classe com estado mutável e regras de transição
public class Encomenda {
    private final String codigoRastreio;
    private final Endereco destino;
    private final double pesoKg;
    private final LocalDateTime dataRegistro;
    private StatusEncomenda status;
    private final List<String> historico = new ArrayList<>();

    public Encomenda(String codigoRastreio, Endereco destino, double pesoKg) {
        this.codigoRastreio = Objects.requireNonNull(codigoRastreio, "Código obrigatório");
        this.destino = Objects.requireNonNull(destino, "Destino obrigatório");

        if (!codigoRastreio.matches("BR\\d{9}")) {
            throw new IllegalArgumentException("Código fora do padrão: " + codigoRastreio);
        }
        if (pesoKg <= 0 || pesoKg > 30) {
            throw new IllegalArgumentException("Peso fora do limite (0-30kg): " + pesoKg);
        }

        this.pesoKg = pesoKg;
        this.dataRegistro = LocalDateTime.now();
        this.status = StatusEncomenda.PENDENTE;
        registrar("Encomenda registrada");
    }

    // Getters — nenhum setter de status
    public String getCodigoRastreio() { return this.codigoRastreio; }
    public Endereco getDestino() { return this.destino; }
    public double getPesoKg() { return this.pesoKg; }
    public StatusEncomenda getStatus() { return this.status; }

    // Visão não modificável — protege a lista interna
    public List<String> getHistorico() {
        return Collections.unmodifiableList(this.historico);
    }

    // Transições de estado — a única forma de mudar o status
    public void despachar() {
        exigirStatus(StatusEncomenda.PENDENTE, "despachar");
        this.status = StatusEncomenda.EM_TRANSITO;
        registrar("Despachada para " + this.destino.cidade());
    }

    public void confirmarEntrega() {
        exigirStatus(StatusEncomenda.EM_TRANSITO, "confirmar entrega");
        this.status = StatusEncomenda.ENTREGUE;
        registrar("Entregue em " + this.destino.cepFormatado());
    }

    public void marcarExtraviada(String motivo) {
        if (this.status.isFinal()) {
            throw new IllegalStateException("Encomenda já finalizada: " + this.status);
        }
        this.status = StatusEncomenda.EXTRAVIADA;
        registrar("Extraviada: " + motivo);
    }

    public double calcularFrete() {
        double base = 10.0 + this.pesoKg * 2.5;
        // switch expression (Java 14+) com enum
        double adicionalRegiao = switch (this.destino.uf()) {
            case "PE", "PB", "AL", "RN" -> 0;
            case "SP", "RJ", "MG", "ES" -> 12.50;
            default -> 25.00;
        };
        return base + adicionalRegiao;
    }

    private void exigirStatus(StatusEncomenda esperado, String acao) {
        if (this.status != esperado) {
            throw new IllegalStateException(
                "Não é possível %s: status atual é %s, esperado %s"
                    .formatted(acao, this.status, esperado)
            );
        }
    }

    private void registrar(String evento) {
        this.historico.add("[%s] %s".formatted(LocalDateTime.now(), evento));
    }

    @Override
    public String toString() {
        return "%s → %s/%s | %.1fkg | %s | Frete: R$ %.2f".formatted(
            this.codigoRastreio, this.destino.cidade(), this.destino.uf(),
            this.pesoKg, this.status.getDescricao(), calcularFrete()
        );
    }
}
```

```java
// ===== USO =====
package application;

import entities.*;

public class SistemaRastreamento {
    public static void main(String[] args) {
        var destinoRecife = new Endereco("Av. Boa Viagem, 100", "Recife", "PE", "51011-000");
        var destinoSP = new Endereco("Av. Paulista, 1000", "São Paulo", "SP", "01310100");

        var enc1 = new Encomenda("BR123456789", destinoRecife, 2.5);
        var enc2 = new Encomenda("BR987654321", destinoSP, 12.0);

        System.out.println("=== ENCOMENDAS REGISTRADAS ===");
        System.out.println(enc1);
        System.out.println(enc2);

        System.out.println("\n=== DESPACHO ===");
        enc1.despachar();
        enc2.despachar();
        System.out.println(enc1);
        System.out.println(enc2);

        System.out.println("\n=== ENTREGA ===");
        enc1.confirmarEntrega();
        System.out.println(enc1);

        System.out.println("\n=== HISTÓRICO DA ENCOMENDA 1 ===");
        enc1.getHistorico().forEach(System.out::println);

        System.out.println("\n=== TENTATIVA INVÁLIDA ===");
        try {
            enc1.despachar();   // já foi entregue
        } catch (IllegalStateException e) {
            System.out.println("Bloqueado: " + e.getMessage());
        }
    }
}
```

**Saída:**

```
=== ENCOMENDAS REGISTRADAS ===
BR123456789 → Recife/PE | 2.5kg | Aguardando coleta | Frete: R$ 16.25
BR987654321 → São Paulo/SP | 12.0kg | Aguardando coleta | Frete: R$ 52.50

=== DESPACHO ===
BR123456789 → Recife/PE | 2.5kg | Em rota de entrega | Frete: R$ 16.25
BR987654321 → São Paulo/SP | 12.0kg | Em rota de entrega | Frete: R$ 52.50

=== ENTREGA ===
BR123456789 → Recife/PE | 2.5kg | Entregue ao destinatário | Frete: R$ 16.25

=== HISTÓRICO DA ENCOMENDA 1 ===
[2026-11-12T14:30:15.123] Encomenda registrada
[2026-11-12T14:30:15.456] Despachada para Recife
[2026-11-12T14:30:15.789] Entregue em 51011-000

=== TENTATIVA INVÁLIDA ===
Bloqueado: Não é possível despachar: status atual é ENTREGUE, esperado PENDENTE
```

---

## 9. Checklist de Boas Práticas

### Construtores
- [ ] Sempre inicialize **todos** os atributos no construtor
- [ ] Centralize a validação no construtor canônico (o mais completo)
- [ ] Use `this()` para evitar repetição entre construtores
- [ ] Use `Objects.requireNonNull` para parâmetros obrigatórios
- [ ] Lance `IllegalArgumentException` para valores fora do domínio
- [ ] Prefira `final` nos campos que não devem mudar após a construção
- [ ] Nunca chame um método sobrescrevível (`public`/`protected` não-final) dentro do construtor

### Palavra `this`
- [ ] Use `this` quando há ambiguidade (especialmente em construtores)
- [ ] Use `this()` para chamar outro construtor da mesma classe
- [ ] Use `super()` para chamar o construtor da classe pai
- [ ] Lembre: `this()` e `super()` devem ser a primeira instrução, e não coexistem

### Sobrecarga
- [ ] Varie número, tipo ou ordem de parâmetros (tipo de retorno não conta)
- [ ] Nomes iguais devem ter significado semelhante
- [ ] Use constructor chaining para evitar repetição
- [ ] Evite sobrecargas ambíguas entre tipos numéricos próximos

### Encapsulamento
- [ ] Todos os atributos devem ser `private`
- [ ] Prefira **operações do domínio** a setters genéricos
- [ ] Valide dados no setter quando ele existir
- [ ] Atributos de identidade e fatos históricos não têm setter
- [ ] Nunca devolva coleções internas diretamente — use `Collections.unmodifiableList`
- [ ] Valores derivados são métodos calculados, não campos

### Java 17
- [ ] Use `record` para objetos de valor imutáveis (DTO, endereço, coordenada)
- [ ] Use construtor compacto do record para validar e normalizar
- [ ] Use `switch` expression (`->`) em vez de `switch` com `break`
- [ ] Use `"...".formatted(...)` em vez de `String.format("...", ...)`
- [ ] Use `var` para variáveis locais quando o tipo já é óbvio no lado direito
- [ ] Use `isBlank()` em vez de `trim().isEmpty()`

### Modificadores de Acesso
- [ ] Comece tudo `private`; promova só o necessário
- [ ] Use `protected` apenas quando subclasses realmente precisam
- [ ] `public` é um contrato — pense duas vezes antes de expor

---

## 10. Exercícios Práticos

### Exercício 1: Classe Transportadora

Implemente `Transportadora` com:
- Atributos: `cnpj` (imutável), `nome`, `raioOperacaoKm`, `ativa`
- Construtores: padrão, com CNPJ+nome, completo (use `this()`)
- Validação: CNPJ com 14 dígitos, raio positivo
- Getters para todos; setter só para `nome`
- Métodos: `ativar()`, `desativar()`, `atendeDestino(double distanciaKm)`
- `toString()` formatado

---

### Exercício 2: Record Endereco

Implemente um `record Endereco` com:
- Componentes: `logradouro`, `numero`, `cidade`, `uf`, `cep`
- Construtor compacto validando UF (2 letras maiúsculas) e CEP (8 dígitos)
- Normalização: remover pontuação do CEP, converter UF para maiúsculas
- Método: `cepFormatado()` devolvendo `00000-000`
- Método: `enderecoCompleto()` concatenando tudo

---

### Exercício 3: Classe Veiculo

Implemente `Veiculo` com:
- Atributos: `placa` (imutável), `modelo`, `capacidadeMaximaKg` (imutável), `cargaAtualKg`
- Construtor completo com `this()` e validação de placa Mercosul
- Getters para todos; sem setter para carga
- Métodos: `carregar(peso)`, `descarregar(peso)`, `espacoLivreKg()`, `estaCheio()`
- `toString()` mostrando o percentual de ocupação

---

### Exercício 4: Classe Motorista

Implemente `Motorista` com:
- Atributos: `cpf` (imutável), `nome`, `cnh`, `categoriaCnh`, `disponivel`
- Construtores: completo e sem categoria (padrão "B")
- Validação: CPF com 11 dígitos, CNH com 11 dígitos
- Método: `podeDirigir(Veiculo v)` — categoria C/D/E para acima de 3500kg
- Método: `atribuirRota()` / `liberar()` alterando disponibilidade

---

### Exercício 5: Enum StatusEntrega com Construtor

Implemente `enum StatusEntrega` com:
- Valores: PENDENTE, COLETADA, EM_TRANSITO, SAIU_PARA_ENTREGA, ENTREGUE, DEVOLVIDA
- Atributos no enum: `descricao`, `ordem`, `finaliza`
- Construtor do enum inicializando os três
- Método: `podeTransicionarPara(StatusEntrega proximo)` validando a ordem
- Método estático: `statusAtivos()` devolvendo os não finalizados

---

### Exercício 6: Classe Rota

Implemente `Rota` com:
- Atributos: `codigo` (imutável), `origem`, `destino` (records `Endereco`), `distanciaKm`, lista de encomendas
- Construtor com validação (origem ≠ destino, distância positiva)
- Método: `adicionarEncomenda(Encomenda e)` respeitando limite de 50
- Método: `pesoTotalKg()`, `freteTotal()`
- Getter da lista devolvendo visão não modificável

---

### Exercício 7: Record Coordenada e Cálculo de Distância

Implemente `record Coordenada(double latitude, double longitude)` com:
- Construtor compacto validando os intervalos (-90/90 e -180/180)
- Método: `distanciaAte(Coordenada outra)` usando a fórmula de Haversine
- Método: `estaDentroDoRaio(Coordenada centro, double raioKm)`

---

### Exercício 8: Classe CentroDistribuicao

Implemente `CentroDistribuicao` com:
- Atributos: `codigo` (imutável), `endereco` (record), `capacidadeEncomendas`, lista de encomendas
- Construtores sobrecarregados com `this()`
- Métodos: `receber(Encomenda)`, `expedir(String codigoRastreio)`
- Método: `taxaOcupacao()` em percentual
- Lançar exceção ao receber com o centro lotado

---

### Exercício 9: Classe Fatura

Implemente `Fatura` com:
- Atributos: `numero` (imutável), `cliente`, `valorBruto`, `desconto`, `dataEmissao` (imutável)
- Construtores: com e sem desconto (padrão 0)
- Validação: valor bruto positivo, desconto entre 0 e valor bruto
- Método: `valorLiquido()`, `aplicarDesconto(double)` com validação
- Método: `estaVencida(LocalDate vencimento)`

---

### Exercício 10: Sistema de Entrega Completo

Integre tudo:
- `record Endereco` com construtor compacto
- `enum StatusEncomenda` com construtor e descrição
- `class Encomenda` com transições de estado validadas
- `class Veiculo` com controle de capacidade
- `class Rota` que recebe um `Veiculo` e uma lista de `Encomenda`
- Simule: registrar encomendas → montar rota → despachar → entregar
- Exiba o histórico de cada encomenda ao final

---

## 11. Referências Rápidas

### Constructor Chaining

```java
public class Encomenda {
    private String codigo;
    private String destino;
    private double pesoKg;

    public Encomenda() {
        this("SEM-CODIGO", "NÃO INFORMADO");
    }

    public Encomenda(String codigo, String destino) {
        this(codigo, destino, 1.0);
    }

    // Construtor canônico — validação mora aqui
    public Encomenda(String codigo, String destino, double pesoKg) {
        this.codigo = Objects.requireNonNull(codigo);
        this.destino = Objects.requireNonNull(destino);
        if (pesoKg <= 0) throw new IllegalArgumentException("Peso inválido");
        this.pesoKg = pesoKg;
    }
}
```

### Record com Construtor Compacto

```java
public record Endereco(String cidade, String uf, String cep) {
    public Endereco {
        Objects.requireNonNull(cidade);
        if (!uf.matches("[A-Z]{2}")) {
            throw new IllegalArgumentException("UF inválida");
        }
        cep = cep.replace("-", "");   // normalização
    }
}
```

### Validação com `Objects`

```java
this.destino = Objects.requireNonNull(destino, "Destino obrigatório");
this.status  = Objects.requireNonNullElse(status, StatusEncomenda.PENDENTE);

if (pesoKg <= 0) {
    throw new IllegalArgumentException("Peso deve ser positivo: " + pesoKg);
}
```

### Padrão JavaBeans

```
Atributo private  → getter getDestino()   → setter setDestino()
Booleano private  → getter isDisponivel() → setter setDisponivel()
Record            → acessor destino()     → sem setter (imutável)
```

### Proteção de Coleção Interna

```java
private final List<Encomenda> encomendas = new ArrayList<>();

public List<Encomenda> getEncomendas() {
    return Collections.unmodifiableList(this.encomendas);
}

public void adicionar(Encomenda e) {
    // validação aqui
    this.encomendas.add(e);
}
```

### `toString()` com `formatted` (Java 15+)

```java
@Override
public String toString() {
    return "%s → %s | %.1fkg | R$ %.2f".formatted(
        this.codigo, this.destino, this.pesoKg, calcularFrete()
    );
}
```

### `switch` Expression (Java 14+)

```java
double adicional = switch (uf) {
    case "PE", "PB", "AL" -> 0;
    case "SP", "RJ", "MG" -> 12.50;
    default -> 25.00;
};
```

---

### Referências

- [Java 17 Language Updates (Oracle)](https://docs.oracle.com/en/java/javase/17/language/index.html)
- [JEP 395: Records](https://openjdk.org/jeps/395)
- [JEP 361: Switch Expressions](https://openjdk.org/jeps/361)
- [Docs: `java.util.Objects`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Objects.html)
- Livro: **Effective Java** (Joshua Bloch) — itens 1 a 9 sobre criação de objetos
- Ver também: [21-CLASSES-E-METODOS-JAVA.MD](21-CLASSES-E-METODOS-JAVA.MD), [08-ENCAPSULAMENTO-JAVA.MD](08-ENCAPSULAMENTO-JAVA.MD), [38-JAVA-17.MD](38-JAVA-17.MD)
