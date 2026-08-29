# Casting e Enums em Java - LoL Edition

Casting permite converter entre tipos. Enums permitem criar conjuntos de valores predefinidos.

---

## 1. Casting: Conversão de Tipos

### Casting Automático (Upcasting)

Java permite "subir" na hierarquia automaticamente:

```java
// Subclass para Superclass (automático e seguro)
Guerreiro garen = new Guerreiro("Garen");
Campeao campeao = garen;  // ✓ Automático

// É como colocar uma caixa pequena em uma caixa grande
// Sempre é seguro
```

### Casting Manual (Downcasting)

Quando quer "descer" na hierarquia, precisa fazer manualmente:

```java
Campeao campeao = new Guerreiro("Garen");

// ❌ Erro: não pode fazer automaticamente
Guerreiro garen = campeao;  // ERRO!

// ✓ Precisa fazer casting manual
Guerreiro garen = (Guerreiro) campeao;  // OK
```

### Problema: ClassCastException

```java
Campeao campeao = new Mago("Lux");

// ❌ ClassCastException em runtime!
Guerreiro garen = (Guerreiro) campeao;
// campeao aponta para Mago, não é Guerreiro!
```

### Solução: Verificar com instanceof

```java
Campeao campeao = new Mago("Lux");

// ✓ Verificar ANTES de fazer cast
if (campeao instanceof Guerreiro) {
    Guerreiro garen = (Guerreiro) campeao;
}

if (campeao instanceof Mago) {
    Mago lux = (Mago) campeao;
}
```

### Casting com Tipos Primitivos

```java
// Casting de tipos numéricos
int inteiro = 42;
double decimal = (double) inteiro;  // 42.0

double d = 3.14;
int i = (int) d;  // 3 (perde a parte decimal)

// Casting de String para número
String numero = "123";
int valor = Integer.parseInt(numero);  // 123
double valor2 = Double.parseDouble("3.14");  // 3.14

// ❌ Erro se formato inválido
int erro = Integer.parseInt("abc");  // NumberFormatException!
```

---

## 2. Pattern Matching (Java 16+)

Simplifica o pattern de instanceof + cast:

```java
// ANTES (verbose)
Campeao c = new Guerreiro("Garen");
if (c instanceof Guerreiro) {
    Guerreiro g = (Guerreiro) c;
    g.atacar();
}

// DEPOIS (Java 16+ - muito melhor)
Campeao c = new Guerreiro("Garen");
if (c instanceof Guerreiro g) {
    g.atacar();  // Já convertido para Guerreiro
}
```

---

## 3. Enums: Valores Predefinidos

### Por Que Usar?

```java
// ❌ Ruim: Strings em toda parte
public class Campeao {
    private String tipo;  // "Tank", "Mago", "ADC", "Suporte"
}

// Pode ter erro de digitação
campeao.tipo = "Tanque";  // Digitou errado, ninguém avisa!

// ✓ Bom: Enum (valores fixos)
public enum TipoCampeao {
    TANK, MAGO, ADC, SUPORTE
}

public class Campeao {
    private TipoCampeao tipo;  // Type-safe
}
```

### Definindo Enums

```java
// Simples
public enum TipoCampeao {
    TANK, MAGO, ADC, SUPORTE
}

// Usando:
Campeao garen = new Campeao("Garen", TipoCampeao.TANK);
Campeao lux = new Campeao("Lux", TipoCampeao.MAGO);

// Comparar
if (garen.getTipo() == TipoCampeao.TANK) {
    System.out.println("Garen é um Tank!");
}
```

### Enum com Atributos

```java
public enum TipoCampeao {
    // Cada enum tem atributos
    TANK("Resistência alta", 800),
    MAGO("Dano mágico alto", 400),
    ADC("Dano físico alto", 500),
    SUPORTE("Suporte do time", 500);
    
    private String descricao;
    private int hpBase;
    
    // Construtor do enum
    TipoCampeao(String descricao, int hpBase) {
        this.descricao = descricao;
        this.hpBase = hpBase;
    }
    
    public String getDescricao() { return descricao; }
    public int getHpBase() { return hpBase; }
}

// Usando:
TipoCampeao tipo = TipoCampeao.TANK;
System.out.println(tipo.getDescricao());  // "Resistência alta"
System.out.println(tipo.getHpBase());     // 800
```

### Enum com Métodos

```java
public enum TipoCampeao {
    TANK("Resistência") {
        @Override
        public int calcularDano(int base) {
            return base / 2;  // Tank faz menos dano
        }
    },
    MAGO("Dano Mágico") {
        @Override
        public int calcularDano(int base) {
            return base * 2;  // Mago faz mais dano
        }
    };
    
    private String descricao;
    
    TipoCampeao(String descricao) {
        this.descricao = descricao;
    }
    
    // Método abstrato
    public abstract int calcularDano(int base);
    
    public String getDescricao() { return descricao; }
}

// Usando:
System.out.println(TipoCampeao.TANK.calcularDano(100));   // 50
System.out.println(TipoCampeao.MAGO.calcularDano(100));   // 200
```

---

## 4. Iterando Enums

```java
public enum TipoCampeao {
    TANK, MAGO, ADC, SUPORTE
}

// Iterar TODOS os valores
for (TipoCampeao tipo : TipoCampeao.values()) {
    System.out.println(tipo);
}

// Output:
// TANK
// MAGO
// ADC
// SUPORTE

// Converter String para enum
String entrada = "MAGO";
TipoCampeao tipo = TipoCampeao.valueOf(entrada);  // TipoCampeao.MAGO

// Comparar
TipoCampeao tipo1 = TipoCampeao.TANK;
System.out.println(tipo1.equals(TipoCampeao.TANK));  // true
```

---

## 5. Exemplo Prático: Sistema de Raridade

```java
public enum Raridade {
    COMUM("Comum", "gray"),
    INCOMUM("Incomum", "green"),
    RARO("Raro", "blue"),
    LENDARIO("Lendário", "orange");
    
    private String nome;
    private String cor;
    
    Raridade(String nome, String cor) {
        this.nome = nome;
        this.cor = cor;
    }
    
    public double getMultiplicadorStats() {
        switch (this) {
            case COMUM: return 1.0;
            case INCOMUM: return 1.2;
            case RARO: return 1.5;
            case LENDARIO: return 2.0;
            default: return 1.0;
        }
    }
    
    public String getNome() { return nome; }
    public String getCor() { return cor; }
}

public class Skin {
    private String nome;
    private Raridade raridade;
    
    public Skin(String nome, Raridade raridade) {
        this.nome = nome;
        this.raridade = raridade;
    }
    
    public void descrever() {
        System.out.println("Skin: " + nome);
        System.out.println("Raridade: " + raridade.getNome() + " (" + raridade.getCor() + ")");
        System.out.println("Multiplicador: " + raridade.getMultiplicadorStats() + "x");
    }
}

// Usando:
Skin skin1 = new Skin("Garen Darius", Raridade.RARO);
skin1.descrever();

Skin skin2 = new Skin("Lux Elementalist", Raridade.LENDARIO);
skin2.descrever();
```

---

## 6. Exemplo Completo: Casting + Enum + Polimorfismo

```java
// Enum para tipo
public enum TipoCampeao {
    TANK, MAGO, ADC, SUPORTE
}

// Classe base
public abstract class Campeao {
    protected String nome;
    protected TipoCampeao tipo;
    
    public Campeao(String nome, TipoCampeao tipo) {
        this.nome = nome;
        this.tipo = tipo;
    }
    
    public abstract void atacar();
    public String getNome() { return nome; }
    public TipoCampeao getTipo() { return tipo; }
}

// Subclasses
public class Tank extends Campeao {
    public Tank(String nome) {
        super(nome, TipoCampeao.TANK);
    }
    
    @Override
    public void atacar() {
        System.out.println(nome + " carrega!");
    }
    
    public void escudar() {
        System.out.println(nome + " ativa escudo!");
    }
}

public class Mago extends Campeao {
    public Mago(String nome) {
        super(nome, TipoCampeao.MAGO);
    }
    
    @Override
    public void atacar() {
        System.out.println(nome + " lança feitiço!");
    }
    
    public void explodir() {
        System.out.println(nome + " causa explosão!");
    }
}

// Gerenciador de time
public class Time {
    private List<Campeao> integrantes;
    
    public Time() {
        this.integrantes = new ArrayList<>();
    }
    
    public void adicionarCampeao(Campeao c) {
        integrantes.add(c);
    }
    
    // Polimorfismo: todos atacam
    public void atacarTodos() {
        for (Campeao c : integrantes) {
            c.atacar();
        }
    }
    
    // Casting: apenas Tanks
    public void escudarTime() {
        for (Campeao c : integrantes) {
            if (c instanceof Tank) {
                Tank tank = (Tank) c;  // Cast
                tank.escudar();
            }
        }
    }
    
    // Casting: apenas Magos (Java 16+)
    public void explodir() {
        for (Campeao c : integrantes) {
            if (c instanceof Mago mago) {
                mago.explodir();
            }
        }
    }
    
    // Agrupar por tipo (enum)
    public void listarPorTipo() {
        for (TipoCampeao tipo : TipoCampeao.values()) {
            System.out.println("\n=== " + tipo + " ===");
            for (Campeao c : integrantes) {
                if (c.getTipo() == tipo) {
                    System.out.println(c.getNome());
                }
            }
        }
    }
}

// Executar tudo:
public class Partida {
    public static void main(String[] args) {
        Time time = new Time();
        
        time.adicionarCampeao(new Tank("Garen"));
        time.adicionarCampeao(new Mago("Lux"));
        time.adicionarCampeao(new Tank("Malphite"));
        time.adicionarCampeao(new Mago("Ahri"));
        
        System.out.println("=== Todos Atacam ===");
        time.atacarTodos();
        
        System.out.println("\n=== Apenas Tanks Escudam ===");
        time.escudarTime();
        
        System.out.println("\n=== Apenas Magos Explodem ===");
        time.explodir();
        
        System.out.println("\n=== Listagem por Tipo ===");
        time.listarPorTipo();
    }
}

// Output:
// === Todos Atacam ===
// Garen carrega!
// Lux lança feitiço!
// Malphite carrega!
// Ahri lança feitiço!
//
// === Apenas Tanks Escudam ===
// Garen ativa escudo!
// Malphite ativa escudo!
//
// === Apenas Magos Explodem ===
// Lux causa explosão!
// Ahri causa explosão!
//
// === Listagem por Tipo ===
//
// === TANK ===
// Garen
// Malphite
//
// === MAGO ===
// Lux
// Ahri
//
// === ADC ===
//
// === SUPORTE ===
```

---

## 7. Comparação: Enum vs. Constantes

```java
// ❌ Ruim: constantes soltas
public class Raridade {
    public static final int COMUM = 0;
    public static final int RARO = 1;
    public static final int LENDARIO = 2;
}

int raridade = 5;  // Inválido, mas Java não avisa!

// ✓ Bom: Enum
public enum Raridade {
    COMUM, RARO, LENDARIO
}

Raridade raridade = Raridade.COMUM;  // Type-safe!
```

---

## 8. Erros Comuns

### ClassCastException

```java
// ❌ Erro comum
Campeao c = new Mago("Lux");
Tank tank = (Tank) c;  // ClassCastException!

// ✓ Verificar antes
if (c instanceof Tank) {
    Tank tank = (Tank) c;
}
```

### Enum equals vs ==

```java
// Ambos funcionam com Enum
TipoCampeao tipo1 = TipoCampeao.TANK;
TipoCampeao tipo2 = TipoCampeao.TANK;

// == funciona perfeitamente
if (tipo1 == tipo2) { }  // ✓ OK

// equals também funciona
if (tipo1.equals(tipo2)) { }  // ✓ OK

// Prefira ==, é mais eficiente
```

### Tentar Instanciar Enum

```java
// ❌ Erro
TipoCampeao tipo = new TipoCampeao();  // Não compila!

// Enums são criados implicitamente
TipoCampeao tipo = TipoCampeao.TANK;  // ✓ Correto
```

---

## 9. Performance: Casting vs. instanceof

```java
// ✓ Rápido: apenas verificar tipo
if (c instanceof Tank) {
    // fazer algo
}

// ✓ Necessário: fazer cast depois de verificar
if (c instanceof Tank) {
    Tank t = (Tank) c;
    t.metodoEspecifico();
}

// ❌ Caro: múltiplos checks
Tank t;
try {
    t = (Tank) c;  // Exception se não for Tank
} catch (ClassCastException e) {
    // Exceções são caras!
}
```

---

## 10. Padrão: Switch com Enum

```java
TipoCampeao tipo = TipoCampeao.TANK;

switch (tipo) {
    case TANK:
        System.out.println("Resistência alta!");
        break;
    case MAGO:
        System.out.println("Dano alto!");
        break;
    case ADC:
        System.out.println("Dano físico!");
        break;
    case SUPORTE:
        System.out.println("Cura!");
        break;
}

// Ou com expressão (Java 14+)
String descricao = switch (tipo) {
    case TANK -> "Resistência";
    case MAGO -> "Dano mágico";
    case ADC -> "Dano físico";
    case SUPORTE -> "Cura";
};
```

---

## Resumo

| Conceito | Uso |
|----------|-----|
| **Casting** | Converter entre tipos |
| **instanceof** | Verificar tipo antes de cast |
| **Enum** | Valores predefinidos, type-safe |
| **Pattern Matching** | Simplificar instanceof + cast |

---

## Dicas Finais

✓ **Sempre verifique com instanceof** antes de downcasting  
✓ Use **Enums** para valores predefinidos (muito melhor!)  
✓ **Java 16+**: aproveite pattern matching  
✓ Use `==` para comparar enums  
✓ Enums com métodos são muito poderosos  

Casting seguro e Enums garantem código robusto! 🎮
