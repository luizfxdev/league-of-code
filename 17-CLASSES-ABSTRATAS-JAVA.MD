# Classes Abstratas em Java - LoL Edition

Classes abstratas permitem criar uma **classe base com comportamento parcial** que força subclasses a completar a implementação.

---

## 1. O Conceito

### O Problema

```java
// ❌ Não faz sentido instanciar uma "Criatura" genérica
Campeao campeao = new Campeao("Generic");
campeao.atacar();  // O que um campeão genérico faria?

// Faz mais sentido ter tipos específicos
Guerreiro garen = new Guerreiro("Garen");
Mago lux = new Mago("Lux");
```

### A Solução: Abstract

Uma **classe abstrata** não pode ser instanciada, apenas estendida:

```java
// Classe abstrata: não pode fazer new, só estender
public abstract class Campeao {
    protected String nome;
    
    // Método abstrato: subclasses DEVEM implementar
    public abstract void atacar();
}

// ❌ Erro: não pode instanciar classe abstrata
Campeao c = new Campeao();  // ERRO!

// ✓ OK: instanciar subclass concreta
Guerreiro garen = new Guerreiro();
```

---

## 2. Métodos Abstratos

### Definição

Um **método abstrato** não tem corpo, obrigando subclasses a implementar:

```java
public abstract class Campeao {
    // Método abstrato: sem corpo
    public abstract void atacar();
    
    // Método concreto: com implementação
    public void ganharExperiencia() {
        System.out.println("Ganhando experiência...");
    }
}

public class Guerreiro extends Campeao {
    // Obrigado a implementar atacar()
    @Override
    public void atacar() {
        System.out.println("Golpe de espada!");
    }
    
    // Herda ganharExperiencia()
}

// Usando:
Guerreiro garen = new Guerreiro();
garen.atacar();                 // Implementação do Guerreiro
garen.ganharExperiencia();      // Herdado de Campeao
```

### Múltiplos Métodos Abstratos

```java
public abstract class Campeao {
    // Método abstrato 1
    public abstract void atacar();
    
    // Método abstrato 2
    public abstract void defender();
    
    // Método concreto
    public void ganharExp() {
        System.out.println("Exp gained!");
    }
}

public class Guerreiro extends Campeao {
    @Override
    public void atacar() {
        System.out.println("Ataca com força!");
    }
    
    @Override
    public void defender() {
        System.out.println("Levanta escudo!");
    }
    
    // Herda ganharExp()
}
```

---

## 3. Métodos Concretos em Classes Abstratas

### Mix: Abstrato + Concreto

```java
public abstract class Campeao {
    protected String nome;
    protected int hp;
    
    // Construtor (classes abstratas podem ter!)
    public Campeao(String nome) {
        this.nome = nome;
        this.hp = 100;
    }
    
    // Método ABSTRATO: cada subclass decide
    public abstract void atacar();
    
    // Método CONCRETO: todos compartilham
    public void ganharExp() {
        System.out.println(nome + " ganhou experiência!");
    }
    
    // Outro método CONCRETO
    public void levarDano(int dano) {
        hp -= dano;
        if (hp <= 0) {
            morrer();
        }
    }
    
    // Método ABSTRATO
    public abstract void morrer();
    
    // Getter concreto
    public String getNome() { return nome; }
}

public class Guerreiro extends Campeao {
    public Guerreiro(String nome) {
        super(nome);
    }
    
    @Override
    public void atacar() {
        System.out.println(nome + " usa espada!");
    }
    
    @Override
    public void morrer() {
        System.out.println(nome + " caiu em glória!");
    }
}

public class Mago extends Campeao {
    public Mago(String nome) {
        super(nome);
    }
    
    @Override
    public void atacar() {
        System.out.println(nome + " lança feitiço!");
    }
    
    @Override
    public void morrer() {
        System.out.println(nome + " desaparece em chamas!");
    }
}

// Usando:
Campeao[] campeoes = {
    new Guerreiro("Garen"),
    new Mago("Lux")
};

for (Campeao c : campeoes) {
    c.atacar();          // Polimorfismo (abstrato)
    c.ganharExp();       // Herdado (concreto)
}
```

---

## 4. Classes Abstratas vs. Interfaces

### Comparação Detalhada

```java
// INTERFACE: Define o que uma classe PODE fazer
public interface Atacavel {
    void atacar();  // Qualquer classe pode implementar
}

// CLASSE ABSTRATA: Define a base para uma hierarquia
public abstract class Criatura {
    protected String nome;
    
    public abstract void atacar();  // Subclasses devem implementar
    
    public void mover() {           // Comportamento compartilhado
        System.out.println("Movendo...");
    }
}

// Exemplo: Guerreiro é uma Criatura E implementa Atacavel
public class Guerreiro extends Criatura implements Atacavel {
    @Override
    public void atacar() {
        System.out.println("Atacando!");
    }
}
```

### Quando Usar Qual?

```java
// ✓ Use INTERFACE para uma CAPACIDADE
public interface Curavel {
    void curar(Criatura alvo);
}

// Um Bardo pode ser Criatura, Atacavel, E Curavel
public class Bardo extends Criatura implements Atacavel, Curavel {
    // ...
}

// ✓ Use CLASSE ABSTRATA para uma HIERARQUIA
public abstract class Criatura {
    protected String nome;
    abstract void atacar();
}

// ✓ Combine: classe abstrata + interfaces
public abstract class Campeao implements Atacavel, Curavel {
    public abstract void atacar();
    public abstract void curar(Campeao alvo);
}
```

---

## 5. Exemplo Prático: Hierarquia Completa

```java
// Classe base abstrata
public abstract class Campeao {
    protected String nome;
    protected int hp;
    protected int nivel;
    
    public Campeao(String nome, int hp) {
        this.nome = nome;
        this.hp = hp;
        this.nivel = 1;
    }
    
    // Método abstrato: cada tipo ataca diferente
    public abstract void atacar(Campeao alvo);
    
    // Método abstrato: cada tipo cura diferente (ou não cura)
    public abstract void curar(Campeao alvo);
    
    // Método concreto: todos ganham exp igual
    public void ganharExp(int exp) {
        nivel += exp / 100;
        System.out.println(nome + " subiu para nível " + nivel);
    }
    
    // Método concreto: todos levam dano igual
    public void levarDano(int dano) {
        hp -= dano;
        System.out.println(nome + " levou " + dano + " de dano!");
        if (hp <= 0) morrer();
    }
    
    // Método abstrato: cada tipo morre diferente
    public abstract void morrer();
    
    public String getNome() { return nome; }
    public int getHp() { return hp; }
}

// Subclass 1: Tank
public class Tank extends Campeao {
    private int resistencia;
    
    public Tank(String nome) {
        super(nome, 800);
        this.resistencia = 200;
    }
    
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(nome + " carrega com investida!");
        alvo.levarDano(100);
    }
    
    @Override
    public void curar(Campeao alvo) {
        System.out.println(nome + " não cura!");
    }
    
    @Override
    public void morrer() {
        System.out.println(nome + " cai com honra!");
    }
}

// Subclass 2: Mago
public class Mago extends Campeao {
    private int inteligencia;
    
    public Mago(String nome) {
        super(nome, 400);
        this.inteligencia = 200;
    }
    
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(nome + " lança explosão!");
        alvo.levarDano(200);
    }
    
    @Override
    public void curar(Campeao alvo) {
        System.out.println(nome + " cura " + alvo.getNome());
        alvo.hp += 150;
    }
    
    @Override
    public void morrer() {
        System.out.println(nome + " desaparece em chamas!");
    }
}

// Subclass 3: Suporte
public class Suporte extends Campeao {
    private int sabedoria;
    
    public Suporte(String nome) {
        super(nome, 500);
        this.sabedoria = 180;
    }
    
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(nome + " ataca com fraqueza!");
        alvo.levarDano(50);
    }
    
    @Override
    public void curar(Campeao alvo) {
        System.out.println(nome + " cura " + alvo.getNome() + " generosamente!");
        alvo.hp += 250;
    }
    
    @Override
    public void morrer() {
        System.out.println(nome + " protege os aliados até o fim!");
    }
}

// Simulação:
public class Partida {
    public static void main(String[] args) {
        Campeao[] time = {
            new Tank("Garen"),
            new Mago("Lux"),
            new Suporte("Lulu")
        };
        
        System.out.println("=== Batalha Iniciada ===\n");
        
        // Todos atacam (polimorfismo com método abstrato)
        for (Campeao c : time) {
            c.atacar(new Tank("Inimigo"));
        }
        
        System.out.println();
        
        // Todos ganham exp (método concreto herdado)
        for (Campeao c : time) {
            c.ganharExp(300);
        }
        
        System.out.println();
        
        // Todos levam dano (método concreto herdado)
        for (Campeao c : time) {
            c.levarDano(50);
        }
        
        System.out.println();
        
        // Todos morrem (polimorfismo com método abstrato)
        for (Campeao c : time) {
            if (c.getHp() <= 0) {
                c.morrer();
            }
        }
    }
}

// Output:
// === Batalha Iniciada ===
//
// Garen carrega com investida!
// Lux lança explosão!
// Lulu ataca com fraqueza!
//
// Garen subiu para nível 4
// Lux subiu para nível 4
// Lulu subiu para nível 4
//
// Garen levou 50 de dano!
// Lux levou 50 de dano!
// Lulu levou 50 de dano!
//
// Garen não cai tão fácil!
// Lux desaparece em chamas!
// Lulu protege os aliados até o fim!
```

---

## 6. Padrão: Template Method

Classes abstratas são perfeitas para o padrão **Template Method**:

```java
public abstract class ProcessoAtaque {
    // Método template: define o fluxo
    public final void executarAtaque(Campeao alvo) {
        preparar();
        calcularDano();
        aplicarDano(alvo);
        animacao();
    }
    
    // Métodos que subclasses devem implementar
    protected abstract void preparar();
    protected abstract void calcularDano();
    protected abstract void animacao();
    
    // Método concreto compartilhado
    protected void aplicarDano(Campeao alvo) {
        System.out.println("Aplicando dano...");
        alvo.levarDano(100);
    }
}

public class AtaqueGuerrreiro extends ProcessoAtaque {
    @Override
    protected void preparar() {
        System.out.println("Levantando espada...");
    }
    
    @Override
    protected void calcularDano() {
        System.out.println("Calculando força do golpe...");
    }
    
    @Override
    protected void animacao() {
        System.out.println("🗡️ Golpe poderoso!");
    }
}

public class AtaqueMago extends ProcessoAtaque {
    @Override
    protected void preparar() {
        System.out.println("Canalizando mana...");
    }
    
    @Override
    protected void calcularDano() {
        System.out.println("Amplificando feitiço...");
    }
    
    @Override
    protected void animacao() {
        System.out.println("✨ Explosão mágica!");
    }
}

// Usando:
ProcessoAtaque ataque1 = new AtaqueGuerrreiro();
ataque1.executarAtaque(inimigo);

ProcessoAtaque ataque2 = new AtaqueMago();
ataque2.executarAtaque(inimigo);
```

---

## 7. Comparação: Abstract vs. Interface vs. Herança Normal

```java
// HERANÇA NORMAL
public class Animal {
    public void fazer_som() { }  // Implementação padrão
}

// CLASS ABSTRATA
public abstract class Animal {
    public abstract void fazer_som();  // Força implementação
    
    public void mover() { }  // Implementação compartilhada
}

// INTERFACE
public interface Animal {
    void fazer_som();  // Apenas assinatura
}

// QUAL USAR?
// - Herança normal: compartilhar código completo
// - Classe abstrata: criar base com métodos abstratos + concretos
// - Interface: definir contrato (capacidade)
```

---

## 8. Erros Comuns

### Tentar Instanciar Classe Abstrata

```java
// ❌ Erro: classe abstrata
Campeao c = new Campeao("Genérico");

// ✓ OK: subclass concreta
Campeao c = new Guerreiro("Garen");
```

### Não Implementar Método Abstrato

```java
public abstract class Campeao {
    public abstract void atacar();
}

// ❌ Erro: não implementou atacar()
public class Guerreiro extends Campeao {
    // Falta atacar()!
}

// ✓ OK
public class Guerreiro extends Campeao {
    @Override
    public void atacar() {
        System.out.println("Atacando!");
    }
}
```

### Esquecer `abstract` na Subclass

```java
public abstract class Campeao {
    public abstract void atacar();
    public abstract void defender();
}

// ❌ Se não implementa todos os abstratos, precisa ser abstract também
public abstract class CampeaoHibrido extends Campeao {
    @Override
    public void atacar() { }
    
    // Não implementou defender(), então precisa de abstract
}

// ✓ Correto
public abstract class CampeaoHibrido extends Campeao {
    @Override
    public void atacar() { }
    
    // defendr() ainda é abstrato, então classe é abstract
}

// ✓ Ou implementar tudo
public class CampeaoCompleto extends Campeao {
    @Override
    public void atacar() { }
    
    @Override
    public void defender() { }
}
```

---

## 9. Modificadores em Classes Abstratas

```java
public abstract class Campeao {
    // Public: qualquer um acessa
    public abstract void atacar();
    
    // Protected: subclasses acessam
    protected abstract void calcularDano();
    
    // Package-private: mesmo pacote
    abstract void metodoInterno();
    
    // Private: ERRO! Não faz sentido (subclass não consegue implementar)
    // private abstract void isso_eh_erro();  // Não compila
}
```

---

## Resumo

| Conceito | Uso |
|----------|-----|
| **Abstract** | Marca classe ou método como incompleto |
| **Método Abstrato** | Força subclasses a implementar |
| **Método Concreto** | Compartilhado com todas subclasses |
| **Construtor** | Classes abstratas podem ter |
| **Polimorfismo** | Trabalhar via referência abstrata |

---

## Dicas Finais

✓ Use classes abstratas para **hierarquias com lógica compartilhada**  
✓ Use interfaces para **capacidades independentes**  
✓ Classes abstratas são **mais específicas que interfaces**  
✓ Um objeto pode ser **uma classe abstrata E implementar interfaces**  
✓ Sempre implemente **TODOS os métodos abstratos**  

Classes abstratas estruturam código robusto! 🎮
