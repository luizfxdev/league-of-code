# Interfaces em Java - LoL Edition

Interfaces definem **contratos**: "Se você quer ser X, você PRECISA ter esses métodos".

---

## 1. O Conceito de Interface

### O Problema

```java
// Nem todos os campeões podem curar
public class Campeao {
    public void curar(Campeao alvo) {
        // Nem todo campeão tem essa habilidade!
    }
}

// Nem todos os campeões podem atordoar
public class Campeao {
    public void atordoar(Campeao alvo) {
        // Nem todo campeão tem essa habilidade!
    }
}
```

### A Solução: Interfaces

Uma interface define **o que uma classe PODE fazer**, sem implementar:

```java
// Interface: "Se você quer curar, implemente isso"
public interface Curavel {
    void curar(Campeao alvo);
    int getManaGastoParaCurar();
}

// Interface: "Se você quer atordoar, implemente isso"
public interface Atordoavel {
    void atordoar(Campeao alvo);
    int getDuracao();
}

// Classe que ESCOLHE ser curável
public class Suporte extends Campeao implements Curavel {
    @Override
    public void curar(Campeao alvo) {
        System.out.println(getNome() + " curou " + alvo.getNome());
    }
    
    @Override
    public int getManaGastoParaCurar() {
        return 100;
    }
}

// Classe que ESCOLHE ser atordoável
public class CampeaoCC extends Campeao implements Atordoavel {
    @Override
    public void atordoar(Campeao alvo) {
        System.out.println(getNome() + " atordoou " + alvo.getNome());
    }
    
    @Override
    public int getDuracao() {
        return 2;  // 2 segundos
    }
}
```

---

## 2. Anatomia de uma Interface

### Definição

```java
public interface Atacavel {
    // Constantes (sempre public static final)
    int DANO_MINIMO = 10;
    int DANO_MAXIMO = 500;
    
    // Métodos abstratos (sem corpo)
    void atacar(Campeao alvo);
    int calcularDano();
    
    // Método com implementação (Java 8+)
    default void verificarDano() {
        System.out.println("Verificando dano...");
    }
}
```

**Características:**

- ✓ Todos os métodos são **públicos** por padrão
- ✓ Todos os atributos são **constantes** (public static final)
- ✓ NÃO tem construtor
- ✓ NÃO tem atributos de instância
- ✓ NÃO tem métodos private (antes de Java 9)

---

## 3. Implementando Interfaces

### Uma Classe, Uma Interface

```java
public interface Atacavel {
    void atacar(Campeao alvo);
}

public class Guerreiro extends Campeao implements Atacavel {
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(getNome() + " faz um golpe!");
    }
}

// Usando:
Atacavel atacavel = new Guerreiro();
atacavel.atacar(new Guerreiro());
```

### Uma Classe, Múltiplas Interfaces

```java
public interface Atacavel {
    void atacar(Campeao alvo);
}

public interface Curavel {
    void curar(Campeao alvo);
}

// Um Suporte pode atacar E curar
public class Suporte extends Campeao implements Atacavel, Curavel {
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(getNome() + " ataca!");
    }
    
    @Override
    public void curar(Campeao alvo) {
        System.out.println(getNome() + " cura " + alvo.getNome());
    }
}

// Usando:
Suporte lulu = new Suporte("Lulu");
lulu.atacar(new Guerreiro());  // Atacar
lulu.curar(new Guerreiro());   // Curar
```

---

## 4. Polimorfismo com Interfaces

### O Poder Real das Interfaces

```java
public interface Atacavel {
    void atacar(Campeao alvo);
}

public class Guerreiro extends Campeao implements Atacavel {
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(getNome() + " golpeia!");
    }
}

public class Mago extends Campeao implements Atacavel {
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(getNome() + " lança feitiço!");
    }
}

public class Arqueiro extends Campeao implements Atacavel {
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(getNome() + " dispara seta!");
    }
}

// Agora você pode agrupar tudo que implementa Atacavel
public class Combate {
    public void iniciarBatalha(Atacavel[] atacantes, Campeao inimigo) {
        for (Atacavel a : atacantes) {
            a.atacar(inimigo);  // Polimorfismo!
        }
    }
}

// Usando:
Atacavel[] time = {
    new Guerreiro("Garen"),
    new Mago("Lux"),
    new Arqueiro("Ashe")
};

Combate combate = new Combate();
combate.iniciarBatalha(time, new Guerreiro("Inimigo"));

// Output:
// Garen golpeia!
// Lux lança feitiço!
// Ashe dispara seta!
```

### Vantagem: Adicionar Novos Tipos é Fácil

```java
// Quando você precisa de um novo tipo, basta implementar a interface
public class Ladrao extends Campeao implements Atacavel {
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(getNome() + " apunhala!");
    }
}

// Funciona imediatamente com o código existente!
Atacavel[] time = {
    new Guerreiro("Garen"),
    new Ladrao("Katarina")  // Novo tipo
};

combate.iniciarBatalha(time, inimigo);  // Sem modificar Combate!
```

---

## 5. Herança de Interfaces

### Uma Interface Pode Herdar de Outra

```java
// Interface base
public interface Criatura {
    void mover();
    void morrer();
}

// Interface que estende
public interface Combatente extends Criatura {
    void atacar(Criatura alvo);
}

// Classe que implementa a interface filha
public class Campeao implements Combatente {
    @Override
    public void mover() {
        System.out.println("Movendo...");
    }
    
    @Override
    public void morrer() {
        System.out.println("Morreu!");
    }
    
    @Override
    public void atacar(Criatura alvo) {
        System.out.println("Atacando!");
    }
}
```

### Uma Interface Pode Herdar de Múltiplas Interfaces

```java
public interface Atacavel {
    void atacar(Criatura alvo);
}

public interface Curavel {
    void curar(Criatura alvo);
}

// Combatente herda de ambas
public interface Combatente extends Atacavel, Curavel {
    void esquivar();
}

// Classe implementa Combatente (e herda de Atacavel + Curavel)
public class Suporte extends Campeao implements Combatente {
    @Override
    public void atacar(Criatura alvo) { }
    
    @Override
    public void curar(Criatura alvo) { }
    
    @Override
    public void esquivar() { }
}
```

---

## 6. Métodos Default em Interfaces (Java 8+)

### Problema: Adicionar Método Quebra Código

```java
// Versão 1
public interface Atacavel {
    void atacar(Campeao alvo);
}

// Versão 2: Novo método!
public interface Atacavel {
    void atacar(Campeao alvo);
    void preparar();  // ❌ Classes existentes quebram!
}

// Todas as implementações PRECISAM adicionar preparar()
public class Guerreiro implements Atacavel {
    @Override
    public void atacar(Campeao alvo) { }
    
    @Override
    public void preparar() { }  // Obrigado a implementar
}
```

### Solução: Default

```java
public interface Atacavel {
    void atacar(Campeao alvo);
    
    // Método com implementação padrão
    default void preparar() {
        System.out.println("Preparando ataque...");
    }
}

// Classes existentes continuam funcionando
public class Guerreiro implements Atacavel {
    @Override
    public void atacar(Campeao alvo) { }
    
    // preparar() já tem implementação padrão
}

// Usando:
Guerreiro garen = new Guerreiro();
garen.preparar();  // Usa implementação default
```

---

## 7. Exemplo Prático Completo

```java
// Interfaces
public interface Atacavel {
    void atacar(Campeao alvo);
}

public interface Curavel {
    void curar(Campeao alvo);
    int getManaGasto();
}

public interface Esquivavel {
    void esquivar();
    int getTaxaEsquiva();
}

// Classes
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
        hp -= dano;
    }
    
    public String getNome() { return nome; }
    public int getHp() { return hp; }
}

// Tank: Ataca e Esquiva
public class Tank extends Campeao implements Atacavel, Esquivavel {
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(nome + " carrega com investida!");
        alvo.levarDano(100);
    }
    
    @Override
    public void esquivar() {
        System.out.println(nome + " usa escudo para esquivar!");
    }
    
    @Override
    public int getTaxaEsquiva() {
        return 30;  // 30%
    }
}

// Mago: Ataca e Cura
public class Mago extends Campeao implements Atacavel, Curavel {
    @Override
    public void atacar(Campeao alvo) {
        System.out.println(nome + " lança explosão mágica!");
        alvo.levarDano(150);
        mana -= 50;
    }
    
    @Override
    public void curar(Campeao alvo) {
        System.out.println(nome + " recupera " + alvo.getNome());
        alvo.hp += 100;
        mana -= getManaGasto();
    }
    
    @Override
    public int getManaGasto() {
        return 80;
    }
}

// Suporte: Cura e Esquiva
public class Suporte extends Campeao implements Curavel, Esquivavel {
    @Override
    public void curar(Campeao alvo) {
        System.out.println(nome + " ativa cura em área!");
        alvo.hp += 150;
    }
    
    @Override
    public int getManaGasto() {
        return 60;
    }
    
    @Override
    public void esquivar() {
        System.out.println(nome + " usa agilidade para esquivar!");
    }
    
    @Override
    public int getTaxaEsquiva() {
        return 40;  // 40%
    }
}

// Usando tudo junto:
public class Partida {
    public static void main(String[] args) {
        Campeao[] time = {
            new Tank("Garen"),
            new Mago("Lux"),
            new Suporte("Lulu")
        };
        
        Campeao[] inimigos = {
            new Tank("Malphite"),
            new Mago("Ahri")
        };
        
        System.out.println("=== Batalha Iniciada ===\n");
        
        // Todos que implementam Atacavel atacam
        for (Campeao c : time) {
            if (c instanceof Atacavel) {
                ((Atacavel) c).atacar(inimigos[0]);
            }
        }
        
        System.out.println();
        
        // Todos que podem esquivar fazem isso
        for (Campeao c : time) {
            if (c instanceof Esquivavel) {
                ((Esquivavel) c).esquivar();
            }
        }
        
        System.out.println();
        
        // Todos que podem curar fazem isso
        for (Campeao c : time) {
            if (c instanceof Curavel) {
                ((Curavel) c).curar(time[0]);
            }
        }
    }
}

// Output:
// === Batalha Iniciada ===
//
// Garen carrega com investida!
// Lux lança explosão mágica!
//
// Garen usa escudo para esquivar!
// Lulu usa agilidade para esquivar!
//
// Lux recupera Garen
// Lulu ativa cura em área!
```

---

## 8. Interface vs. Classe Abstrata

### Diferenças

| Aspecto | Interface | Classe Abstrata |
|---------|-----------|-----------------|
| **Herança** | Implementa (pode múltiplas) | Estende (apenas 1) |
| **Atributos** | Apenas constantes | Qualquer tipo |
| **Métodos** | Todos abstratos (antes de Java 8) | Mix: abstratos + concretos |
| **Construtor** | Não tem | Tem |
| **Modificadores** | Public por padrão | Podem ser private |
| **Propósito** | "Contrato" / "Capacidade" | "Classe base" / "Template" |

### Quando Usar?

```java
// Use INTERFACE para uma capacidade/contrato
public interface Atacavel {
    void atacar(Campeao alvo);
}

// Use CLASSE ABSTRATA para uma hierarquia
public abstract class Campeao {
    protected String nome;
    
    abstract void atacar(Campeao alvo);
}
```

---

## 9. Padrão: Verificação com instanceof

```java
Campeao[] times = { ... };

for (Campeao c : times) {
    // Verificar antes de usar a interface
    if (c instanceof Atacavel) {
        Atacavel atacavel = (Atacavel) c;
        atacavel.atacar(inimigo);
    }
    
    if (c instanceof Curavel) {
        Curavel curavel = (Curavel) c;
        curavel.curar(aliado);
    }
}

// Java 16+: Pattern Matching
for (Campeao c : times) {
    if (c instanceof Atacavel atacavel) {  // Cast automático
        atacavel.atacar(inimigo);
    }
}
```

---

## 10. Erros Comuns

### Esquecer @Override

```java
public interface Atacavel {
    void atacar(Campeao alvo);
}

public class Guerreiro implements Atacavel {
    // ❌ Sem @Override, erros passam despercebidos
    public void atacar(Campeao alvo) {  // Typo: deveria ser atacar
        System.out.println("Atacando!");
    }
    
    // ✓ Com @Override, Java avisa
    @Override
    public void atacar(Campeao alvo) {
        System.out.println("Atacando!");
    }
}
```

### Cast Sem Verificação

```java
// ❌ ClassCastException em runtime
Campeao c = new Guerreiro();
Curavel curavel = (Curavel) c;  // Erro se Guerreiro não implementa Curavel
curavel.curar(aliado);

// ✓ Verificar primeiro
if (c instanceof Curavel) {
    Curavel curavel = (Curavel) c;
    curavel.curar(aliado);
}
```

### Esquecer de Implementar Todos os Métodos

```java
public interface Atacavel {
    void atacar(Campeao alvo);
    int getDano();
}

public class Guerreiro implements Atacavel {
    @Override
    public void atacar(Campeao alvo) { }
    
    // ❌ Erro de compilação: falta getDano()
}

public class Guerreiro implements Atacavel {
    @Override
    public void atacar(Campeao alvo) { }
    
    @Override
    public int getDano() { return 100; }  // ✓ Correto
}
```

---

## Resumo

| Conceito | Uso |
|----------|-----|
| **Interface** | Define contratos, capacidades |
| **Implements** | Uma classe implementa interface |
| **Polimorfismo** | Trabalhar com interface, não implementação |
| **Default** | Métodos com implementação padrão |
| **instanceof** | Verificar tipo antes de cast |

---

## Dicas Finais

✓ Use interfaces para **flexibilidade e extensibilidade**  
✓ Um objeto pode ser múltiplas interfaces (muito prático!)  
✓ Sempre use `@Override` para segurança  
✓ Prefira **programar para a interface, não para a implementação**  
✓ Use `instanceof` para verificar tipo antes de cast  

Interfaces são a chave para código flexível! 🎮
