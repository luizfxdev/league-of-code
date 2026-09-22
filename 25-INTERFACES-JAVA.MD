# Interfaces em Java 


---

## Sumário

1. [O que é uma interface](#1-o-que-é-uma-interface)
2. [Para que serve: baixo acoplamento](#2-para-que-serve-baixo-acoplamento)
3. [Problema exemplo: locação de veículo da frota](#3-problema-exemplo-locação-de-veículo-da-frota)
4. [Acoplamento forte vs fraco](#4-acoplamento-forte-vs-fraco)
5. [Inversão de controle e injeção de dependência](#5-inversão-de-controle-e-injeção-de-dependência)
6. [Exercício: contrato de frete parcelado](#6-exercício-contrato-de-frete-parcelado)
7. [Herdar vs cumprir contrato](#7-herdar-vs-cumprir-contrato)
8. [Interface + classe abstrata](#8-interface--classe-abstrata)
9. [Herança múltipla e o problema do diamante](#9-herança-múltipla-e-o-problema-do-diamante)
10. [Interface Comparable](#10-interface-comparable)
11. [Default methods (Java 8+)](#11-default-methods-java-8)
12. [Outros recursos: static, private e funcionais](#12-outros-recursos-static-private-e-funcionais)
13. [Interface vs classe abstrata vs classe concreta](#13-interface-vs-classe-abstrata-vs-classe-concreta)
14. [Boas práticas e erros comuns](#14-boas-práticas-e-erros-comuns)
15. [Exercícios propostos](#15-exercícios-propostos)
16. [Resumo rápido](#16-resumo-rápido)

---

## 1. O que é uma interface

**Interface é um tipo que define um conjunto de operações que uma classe deve implementar.**

Ela diz **o que** deve ser feito, nunca **como**.

### Analogia logística

Pense em um **contrato de transporte**. O contrato diz:

- "quem transportar esta carga deve **coletar**, **transportar** e **entregar**".

Não importa se quem cumpre é um **caminhão**, uma **van**, uma **moto** ou um **drone**. Todos assinam o mesmo contrato, cada um cumpre do seu jeito.

```java
public interface Transportavel {
    void coletar();
    void transportar();
    void entregar();
}
```

```java
public class Caminhao implements Transportavel {
    @Override
    public void coletar() { System.out.println("Caminhão coletando no CD"); }

    @Override
    public void transportar() { System.out.println("Caminhão na rodovia"); }

    @Override
    public void entregar() { System.out.println("Caminhão descarregando na doca"); }
}
```

```java
public class Moto implements Transportavel {
    @Override
    public void coletar() { System.out.println("Moto coletando na loja"); }

    @Override
    public void transportar() { System.out.println("Moto pelo trânsito urbano"); }

    @Override
    public void entregar() { System.out.println("Moto entregando na porta"); }
}
```

### Regras básicas

| Regra | Detalhe |
|---|---|
| Métodos são `public abstract` por padrão | Não precisa escrever |
| Campos são `public static final` | Só constantes |
| Não instancia | `new Transportavel()` não compila |
| Classe usa `implements` | Pode implementar **várias** |
| Interface usa `extends` | Pode estender **várias** interfaces |
| Classe concreta | **Obrigada** a implementar todos os métodos abstratos |

### Polimorfismo com interface

```java
Transportavel t = new Caminhao();
t.coletar();

t = new Moto();
t.coletar();
```

A variável é do tipo da **interface**; o objeto real é uma classe concreta. O método executado é o da classe concreta (*upcasting* + polimorfismo).

---

## 2. Para que serve: baixo acoplamento

> Criar sistemas **flexíveis** e com **baixo acoplamento**.

A interface estabelece um **contrato** que a classe deve cumprir. Quem *usa* o contrato não precisa saber quem o *cumpre*.

### Analogia

O setor de **expedição** precisa que alguém calcule o frete. Ele não quer saber se é a tabela da transportadora A ou B. Só quer chamar `calcular()`. Se amanhã trocar de transportadora, a expedição **não muda**.

---

## 3. Problema exemplo: locação de veículo da frota

Sua empresa aluga veículos para complementar a frota.

**Regras:**

- Até **12 horas**: cobra por **hora** (arredondando para cima).
- Acima de 12 horas: cobra por **dia** (arredondando para cima).
- Imposto (Brasil): **20%** para valores até 100,00; **15%** para valores acima de 100,00.
- Gerar a fatura: valor básico, imposto e total.

### Exemplo 1

```
Modelo: Sprinter
Retirada: 25/06/2018 10:30
Retorno:  25/06/2018 14:40
Preço/hora: 10.00 | Preço/dia: 130.00

Duração = 4h10 → 5 horas
Básico  = 5 * 10 = 50.00
Imposto = 50 * 20% = 10.00
Total   = 60.00
```

### Exemplo 2

```
Retirada: 25/06/2018 10:30
Retorno:  27/06/2018 11:40

Duração = 2 dias + 1h10 → 3 dias
Básico  = 3 * 130 = 390.00
Imposto = 390 * 15% = 58.50
Total   = 448.50
```

### Modelo de domínio

```
Veiculo 1 ◄──── Locacao ────► 0..1 Fatura
 modelo         retirada         pagamentoBasico
                devolucao        imposto
                                 /pagamentoTotal (calculado)
```

```java
public class Veiculo {
    private final String modelo;

    public Veiculo(String modelo) { this.modelo = modelo; }

    public String getModelo() { return modelo; }
}
```

```java
import java.time.LocalDateTime;

public class Locacao {
    private final LocalDateTime retirada;
    private final LocalDateTime devolucao;
    private final Veiculo veiculo;
    private Fatura fatura;

    public Locacao(LocalDateTime retirada, LocalDateTime devolucao, Veiculo veiculo) {
        this.retirada = retirada;
        this.devolucao = devolucao;
        this.veiculo = veiculo;
    }

    public LocalDateTime getRetirada() { return retirada; }
    public LocalDateTime getDevolucao() { return devolucao; }
    public Veiculo getVeiculo() { return veiculo; }
    public Fatura getFatura() { return fatura; }
    public void setFatura(Fatura fatura) { this.fatura = fatura; }
}
```

```java
public class Fatura {
    private final double pagamentoBasico;
    private final double imposto;

    public Fatura(double pagamentoBasico, double imposto) {
        this.pagamentoBasico = pagamentoBasico;
        this.imposto = imposto;
    }

    public double getPagamentoBasico() { return pagamentoBasico; }
    public double getImposto() { return imposto; }
    public double getPagamentoTotal() { return pagamentoBasico + imposto; }
}
```

### A interface do imposto

O imposto depende do **país**. Hoje é Brasil; amanhã pode ser Argentina. Esse "cálculo de imposto" vira um **contrato**:

```java
public interface ServicoImposto {
    double calcular(double valor);
}
```

```java
public class ImpostoBrasil implements ServicoImposto {
    @Override
    public double calcular(double valor) {
        return valor <= 100.0 ? valor * 0.20 : valor * 0.15;
    }
}
```

### O serviço que gera a fatura

```java
import java.time.Duration;

public class ServicoLocacao {
    private final double precoPorHora;
    private final double precoPorDia;
    private final ServicoImposto servicoImposto;

    public ServicoLocacao(double precoPorHora, double precoPorDia, ServicoImposto servicoImposto) {
        this.precoPorHora = precoPorHora;
        this.precoPorDia = precoPorDia;
        this.servicoImposto = servicoImposto;
    }

    public void gerarFatura(Locacao locacao) {
        double minutos = Duration.between(locacao.getRetirada(), locacao.getDevolucao()).toMinutes();
        double horas = minutos / 60.0;

        double pagamentoBasico = horas <= 12.0
                ? precoPorHora * Math.ceil(horas)
                : precoPorDia * Math.ceil(horas / 24.0);

        double imposto = servicoImposto.calcular(pagamentoBasico);
        locacao.setFatura(new Fatura(pagamentoBasico, imposto));
    }
}
```

### Programa principal

```java
import java.time.LocalDateTime;

public class Program {
    public static void main(String[] args) {
        Locacao locacao = new Locacao(
                LocalDateTime.of(2018, 6, 25, 10, 30),
                LocalDateTime.of(2018, 6, 25, 14, 40),
                new Veiculo("Sprinter"));

        ServicoLocacao servico = new ServicoLocacao(10.0, 130.0, new ImpostoBrasil());
        servico.gerarFatura(locacao);

        Fatura f = locacao.getFatura();
        System.out.printf("Pagamento básico: %.2f%n", f.getPagamentoBasico());
        System.out.printf("Imposto: %.2f%n", f.getImposto());
        System.out.printf("Pagamento total: %.2f%n", f.getPagamentoTotal());
    }
}
```

Saída:

```
Pagamento básico: 50.00
Imposto: 10.00
Pagamento total: 60.00
```

---

## 4. Acoplamento forte vs fraco

### Sem interface (acoplamento forte)

```
ServicoLocacao ───────► ImpostoBrasil
```

```java
public class ServicoLocacao {
    private ImpostoBrasil servicoImposto;
}
```

- `ServicoLocacao` **conhece** a classe concreta.
- Se a classe concreta mudar, é preciso **mudar** `ServicoLocacao`.
- Para operar em outro país, é preciso **reescrever** o serviço.

### Com interface (acoplamento fraco)

```
ServicoLocacao ───────► «interface» ServicoImposto
                                ▲
                                ┆ implements
                          ImpostoBrasil
```

```java
public class ServicoLocacao {
    private ServicoImposto servicoImposto;
}
```

- `ServicoLocacao` **não conhece** a implementação.
- Se a classe concreta mudar, `ServicoLocacao` **não muda nada**.
- Para outro país, cria-se `ImpostoArgentina implements ServicoImposto` e injeta-se no lugar.

### Analogia

| | Analogia logística |
|---|---|
| Forte | A expedição só trabalha com **uma** transportadora específica; se ela fechar, o setor para |
| Fraco | A expedição só exige "quem cumpra o contrato de frete"; troca de transportadora sem mudar processo |

---

## 5. Inversão de controle e injeção de dependência

### Inversão de controle (IoC)

> Retirar da classe a **responsabilidade de instanciar** suas dependências.

**Sem IoC:** a classe cria a dependência por conta própria.

```java
public class ServicoLocacao {
    private ServicoImposto servicoImposto = new ImpostoBrasil();
}
```

Aqui ela decide sozinha qual imposto usar, e voltamos ao acoplamento forte.

**Com IoC:** alguém de fora entrega a dependência pronta.

### Injeção de dependência (DI)

> Forma de realizar IoC: um componente **externo** instancia a dependência e a **injeta** no objeto "pai".

Formas de fazer:

| Forma | Como |
|---|---|
| **Construtor** | Recebe pelo construtor (a mais recomendada) |
| **Factory / Builder** | Uma classe de instanciação monta os objetos |
| **Container / framework** | Spring, CDI etc. |

### Injeção por construtor

```java
public ServicoLocacao(double precoPorHora, double precoPorDia, ServicoImposto servicoImposto) {
    this.precoPorHora = precoPorHora;
    this.precoPorDia = precoPorDia;
    this.servicoImposto = servicoImposto;
}
```

```java
ServicoLocacao servico = new ServicoLocacao(10.0, 130.0, new ImpostoBrasil());
```

O `new ImpostoBrasil()` passado onde se espera `ServicoImposto` é um **upcasting**.

### Analogia

A **empresa de logística** não fabrica a própria frota. Quem monta a operação (o gerente, o `main`) escolhe a transportadora e **entrega** para a operação. A operação só usa.

---

## 6. Exercício: contrato de frete parcelado

Uma empresa de logística fecha **contratos de frete** com clientes e parcela o pagamento. O pagamento é feito por um serviço online que cobra:

- **juros simples de 1% ao mês**, por parcela (parcela 1 = 1%, parcela 2 = 2%...);
- **taxa de pagamento de 2%** sobre o valor já com juros.

O programa lê o número do contrato, a data e o valor total, depois o número de parcelas, e gera as parcelas (a 1ª vence 1 mês após a data do contrato, a 2ª dois meses, etc.).

### Exemplo

```
Numero: 8028
Data: 25/06/2018
Valor do contrato: 600.00
Parcelas: 3

25/07/2018 - 206.04
25/08/2018 - 208.08
25/09/2018 - 210.12
```

**Cálculo** (cota = 600 / 3 = 200):

| Parcela | Juros simples | + Taxa 2% |
|---|---|---|
| 1 | 200 + 1% × 1 = 202 | 202 + 2% = **206.04** |
| 2 | 200 + 1% × 2 = 204 | 204 + 2% = **208.08** |
| 3 | 200 + 1% × 3 = 206 | 206 + 2% = **210.12** |

### Entidades

```
Contrato ──────────► * Parcela
 numero    parcelas     dataVencimento
 data                   valor
 valorTotal
```

```java
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

public class Contrato {
    private final int numero;
    private final LocalDate data;
    private final double valorTotal;
    private final List<Parcela> parcelas = new ArrayList<>();

    public Contrato(int numero, LocalDate data, double valorTotal) {
        this.numero = numero;
        this.data = data;
        this.valorTotal = valorTotal;
    }

    public int getNumero() { return numero; }
    public LocalDate getData() { return data; }
    public double getValorTotal() { return valorTotal; }
    public List<Parcela> getParcelas() { return parcelas; }
}
```

```java
import java.time.LocalDate;

public class Parcela {
    private final LocalDate dataVencimento;
    private final double valor;

    public Parcela(LocalDate dataVencimento, double valor) {
        this.dataVencimento = dataVencimento;
        this.valor = valor;
    }

    public LocalDate getDataVencimento() { return dataVencimento; }
    public double getValor() { return valor; }
}
```

### Interface do serviço de pagamento

```
ServicoContrato ───► «interface» ServicoPagamentoOnline
                            ▲
                            ┆
                       ServicoPaypal
```

```java
public interface ServicoPagamentoOnline {
    double taxaPagamento(double valor);
    double juros(double valor, int meses);
}
```

```java
public class ServicoPaypal implements ServicoPagamentoOnline {
    @Override
    public double taxaPagamento(double valor) {
        return valor * 0.02;
    }

    @Override
    public double juros(double valor, int meses) {
        return valor * 0.01 * meses;
    }
}
```

Entendendo a interface:

```java
ServicoPagamentoOnline s = new ServicoPaypal();
s.juros(200, 1);        // 2.0
s.juros(200, 2);        // 4.0
s.juros(200, 3);        // 6.0
s.taxaPagamento(202);   // 4.04
s.taxaPagamento(204);   // 4.08
s.taxaPagamento(206);   // 4.12
```

### O que `processarContrato` faz

```
contrato (sem parcelas)  ──processarContrato(contrato, 3)──►  contrato + 3 parcelas
```

```java
public class ServicoContrato {
    private final ServicoPagamentoOnline servicoPagamento;

    public ServicoContrato(ServicoPagamentoOnline servicoPagamento) {
        this.servicoPagamento = servicoPagamento;
    }

    public void processarContrato(Contrato contrato, int meses) {
        double cota = contrato.getValorTotal() / meses;

        for (int i = 1; i <= meses; i++) {
            double comJuros = cota + servicoPagamento.juros(cota, i);
            double total = comJuros + servicoPagamento.taxaPagamento(comJuros);
            contrato.getParcelas().add(new Parcela(contrato.getData().plusMonths(i), total));
        }
    }
}
```

### Programa principal

```java
import java.time.LocalDate;

public class Program {
    public static void main(String[] args) {
        Contrato contrato = new Contrato(8028, LocalDate.of(2018, 6, 25), 600.00);

        ServicoContrato servico = new ServicoContrato(new ServicoPaypal());
        servico.processarContrato(contrato, 3);

        contrato.getParcelas().forEach(p ->
                System.out.printf("%s - %.2f%n", p.getDataVencimento(), p.getValor()));
    }
}
```

### Ganho real

Se a empresa trocar o gateway (ex.: Pix, boleto, outro provedor), basta criar uma nova classe `implements ServicoPagamentoOnline` e passar no `main`. **`ServicoContrato` não é alterado.**

---

## 7. Herdar vs cumprir contrato

### O que têm em comum

- Relação **"é-um"**
- **Generalização / especialização**
- **Polimorfismo**

```
Veiculo (classe)                     «interface» ServicoImposto
   ▲          ▲                             ▲              ▲
Caminhao    Moto                      ImpostoBrasil   ImpostoArgentina
```

### Diferença fundamental

| | Objetivo |
|---|---|
| **Herança** | **Reuso** de código e estrutura |
| **Interface** | **Contrato** a ser cumprido |

### Regra prática

- `Caminhao` **é um** `Veiculo` → herança (reaproveita placa, capacidade, etc.).
- `Caminhao` **é** `Rastreavel` (sabe informar posição) → interface (capacidade, não estrutura).

```java
public abstract class Veiculo {
    private final String placa;

    protected Veiculo(String placa) { this.placa = placa; }

    public String getPlaca() { return placa; }

    public abstract double capacidadeKg();
}
```

```java
public interface Rastreavel {
    double latitude();
    double longitude();
}
```

```java
public class Caminhao extends Veiculo implements Rastreavel {
    public Caminhao(String placa) { super(placa); }

    @Override public double capacidadeKg() { return 12_000; }
    @Override public double latitude() { return -23.55; }
    @Override public double longitude() { return -46.63; }
}
```

Um **container refrigerado** também pode ser `Rastreavel` sem ser um `Veiculo`. Interface permite isso; herança não.

---

## 8. Interface + classe abstrata

**Problema:** quero um contrato (interface) **e** também uma estrutura/lógica comum reutilizável para as implementações.

**Solução:** interface define o contrato; classe abstrata implementa parte dele e reaproveita código; classes concretas completam o resto.

```
«interface» ServicoNotificacao
           ▲
           ┆
ServicoNotificacaoAbstrato   (implementa o que é comum)
      ▲               ▲
NotificacaoSms   NotificacaoMock
```

### Exemplo: avisar o cliente sobre a entrega

```java
public record Entrega(String codigo, String destino) { }
```

```java
public interface ServicoNotificacao {
    void enviar(Entrega entrega, String mensagem);
    void notificarEntregaConcluida(Entrega entrega);
}
```

```java
public abstract class ServicoNotificacaoAbstrato implements ServicoNotificacao {
    @Override
    public void notificarEntregaConcluida(Entrega entrega) {
        enviar(entrega, montarMensagem(entrega));
    }

    protected String montarMensagem(Entrega entrega) {
        return "Pedido " + entrega.codigo() + " entregue em " + entrega.destino() + ".";
    }
}
```

```java
public class NotificacaoSms extends ServicoNotificacaoAbstrato {
    @Override
    public void enviar(Entrega entrega, String mensagem) {
        System.out.println("[SMS] " + mensagem);
    }
}
```

```java
public class NotificacaoMock extends ServicoNotificacaoAbstrato {
    @Override
    public void enviar(Entrega entrega, String mensagem) {
        System.out.println("[MOCK] " + mensagem);
    }
}
```

```java
ServicoNotificacao servico = new NotificacaoMock();
servico.notificarEntregaConcluida(new Entrega("PED-1001", "Campinas/SP"));
```

- Em **desenvolvimento/testes** injeta-se `NotificacaoMock`.
- Em **produção** injeta-se `NotificacaoSms`.
- A regra de montar a mensagem fica **uma vez só** na classe abstrata.

---

## 9. Herança múltipla e o problema do diamante

### O problema

Herança múltipla de **classes** pode gerar **ambiguidade** quando o mesmo método existe em mais de uma superclasse.

```
            Equipamento
            processar(doc)
             ▲          ▲
          Leitor     Impressora        (ambos sobrescrevem processar)
             ▲          ▲
          ColetorEtiquetadora          → qual processar() herdar?
```

Por isso, **Java não permite herdar mais de uma classe**.

### A saída: implementar várias interfaces

Uma classe **pode** implementar quantas interfaces quiser.

> Isso **não** é herança múltipla, porque **não há reuso** de código entre a classe e as interfaces. A classe não herda, ela **cumpre o contrato**.

### Exemplo: coletor de dados que lê código de barras e imprime etiqueta

```
        Equipamento (abstrata)         «interface» Leitor        «interface» Impressora
                ▲                            ▲                          ▲
                └──────────── ColetorEtiquetadora ──────────────────────┘
```

```java
public abstract class Equipamento {
    private final String numeroSerie;

    protected Equipamento(String numeroSerie) { this.numeroSerie = numeroSerie; }

    public String getNumeroSerie() { return numeroSerie; }

    public abstract void processar(String doc);
}
```

```java
public interface Leitor {
    String ler();
}
```

```java
public interface Impressora {
    void imprimir(String doc);
}
```

```java
public class ColetorEtiquetadora extends Equipamento implements Leitor, Impressora {
    public ColetorEtiquetadora(String numeroSerie) { super(numeroSerie); }

    @Override
    public void processar(String doc) { System.out.println("Processando: " + doc); }

    @Override
    public String ler() { return "7891234567890"; }

    @Override
    public void imprimir(String doc) { System.out.println("Etiqueta impressa: " + doc); }
}
```

```java
ColetorEtiquetadora c = new ColetorEtiquetadora("SN-001");
Leitor leitor = c;
Impressora impressora = c;

impressora.imprimir(leitor.ler());
```

Um mesmo objeto pode ser visto **como Leitor** ou **como Impressora**, dependendo do que quem o usa precisa.

---

## 10. Interface Comparable

Interface pronta da linguagem, usada para definir **ordem natural** entre objetos.

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

### Retorno de `compareTo`

| Retorno | Significado |
|---|---|
| Negativo | `this` é **menor** que `o` |
| Zero | São **iguais** |
| Positivo | `this` é **maior** que `o` |

```java
System.out.println("maria".compareTo("alex"));   // 12
System.out.println("alex".compareTo("maria"));   // -12
System.out.println("maria".compareTo("maria"));  // 0
```

### Problema 1: ordenar nomes de motoristas

Arquivo `motoristas.txt`:

```
Maria Brown
Alex Green
Bob Grey
Anna White
```

`String` já implementa `Comparable`, então funciona direto:

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class Program {
    public static void main(String[] args) {
        List<String> lista = new ArrayList<>();
        String caminho = "C:\\temp\\motoristas.txt";

        try (BufferedReader br = new BufferedReader(new FileReader(caminho))) {
            String nome = br.readLine();
            while (nome != null) {
                lista.add(nome);
                nome = br.readLine();
            }
            Collections.sort(lista);
            lista.forEach(System.out::println);
        } catch (IOException e) {
            System.out.println("Erro: " + e.getMessage());
        }
    }
}
```

### Problema 2: ordenar motoristas (nome, salário) de um CSV

Arquivo `motoristas.csv`:

```
Maria Brown,4300.00
Alex Green,3100.00
Bob Grey,3100.00
Anna White,3500.00
```

`Collections.sort(lista)` **não compila** para uma classe sua, a menos que ela implemente `Comparable`:

```java
public class Motorista implements Comparable<Motorista> {
    private String nome;
    private Double salario;

    public Motorista(String nome, Double salario) {
        this.nome = nome;
        this.salario = salario;
    }

    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
    public Double getSalario() { return salario; }
    public void setSalario(Double salario) { this.salario = salario; }

    @Override
    public int compareTo(Motorista outro) {
        return nome.compareTo(outro.getNome());
    }
}
```

```java
public class Program {
    public static void main(String[] args) {
        List<Motorista> lista = new ArrayList<>();
        String caminho = "C:\\temp\\motoristas.csv";

        try (BufferedReader br = new BufferedReader(new FileReader(caminho))) {
            String linha = br.readLine();
            while (linha != null) {
                String[] campos = linha.split(",");
                lista.add(new Motorista(campos[0], Double.parseDouble(campos[1])));
                linha = br.readLine();
            }
            Collections.sort(lista);
            for (Motorista m : lista) {
                System.out.println(m.getNome() + ", " + m.getSalario());
            }
        } catch (IOException e) {
            System.out.println("Erro: " + e.getMessage());
        }
    }
}
```

O `Collections.sort` só precisa saber que o objeto **cumpre o contrato** `Comparable`. Ele chama `compareTo` sem conhecer `Motorista`. É baixo acoplamento na prática.

### E se eu precisar de mais de um critério?

`Comparable` define **uma** ordem natural. Para outras ordens, use `Comparator`:

```java
lista.sort(Comparator.comparing(Motorista::getSalario).reversed());

lista.sort(Comparator.comparing(Motorista::getSalario)
        .thenComparing(Motorista::getNome));
```

| | `Comparable` | `Comparator` |
|---|---|---|
| Onde fica | Dentro da própria classe | Fora, em qualquer lugar |
| Quantidade | Uma ordem natural | Quantas quiser |
| Exemplo logístico | Entrega ordenada por prazo | Entregas por peso, por cliente, por rota |

---

## 11. Default methods (Java 8+)

### Antes do Java 8

Interface só tinha métodos **abstratos**. Toda classe repetia a mesma implementação, ou era preciso criar uma classe abstrata só para reaproveitar código.

### A partir do Java 8

Interfaces podem ter **métodos concretos** com a palavra `default` (*defender methods*).

**Vantagens:**

1. Evita repetir implementação em toda classe.
2. Evita criar classe abstrata só para reuso.
3. Mantém **retrocompatibilidade** (dá para adicionar método novo em interface existente sem quebrar quem já a implementa).
4. Permite que **interfaces funcionais** (com um único método abstrato) ofereçam operações padrão reutilizáveis.

### Problema exemplo: juros sobre fatura de frete

Cliente atrasou o pagamento do frete. Calcular o valor a pagar após N meses, com **juros compostos**:

```
pagamento = valor * (1 + taxa / 100) ^ meses
```

- **Brasil:** 2% ao mês. `200 * 1.02³ = 212.2416` → **212.24**
- **EUA:** 1% ao mês. `200 * 1.01³ = 206.0602` → **206.06**

A **fórmula é a mesma**, só muda a **taxa**. Sem `default`, cada país repetiria a fórmula. Com `default`, ela fica **na interface**:

```java
public interface ServicoJuros {
    double getTaxaJuros();

    default double calcularPagamento(double valor, int meses) {
        if (meses < 1) {
            throw new IllegalArgumentException("Meses deve ser maior ou igual a 1");
        }
        return valor * Math.pow(1.0 + getTaxaJuros() / 100.0, meses);
    }
}
```

```java
public class JurosBrasil implements ServicoJuros {
    @Override
    public double getTaxaJuros() { return 2.0; }
}
```

```java
public class JurosEUA implements ServicoJuros {
    @Override
    public double getTaxaJuros() { return 1.0; }
}
```

```java
ServicoJuros brasil = new JurosBrasil();
ServicoJuros eua = new JurosEUA();

System.out.printf("%.2f%n", brasil.calcularPagamento(200.0, 3));  // 212.24
System.out.printf("%.2f%n", eua.calcularPagamento(200.0, 3));     // 206.06
```

Cada país só informa a **taxa**; a regra de cálculo é **reaproveitada**.

### Considerações importantes

- Agora interfaces **podem prover reuso**.
- Agora existe uma forma de **herança múltipla de comportamento**.
- Se duas interfaces tiverem `default` com a **mesma assinatura**, o compilador **exige** que a classe sobrescreva o método.

#### Resolvendo conflito de default

```java
public interface Rastreavel {
    default String identificar() { return "GPS"; }
}

public interface Identificavel {
    default String identificar() { return "RFID"; }
}
```

```java
public class Carga implements Rastreavel, Identificavel {
    @Override
    public String identificar() {
        return Rastreavel.super.identificar() + " + " + Identificavel.super.identificar();
    }
}
```

`Interface.super.metodo()` escolhe explicitamente qual implementação chamar.

### Interfaces ainda são diferentes de classes abstratas

Interfaces **não possuem**:

- Construtores
- Atributos de instância (só constantes `static final`)
- Estado próprio

Se você precisa de **estado** (ex.: `placa`, `numeroSerie`) ou **construtor**, use classe abstrata.

---

## 12. Outros recursos: static, private e funcionais

### Método `static` em interface (Java 8)

Pertence à interface, não ao objeto. Chamado como `Interface.metodo()`.

```java
public interface ServicoFrete {
    double calcular(double pesoKg, double distanciaKm);

    static ServicoFrete padrao() {
        return (peso, km) -> peso * 0.05 + km * 1.2;
    }
}
```

```java
double valor = ServicoFrete.padrao().calcular(500, 300);
```

### Método `private` em interface (Java 9)

Serve para **dividir lógica** entre vários `default`, sem expor.

```java
public interface ServicoJuros {
    double getTaxaJuros();

    default double calcularPagamento(double valor, int meses) {
        validar(meses);
        return valor * fator(meses);
    }

    private void validar(int meses) {
        if (meses < 1) throw new IllegalArgumentException("meses >= 1");
    }

    private double fator(int meses) {
        return Math.pow(1.0 + getTaxaJuros() / 100.0, meses);
    }
}
```

### Interface funcional

Interface com **exatamente um** método abstrato. Pode ser implementada com **lambda**.

```java
@FunctionalInterface
public interface RegraFrete {
    double calcular(double pesoKg, double distanciaKm);

    default RegraFrete comDesconto(double percentual) {
        return (peso, km) -> calcular(peso, km) * (1 - percentual / 100.0);
    }
}
```

```java
RegraFrete regra = (peso, km) -> peso * 0.05 + km * 1.2;
RegraFrete comDesconto = regra.comDesconto(10);

System.out.println(comDesconto.calcular(500, 300));
```

Repare como o `default` deu uma operação extra reutilizável a uma interface funcional (vantagem 4 da seção anterior).

---

## 13. Interface vs classe abstrata vs classe concreta

| Característica | Interface | Classe abstrata | Classe concreta |
|---|---|---|---|
| Instanciável | Não | Não | Sim |
| Métodos abstratos | Sim | Sim | Não |
| Métodos com corpo | `default`, `static`, `private` | Sim | Sim |
| Atributos de instância | **Não** (só constantes) | Sim | Sim |
| Construtor | **Não** | Sim | Sim |
| Herda/estende | Várias interfaces (`extends`) | **Uma** classe | **Uma** classe |
| Implementa | — | Várias interfaces | Várias interfaces |
| Foco | **Contrato** | Reuso + contrato parcial | Implementação completa |
| Analogia | Edital/contrato de transporte | Modelo-base de veículo | Caminhão específico |

### Quando usar o quê

| Situação | Use |
|---|---|
| Definir **o que** algo deve fazer, sem se importar com quem faz | **Interface** |
| Trocar implementações (país, gateway, transportadora) | **Interface** |
| Classes compartilham **estado** e **lógica** | **Classe abstrata** |
| Contrato + código comum | **Interface + classe abstrata** |
| Dar capacidade a tipos sem relação (rastreável, imprimível) | **Interface** |
| Precisa de construtor ou atributos | **Classe abstrata** |

---

## 14. Boas práticas e erros comuns

### Boas práticas

- **Programe para a interface**, não para a implementação:

  ```java
  ServicoImposto imposto = new ImpostoBrasil();
  List<Motorista> lista = new ArrayList<>();
  ```

- Prefira **injeção por construtor** e campos `final`.
- Interfaces **pequenas e coesas** (*Interface Segregation*): `Leitor` e `Impressora` separadas, em vez de uma `DispositivoTudoEmUm`.
- Nomeie pela **capacidade** (`Rastreavel`, `Comparable`) ou pelo **papel** (`ServicoImposto`).
- Use `@Override` sempre.
- Use `default` para **evoluir** interfaces existentes, não para transformá-las em classes.
- Em produção, para dinheiro use **`BigDecimal`** em vez de `double` (os exemplos usam `double` para ficar didático e bater com o material).

### Erros comuns

| Erro | Causa | Correção |
|---|---|---|
| `class X is not abstract and does not override abstract method` | Faltou implementar método da interface | Implementar todos ou declarar `X` como `abstract` |
| `attempting to assign weaker access privileges` | Implementou sem `public` | Métodos de interface são `public`; a implementação também |
| `types A and B are incompatible; both define default method` | Conflito de `default` | Sobrescrever e usar `A.super.metodo()` |
| `Interface is abstract; cannot be instantiated` | `new Interface()` | Instanciar uma classe concreta |
| `Collections.sort` não compila para sua classe | Não implementa `Comparable` | Implementar `Comparable<T>` ou passar um `Comparator` |
| Classe cria a própria dependência com `new` | Acoplamento forte | Receber a interface pelo construtor |

---

## 15. Exercícios propostos

1. **Frete por modal.** Crie a interface `CalculadoraFrete` com `double calcular(double pesoKg, double distanciaKm)`. Implemente `FreteRodoviario`, `FreteAereo` e `FreteFluvial` com regras diferentes. Escreva um `ServicoCotacao` que receba a calculadora pelo construtor.

2. **Imposto por estado.** Reaproveite `ServicoImposto` e crie `ImpostoSP` e `ImpostoMG` com alíquotas diferentes. Rode a locação do exemplo com ambos, sem alterar `ServicoLocacao`.

3. **Rastreamento.** Crie as interfaces `Rastreavel` (posição) e `Refrigerado` (temperatura). Crie `Caminhao extends Veiculo implements Rastreavel` e `ContainerFrio implements Rastreavel, Refrigerado`. Monte uma `List<Rastreavel>` e imprima as posições.

4. **Ordenação de entregas.** Faça `Entrega implements Comparable<Entrega>` ordenando por data prevista. Depois use `Comparator` para ordenar por peso decrescente e, em caso de empate, por código.

5. **Default method.** Crie `ServicoMulta` com `getPercentualDiario()` e um `default` que calcula a multa de atraso: `valor * percentualDiario/100 * diasAtraso`. Implemente `MultaPadrao` (0,5%) e `MultaContratoPremium` (0,2%).

6. **Abstrata + interface.** Crie `ServicoNotificacao` com `enviar()` e `notificarSaidaParaEntrega()`. Faça `NotificacaoEmail` e `NotificacaoWhatsApp` herdarem de uma classe abstrata que monta a mensagem padrão.

7. **Conflito de default.** Crie duas interfaces com `default String descricao()` e uma classe que implemente ambas resolvendo o conflito com `Interface.super`.

---

## 16. Resumo rápido

| Conceito | Em uma frase | Logística |
|---|---|---|
| **Interface** | Contrato: define **o quê**, não **como** | "Quem transportar deve coletar, transportar e entregar" |
| **`implements`** | A classe promete cumprir o contrato | `Caminhao implements Transportavel` |
| **Baixo acoplamento** | Usar o contrato sem conhecer quem cumpre | Expedição não depende de uma transportadora específica |
| **IoC** | A classe não instancia suas dependências | A operação recebe a transportadora escolhida |
| **DI por construtor** | Dependência entra pelo construtor | `new ServicoLocacao(..., new ImpostoBrasil())` |
| **Herança** | Reuso de código/estrutura | `Caminhao extends Veiculo` |
| **Interface** | Contrato a cumprir | `Caminhao implements Rastreavel` |
| **Diamante** | Ambiguidade na herança múltipla de classes | Java só permite **uma** superclasse |
| **Várias interfaces** | Uma classe cumpre vários contratos | `ColetorEtiquetadora implements Leitor, Impressora` |
| **`Comparable`** | Define a ordem natural | Ordenar motoristas por nome |
| **`Comparator`** | Ordens alternativas, fora da classe | Ordenar por salário, peso, prazo |
| **`default`** | Método concreto na interface | Fórmula de juros compartilhada; cada país informa a taxa |
| **Interface funcional** | Um só método abstrato, aceita lambda | `RegraFrete` |

### Fluxo mental para decidir

```
Preciso de um contrato que várias classes cumpram?         → Interface
Além do contrato, preciso de estado/lógica compartilhada?  → Interface + classe abstrata
Preciso trocar a implementação sem mexer em quem usa?      → Interface + injeção por construtor
Preciso ordenar objetos meus?                              → Comparable (ou Comparator)
Preciso de um comportamento padrão na interface?           → default
```