# Programação Orientada a Objetos (POO) em Java com Pokémon

## 1. Conceitos Fundamentais

### O que é POO?

Programação Orientada a Objetos é um paradigma que organiza o código em **objetos** (instâncias de classes) que possuem **estado** (dados) e **comportamento** (métodos).

**Analogia Pokémon:**
- Um Pokémon é um objeto
- Pikachu é uma instância da classe Pokémon
- Atributos: nome, nível, tipo, HP
- Métodos: atacar(), ganharExp(), evoluir()

---

## 2. Classes e Objetos

### Definição

**Classe** = Molde, blueprint, tipo de dado estruturado
**Objeto** = Instância concreta da classe

```java
// Classe (o molde)
public class Pokemon {
    public String nome;
    public String tipo;
    public int nivel;
    public int hp;
}

// Criando objetos (instâncias)
Pokemon pikachu = new Pokemon();
Pokemon charmander = new Pokemon();

pikachu.nome = "Pikachu";
pikachu.tipo = "Elétrico";
pikachu.nivel = 5;
pikachu.hp = 35;

charmander.nome = "Charmander";
charmander.tipo = "Fogo";
charmander.nivel = 5;
charmander.hp = 39;
```

### Estrutura em Memória

```
Stack                          Heap
???????????????               ????????????????????
? pikachu ???????????????????>? Pokemon          ?
? charmander ?????????????    ? nome: "Pikachu"  ?
???????????????          ?    ? tipo: "Elétrico" ?
                         ?    ? nivel: 5         ?
                         ?    ? hp: 35           ?
                         ?    ????????????????????
                         ?
                         ????>????????????????????
                              ? Pokemon          ?
                              ? nome: "Charmander"
                              ? tipo: "Fogo"     ?
                              ? nivel: 5         ?
                              ? hp: 39           ?
                              ????????????????????
```

---

## 3. Atributos (Propriedades)

### O que são?

Dados que descrevem o estado do objeto. Em Pokémon: nome, HP, ataque, defesa, tipo.

```java
public class Pokemon {
    public String nome;           // String
    public String tipo;           // String  
    public int nivel;             // int
    public int hp;                // int
    public int ataque;            // int
    public int defesa;            // int
    public double peso;           // double
    public boolean capturado;     // boolean
}
```

### Acessando Atributos

```java
Pokemon pikachu = new Pokemon();
pikachu.nome = "Pikachu";           // Escrita
System.out.println(pikachu.nome);   // Leitura: "Pikachu"
pikachu.hp = pikachu.hp - 10;       // Modificação
```

**Referência Clean Code (Robert Martin):**
- Use nomes descritivos
- Evite prefixos/sufixos redundantes: ? `pokemonNome`, ? `nome`
- Um atributo = uma responsabilidade

---

## 4. Métodos (Comportamentos)

### O que são?

Funções que definem as ações que um objeto pode executar.

```java
public class Pokemon {
    public String nome;
    public int hp;
    public int ataque;
    
    public void atacar(Pokemon adversario) {
        int dano = this.ataque;
        adversario.hp -= dano;
        System.out.println(this.nome + " atacou " + adversario.nome + "!");
    }
    
    public void ganharExp(int exp) {
        System.out.println(this.nome + " ganhou " + exp + " de experiência!");
    }
    
    public int calcularDano() {
        return this.ataque + (int)(Math.random() * 10);
    }
}
```

### Uso

```java
Pokemon pikachu = new Pokemon();
pikachu.nome = "Pikachu";
pikachu.hp = 35;
pikachu.ataque = 55;

Pokemon squirtle = new Pokemon();
squirtle.nome = "Squirtle";
squirtle.hp = 44;

pikachu.atacar(squirtle);  // Pikachu atacou Squirtle!
pikachu.ganharExp(100);    // Pikachu ganhou 100 de experiência!
```

### Diferença entre `this` e sem `this`

```java
public void atacar(Pokemon adversario) {
    // Com this (explícito, mais legível):
    int dano1 = this.ataque;
    
    // Sem this (implícito, funciona igual):
    int dano2 = ataque;
    
    // Use this quando há ambiguidade:
    this.ataque = this.ataque + 5;  // Claro que é o atributo da classe
}
```

---

## 5. Construtores

### O que é?

Método especial que inicializa o objeto. Executado automaticamente com `new`.

```java
public class Pokemon {
    public String nome;
    public String tipo;
    public int nivel;
    public int hp;
    
    // Construtor sem parâmetros (padrão)
    public Pokemon() {
        this.nivel = 1;
        this.hp = 20;
    }
    
    // Construtor com parâmetros (overload)
    public Pokemon(String nome, String tipo, int hp) {
        this.nome = nome;
        this.tipo = tipo;
        this.hp = hp;
        this.nivel = 1;
    }
}

// Uso
Pokemon pikachu = new Pokemon();  // Usa construtor padrão
Pokemon charmander = new Pokemon("Charmander", "Fogo", 39);
```

### Sobrecarga de Construtores (Overloading)

```java
public class Pokemon {
    public String nome;
    public int hp;
    public int nivel;
    
    // Construtor 1: sem parâmetros
    public Pokemon() {
        this("Pokemon", 20, 1);
    }
    
    // Construtor 2: nome + HP
    public Pokemon(String nome, int hp) {
        this(nome, hp, 1);
    }
    
    // Construtor 3: nome + HP + nível
    public Pokemon(String nome, int hp, int nivel) {
        this.nome = nome;
        this.hp = hp;
        this.nivel = nivel;
    }
}

// Cada construtor é uma forma diferente de criar o objeto
Pokemon p1 = new Pokemon();
Pokemon p2 = new Pokemon("Pikachu", 35);
Pokemon p3 = new Pokemon("Charmander", 39, 5);
```

---

## 6. Encapsulamento

### O Problema do Acesso Público

```java
public class Pokemon {
    public int hp = 100;
}

Pokemon pikachu = new Pokemon();
pikachu.hp = -500;  // ? Inválido! HP nunca deve ser negativo
```

### A Solução: Getters e Setters com Validação

```java
public class Pokemon {
    private int hp;  // private: só acessa dentro da classe
    
    // Getter: ler o valor com segurança
    public int getHp() {
        return this.hp;
    }
    
    // Setter: escrever o valor com validação
    public void setHp(int hp) {
        if (hp < 0) {
            this.hp = 0;  // Não permite negativo
        } else if (hp > 100) {
            this.hp = 100;  // Limita ao máximo
        } else {
            this.hp = hp;
        }
    }
}

Pokemon pikachu = new Pokemon();
pikachu.setHp(50);      // ? Válido
pikachu.setHp(-100);    // ? Ajustado para 0
System.out.println(pikachu.getHp());  // 0
```

### Modificadores de Acesso

| Modificador | Mesmo Pacote | Pacotes Diferentes | Observação |
|------------|-----------|--------|-----------|
| `public` | ? | ? | Acessível de qualquer lugar |
| `private` | ? | ? | Acessível apenas dentro da classe |
| `protected` | ? | ? (subclasses) | Herdável, restrito |
| (padrão) | ? | ? | Padrão do pacote |

```java
public class Pokemon {
    private String nome;           // Só Pokemon acessa
    protected int ataque;          // Pokemon + subclasses acessam
    public void atacar() {}        // Qualquer um acessa
    void recuperarEnergia() {}     // Só o pacote acessa
}
```

---

## 7. toString() e Object

### toString()

Converte o objeto em String para exibição legível.

```java
public class Pokemon {
    private String nome;
    private String tipo;
    private int hp;
    
    @Override
    public String toString() {
        return String.format(
            "Pokemon{nome='%s', tipo='%s', hp=%d}",
            this.nome, this.tipo, this.hp
        );
    }
}

Pokemon pikachu = new Pokemon("Pikachu", "Elétrico", 35);
System.out.println(pikachu);  
// Saída: Pokemon{nome='Pikachu', tipo='Elétrico', hp=35}
```

### Métodos herdados de Object

Toda classe em Java herda de `Object`:

```java
public class Pokemon {
    // getClass(): retorna a classe do objeto
    Pokemon p = new Pokemon();
    Class<?> classe = p.getClass();  // class Pokemon
    
    // equals(): compara igualdade entre objetos
    Pokemon p1 = new Pokemon("Pikachu", 35);
    Pokemon p2 = new Pokemon("Pikachu", 35);
    System.out.println(p1.equals(p2));  // false (referências diferentes)
    
    // hashCode(): código único do objeto
    System.out.println(p1.hashCode());  // e.g., 1550089733
    
    // toString(): representação em String (já sobrescrito acima)
}
```

---

## 8. Membros Estáticos (Static)

### O que é?

Membros que pertencem à **classe**, não ao **objeto**. Compartilhados por todas as instâncias.

```java
public class Pokemon {
    private String nome;
    private int nivel;
    
    // Atributo estático: um valor compartilhado por todos os Pokémon
    public static int totalCapturados = 0;
    
    public Pokemon(String nome) {
        this.nome = nome;
        this.nivel = 1;
        Pokemon.totalCapturados++;  // Incrementa o contador global
    }
    
    public static void exibirTotal() {
        System.out.println("Total capturado: " + totalCapturados);
    }
}

// Uso
Pokemon pikachu = new Pokemon("Pikachu");
Pokemon charmander = new Pokemon("Charmander");
Pokemon.exibirTotal();  // Total capturado: 2
```

### Estático vs Instância

```java
public class Calculadora {
    // Atributo de instância: cada calculadora tem sua memória
    private double memoriaLocal = 0;
    
    // Atributo estático: compartilhado entre todas as calculadoras
    public static final double PI = 3.14159;
    
    // Método de instância: usa dados do objeto
    public void adicionar(double valor) {
        this.memoriaLocal += valor;
    }
    
    // Método estático: não acessa dados da instância
    public static double circunferencia(double raio) {
        return 2.0 * PI * raio;
    }
}

Calculadora calc1 = new Calculadora();
Calculadora calc2 = new Calculadora();

calc1.adicionar(5);  // Afeta apenas calc1
// calc2.memoriaLocal permanece 0

Calculadora.circunferencia(10);  // Não precisa de instância
```

### Constantes Estáticas

```java
public class Tipo {
    public static final String ELETRICO = "Elétrico";
    public static final String FOGO = "Fogo";
    public static final String AGUA = "Água";
    public static final String GRAMA = "Grama";
}

Pokemon pikachu = new Pokemon("Pikachu", Tipo.ELETRICO);
```

---

## 9. Herança

### O que é?

Reutilização de código. Uma classe **herda** atributos e métodos de outra.

```java
// Classe base (superclasse)
public class Animal {
    protected String nome;
    protected int idade;
    
    public void dormir() {
        System.out.println(this.nome + " está dormindo...");
    }
}

// Classe derivada (subclasse) - herda de Animal
public class Pokemon extends Animal {
    private String tipo;
    
    public void atacar() {
        System.out.println(this.nome + " atacou!");
    }
}

// Uso
Pokemon pikachu = new Pokemon();
pikachu.nome = "Pikachu";
pikachu.dormir();   // Herdado de Animal
pikachu.atacar();   // Próprio de Pokemon
```

### Hierarquia em Pokémon

```java
// Nível 1: Animal (super genérico)
public class Animal {
    protected String nome;
    public void fazer_som() { }
}

// Nível 2: Criatura (mais específico)
public class Criatura extends Animal {
    protected int hp;
    protected String tipo;
}

// Nível 3: Pokemon (ainda mais específico)
public class Pokemon extends Criatura {
    private int nivel;
    public void atacar() { }
}

// Pikachu (concreto)
Pokemon pikachu = new Pokemon();
```

### Super - Acessando a classe pai

```java
public class Criatura {
    protected String nome = "Criatura";
    
    public void apresentar() {
        System.out.println("Olá, sou uma " + this.nome);
    }
}

public class Pokemon extends Criatura {
    public Pokemon(String nome) {
        this.nome = nome;  // Atributo herdado
    }
    
    @Override
    public void apresentar() {
        super.apresentar();  // Chama o método da superclasse
        System.out.println("E sou um Pokémon do tipo elétrico!");
    }
}

Pokemon pikachu = new Pokemon("Pikachu");
pikachu.apresentar();
// Olá, sou uma Pikachu
// E sou um Pokémon do tipo elétrico!
```

---

## 10. Polimorfismo

### O que é?

"Muitas formas". Objetos podem ser usados através de suas superclasses.

```java
public class Criatura {
    protected String nome;
    public void atacar() { }
}

public class Pokemon extends Criatura {
    @Override
    public void atacar() {
        System.out.println(this.nome + " usou um movimento Pokémon!");
    }
}

public class Dragao extends Criatura {
    @Override
    public void atacar() {
        System.out.println(this.nome + " lança fogo dragão!");
    }
}

// Polimorfismo: uma lista contém tipos diferentes
List<Criatura> criaturas = new ArrayList<>();
criaturas.add(new Pokemon("Pikachu"));
criaturas.add(new Dragao("Dragonite"));

// Mesmo método, comportamentos diferentes
for (Criatura criatura : criaturas) {
    criatura.atacar();
}
// Pikachu usou um movimento Pokémon!
// Dragonite lança fogo dragão!
```

### Tipo em Tempo de Execução (instanceof)

```java
Criatura pikachu = new Pokemon("Pikachu");

if (pikachu instanceof Pokemon) {
    System.out.println("É um Pokémon!");
}

if (pikachu instanceof Dragao) {
    System.out.println("É um dragão!");
}
```

---

## 11. Abstração

### Classes Abstratas

Não podem ser instanciadas diretamente. Definem um contrato.

```java
// Classe abstrata
public abstract class Criatura {
    protected String nome;
    
    // Método abstrato: subclasses DEVEM implementar
    public abstract void atacar();
    
    // Método concreto: já implementado
    public void dormir() {
        System.out.println(this.nome + " dormiu!");
    }
}

// Deve implementar o método abstrato
public class Pokemon extends Criatura {
    @Override
    public void atacar() {
        System.out.println(this.nome + " atacou!");
    }
}

// ? Não pode fazer isso
// Criatura c = new Criatura();

// ? Mas pode fazer assim
Criatura pikachu = new Pokemon("Pikachu");
pikachu.atacar();
```

### Interfaces

Contrato que define apenas métodos (sem implementação).

```java
public interface Combatente {
    void atacar(Combatente inimigo);
    void defender();
    int calcularDano();
}

public class Pokemon implements Combatente {
    private String nome;
    private int ataque;
    private int defesa;
    
    @Override
    public void atacar(Combatente inimigo) {
        System.out.println(this.nome + " atacou!");
    }
    
    @Override
    public void defender() {
        System.out.println(this.nome + " se defendeu!");
    }
    
    @Override
    public int calcularDano() {
        return this.ataque;
    }
}

// Uso polimórfico
Combatente pikachu = new Pokemon("Pikachu", 55, 40);
pikachu.atacar(null);
pikachu.defender();
```

---

## 12. Composição vs Herança

### Quando usar?

**Herança ("é um"):** Pokemon **é um** Animal
```java
public class Pokemon extends Animal { }
```

**Composição ("tem um"):** Pokemon **tem um** Treinador
```java
public class Pokemon {
    private Treinador treinador;  // Composição
}
```

### Exemplo Prático

```java
// ? Herança excessiva (errado)
public class Pikachu extends Pokemon {
    // Pikachu herda tudo de Pokemon
}

// ? Composição (correto - "um Pikachu tem características específicas")
public class Pokemon {
    private String especie = "Pikachu";  // Atributo
    private Habilidade habilidade;       // Composição
}

public class Habilidade {
    private String nome = "Esfera de Eletricidade";
    private int poder = 90;
}
```

---

## 13. Princípios SOLID (Clean Code)

### S - Single Responsibility Principle

Uma classe, uma responsabilidade.

```java
// ? Violando SRP
public class Pokemon {
    public void atacar() { }
    public void guardarEmBancoDados() { }
    public void renderizarGrafico() { }
}

// ? Respeitando SRP
public class Pokemon {
    private String nome;
    public void atacar() { }
}

public class PokemonRepositorio {
    public void salvar(Pokemon p) { }
}

public class PokemonRenderer {
    public void renderizar(Pokemon p) { }
}
```

### O - Open/Closed Principle

Aberto para extensão, fechado para modificação.

```java
// ? Violando OCP - precisa modificar a classe a cada novo tipo
public class Ataque {
    public int calcularDano(String tipo) {
        if (tipo.equals("Elétrico")) return 50;
        if (tipo.equals("Fogo")) return 60;
        // Sempre que novo tipo aparece, modifica aqui
    }
}

// ? Respeitando OCP
public abstract class Ataque {
    public abstract int calcularDano();
}

public class AtaqueEletrico extends Ataque {
    @Override
    public int calcularDano() { return 50; }
}

public class AtaqueFogo extends Ataque {
    @Override
    public int calcularDano() { return 60; }
}
```

### L - Liskov Substitution Principle

Objetos de subclasses devem ser substituíveis pelos da superclasse.

```java
// ? LSP respeitado
public class Pokemon extends Criatura {
    @Override
    public void atacar() {
        System.out.println("Pikachu atacou!");
    }
}

// Pode substituir Criatura por Pokemon sem quebrar o programa
Criatura c = new Pokemon();
c.atacar();  // Funciona perfeitamente
```

### I - Interface Segregation Principle

Interfaces pequenas, específicas.

```java
// ? Interface gigante (errado)
public interface Criatura {
    void atacar();
    void defender();
    void voar();
    void nadar();
    void cavar();
}

// ? Interfaces pequenas
public interface Combatente {
    void atacar();
    void defender();
}

public interface Voador {
    void voar();
}

public interface Aquatico {
    void nadar();
}

public class Dragonite implements Combatente, Voador {
    // Implementa apenas o necessário
}
```

### D - Dependency Inversion Principle

Dependa de abstrações, não de implementações concretas.

```java
// ? Dependência de implementação concreta
public class Treinador {
    private PokemonBancoDados banco = new PokemonBancoDados();
    
    public void salvarPokemon(Pokemon p) {
        banco.salvar(p);
    }
}

// ? Dependência de abstração
public interface RepositorioPokemon {
    void salvar(Pokemon p);
}

public class Treinador {
    private RepositorioPokemon repositorio;
    
    public Treinador(RepositorioPokemon repositorio) {
        this.repositorio = repositorio;
    }
    
    public void salvarPokemon(Pokemon p) {
        this.repositorio.salvar(p);
    }
}
```

---

## 14. Exemplo Completo: Sistema de Pokémon

```java
// ===== ABSTRAÇÕES =====

public abstract class Criatura {
    protected String nome;
    protected int hp;
    protected int maxHp;
    protected int ataque;
    protected int defesa;
    
    public abstract void atacar(Criatura alvo);
    
    public void receberDano(int dano) {
        int danoReduzido = Math.max(1, dano - this.defesa / 2);
        this.hp = Math.max(0, this.hp - danoReduzido);
    }
    
    public boolean estaViva() {
        return this.hp > 0;
    }
    
    @Override
    public String toString() {
        return String.format("%s [HP: %d/%d]", this.nome, this.hp, this.maxHp);
    }
}

// ===== IMPLEMENTAÇÕES =====

public class Pokemon extends Criatura {
    private String tipo;
    
    public Pokemon(String nome, String tipo, int hp, int ataque, int defesa) {
        this.nome = nome;
        this.tipo = tipo;
        this.hp = hp;
        this.maxHp = hp;
        this.ataque = ataque;
        this.defesa = defesa;
    }
    
    @Override
    public void atacar(Criatura alvo) {
        int dano = this.ataque + (int)(Math.random() * 10);
        System.out.printf("%s atacou %s com %d de dano!%n", 
            this.nome, alvo.nome, dano);
        alvo.receberDano(dano);
    }
}

// ===== LÓGICA DE JOGO =====

public class Batalha {
    private Criatura criatura1;
    private Criatura criatura2;
    
    public Batalha(Criatura c1, Criatura c2) {
        this.criatura1 = c1;
        this.criatura2 = c2;
    }
    
    public void iniciar() {
        System.out.println("=== BATALLA INICIADA ===");
        System.out.println(criatura1 + " vs " + criatura2);
        System.out.println();
        
        int turno = 1;
        while (criatura1.estaViva() && criatura2.estaViva()) {
            System.out.println("--- TURNO " + turno + " ---");
            
            criatura1.atacar(criatura2);
            System.out.println(criatura2);
            
            if (!criatura2.estaViva()) break;
            
            criatura2.atacar(criatura1);
            System.out.println(criatura1);
            
            System.out.println();
            turno++;
        }
        
        String vencedor = criatura1.estaViva() ? criatura1.nome : criatura2.nome;
        System.out.println("=== " + vencedor.toUpperCase() + " VENCEU! ===");
    }
}

// ===== USO =====

public class Main {
    public static void main(String[] args) {
        Pokemon pikachu = new Pokemon("Pikachu", "Elétrico", 35, 55, 40);
        Pokemon charmander = new Pokemon("Charmander", "Fogo", 39, 52, 43);
        
        Batalha batalha = new Batalha(pikachu, charmander);
        batalha.iniciar();
    }
}
```

---

## 15. Padrões de Código Clean

### Nomes Significativos

```java
// ? Ruim
public class P {
    public int x;
    public void a(P p) { }
}

// ? Bom
public class Pokemon {
    private int hp;
    public void atacar(Pokemon alvo) { }
}
```

### Métodos Pequenos e Focados

```java
// ? Método grande
public void batalha(Pokemon p1, Pokemon p2) {
    // 50 linhas de lógica
}

// ? Métodos pequenos e compostos
public void batalha(Pokemon p1, Pokemon p2) {
    inicializarBatalha(p1, p2);
    executarTurnos(p1, p2);
    finalizarBatalha(p1, p2);
}

private void inicializarBatalha(Pokemon p1, Pokemon p2) { }
private void executarTurnos(Pokemon p1, Pokemon p2) { }
private void finalizarBatalha(Pokemon p1, Pokemon p2) { }
```

### Evite Magic Numbers

```java
// ? O que é 0.5? Por que 100?
public int calcularDano(Pokemon atacante) {
    return (int)(atacante.ataque * 0.5 + 100);
}

// ? Claro
private static final double MULTIPLICADOR_DANO = 0.5;
private static final int DANO_BASE = 100;

public int calcularDano(Pokemon atacante) {
    return (int)(atacante.ataque * MULTIPLICADOR_DANO + DANO_BASE);
}
```

### Uso de Exceções Apropriadamente

```java
// ? Retorna null (confuso)
public Pokemon buscarPokemon(String nome) {
    if (nome == null) return null;
    return banco.buscar(nome);
}

// ? Lança exceção (claro)
public Pokemon buscarPokemon(String nome) {
    if (nome == null || nome.isEmpty()) {
        throw new IllegalArgumentException("Nome inválido");
    }
    Pokemon p = banco.buscar(nome);
    if (p == null) {
        throw new PokemonNaoEncontradoException(nome);
    }
    return p;
}

class PokemonNaoEncontradoException extends RuntimeException {
    public PokemonNaoEncontradoException(String nome) {
        super("Pokémon " + nome + " não encontrado");
    }
}
```

---

## 16. Exercícios Práticos

### Exercício 1: Classe Rectangle

Crie uma classe `Rectangle` com largura e altura. Implemente:
- Getters e setters com validação (não permitir valores ? 0)
- Método `calcularArea()`
- Método `calcularPerimetro()`
- Sobrescrita de `toString()`

**Entrada:**
```
Largura: 3.0
Altura: 4.0
```

**Saída:**
```
AREA = 12.00
PERIMETRO = 14.00
Rectangle{largura=3.0, altura=4.0}
```

---

### Exercício 2: Classe Employee

Crie uma classe `Employee` com nome, salário bruto e imposto. Implemente:
- Getters e setters
- Método `calcularSalarioLiquido()` (bruto - imposto)
- Método `aumentarSalario(porcentagem)` - aumenta apenas o bruto
- Sobrescrita de `toString()`

**Entrada:**
```
Nome: João Silva
Salário bruto: 6000.00
Imposto: 1000.00
Aumento de 10%
```

**Saída:**
```
Antes: João Silva, $ 5000.00
Depois: João Silva, $ 5600.00
```

---

### Exercício 3: Classe Student

Crie uma classe `Student` com nome e 3 notas. Implemente:
- Construtor que recebe nome e 3 notas
- Método `calcularNotaFinal()` (média ponderada: 30%, 35%, 35%)
- Método `passou()` retorna true se nota ? 60
- Método `pontosFaltantes()` retorna quantos pontos faltam para passar
- Sobrescrita de `toString()`

**Entrada:**
```
Alex Green
27.00
31.00
32.00
```

**Saída:**
```
FINAL GRADE = 90.00
PASS

---

Entrada:
Alex Green
17.00
20.00
15.00

Saída:
FINAL GRADE = 52.00
FAILED
MISSING 8.00 POINTS
```

---

### Exercício 4: Classe CurrencyConverter

Crie uma classe utilitária `CurrencyConverter` com:
- Atributo estático `IOF = 0.06` (imposto)
- Método estático `dollarToReal(amount, dollarPrice)` 
- Cálculo: amount * dollarPrice * (1 + IOF)

**Entrada:**
```
Preço do dólar: 3.10
Dólares a comprar: 200.00
```

**Saída:**
```
Amount to be paid in reais = 657.20
```

---

### Exercício 5: Classe Triangulo

Crie uma classe `Triangle` com três lados (a, b, c). Implemente:
- Getters e setters
- Método `area()` usando fórmula de Heron
- Classe `Program` que lê 2 triângulos e diz qual tem maior área

**Fórmula de Heron:**
```
p = (a + b + c) / 2
area = sqrt(p * (p-a) * (p-b) * (p-c))
```

---

### Exercício 6: Pokemon com Herança

Crie:
- Classe abstrata `Criatura` com nome, hp, ataque, defesa
- Classe `Pokemon(extends Criatura)` com tipo
- Classe `Dragao(extends Criatura)` com força de asas
- Implemente `atacar(Criatura alvo)` em ambas (com dano diferente)
- Teste polimorfismo em uma lista

---

### Exercício 7: Conversor de Moedas (Dependency Injection)

Crie:
- Interface `ConversorMoeda` com `converter(valor, taxa)`
- Classe `ConversorDolar implements ConversorMoeda`
- Classe `ConversorEuro implements ConversorMoeda`
- Classe `Carteira` que recebe o conversor no construtor (DI)

---

### Exercício 8: Sistema de Pokedex

Crie:
- Classe `Pokemon` com nome, tipo, hp, ataque
- Interface `RepositorioPokemon` com `salvar()`, `buscar()`, `listarTodos()`
- Classe `PokedexEmMemoria implements RepositorioPokemon`
- Classe `Pokedex` que usa o repositório (composição)

---

### Exercício 9: Validação e Exceções

Crie:
- Classe `Pokemon` que valida: nome não vazio, hp > 0, ataque ? 0
- Exceção customizada `PokemonInvalidoException`
- Lançar a exceção quando dados inválidos

**Exemplo:**
```java
try {
    new Pokemon("", 100, 50);  // Nome vazio
} catch (PokemonInvalidoException e) {
    System.out.println(e.getMessage());
}
```

---

### Exercício 10: Batalha entre Pokémons

Crie um sistema completo:
- Classe `Pokemon` com hp, ataque, defesa
- Classe `Batalha` que simula dois Pokémons lutando
- Cada turno: um ataca (dano aleatório 10%), outro se defende
- Continua até um morrer
- Exiba status após cada turno

**Saída esperada:**
```
=== BATALHA INICIADA ===
Pikachu [HP: 35/35] vs Charmander [HP: 39/39]

--- TURNO 1 ---
Pikachu atacou Charmander com 50 de dano!
Charmander [HP: -11/39] 

PIKACHU VENCEU!
```

---

## 17. Referências Bibliográficas

### Livros Recomendados

1. **Clean Code** - Robert Martin
   - Nomes significativos
   - Funções pequenas
   - Tratamento de erros

2. **Effective Java** - Joshua Bloch
   - Patterns e boas práticas
   - Quando usar static
   - Encapsulamento

3. **Head First Design Patterns** - Freeman & Freeman
   - Padrões de design
   - Abstração e interfaces
   - Polimorfismo prático

4. **The Pragmatic Programmer** - Hunt & Thomas
   - Princípios gerais
   - Código mantível

### Documentação Oficial

- [Java Tutorials - Oracle](https://docs.oracle.com/javase/tutorial/)
- [Effective Java - Third Edition](https://www.oracle.com/java/)

---

## Checklist de Aprendizado

- [ ] Entendo a diferença entre classe e objeto
- [ ] Consigo criar construtores com sobrecarga
- [ ] Sei usar getters e setters com validação
- [ ] Implementei herança com `extends`
- [ ] Usei polimorfismo com List<>
- [ ] Criei classes abstratas e interfaces
- [ ] Apliquei static para utilitários
- [ ] Sobrescrevi `toString()` apropriadamente
- [ ] Respeitei SRP (Single Responsibility)
- [ ] Testei exceções customizadas

---

