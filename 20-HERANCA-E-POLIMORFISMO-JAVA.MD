# Herança e Polimorfismo em Java ? Guia Prático com League of Legends

## 1. Herança (Inheritance)

**Conceito:** Uma classe herda todos os dados e comportamentos de outra.

**Analogia LoL:**
Imagine campeões especializados que herdam de um "Campeão Base":
- **Campeão Base** tem: atributos genéricos (HP, mana, velocidade, AD)
- **Campeão Top** herda tudo e adiciona: resistência a CC
- **Campeão ADC** herda tudo e adiciona: alcance de ataque

### Sintaxe Básica

```java
// Superclasse (classe base)
public class Campeao {
    protected String nome;
    protected int hp;
    protected double dano;
    
    public Campeao(String nome, int hp, double dano) {
        this.nome = nome;
        this.hp = hp;
        this.dano = dano;
    }
    
    public void atacar() {
        System.out.println(nome + " ataca com " + dano + " de dano!");
    }
}

// Subclasse (classe derivada)
public class Mago extends Campeao {
    private double poderMagico;
    
    public Mago(String nome, int hp, double dano, double poderMagico) {
        super(nome, hp, dano); // Chama construtor da superclasse
        this.poderMagico = poderMagico;
    }
    
    public void lancarUltimate() {
        System.out.println(nome + " lança ultimate com " + poderMagico + " de poder!");
    }
}
```

### O que foi Herdado?

```java
Mago ahri = new Mago("Ahri", 500, 80.0, 150.0);

// Ahri herdou isso:
ahri.nome;        // "Ahri"
ahri.hp;          // 500
ahri.dano;        // 80.0
ahri.atacar();    // Método da superclasse

// Ahri adicionou isso:
ahri.poderMagico; // 150.0
ahri.lancarUltimate(); // Método próprio
```

---

## 2. Modificadores de Acesso: `public`, `private`, `protected`

| Modificador | Mesma Classe | Mesmo Pacote | Subclasse | Outro Pacote |
|-------------|--------------|--------------|-----------|--------------|
| `public` | ? | ? | ? | ? |
| `protected` | ? | ? | ? | ? |
| (padrão) | ? | ? | ? | ? |
| `private` | ? | ? | ? | ? |

### Por Que `protected`?

```java
public class Conta {
    private double saldo;      // Privado: só Conta pode acessar
    
    protected void sacar(double valor) {  // Subclasses podem usar
        saldo -= valor;
    }
}

public class ContaPoupanca extends Conta {
    @Override
    protected void sacar(double valor) {
        // Pode sobrepor o método protegido
        super.sacar(valor);     // Chama a versão da superclasse
        System.out.println("Taxa de saque: R$ 2.00");
    }
}
```

Se `sacar()` fosse **private**, `ContaPoupanca` não conseguiria **sobrepor** o método.
Se fosse **public**, qualquer classe poderia acessar. **protected** é o meio termo.

---

## 3. Upcasting e Downcasting

### Upcasting ? Subclasse ? Superclasse (Seguro ?)

```java
Mago ahri = new Mago("Ahri", 500, 80.0, 150.0);

// Upcasting: automático, sempre seguro
Campeao c = ahri;  // Ahri é um Campeao

c.atacar();  // Ok - Campeao tem atacar()
// c.lancarUltimate(); // ERRO - Campeao não tem este método
```

**Analogia:** Um Mago **é um** Campeão. Sumir com os atributos mágicos não quebra o contrato.

### Downcasting ? Superclasse ? Subclasse (Perigoso ??)

```java
Campeao c = new Mago("Ahri", 500, 80.0, 150.0);

// Downcasting: precisa de cast explícito
Mago mago = (Mago) c;  // Cast explícito
mago.lancarUltimate(); // Ok agora

// MAS CUIDADO:
Campeao c2 = new Campeao("Teste", 100, 50.0); // Não é Mago!
Mago mago2 = (Mago) c2; // CRASH! ClassCastException em runtime
```

### Usando `instanceof` para Verificar

```java
Campeao c = new Mago("Ahri", 500, 80.0, 150.0);

if (c instanceof Mago) {
    Mago mago = (Mago) c;
    mago.lancarUltimate(); // Seguro!
}

if (c instanceof Guerreiro) {
    Guerreiro g = (Guerreiro) c;
    g.ativarEscudo(); // Não executa
}
```

---

## 4. Sobrescrita (Override) de Métodos

Quando uma subclasse oferece sua própria implementação de um método da superclasse.

### Exemplo: Saque com Taxa

```java
public class Conta {
    protected double saldo = 1000.0;
    
    public void sacar(double valor) {
        saldo -= valor + 5.0;  // Taxa de R$ 5.00
        System.out.println("Saque: " + valor + ", Taxa: 5.00");
    }
}

public class ContaPoupanca extends Conta {
    @Override  // Anotação: informa ao compilador que está sobrescrevendo
    public void sacar(double valor) {
        saldo -= valor;  // Sem taxa!
        System.out.println("Saque: " + valor + ", Taxa: NENHUMA");
    }
}
```

**Teste:**

```java
Conta c1 = new Conta();
c1.sacar(100.0);  // Saque: 100.0, Taxa: 5.00

Conta c2 = new ContaPoupanca();
c2.sacar(100.0);  // Saque: 100.0, Taxa: NENHUMA
```

### A Palavra `super`

Acessa a implementação da superclasse.

```java
public class ContaEmpresa extends Conta {
    @Override
    public void sacar(double valor) {
        super.sacar(valor);         // Chama o saque normal da superclasse
        saldo -= 2.0;               // Cobra taxa adicional
        System.out.println("Taxa empresarial: 2.00");
    }
}
```

---

## 5. Classe `final` e Método `final`

### Classe `final` ? Não pode ser Herdada

```java
public final class ContaPoupanca {
    // Impede que surjam subclasses de ContaPoupanca
}

// Isto gera ERRO de compilação:
public class ContaPoupancaPremium extends ContaPoupanca {
    // ERRO!
}
```

**Por quê usar?**
- Segurança: Garante que ninguém mude o comportamento crítico
- Performance: Compilador otimiza métodos `final`
- Exemplo real: String é `final` (protege dados)

### Método `final` ? Não pode ser Sobreposto

```java
public class Conta {
    public final void transferir(double valor, Conta destino) {
        // Implementação crítica que NÃO deve ser alterada
    }
}

public class ContaPoupanca extends Conta {
    @Override
    public final void transferir(double valor, Conta destino) {
        // ERRO! Método é final
    }
}
```

---

## 6. Polimorfismo

**Definição:** Uma variável de tipo genérico (superclasse) pode apontar para objetos de tipos específicos (subclasses), tendo comportamentos diferentes.

### Exemplo Clássico

```java
List<Conta> contas = new ArrayList<>();

contas.add(new Conta());           // Conta comum
contas.add(new ContaPoupanca());   // Poupança
contas.add(new ContaEmpresa());    // Empresa

// Tratamento uniforme, mas comportamento diferente:
for (Conta c : contas) {
    c.sacar(100.0);  // Cada uma saca de forma diferente!
}

// Saída:
// Saque: 100.0, Taxa: 5.00
// Saque: 100.0, Taxa: NENHUMA
// Saque: 100.0, Taxa empresarial: 2.00
```

**Por que é poderoso?**

Você não precisa saber o tipo específico de cada conta. Apenas chama `c.sacar()` e Java descobre automaticamente qual implementação usar. Isso é **polimorfismo em ação**.

---

## 7. Classes Abstratas

Classe que **não pode ser instanciada**, apenas herdada.

```java
public abstract class Veiculo {
    protected String placa;
    protected double velocidade;
    
    public Veiculo(String placa) {
        this.placa = placa;
    }
    
    // Método normal
    public void buzinar() {
        System.out.println("Bibi!");
    }
    
    // Método abstrato (sem implementação)
    public abstract void acelerar();
}

// Subclasses DEVEM implementar os métodos abstratos
public class Carro extends Veiculo {
    @Override
    public void acelerar() {
        velocidade += 20.0;
        System.out.println("Carro acelerando: " + velocidade);
    }
}

public class Bicicleta extends Veiculo {
    @Override
    public void acelerar() {
        velocidade += 5.0;
        System.out.println("Bicicleta pedalando: " + velocidade);
    }
}
```

### Tentativas Inválidas

```java
// Isto é ERRO:
Veiculo v = new Veiculo("ABC-1234");  // ERRO! Classe abstrata não pode ser instanciada

// Isto é OK:
Veiculo v = new Carro("ABC-1234");    // Ok! Carro é concreto
Veiculo v2 = new Bicicleta("XYZ-9999"); // Ok! Bicicleta é concreto
```

### Por Que Abstratas?

**Cenário:** Um sistema de banco com contas poupança e empresarial.

```java
// Sem abstração (RUIM):
Conta c = new Conta();  // O que é uma "conta comum"? Ambíguo!

// Com abstração (BOM):
// Conta é abstrata ? força criar ContaPoupanca ou ContaEmpresa
```

---

## 8. Métodos Abstratos

Métodos **sem implementação** na superclasse. Subclasses **obrigadas** a implementar.

```java
public abstract class Funcionario {
    protected String nome;
    
    public Funcionario(String nome) {
        this.nome = nome;
    }
    
    // Método concreto
    public void apresentar() {
        System.out.println("Olá, sou " + nome);
    }
    
    // Método abstrato - subclasses DEVEM implementar
    public abstract double calcularSalario();
}

public class FuncionarioComum extends Funcionario {
    private double salarioBase;
    
    public FuncionarioComum(String nome, double salarioBase) {
        super(nome);
        this.salarioBase = salarioBase;
    }
    
    @Override
    public double calcularSalario() {
        return salarioBase;
    }
}

public class FuncionarioTerceirizado extends Funcionario {
    private double valorHora;
    private int horasTrabalhadas;
    
    public FuncionarioTerceirizado(String nome, double valorHora, int horas) {
        super(nome);
        this.valorHora = valorHora;
        this.horasTrabalhadas = horas;
    }
    
    @Override
    public double calcularSalario() {
        return valorHora * horasTrabalhadas + (valorHora * horasTrabalhadas * 0.1); // +10% bônus
    }
}
```

**Teste:**

```java
List<Funcionario> funcionarios = new ArrayList<>();

funcionarios.add(new FuncionarioComum("Maria", 3000.0));
funcionarios.add(new FuncionarioTerceirizado("João", 50.0, 100));

double totalSalarios = 0;
for (Funcionario f : funcionarios) {
    double salario = f.calcularSalario();  // Polimorfismo!
    System.out.println(f.nome + ": R$ " + salario);
    totalSalarios += salario;
}

System.out.println("Total: R$ " + totalSalarios);

// Saída:
// Maria: R$ 3000.0
// João: R$ 5500.0
// Total: R$ 8500.0
```

---

## 9. Hierarquia Prática: Contas Bancárias

```
         Conta (abstrata)
           ? ?
           | |
           | ?? ContaEmpresa
           ????? ContaPoupanca
```

### Implementação Completa

```java
// ============= SUPERCLASSE ABSTRATA =============
public abstract class Conta {
    protected int numero;
    protected String titular;
    protected double saldo;
    
    public Conta(int numero, String titular, double saldo) {
        this.numero = numero;
        this.titular = titular;
        this.saldo = saldo;
    }
    
    // Método concreto (todos usam igual)
    public void depositar(double valor) {
        saldo += valor;
        System.out.println("Deposito de R$ " + valor + " realizado");
    }
    
    // Método abstrato (cada um implementa diferente)
    public abstract void sacar(double valor);
    
    public double getSaldo() {
        return saldo;
    }
    
    public abstract String toString();
}

// ============= SUBCLASSE: POUPANÇA =============
public class ContaPoupanca extends Conta {
    private double taxaJuros;
    
    public ContaPoupanca(int numero, String titular, double saldo, double taxa) {
        super(numero, titular, saldo);
        this.taxaJuros = taxa;
    }
    
    @Override
    public void sacar(double valor) {
        saldo -= valor;  // Sem taxa!
    }
    
    public void atualizarSaldo() {
        saldo += saldo * taxaJuros / 100.0;
        System.out.println("Juros aplicados: R$ " + (saldo * taxaJuros / 100.0));
    }
    
    @Override
    public String toString() {
        return "ContaPoupanca{" +
                "numero=" + numero +
                ", titular='" + titular + '\'' +
                ", saldo=" + saldo +
                '}';
    }
}

// ============= SUBCLASSE: EMPRESA =============
public class ContaEmpresa extends Conta {
    private double limitoEmprestimo;
    
    public ContaEmpresa(int numero, String titular, double saldo, double limito) {
        super(numero, titular, saldo);
        this.limitoEmprestimo = limito;
    }
    
    @Override
    public void sacar(double valor) {
        if (valor <= saldo + limitoEmprestimo) {
            saldo -= valor + 2.0;  // Taxa de R$ 2.00
            System.out.println("Saque empresarial realizado");
        }
    }
    
    public void tomarEmprestimo(double valor) {
        if (valor <= limitoEmprestimo) {
            saldo += valor;
            limitoEmprestimo -= valor;
            System.out.println("Emprestimo concedido: R$ " + valor);
        }
    }
    
    @Override
    public String toString() {
        return "ContaEmpresa{" +
                "numero=" + numero +
                ", titular='" + titular + '\'' +
                ", saldo=" + saldo +
                ", limiteDisponivel=" + limitoEmprestimo +
                '}';
    }
}

// ============= TESTE: POLIMORFISMO =============
public class Main {
    public static void main(String[] args) {
        List<Conta> contas = new ArrayList<>();
        
        contas.add(new ContaPoupanca(101, "Maria", 1000.0, 2.0));
        contas.add(new ContaEmpresa(102, "Tech Corp", 5000.0, 10000.0));
        contas.add(new ContaPoupanca(103, "João", 500.0, 2.0));
        
        // Operações polimórficas
        for (Conta c : contas) {
            c.depositar(200.0);
            c.sacar(100.0);
            System.out.println(c);
            System.out.println();
        }
        
        // Saída:
        // Deposito de R$ 200.0 realizado
        // Saque realizado
        // ContaPoupanca{numero=101, titular='Maria', saldo=1100.0}
        // ...
    }
}
```

---

## 10. Exercício Completo: Sistema de Funcionários

```
       Funcionario (abstrata)
          ? ?
          | |
    Comum ?? Terceirizado
```

```java
public abstract class Funcionario {
    protected String nome;
    protected int id;
    protected double valorHora;
    
    public Funcionario(String nome, int id, double valorHora) {
        this.nome = nome;
        this.id = id;
        this.valorHora = valorHora;
    }
    
    public abstract double calcularSalario();
    
    public abstract String toString();
}

public class FuncionarioComum extends Funcionario {
    private int horasTrabalhadasMes;
    
    public FuncionarioComum(String nome, int id, double valorHora, int horas) {
        super(nome, id, valorHora);
        this.horasTrabalhadasMes = horas;
    }
    
    @Override
    public double calcularSalario() {
        return valorHora * horasTrabalhadasMes;
    }
    
    @Override
    public String toString() {
        return nome + ", Salário: R$ " + String.format("%.2f", calcularSalario());
    }
}

public class FuncionarioTerceirizado extends Funcionario {
    private int horasTrabalhadasMes;
    private double despesaAdicional;
    
    public FuncionarioTerceirizado(String nome, int id, double valorHora, int horas, double despesa) {
        super(nome, id, valorHora);
        this.horasTrabalhadasMes = horas;
        this.despesaAdicional = despesa;
    }
    
    @Override
    public double calcularSalario() {
        double salarioBase = valorHora * horasTrabalhadasMes;
        double bonus = despesaAdicional * 1.1;  // 110% da despesa
        return salarioBase + bonus;
    }
    
    @Override
    public String toString() {
        return nome + ", Salário: R$ " + String.format("%.2f", calcularSalario());
    }
}

public class Main {
    public static void main(String[] args) {
        List<Funcionario> funcionarios = new ArrayList<>();
        
        funcionarios.add(new FuncionarioComum("Alex", 1, 20.0, 50));
        funcionarios.add(new FuncionarioTerceirizado("Bob", 2, 15.0, 100, 200.0));
        funcionarios.add(new FuncionarioComum("Maria", 3, 20.0, 60));
        
        double total = 0;
        for (Funcionario f : funcionarios) {
            System.out.println(f);
            total += f.calcularSalario();
        }
        
        System.out.println("\nTotal de Folha: R$ " + String.format("%.2f", total));
    }
}
```

---

## 11. Resumo: Quando Usar O Quê

| Conceito | Use Quando | Exemplo |
|----------|-----------|---------|
| **Herança** | Classe A **é um** tipo de Classe B | Poupança **é uma** Conta |
| **Classe Abstrata** | Precisa de contrato comum mas implementações diferentes | Veiculo base para Carro, Moto |
| **Método Abstrato** | Método muda por tipo | `calcularSalario()` diferente por tipo |
| **Polimorfismo** | Tratar diferentes tipos uniformemente | Loop em `List<Conta>` chamando `sacar()` |
| **Override** | Subclasse customiza comportamento | ContaPoupanca customiza `sacar()` |
| **Super** | Subclasse quer usar implementação da superclasse | Chamar `super.sacar()` + taxa extra |
| **Protected** | Subclasses precisam acessar, outros não | Atributo `saldo` protegido |
| **Final** | Evitar sobrescrita perigosa | Método de segurança crítica |

---

## 12. Checklist de Aprendizado

- [ ] Entendo a relação "é-um" (herança)
- [ ] Consigo criar classe com `extends`
- [ ] Sei quando usar `protected` vs `private`
- [ ] Entendo upcasting (seguro) e downcasting (perigoso)
- [ ] Sei usar `instanceof` para verificar tipo
- [ ] Consigo sobrepor (`@Override`) métodos
- [ ] Uso `super` para chamar versão da superclasse
- [ ] Entendo classe `final` e método `final`
- [ ] Entendo polimorfismo na prática
- [ ] Consigo criar classes abstratas
- [ ] Consigo criar métodos abstratos e forçar implementação
- [ ] Consigo aplicar herança em problemas reais
- [ ] Sei quando usar herança vs composição (futura)

---

## Analogia Final: LoL

**Sem herança (RUIM):**
```java
class MagoAhri { // Toda a implementação do zero
    void atacar() { ... }
    void andar() { ... }
    void tomarDano() { ... }
    void lancarMagia() { ... }
}

class MagoLux { // Repetindo tudo de novo!
    void atacar() { ... }
    void andar() { ... }
    void tomarDano() { ... }
    void lancarMagia() { ... }
}
```

**Com herança (BOM):**
```java
abstract class Campeao { // Base compartilhada
    void atacar() { ... }
    void andar() { ... }
    void tomarDano() { ... }
    abstract void lancarHabilidade();
}

class MagoAhri extends Campeao {
    @Override
    void lancarHabilidade() { // Só customiza o diferente
        // Implantação específica de Ahri
    }
}

class MagoLux extends Campeao {
    @Override
    void lancarHabilidade() { // Só customiza o diferente
        // Implantação específica de Lux
    }
}
```

**Reuso + Polimorfismo + Segurança = Sucesso!**