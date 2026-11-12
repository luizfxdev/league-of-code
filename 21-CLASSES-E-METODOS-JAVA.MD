# Classes e Métodos em Java — Guia Prático para Sistemas de Logística

## 1. O que é uma Classe?

Uma classe é um molde (blueprint) que define como criar objetos. Ela agrupa dados (atributos) e comportamentos (métodos) relacionados.

### Sintaxe Básica

```java
public class Veiculo {
    // Atributos (dados)
    private String placa;
    private String modelo;
    private double capacidadeCarga;
    
    // Construtor
    public Veiculo(String placa, String modelo, double capacidadeCarga) {
        this.placa = placa;
        this.modelo = modelo;
        this.capacidadeCarga = capacidadeCarga;
    }
    
    // Getters
    public String getPlaca() {
        return placa;
    }
}
```

### Componentes de uma Classe

| Componente | O que é | Exemplo |
|-----------|---------|---------|
| **Atributos** | Dados que a classe armazena | `private String placa` |
| **Construtor** | Inicializa um objeto quando criado | `public Veiculo(...)` |
| **Getters** | Retornam valores dos atributos | `public String getPlaca()` |
| **Setters** | Modificam valores dos atributos | `public void setPlaca(String p)` |
| **Métodos** | Definem comportamentos | `public void acelerar()` |

---

## 2. Visibilidade: Public, Private, Protected

Controla quem pode acessar cada parte da classe.

```java
public class Entrega {
    public String id;              // Acessível de qualquer lugar
    private String endereco;        // Apenas dentro desta classe
    protected String motorista;     // Esta classe + subclasses + mesmo pacote
}
```

### Boas Práticas em Logística

```java
public class Pedido {
    // Dados sensíveis ficam private
    private double valorTotal;
    private LocalDateTime dataCriacao;
    private StatusPedido status;
    
    // Acesso controlado via métodos
    public double getValorTotal() {
        return valorTotal;
    }
    
    public void setStatus(StatusPedido novoStatus) {
        if (podeAlterarStatus(novoStatus)) {
            this.status = novoStatus;
        }
    }
    
    private boolean podeAlterarStatus(StatusPedido novo) {
        // Lógica de validação encapsulada
        return status != StatusPedido.ENTREGUE;
    }
}
```

**Benefício:** O estado interno não é corrompido por acessos inadequados.

---

## 3. Construtores

Método especial que **executa automaticamente** quando você cria um novo objeto com `new`. Responsável por **inicializar o estado** do objeto de forma consistente.

### O Problema: Objetos Vazios e Inconsistentes

```java
// ❌ Sem construtor customizado
public class Transportadora {
    private String cnpj;
    private String nome;
    private double raioOperacao;
}

Transportadora t = new Transportadora();  // Objeto vazio
System.out.println(t.cnpj);          // null
System.out.println(t.nome);          // null
System.out.println(t.raioOperacao);  // 0.0

// Problema: uma transportadora sem CNPJ ou nome é inconsistente!
// Pode gerar erros no sistema
```

### A Solução: Construtor Customizado

```java
// ✅ Com construtor - força inicializar dados obrigatórios
public class Transportadora {
    private String cnpj;
    private String nome;
    private double raioOperacao;
    
    // Construtor obriga a fornecer os dados essenciais
    public Transportadora(String cnpj, String nome, double raioOperacao) {
        this.cnpj = cnpj;      // this diferencia atributo de parâmetro
        this.nome = nome;
        this.raioOperacao = raioOperacao;
    }
}

// Agora é impossível criar uma Transportadora sem dados
Transportadora t = new Transportadora("12345678000100", "Loggi", 500.0);
// Objeto sempre nasce COMPLETO e CONSISTENTE
```

### Construtor Padrão vs Customizado

```java
// Se você NÃO criar construtor, Java fornece um vazio automaticamente
public class Entrega {
    private String id;
}

// Java fornece implicitamente:
// public Entrega() { }

Entrega e = new Entrega();  // Funciona

// ---

// Mas se você criar um construtor customizado...
public class Entrega {
    private String id;
    
    public Entrega(String id) {
        this.id = id;
    }
    
    // O construtor padrão DESAPARECE!
}

Entrega e1 = new Entrega("ENT001");  // ✅ OK
Entrega e2 = new Entrega();          // ❌ ERRO!
```

**Solução:** Forneça o construtor padrão explicitamente se necessário:

```java
public class Entrega {
    private String id;
    
    // Construtor padrão (sem parâmetros)
    public Entrega() {
        this.id = gerarId();
    }
    
    // Construtor customizado
    public Entrega(String id) {
        this.id = id;
    }
    
    private String gerarId() {
        return "ENT-" + System.currentTimeMillis();
    }
}

Entrega e1 = new Entrega();
Entrega e2 = new Entrega("ENT001");
```

### Fluxo de Construção

```
Transportadora t = new Transportadora("12345678000100", "Loggi", 500.0);

1. new Transportadora(...)    → Aloca memória na heap
2. Executa construtor         → Inicializa cnpj, nome, raioOperacao
3. Retorna referência         → t aponta para o objeto criado
```

### A Palavra `this` em Construtores

`this` é uma referência ao **próprio objeto**. Diferencia atributos de parâmetros.

```java
public class Motorista {
    private String nome;
    private String cnh;
    
    // ❌ Confuso: qual nome? O parâmetro ou o atributo?
    public void inicializar1(String nome) {
        nome = nome;  // Erro lógico!
    }
    
    // ✅ Claro: this diferencia
    public Motorista(String nome, String cnh) {
        this.nome = nome;  // this.nome (atributo) = nome (parâmetro)
        this.cnh = cnh;    // this.cnh (atributo) = cnh (parâmetro)
    }
}
```

**Stack vs Heap:**
```
Stack (Parâmetros)     Heap (Objeto)
nome ("João")      →   this.nome = "João"
cnh ("1234567890") →   this.cnh = "1234567890"
```

### Constructor Chaining: Evitando Repetição

Múltiplos construtores podem chamar um ao outro com `this()`.

```java
public class Transportadora {
    private String cnpj;
    private String nome;
    private double raioOperacao;
    private boolean ativa;
    
    // Construtor completo (faz o trabalho real)
    public Transportadora(String cnpj, String nome, double raioOperacao) {
        this.cnpj = cnpj;
        this.nome = nome;
        this.raioOperacao = raioOperacao;
        this.ativa = true;
    }
    
    // Construtor simplificado chama o completo com this()
    public Transportadora(String cnpj, String nome) {
        this(cnpj, nome, 100.0);  // Raio padrão
    }
    
    // Construtor mínimo
    public Transportadora(String cnpj) {
        this(cnpj, "Sem Nome");  // Chama o outro construtor
    }
    
    // Construtor padrão
    public Transportadora() {
        this("00000000000000");  // Chama o construtor mínimo
    }
}

// Uso - cada construtor oferece uma opção
Transportadora t1 = new Transportadora();
Transportadora t2 = new Transportadora("12345678000100");
Transportadora t3 = new Transportadora("12345678000100", "Loggi");
Transportadora t4 = new Transportadora("12345678000100", "Loggi", 500.0);

// Vantagem: código não repetido
// Desvantagem: nenhuma (sem this(), teria repetição)
```

**Comparação: Com vs Sem Constructor Chaining**

```java
// ❌ Ruim: código repetido
public Transportadora(String cnpj, String nome) {
    this.cnpj = cnpj;
    this.nome = nome;
    this.raioOperacao = 100.0;
    this.ativa = true;
}

public Transportadora() {
    this.cnpj = "00000000000000";
    this.nome = "Sem Nome";
    this.raioOperacao = 100.0;
    this.ativa = true;
}

// ✅ Bom: encadeamento
public Transportadora(String cnpj, String nome) {
    this(cnpj, nome, 100.0);
}

public Transportadora() {
    this("00000000000000", "Sem Nome");
}

public Transportadora(String cnpj, String nome, double raio) {
    this.cnpj = cnpj;
    this.nome = nome;
    this.raioOperacao = raio;
    this.ativa = true;
}
```

### Sobrecarga de Construtores

Uma classe pode ter **múltiplos construtores** com listas de parâmetros diferentes.

```java
public class Entrega {
    private String id;
    private String endereco;
    private double peso;
    private LocalDateTime data;
    
    // Construtor 1: mínimo
    public Entrega(String endereco) {
        this(endereco, 1.0);
    }
    
    // Construtor 2: sem data
    public Entrega(String endereco, double peso) {
        this(endereco, peso, LocalDateTime.now());
    }
    
    // Construtor 3: completo
    public Entrega(String endereco, double peso, LocalDateTime data) {
        this.id = "ENT-" + System.currentTimeMillis();
        this.endereco = endereco;
        this.peso = peso;
        this.data = data;
    }
}

// Cada construtor oferece uma interface diferente
Entrega e1 = new Entrega("Rua A, 100");
Entrega e2 = new Entrega("Rua B, 200", 5.5);
Entrega e3 = new Entrega("Rua C, 300", 10.0, LocalDateTime.now());
```

**Regras para Sobrecarga:**
- ✓ Número diferente de parâmetros
- ✓ Tipo diferente de parâmetros
- ✓ Ordem diferente de parâmetros
- ✗ Apenas tipo de retorno diferente (não funciona)

### Validação no Construtor

Garante que o objeto nasce num estado válido.

```java
public class Veiculo {
    private String placa;
    private double capacidade;
    
    public Veiculo(String placa, double capacidade) {
        // Validar placa
        if (placa == null || placa.isBlank()) {
            throw new IllegalArgumentException("Placa não pode ser vazia");
        }
        
        // Validar capacidade
        if (capacidade <= 0) {
            throw new IllegalArgumentException("Capacidade deve ser positiva");
        }
        
        this.placa = placa;
        this.capacidade = capacidade;
    }
}

// Uso
Veiculo v1 = new Veiculo("ABC1234", 1000);  // OK
Veiculo v2 = new Veiculo("", 500);          // ERRO!
Veiculo v3 = new Veiculo("XYZ9999", -100);  // ERRO!
```

### Exemplo Prático Completo

```java
public class Pedido {
    private String id;
    private String cliente;
    private double valor;
    private StatusPedido status;
    private LocalDateTime dataCriacao;
    
    // Construtor com validação e inicialização completa
    public Pedido(String cliente, double valor) {
        // Validar
        if (cliente == null || cliente.isBlank()) {
            throw new IllegalArgumentException("Cliente obrigatório");
        }
        if (valor <= 0) {
            throw new IllegalArgumentException("Valor deve ser positivo");
        }
        
        // Inicializar
        this.id = "PED-" + System.currentTimeMillis();
        this.cliente = cliente;
        this.valor = valor;
        this.status = StatusPedido.PENDENTE;  // Sempre começa como PENDENTE
        this.dataCriacao = LocalDateTime.now();
        
        System.out.println("✓ Pedido criado: " + this.id);
    }
    
    // Getters
    public String getId() { return id; }
    public String getCliente() { return cliente; }
    public double getValor() { return valor; }
    public StatusPedido getStatus() { return status; }
}

// Uso
Pedido p1 = new Pedido("Empresa XYZ", 1500.00);
// Saída: ✓ Pedido criado: PED-1699876543210

Pedido p2 = new Pedido("", 1000);  // ERRO lançado!
// Exception in thread: Cliente obrigatório
```

### Checklist: Boas Práticas em Construtores

- [ ] Sempre inicialize **todos os atributos** no construtor
- [ ] Use `this` quando há ambiguidade (OBRIGATÓRIO)
- [ ] Use `this()` para evitar repetição de código
- [ ] Ofereça múltiplos construtores para flexibilidade
- [ ] Valide parâmetros antes de usar
- [ ] Lance exceções em dados inválidos
- [ ] Deixe o objeto sempre num estado **consistente**
- [ ] Forneça construtor padrão se necessário

---

## 4. Métodos: Definição e Comportamento

Um método é um bloco de código que realiza uma ação específica.

### Estrutura Completa

```java
[modificador] [tipo_retorno] [nome_metodo] ([parâmetros]) {
    // corpo do método
}
```

### Exemplos

```java
public class Motorista {
    private String nome;
    private String cnh;
    private boolean ativo;
    
    // Sem parâmetros, com retorno
    public String obterNome() {
        return nome;
    }
    
    // Com parâmetro, sem retorno
    public void ativar() {
        this.ativo = true;
    }
    
    // Com parâmetro e retorno
    public boolean validarCNH(String cnh) {
        return cnh != null && cnh.length() == 11;
    }
    
    // Múltiplos parâmetros
    public void registrarViagem(String origem, String destino, double km) {
        System.out.println(nome + " viajou " + km + "km");
    }
}
```

---

## 5. Tipos de Retorno

### void (sem retorno)

```java
public void procesarEntrega(Entrega entrega) {
    entrega.setStatus(StatusPedido.EM_TRANSITO);
    notificarCliente(entrega);
}
```

### Retorna valor

```java
public double calcularFrete(double distancia, double peso) {
    return distancia * 1.5 + peso * 0.8;
}
```

### Retorna objeto

```java
public Veiculo encontrarVeiculoDisponivel(double capacidadeNecessaria) {
    for (Veiculo v : frota) {
        if (v.getCapacidade() >= capacidadeNecessaria && v.estáDisponivel()) {
            return v;
        }
    }
    return null; // Nenhum disponível
}
```

### Retorna lista ou coleção

```java
public List<Entrega> obterEntregasPendentes(String motorista) {
    return entregasDAO.buscarPorMotoristaEStatus(motorista, StatusPedido.PENDENTE);
}
```

---

## 6. Parâmetros e Argumentos

**Parâmetros** = definição (na assinatura)  
**Argumentos** = valores passados (na chamada)

```java
// Definição (parâmetros)
public void atualizarLocalizacao(double latitude, double longitude) {
    this.latitude = latitude;
    this.longitude = longitude;
}

// Chamada (argumentos)
veiculo.atualizarLocalizacao(-8.0028, -34.8813); // Recife
```

### Parâmetros Variáveis (varargs)

```java
public void adicionarParagensRota(String... localidades) {
    for (String local : localidades) {
        rota.add(local);
    }
}

// Uso
veiculo.adicionarParagensRota("Olinda", "Jaboatão", "Paulista", "Igarassu");
```

---

## 7. Métodos Getters e Setters

Fornecem acesso controlado aos atributos.

```java
public class Entrega {
    private String id;
    private double peso;
    private LocalDateTime dataPrevista;
    
    // Getter simples
    public String getId() {
        return id;
    }
    
    // Setter com validação
    public void setPeso(double peso) {
        if (peso > 0) {
            this.peso = peso;
        } else {
            throw new IllegalArgumentException("Peso deve ser positivo");
        }
    }
    
    // Getter com lógica
    public boolean estáAtrasada() {
        return LocalDateTime.now().isAfter(dataPrevista);
    }
    
    // Setter que dispara ação
    public void setDataPrevista(LocalDateTime data) {
        this.dataPrevista = data;
        notificarAlteracaoDataAoCliente();
    }
}
```

---

## 8. Métodos Estáticos

Pertencem à classe, não a instâncias específicas.

```java
public class CalculadoraFrete {
    private static final double TAXA_BASE = 5.0;
    
    // Método estático
    public static double calcularFrete(double km) {
        return TAXA_BASE + (km * 0.50);
    }
    
    // Método estático que cria instâncias
    public static CalculadoraFrete criar() {
        return new CalculadoraFrete();
    }
}

// Chamada — não precisa criar objeto
double frete = CalculadoraFrete.calcularFrete(100);
```

### Uso Prático em Logística

```java
public class StatusEntrega {
    public static final String PENDENTE = "PENDENTE";
    public static final String EM_TRANSITO = "EM_TRANSITO";
    public static final String ENTREGUE = "ENTREGUE";
    public static final String CANCELADO = "CANCELADO";
    
    public static boolean estaFinalizado(String status) {
        return status.equals(ENTREGUE) || status.equals(CANCELADO);
    }
}

// Uso
if (StatusEntrega.estaFinalizado(pedido.getStatus())) {
    arquivar(pedido);
}
```

---

## 9. Métodos com Objetos como Parâmetros

```java
public class Rota {
    private List<Entrega> entregas = new ArrayList<>();
    
    // Recebe um objeto como parâmetro
    public void adicionarEntrega(Entrega entrega) {
        if (entrega.getPeso() <= capacidadeMaxima) {
            entregas.add(entrega);
        }
    }
    
    // Recebe múltiplos objetos
    public void otimizarRota(Motorista motorista, List<Entrega> entregas) {
        validarMotorista(motorista);
        ordenarPorProximidade(entregas);
        atribuirMotorista(motorista);
    }
    
    // Retorna um objeto modificado
    public Rota comPrioritarias() {
        return entregas.stream()
            .filter(e -> e.isPrioritaria())
            .collect(Collectors.toList());
    }
}
```

---

## 10. This e Super

### This: Referência ao Objeto Atual

```java
public class Veiculo {
    private String placa;
    private String modelo;
    
    public Veiculo(String placa, String modelo) {
        this.placa = placa;      // this diferencia variável local de atributo
        this.modelo = modelo;
    }
    
    public void exibir() {
        System.out.println("Veiculo: " + this.placa); // Explícito, legível
    }
}
```

### Super: Referência à Classe Pai

```java
public class Veiculo {
    protected double velocidade;
    
    public void acelerar() {
        velocidade += 10;
    }
}

public class VeiculoEntrega extends Veiculo {
    private double pesoMaximo;
    
    @Override
    public void acelerar() {
        super.acelerar(); // Chama método da classe pai
        System.out.println("Aceleração com carga: " + pesoMaximo);
    }
}
```

---

## 11. Sobrecarga de Métodos (Overloading)

Mesma classe, mesmo nome, parâmetros diferentes.

```java
public class Endereco {
    private String rua;
    private String numero;
    private String complemento;
    
    // Sobrecarga 1: construtor mínimo
    public Endereco(String rua) {
        this.rua = rua;
    }
    
    // Sobrecarga 2: construtor parcial
    public Endereco(String rua, String numero) {
        this.rua = rua;
        this.numero = numero;
    }
    
    // Sobrecarga 3: construtor completo
    public Endereco(String rua, String numero, String complemento) {
        this.rua = rua;
        this.numero = numero;
        this.complemento = complemento;
    }
}

// Uso
Endereco e1 = new Endereco("Av. Dantas Barreto");
Endereco e2 = new Endereco("Av. Dantas Barreto", "100");
Endereco e3 = new Endereco("Av. Dantas Barreto", "100", "Apto 2001");
```

### Métodos Sobrecarregados

```java
public class NotificacaoCliente {
    public void enviar(String email, String mensagem) {
        // Envia por email
    }
    
    public void enviar(String telefone, String mensagem, String formato) {
        // Envia por SMS
    }
    
    public void enviar(Entrega entrega) {
        // Envia notificação padrão
    }
}
```

---

## 12. Herança: Extends

Uma classe herda atributos e métodos de outra.

```java
// Classe pai
public class Veiculo {
    protected String placa;
    protected String modelo;
    protected double velocidadeMaxima;
    
    public void ligar() {
        System.out.println("Veículo ligado");
    }
}

// Classe filha
public class Caminhao extends Veiculo {
    private double capacidadeCarga;
    
    public void carregar(double peso) {
        if (peso <= capacidadeCarga) {
            System.out.println("Carregando " + peso + "kg");
        }
    }
}

// Classe filha
public class Moto extends Veiculo {
    private boolean temBau;
    
    public void buscarDocumentacao() {
        if (temBau) {
            System.out.println("Buscando documentação rápido");
        }
    }
}
```

### Uso

```java
Caminhao c = new Caminhao();
c.placa = "ABC1234";      // Herdado
c.ligar();                 // Herdado
c.carregar(5000);          // Próprio

Moto m = new Moto();
m.ligar();                 // Herdado
m.buscarDocumentacao();    // Próprio
```

---

## 13. @Override: Sobrescrita de Métodos

A classe filha reimplementa um método da classe pai.

```java
public class Veiculo {
    public void calcularTempo(double distancia) {
        double velocidadeMedia = 80;
        System.out.println("Tempo: " + distancia / velocidadeMedia + "h");
    }
}

public class Caminhao extends Veiculo {
    @Override
    public void calcularTempo(double distancia) {
        double velocidadeMedia = 60; // Caminhões mais lentos
        System.out.println("Tempo caminhão: " + distancia / velocidadeMedia + "h");
    }
}

public class Moto extends Veiculo {
    @Override
    public void calcularTempo(double distancia) {
        double velocidadeMedia = 100; // Motos mais rápidas
        System.out.println("Tempo moto: " + distancia / velocidadeMedia + "h");
    }
}
```

### Uso Polimórfico

```java
List<Veiculo> frota = new ArrayList<>();
frota.add(new Caminhao());
frota.add(new Moto());
frota.add(new Veiculo());

for (Veiculo v : frota) {
    v.calcularTempo(100); // Cada um usa sua implementação
}
```

---

## 14. Interfaces: Contrato de Comportamento

Define métodos que a classe deve implementar.

```java
public interface Entregavel {
    void entregar();
    boolean podeSerEntregue();
    String obterStatus();
}

public class EntregaExpresso implements Entregavel {
    @Override
    public void entregar() {
        System.out.println("Entregando expressa");
    }
    
    @Override
    public boolean podeSerEntregue() {
        return true;
    }
    
    @Override
    public String obterStatus() {
        return "ENTREGUE";
    }
}

public class EntregaAgendada implements Entregavel {
    private LocalDateTime dataEntrega;
    
    @Override
    public void entregar() {
        System.out.println("Entregando em " + dataEntrega);
    }
    
    @Override
    public boolean podeSerEntregue() {
        return LocalDateTime.now().isAfter(dataEntrega);
    }
    
    @Override
    public String obterStatus() {
        return "AGENDADA";
    }
}
```

### Uso

```java
List<Entregavel> entregas = new ArrayList<>();
entregas.add(new EntregaExpresso());
entregas.add(new EntregaAgendada());

for (Entregavel e : entregas) {
    if (e.podeSerEntregue()) {
        e.entregar();
    }
}
```

---

## 15. Classes Abstratas

Define comportamento comum, força subclasses a implementar certos métodos.

```java
public abstract class DocumentoFiscal {
    protected String numero;
    protected LocalDateTime data;
    protected double valor;
    
    public void emitir() {
        gerarNumero();
        setarData();
        registrarEmBancoDados();
    }
    
    protected abstract void gerarNumero();
    
    public void setarData() {
        this.data = LocalDateTime.now();
    }
}

public class Nota extends DocumentoFiscal {
    @Override
    protected void gerarNumero() {
        this.numero = "NF-" + System.currentTimeMillis();
    }
}

public class Recibo extends DocumentoFiscal {
    @Override
    protected void gerarNumero() {
        this.numero = "REC-" + System.currentTimeMillis();
    }
}
```

---

## 16. Encapsulamento: Prática Completa

Exemplo real de um sistema de entrega.

```java
public class Entrega {
    private String id;
    private String endereco;
    private double peso;
    private StatusEntrega status;
    private LocalDateTime dataCriacao;
    private LocalDateTime dataEntrega;
    private double valorCobrado;
    
    // Construtor
    public Entrega(String endereco, double peso) {
        this.id = gerarId();
        this.endereco = endereco;
        this.peso = peso;
        this.status = StatusEntrega.PENDENTE;
        this.dataCriacao = LocalDateTime.now();
    }
    
    // Getters básicos
    public String getId() {
        return id;
    }
    
    public String getEndereco() {
        return endereco;
    }
    
    // Getter com validação
    public double getPeso() {
        return peso;
    }
    
    // Setter com regra de negócio
    public void setPeso(double peso) {
        if (peso <= 0) {
            throw new IllegalArgumentException("Peso deve ser positivo");
        }
        if (status != StatusEntrega.PENDENTE) {
            throw new IllegalStateException("Não pode alterar peso após processo iniciado");
        }
        this.peso = peso;
    }
    
    // Método que valida antes de alterar status
    public void marcarComoEmTransito() {
        if (status != StatusEntrega.PENDENTE) {
            throw new IllegalStateException("Apenas entregas pendentes podem ser marcadas");
        }
        this.status = StatusEntrega.EM_TRANSITO;
    }
    
    // Método que calcula
    public double calcularFrete() {
        double taxa = 10.0;
        double taxaPorKg = 2.5;
        return taxa + (peso * taxaPorKg);
    }
    
    // Método que finaliza processo
    public void marcarComoEntregue() {
        if (status != StatusEntrega.EM_TRANSITO) {
            throw new IllegalStateException("Apenas em trânsito pode ser finalizada");
        }
        this.status = StatusEntrega.ENTREGUE;
        this.dataEntrega = LocalDateTime.now();
        this.valorCobrado = calcularFrete();
    }
    
    // Getter que retorna estado calculado
    public boolean estaAtrasada() {
        return status == StatusEntrega.PENDENTE && 
               LocalDateTime.now().isAfter(dataCriacao.plusHours(24));
    }
    
    private String gerarId() {
        return "ENT-" + System.currentTimeMillis();
    }
}
```

---

## 17. Métodos Úteis: toString, equals, hashCode

```java
public class Motorista {
    private String cpf;
    private String nome;
    private String telefone;
    
    // Representação em String
    @Override
    public String toString() {
        return "Motorista{" +
                "cpf='" + cpf + '\'' +
                ", nome='" + nome + '\'' +
                ", telefone='" + telefone + '\'' +
                '}';
    }
    
    // Comparação entre objetos
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Motorista motorista = (Motorista) o;
        return cpf.equals(motorista.cpf);
    }
    
    // Hash para coleções
    @Override
    public int hashCode() {
        return cpf.hashCode();
    }
}

// Uso
Motorista m1 = new Motorista("12345678900", "João", "81999999999");
Motorista m2 = new Motorista("12345678900", "João da Silva", "81988888888");

System.out.println(m1); // Motorista{cpf='12345678900', nome='João', ...}
System.out.println(m1.equals(m2)); // true (mesmo CPF)

Set<Motorista> motoristas = new HashSet<>();
motoristas.add(m1);
motoristas.add(m2); // Não adiciona (hashCode igual)
```

---

## 18. Boas Práticas em Logística

### 1. Classes com Responsabilidade Única

```java
// ❌ Ruim: mistura responsabilidades
public class Entrega {
    public void entregar() { }
    public void enviarEmail() { }
    public void salvarBancoDados() { }
}

// ✅ Bom: cada classe tem um propósito
public class Entrega {
    public void entregar() { }
}

public class NotificadorEntrega {
    public void enviarEmail(Entrega e) { }
}

public class RepositorioEntrega {
    public void salvar(Entrega e) { }
}
```

### 2. Evitar Exposição de Dados

```java
// ❌ Ruim: dados públicos
public class Pedido {
    public List<Entrega> entregas;
}

// Problema: qualquer um modifica
pedido.entregas.clear();

// ✅ Bom: controle de acesso
public class Pedido {
    private List<Entrega> entregas = new ArrayList<>();
    
    public List<Entrega> obterEntregas() {
        return Collections.unmodifiableList(entregas);
    }
    
    public void adicionarEntrega(Entrega e) {
        if (validar(e)) {
            entregas.add(e);
        }
    }
}
```

### 3. Validação no Setter

```java
// ❌ Ruim: sem validação
public void setVelocidade(double v) {
    this.velocidade = v;
}

// ✅ Bom: com validação
public void setVelocidade(double v) {
    if (v < 0 || v > velocidadeMaxima) {
        throw new IllegalArgumentException("Velocidade inválida");
    }
    this.velocidade = v;
}
```

### 4. Documentação Mínima

```java
/**
 * Calcula o frete baseado em distância e peso.
 * 
 * @param distancia em km
 * @param peso em kg
 * @return valor do frete em R$
 */
public double calcularFrete(double distancia, double peso) {
    return distancia * 1.5 + peso * 0.8;
}
```

---

## 19. Exemplo Completo: Sistema de Entrega

```java
// Entidade principal
public class Entrega {
    private String id;
    private String endereco;
    private double peso;
    private StatusEntrega status;
    private LocalDateTime dataCriacao;
    private Motorista motorista;
    
    public Entrega(String endereco, double peso) {
        validar(endereco, peso);
        this.id = "ENT-" + System.currentTimeMillis();
        this.endereco = endereco;
        this.peso = peso;
        this.status = StatusEntrega.PENDENTE;
        this.dataCriacao = LocalDateTime.now();
    }
    
    public void atribuirMotorista(Motorista m) {
        if (status != StatusEntrega.PENDENTE) {
            throw new IllegalStateException("Entregar já atribuída");
        }
        this.motorista = m;
        this.status = StatusEntrega.EM_TRANSITO;
    }
    
    public void finalizar() {
        if (status != StatusEntrega.EM_TRANSITO) {
            throw new IllegalStateException("Não está em trânsito");
        }
        this.status = StatusEntrega.ENTREGUE;
    }
    
    private void validar(String endereco, double peso) {
        if (endereco == null || endereco.isBlank()) {
            throw new IllegalArgumentException("Endereço obrigatório");
        }
        if (peso <= 0) {
            throw new IllegalArgumentException("Peso deve ser positivo");
        }
    }
    
    public double calcularFrete() {
        return 10.0 + (peso * 2.5);
    }
    
    // Getters
    public String getId() { return id; }
    public StatusEntrega getStatus() { return status; }
    public Motorista getMotorista() { return motorista; }
}

// Enum de status
public enum StatusEntrega {
    PENDENTE, EM_TRANSITO, ENTREGUE, CANCELADO
}

// Motorista
public class Motorista {
    private String cpf;
    private String nome;
    private List<Entrega> entregas = new ArrayList<>();
    
    public Motorista(String cpf, String nome) {
        this.cpf = cpf;
        this.nome = nome;
    }
    
    public void receberEntrega(Entrega e) {
        e.atribuirMotorista(this);
        entregas.add(e);
    }
    
    public void finalizarEntrega(Entrega e) {
        if (!entregas.contains(e)) {
            throw new IllegalArgumentException("Entrega não pertence a este motorista");
        }
        e.finalizar();
    }
    
    public int totalEntregas() {
        return entregas.size();
    }
}

// Serviço (coordenador)
public class ServicoEntrega {
    private List<Entrega> todasEntregas = new ArrayList<>();
    private List<Motorista> motoristas = new ArrayList<>();
    
    public void registrarEntrega(String endereco, double peso) {
        Entrega e = new Entrega(endereco, peso);
        todasEntregas.add(e);
    }
    
    public void atribuirMotorista(String entregaId, String motoristaCpf) {
        Entrega e = todasEntregas.stream()
            .filter(en -> en.getId().equals(entregaId))
            .findFirst()
            .orElseThrow();
        
        Motorista m = motoristas.stream()
            .filter(mot -> mot.getCpf().equals(motoristaCpf))
            .findFirst()
            .orElseThrow();
        
        m.receberEntrega(e);
    }
    
    public List<Entrega> obterPendentes() {
        return todasEntregas.stream()
            .filter(e -> e.getStatus() == StatusEntrega.PENDENTE)
            .toList();
    }
}
```

---

## 20. Resumo Prático

| Conceito | O que faz | Exemplo |
|----------|-----------|---------|
| **Classe** | Define um tipo de objeto | `class Entrega {}` |
| **Atributo** | Dado armazenado | `private String id` |
| **Construtor** | Inicializa o objeto | `public Entrega(...)` |
| **Getter** | Acessa atributo | `public String getId()` |
| **Setter** | Modifica atributo | `public void setStatus(...)` |
| **Método** | Ação do objeto | `public void finalizar()` |
| **Private** | Apenas dentro da classe | Encapsulamento |
| **Public** | Acessível de qualquer lugar | Interface da classe |
| **Static** | Pertence à classe, não ao objeto | Constantes, utilitários |
| **Herança** | Uma classe estende outra | `extends` |
| **Interface** | Contrato de métodos | `implements` |
| **Sobrecarga** | Mesmo nome, parâmetros diferentes | Múltiplos construtores |
| **@Override** | Reimplementa método da classe pai | Polimorfismo |
| **Encapsulamento** | Controla acesso aos dados | Private + getters/setters |

---

