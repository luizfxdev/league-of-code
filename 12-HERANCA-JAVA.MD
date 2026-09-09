# Herança em Java

Um dos pilares da POO. Permite que uma classe herde atributos e métodos de outra, evitando repetição de código.

---

## 1. O Problema Sem Herança

```java
// ❌ Código repetido
public class Caminhao {
    private String placa;
    private double capacidadeCarga;
    
    public void ligar() { }
    public void registrarViagem() { }
}

public class Moto {
    private String placa;
    private double capacidadeCarga;
    
    public void ligar() { }
    public void registrarViagem() { }
}

// Mesmos atributos e métodos duplicados em cada classe
```

---

## 2. A Solução: Herança

```java
// Classe PAI (superclasse)
public class Veiculo {
    protected String placa;
    protected double capacidadeCarga;
    
    public void ligar() {
        System.out.println(placa + " ligado.");
    }
    
    public void registrarViagem() {
        System.out.println("Viagem registrada para " + placa);
    }
}

// Classe FILHA (subclasse) - herda de Veiculo
public class Caminhao extends Veiculo {
    private double alturaMaxima;
    
    public void carregar(double peso) {
        System.out.println("Carregando " + peso + "kg");
    }
}

// Classe FILHA - herda de Veiculo
public class Moto extends Veiculo {
    private boolean temBau;
    
    public void entregarDocumento() {
        System.out.println("Entrega rápida de documento");
    }
}
```

**Visualização:**

```
        Veiculo (Superclasse)
        /                    \
       /                      \
  Caminhao                    Moto
  (Subclasse)               (Subclasse)
```

---

## 3. Usando a Herança

```java
Caminhao caminhao = new Caminhao();
caminhao.ligar();              // Herdado ✓
caminhao.registrarViagem();    // Herdado ✓
caminhao.carregar(5000);       // Próprio do Caminhao ✓

Moto moto = new Moto();
moto.ligar();                  // Herdado ✓
moto.registrarViagem();        // Herdado ✓
moto.entregarDocumento();      // Próprio da Moto ✓
```

---

## 4. Construtor em Herança

A subclasse deve inicializar os atributos herdados chamando o construtor da superclasse com `super(...)`.

```java
public class Veiculo {
    protected String placa;
    protected double capacidadeCarga;
    
    public Veiculo(String placa, double capacidadeCarga) {
        this.placa = placa;
        this.capacidadeCarga = capacidadeCarga;
    }
}

public class Caminhao extends Veiculo {
    private double alturaMaxima;
    
    public Caminhao(String placa, double capacidadeCarga, double alturaMaxima) {
        super(placa, capacidadeCarga);  // Chama construtor de Veiculo
        this.alturaMaxima = alturaMaxima;
    }
}
```

Uso:

```java
Caminhao caminhao = new Caminhao("ABC1234", 8000, 4.2);
```

---

## 5. Super: Referência à Superclasse

`super` permite chamar métodos ou o construtor da classe pai a partir da subclasse.

```java
public class Veiculo {
    protected String placa;
    
    public void apresentar() {
        System.out.println("Veículo de placa " + placa);
    }
}

public class Caminhao extends Veiculo {
    @Override
    public void apresentar() {
        super.apresentar();  // Chama o método do pai
        System.out.println("É um caminhão de carga.");
    }
}
```

Uso:

```java
Caminhao caminhao = new Caminhao();
caminhao.apresentar();

// Output:
// Veículo de placa ABC1234
// É um caminhão de carga.
```

---

## 6. Modificadores de Acesso em Herança

| Modificador | A subclasse herda? |
|-------------|---------------------|
| `private` | Não |
| `protected` | Sim |
| `public` | Sim |

```java
public class Veiculo {
    private double custoManutencao;  // Não é herdado
    protected double velocidadeMaxima; // É herdado
    
    protected void revisar() {
        System.out.println("Revisão realizada");
    }
}

public class Caminhao extends Veiculo {
    public void verificar() {
        System.out.println(velocidadeMaxima); // ✓ Acessa protected
        revisar();                            // ✓ Chama método protected
        
        // System.out.println(custoManutencao); // ❌ Erro: é private
    }
}
```

---

## 7. Cadeia de Herança (Multi-nível)

Uma classe pode herdar de outra que já herda de uma terceira.

```java
// Nível 1
public class Veiculo {
    protected String placa;
    
    public void ligar() {
        System.out.println(placa + " ligado.");
    }
}

// Nível 2: herda de Veiculo
public class VeiculoDeCarga extends Veiculo {
    protected double capacidadeCarga;
    
    public void carregar(double peso) {
        System.out.println("Carregando " + peso + "kg");
    }
}

// Nível 3: herda de VeiculoDeCarga
public class CaminhaoRefrigerado extends VeiculoDeCarga {
    private double temperaturaMinima;
    
    public void resfriar() {
        System.out.println("Resfriando até " + temperaturaMinima + "°C");
    }
}
```

Uso:

```java
CaminhaoRefrigerado caminhao = new CaminhaoRefrigerado();
caminhao.ligar();      // De Veiculo
caminhao.carregar(2000); // De VeiculoDeCarga
caminhao.resfriar();   // Próprio
```

```
Veiculo (placa, ligar())
  |
  +-- VeiculoDeCarga (capacidadeCarga, carregar())
        |
        +-- CaminhaoRefrigerado (temperaturaMinima, resfriar())
```

---

## 8. Java Não Suporta Herança Múltipla

```java
// ❌ Erro: não é possível herdar de duas classes
public class VeiculoEspecial extends Veiculo, Equipamento {
    // Erro de compilação!
}
```

Isso evita ambiguidade caso as duas classes tenham métodos com o mesmo nome. Para reaproveitar comportamento de mais de uma fonte, Java usa **interfaces**:

```java
// ✓ Uma classe pode implementar múltiplas interfaces
public class VeiculoEspecial extends Veiculo implements Rastreavel, Segurado {
    // OK!
}
```

---

## 9. Exemplo Completo: Frota de Veículos

```java
public class Veiculo {
    protected String placa;
    protected double capacidadeCarga;
    protected boolean disponivel;
    
    public Veiculo(String placa, double capacidadeCarga) {
        this.placa = placa;
        this.capacidadeCarga = capacidadeCarga;
        this.disponivel = true;
    }
    
    public void ligar() {
        System.out.println(placa + " ligado.");
    }
    
    public String getPlaca() {
        return placa;
    }
}

public class Caminhao extends Veiculo {
    private double alturaMaxima;
    
    public Caminhao(String placa, double capacidadeCarga, double alturaMaxima) {
        super(placa, capacidadeCarga);
        this.alturaMaxima = alturaMaxima;
    }
    
    public void carregar(double peso) {
        if (peso <= capacidadeCarga) {
            System.out.println("Caminhão " + placa + " carregado com " + peso + "kg");
        }
    }
}

public class Moto extends Veiculo {
    private boolean temBau;
    
    public Moto(String placa, boolean temBau) {
        super(placa, 15.0); // capacidade fixa de motos
        this.temBau = temBau;
    }
    
    public void entregarDocumento() {
        System.out.println("Moto " + placa + " entregando documento");
    }
}

public class Van extends Veiculo {
    private int quantidadePacotes;
    
    public Van(String placa, double capacidadeCarga) {
        super(placa, capacidadeCarga);
    }
    
    public void entregarPacotes(int quantidade) {
        this.quantidadePacotes = quantidade;
        System.out.println("Van " + placa + " entregando " + quantidade + " pacotes");
    }
}
```

Uso prático:

```java
public class GerenciadorFrota {
    public static void main(String[] args) {
        Caminhao caminhao = new Caminhao("ABC1234", 8000, 4.2);
        Moto moto = new Moto("XYZ5678", true);
        Van van = new Van("DEF9012", 1200);
        
        caminhao.ligar();           // Herdado de Veiculo
        caminhao.carregar(5000);    // Próprio
        
        moto.ligar();                // Herdado de Veiculo
        moto.entregarDocumento();    // Próprio
        
        van.ligar();                 // Herdado de Veiculo
        van.entregarPacotes(20);     // Próprio
    }
}
```

---

## 10. Erros Comuns

### Esquecer o `super()` no construtor

```java
public class Caminhao extends Veiculo {
    private double alturaMaxima;
    
    // ❌ Erro: atributos do pai não são inicializados
    public Caminhao(String placa, double alturaMaxima) {
        this.alturaMaxima = alturaMaxima;
        // placa nunca foi definida!
    }
    
    // ✓ Correto
    public Caminhao(String placa, double alturaMaxima) {
        super(placa, 0);
        this.alturaMaxima = alturaMaxima;
    }
}
```

### Tentar acessar atributo `private` do pai

```java
public class Veiculo {
    private double custoManutencao;
}

public class Caminhao extends Veiculo {
    public void verificar() {
        System.out.println(custoManutencao); // ❌ Erro: não herdado
    }
}
```

**Solução:** usar `protected` no atributo, ou expor um getter público na superclasse.

---

## 11. Herança x Polimorfismo — Qual a Diferença?

Herança e polimorfismo são conceitos complementares, mas não são a mesma coisa:

- **Herança** é a relação estrutural: define que uma classe (`Caminhao`) reaproveita atributos e métodos de outra (`Veiculo`).
- **Polimorfismo** é o comportamento que a herança viabiliza: uma variável do tipo `Veiculo` pode referenciar um `Caminhao`, uma `Moto` ou uma `Van`, e cada um executa sua própria versão de um método (com `@Override`).

```java
Veiculo v = new Caminhao("ABC1234", 8000, 4.2); // polimorfismo em ação
v.ligar(); // usa o método herdado ou sobrescrito, dependendo do objeto real
```

Em resumo: sem herança não existe polimorfismo de subtipo em Java — a herança monta a hierarquia, e o polimorfismo é o que se ganha ao usá-la.

---

## Resumo

| Conceito | Significado | Exemplo |
|----------|-------------|---------|
| **Herança** | Subclasse herda de superclasse | `class Caminhao extends Veiculo` |
| **Super** | Referência ao construtor/método do pai | `super(placa, capacidade)` |
| **Protected** | Acessível em subclasses | `protected double capacidadeCarga` |
| **Herança múltipla** | Não suportada em Java | Usar interfaces no lugar |
| **Cadeia de herança** | Subclasse de subclasse | `Veiculo → VeiculoDeCarga → CaminhaoRefrigerado` |

---

## Dicas Finais

✓ Use herança para relações **"é um"** (Caminhão é um Veículo)  
✓ Sempre chame `super(...)` no construtor da subclasse  
✓ Use `protected` nos atributos que as subclasses precisam acessar  
✓ Mantenha hierarquias **rasas** (evite muitos níveis de herança)  
✓ Se não há uma relação "é um" clara, considere composição em vez de herança