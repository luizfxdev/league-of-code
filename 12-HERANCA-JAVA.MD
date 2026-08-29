# Herança em Java - LoL Edition

Um dos pilares da POO. Permite que classes herdem atributos e métodos de outras, promovendo reuso de código.

---

## 1. Conceito: Hierarquia de Classes

### O Problema Sem Herança

```java
// ❌ Código repetido
public class CampeaoTank {
    private String nome;
    private int hp;
    private int nivel;
    
    public void atacar() { }
    public void ganharExp() { }
}

public class CampeaoMago {
    private String nome;
    private int hp;
    private int nivel;
    
    public void atacar() { }
    public void ganharExp() { }
}

// REPETIÇÃO! Mesmos atributos e métodos
```

### A Solução: Herança

```java
// Classe PAI (superclasse)
public class Campeao {
    private String nome;
    private int hp;
    private int nivel;
    
    public void atacar() {
        System.out.println(nome + " atacou!");
    }
    
    public void ganharExp() {
        nivel++;
    }
}

// Classe FILHA (subclasse) - herda de Campeao
public class CampeaoTank extends Campeao {
    private int resistencia;
    
    public void escudarAliado() {
        System.out.println("Protegendo aliado!");
    }
}

// Classe FILHA - herda de Campeao
public class CampeaoMago extends Campeao {
    private int manaAtual;
    
    public void lançarFeitiço() {
        System.out.println("Lançando feitiço!");
    }
}
```

**Visualização:**

```
        Campeao (Superclasse)
        /         |        \
       /          |         \
  CampeaoTank  CampeaoMago  CampeaoADC
  (Subclasse) (Subclasse)  (Subclasse)
```

---

## 2. Usando Herança

### Instanciando Subclasses

```java
// Criar um Tank
CampeaoTank garen = new CampeaoTank();
garen.atacar();           // Herdado ✓
garen.ganharExp();        // Herdado ✓
garen.escudarAliado();    // Próprio do Tank ✓

// Criar um Mago
CampeaoMago lux = new CampeaoMago();
lux.atacar();             // Herdado ✓
lux.ganharExp();          // Herdado ✓
lux.lançarFeitiço();      // Próprio do Mago ✓
```

### Construtor em Herança

```java
public class Campeao {
    protected String nome;
    protected int hp;
    
    // Construtor
    public Campeao(String nome, int hp) {
        this.nome = nome;
        this.hp = hp;
    }
}

public class CampeaoTank extends Campeao {
    private int resistencia;
    
    // Construtor do Tank (chama construtor do pai)
    public CampeaoTank(String nome, int hp, int resistencia) {
        super(nome, hp);  // Chama construtor do Campeao
        this.resistencia = resistencia;
    }
}

// Usando:
CampeaoTank garen = new CampeaoTank("Garen", 650, 100);
```

### Super: Referência à Superclasse

```java
public class Campeao {
    protected String nome;
    
    public void apresentar() {
        System.out.println("Sou um campeão!");
    }
}

public class CampeaoTank extends Campeao {
    public void apresentar() {
        super.apresentar();  // Chama método do pai
        System.out.println("E sou um Tank!");
    }
}

// Usando:
CampeaoTank garen = new CampeaoTank();
garen.apresentar();

// Output:
// Sou um campeão!
// E sou um Tank!
```

---

## 3. Override (Sobrescrita)

### Problema: Comportamentos Diferentes

```java
public class Campeao {
    public void atacar() {
        System.out.println("Ataque genérico!");
    }
}

public class CampeaoTank extends Campeao {
    // Tank ataca de forma diferente
    public void atacar() {
        System.out.println("Investida poderosa!");
    }
}

public class CampeaoMago extends Campeao {
    // Mago ataca de forma diferente
    public void atacar() {
        System.out.println("Lança um feitiço!");
    }
}

// Usando:
Campeao tank = new CampeaoTank();
Campeao mago = new CampeaoMago();

tank.atacar();  // "Investida poderosa!"
mago.atacar();  // "Lança um feitiço!"
```

### Override vs. Overload

```java
public class Campeao {
    // Método original
    public void atacar() {
        System.out.println("Ataque básico!");
    }
}

public class CampeaoTank extends Campeao {
    // OVERRIDE: mesmo nome, assinatura, comportamento diferente
    @Override  // Anotação (opcional mas recomendada)
    public void atacar() {
        System.out.println("Investida!");
    }
    
    // OVERLOAD: mesmo nome, parâmetros diferentes
    public void atacar(Campeao alvo) {
        System.out.println("Atacando " + alvo.getNome());
    }
    
    public void atacar(int dano) {
        System.out.println("Ataque com " + dano + " de dano!");
    }
}

// Usando:
CampeaoTank garen = new CampeaoTank();
garen.atacar();              // Override: "Investida!"
garen.atacar(new CampeaoMago());  // Overload: "Atacando ..."
garen.atacar(150);           // Overload: "Ataque com 150 de dano!"
```

---

## 4. Polimorfismo: O Poder da Herança

### O Conceito

A mesma referência pode apontar para objetos de **tipos diferentes**:

```java
// Campeao é a superclasse
Campeao campeao1 = new CampeaoTank();   // ✓ Permitido
Campeao campeao2 = new CampeaoMago();   // ✓ Permitido
Campeao campeao3 = new CampeaoADC();    // ✓ Permitido

// MAS ao contrário, NÃO é permitido
CampeaoTank tank = new Campeao();       // ❌ Erro!
```

### Aplicação Prática

```java
public class Time {
    private Campeao[] integrantes;
    
    public Time(Campeao[] integrantes) {
        this.integrantes = integrantes;
    }
    
    // Método que funciona com QUALQUER subclasse
    public void listarCampeoes() {
        for (Campeao campeao : integrantes) {
            System.out.println(campeao.getNome());
        }
    }
    
    // Todos atacam (mas de formas diferentes)
    public void atacarTodos(Campeao alvo) {
        for (Campeao campeao : integrantes) {
            campeao.atacar(alvo);  // Polimorfismo!
        }
    }
}

// Usando:
Campeao[] time = {
    new CampeaoTank("Garen", 650),
    new CampeaoMago("Lux", 450),
    new CampeaoADC("Ashe", 500)
};

Time timeBlue = new Time(time);
timeBlue.listarCampeoes();   // Funciona com todas subclasses
timeBlue.atacarTodos(inimigo);  // Cada uma ataca à sua maneira
```

### Por Que é Poderoso?

```java
// Sem polimorfismo:
if (campeao instanceof CampeaoTank) {
    ((CampeaoTank) campeao).atacar();
} else if (campeao instanceof CampeaoMago) {
    ((CampeaoMago) campeao).atacar();
} else if (campeao instanceof CampeaoADC) {
    ((CampeaoADC) campeao).atacar();
}
// Horrível! Muito código repetido

// Com polimorfismo (muito melhor):
campeao.atacar();  // Funciona para qualquer subclasse
```

---

## 5. Modificadores de Acesso em Herança

### Private: Não Herda

```java
public class Campeao {
    private int segredo;  // PRIVADO
    protected int hp;     // PROTEGIDO
}

public class CampeaoTank extends Campeao {
    public void testar() {
        // ❌ Não consegue acessar segredo
        System.out.println(segredo);  // Erro!
        
        // ✓ Consegue acessar hp (protected)
        System.out.println(hp);       // OK
    }
}
```

### Protected: Herda

```java
public class Campeao {
    protected String nome;    // Subclasses acessam
    private int hpSecreto;    // Subclasses NÃO acessam
    
    protected void metodoInterno() {
        System.out.println("Método protegido");
    }
}

public class CampeaoTank extends Campeao {
    public void apresentar() {
        System.out.println(nome);  // ✓ Pode acessar protected
        metodoInterno();           // ✓ Pode chamar protected
    }
}
```

---

## 6. Cadeia de Herança (Hierarquia Profunda)

### Herança Multi-nível

```java
// Nível 1: Classe base
public class SerVivo {
    protected int hp;
    
    public void morrer() {
        hp = 0;
    }
}

// Nível 2: Herda de SerVivo
public class Campeao extends SerVivo {
    protected String nome;
    
    public void atacar() {
        System.out.println(nome + " ataca!");
    }
}

// Nível 3: Herda de Campeao
public class CampeaoTank extends Campeao {
    protected int resistencia;
    
    public void escudar() {
        System.out.println("Escudando!");
    }
}

// Usando:
CampeaoTank garen = new CampeaoTank();
garen.atacar();    // De Campeao ✓
garen.escudar();   // De CampeaoTank ✓
garen.morrer();    // De SerVivo ✓
```

**Visualização:**

```
SerVivo (hp)
  |
  +-- Campeao (nome, atacar())
        |
        +-- CampeaoTank (resistencia, escudar())
```

---

## 7. Problema: Herança Múltipla

### Java NÃO Suporta Herança Múltipla

```java
// ❌ ERRO: Não é permitido herdar de 2 classes
public class CampeaoEspecial extends Campeao, Objeto {
    // Erro!
}
```

**Por quê?** Cria ambiguidade (qual método usar se ambas as classes têm?).

### Solução: Interfaces (veremos no próximo arquivo)

```java
// ✓ Uma classe pode implementar múltiplas interfaces
public class CampeaoEspecial extends Campeao implements Atacavel, Curavel {
    // OK!
}
```

---

## 8. Padrão Prático: Hierarquia Completa

```java
// Base
public class Campeao {
    protected String nome;
    protected int hp;
    protected int mana;
    
    public Campeao(String nome, int hp, int mana) {
        this.nome = nome;
        this.hp = hp;
        this.mana = mana;
    }
    
    public void levarDano(int dano) {
        this.hp -= dano;
    }
    
    public void atacar() {
        System.out.println(nome + " ataca!");
    }
    
    public String getNome() { return nome; }
    public int getHp() { return hp; }
}

// Tank: Muita resistência, pouca mobilidade
public class CampeaoTank extends Campeao {
    private int resistencia;
    
    public CampeaoTank(String nome, int hp, int mana, int resistencia) {
        super(nome, hp, mana);
        this.resistencia = resistencia;
    }
    
    @Override
    public void levarDano(int dano) {
        int danoReduzido = (int) (dano * 0.7);  // 30% redução
        super.levarDano(danoReduzido);
    }
    
    public void escudarAliado(Campeao aliado) {
        System.out.println(nome + " protege " + aliado.getNome());
    }
}

// Mago: Pouca resistência, muito dano mágico
public class CampeaoMago extends Campeao {
    private int poderelecao;
    
    public CampeaoMago(String nome, int hp, int mana, int poderelecao) {
        super(nome, hp, mana);
        this.poderelecao = poderelecao;
    }
    
    @Override
    public void atacar() {
        System.out.println(nome + " lança um feitiço!");
    }
    
    public void lancarExplosao(Campeao alvo) {
        System.out.println(nome + " lança explosão em " + alvo.getNome());
        alvo.levarDano(poderelecao);
    }
}

// ADC: Mobilidade média, dano físico alto
public class CampeaoADC extends Campeao {
    private int danofisico;
    
    public CampeaoADC(String nome, int hp, int mana, int danofisico) {
        super(nome, hp, mana);
        this.danofisico = danofisico;
    }
    
    @Override
    public void atacar() {
        System.out.println(nome + " dispara uma flecha!");
    }
    
    public void disparoMultiplo(Campeao[] alvos) {
        for (Campeao alvo : alvos) {
            alvo.levarDano(danofisico);
        }
        System.out.println(nome + " disparou em " + alvos.length + " alvos!");
    }
}

// Usando tudo junto:
public class Partida {
    public static void main(String[] args) {
        Campeao[] timeBlue = {
            new CampeaoTank("Garen", 650, 100, 50),
            new CampeaoMago("Lux", 450, 400, 200),
            new CampeaoADC("Ashe", 500, 200, 150)
        };
        
        Campeao[] timeRed = {
            new CampeaoADC("Jinx", 480, 180, 160),
            new CampeaoTank("Malphite", 700, 80, 60),
            new CampeaoMago("Ahri", 470, 420, 190)
        };
        
        System.out.println("=== Batalha Iniciada ===\n");
        
        // Todos atacam (polimorfismo!)
        for (Campeao c : timeBlue) {
            c.atacar();
        }
        
        System.out.println();
        
        for (Campeao c : timeRed) {
            c.atacar();
        }
    }
}

// Output:
// === Batalha Iniciada ===
//
// Garen ataca!
// Lux lança um feitiço!
// Ashe dispara uma flecha!
//
// Jinx dispara uma flecha!
// Malphite ataca!
// Ahri lança um feitiço!
```

---

## 9. Erros Comuns

### Esquecer Super no Construtor

```java
public class CampeaoTank extends Campeao {
    private int resistencia;
    
    // ❌ Erro: não inicializa atributos do pai
    public CampeaoTank(String nome, int resistencia) {
        this.resistencia = resistencia;
        // nome não foi inicializado!
    }
    
    // ✓ Correto
    public CampeaoTank(String nome, int resistencia) {
        super(nome);  // Inicializa atributos do pai
        this.resistencia = resistencia;
    }
}
```

### Confundir Override com Overload

```java
public class Campeao {
    public void atacar() {
        System.out.println("Ataque");
    }
}

public class CampeaoTank extends Campeao {
    // ✓ Override: mesmo nome, mesma assinatura
    @Override
    public void atacar() {
        System.out.println("Investida");
    }
    
    // ✓ Overload: mesmo nome, assinatura diferente
    public void atacar(Campeao alvo) {
        System.out.println("Ataque em " + alvo.getNome());
    }
}
```

### Type Casting Perigoso

```java
Campeao campeao = new CampeaoTank();

// ❌ ClassCastException se não for realmente um ADC
CampeaoADC adc = (CampeaoADC) campeao;  // Erro em runtime!

// ✓ Verificar antes
if (campeao instanceof CampeaoADC) {
    CampeaoADC adc = (CampeaoADC) campeao;  // Seguro
}
```

---

## Resumo

| Conceito | Significado | Exemplo |
|----------|-------------|---------|
| **Herança** | Subclasse herda de superclasse | `class Tank extends Campeao` |
| **Super** | Referência ao pai | `super.atacar()` |
| **Override** | Mesmo método, comportamento diferente | `@Override public void atacar()` |
| **Polimorfismo** | Mesma referência, tipos diferentes | `Campeao c = new Tank()` |
| **Protected** | Acessível em subclasses | `protected int hp` |

---

## Dicas Finais

✓ Use herança para **relações "é um"** (Tank é um Campeao)  
✓ Sempre use `super()` no construtor da subclasse  
✓ Use `@Override` para clareza  
✓ Prefira **composição** quando não há relação clara (veremos depois)  
✓ Mantenha hierarquias **rasas** (não muito profundas)  

Domina herança e você domina POO! 🎮
