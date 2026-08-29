# Construtores, This, Sobrecarga e Encapsulamento em Java

Aprofundamento em conceitos fundamentais de POO com analogias Pokémon.

---

## 1. Construtores

### O que é um Construtor?

Método especial que **executa automaticamente** quando você cria um novo objeto com `new`. Responsável por **inicializar o estado** do objeto.

**Analogia Pokémon:**
- Quando você captura um Pokémon, ele não nasce vazio
- Tem um nome, tipo, HP definidos no momento da captura
- O construtor é esse processo de inicialização

### Problema: Objetos Vazios

```java
// ? Sem construtor customizado
public class Pokemon {
    public String nome;
    public String tipo;
    public int hp;
}

Pokemon pikachu = new Pokemon();  // Objeto vazio
System.out.println(pikachu.nome);  // null
System.out.println(pikachu.tipo);  // null
System.out.println(pikachu.hp);    // 0

// Problema: faz sentido um Pokémon sem nome?
// É inconsistente ter Pokémon vazio em memória
```

### Solução: Construtor Customizado

```java
// ? Com construtor
public class Pokemon {
    public String nome;
    public String tipo;
    public int hp;
    
    // Construtor: força inicializar nome, tipo e hp
    public Pokemon(String nome, String tipo, int hp) {
        this.nome = nome;
        this.tipo = tipo;
        this.hp = hp;
    }
}

Pokemon pikachu = new Pokemon("Pikachu", "Elétrico", 35);
// Agora o objeto SEMPRE nasce consistente
```

### Construtor Padrão (Default)

Se você não cria um construtor customizado, Java fornece um vazio automaticamente:

```java
public class Pokemon {
    public String nome;
}

// Java fornece implicitamente:
// public Pokemon() { }

Pokemon p = new Pokemon();  // Funciona
```

**Mas:** Se você criar um construtor customizado, o padrão desaparece:

```java
public class Pokemon {
    public String nome;
    
    public Pokemon(String nome) {
        this.nome = nome;
    }
    
    // O construtor padrão NÃO existe mais!
}

Pokemon p1 = new Pokemon("Pikachu");  // ? Funciona
Pokemon p2 = new Pokemon();           // ? ERRO!
```

**Solução:** Forneça o construtor padrão explicitamente:

```java
public class Pokemon {
    public String nome;
    
    // Construtor padrão
    public Pokemon() {
    }
    
    // Construtor customizado
    public Pokemon(String nome) {
        this.nome = nome;
    }
}

Pokemon p1 = new Pokemon();
Pokemon p2 = new Pokemon("Pikachu");
```

### Fluxo de Construção

```
Pokemon pikachu = new Pokemon("Pikachu", "Elétrico", 35);

1. new Pokemon(...)  ? Aloca memória
2. Executa construtor ? Inicializa atributos
3. Retorna referência ? pikachu aponta para o objeto
```

---

## 2. A Palavra `this`

### O que é?

Referência ao **próprio objeto**. Explícita e diferencia atributos de variáveis locais.

**Analogia Pokémon:**
- `this` = o próprio Pokémon falando sobre si mesmo
- "Meu nome" = `this.nome`
- "Meu HP" = `this.hp`

### Uso 1: Diferenciar Atributos de Parâmetros

```java
public class Pokemon {
    public String nome;
    public int hp;
    
    // ? Confuso: qual é qual?
    public void inicializar1(String nome, int hp) {
        nome = nome;  // Qual nome? O parâmetro ou o atributo?
        hp = hp;      // Qual hp?
    }
    
    // ? Claro: this diferencia
    public void inicializar2(String nome, int hp) {
        this.nome = nome;  // this.nome = parâmetro nome
        this.hp = hp;      // this.hp = parâmetro hp
    }
}
```

### Uso 2: Em Construtores

```java
public class Pokemon {
    private String nome;
    private String tipo;
    private int hp;
    
    public Pokemon(String nome, String tipo, int hp) {
        this.nome = nome;  // Atributo recebe parâmetro
        this.tipo = tipo;
        this.hp = hp;
    }
}

// Stack                 Heap
// nome ("Pikachu") ---> this.nome = "Pikachu"
// tipo ("Elétrico")---> this.tipo = "Elétrico"
// hp (35)         ---> this.hp = 35
```

### Uso 3: Chamar Outro Construtor (Constructor Chaining)

```java
public class Pokemon {
    private String nome;
    private String tipo;
    private int hp;
    private int nivel;
    
    // Construtor completo
    public Pokemon(String nome, String tipo, int hp, int nivel) {
        this.nome = nome;
        this.tipo = tipo;
        this.hp = hp;
        this.nivel = nivel;
    }
    
    // Construtor simplificado chama o completo com this()
    public Pokemon(String nome, String tipo) {
        this(nome, tipo, 20, 1);  // Chama o outro construtor
    }
    
    // Construtor padrão
    public Pokemon() {
        this("Pokémon", "Normal");  // Chama o construtor simplificado
    }
}

Pokemon p1 = new Pokemon();                            // padrão
Pokemon p2 = new Pokemon("Pikachu", "Elétrico");      // simplificado
Pokemon p3 = new Pokemon("Pikachu", "Elétrico", 35, 5); // completo
```

**Vantagem:** Evita repetição de código

```java
// ? Repetição (errado)
public Pokemon(String nome, String tipo) {
    this.nome = nome;
    this.tipo = tipo;
    this.hp = 20;
    this.nivel = 1;
}

public Pokemon() {
    this.nome = "Pokémon";
    this.tipo = "Normal";
    this.hp = 20;
    this.nivel = 1;
}

// ? Sem repetição (certo)
public Pokemon(String nome, String tipo, int hp, int nivel) {
    this.nome = nome;
    this.tipo = tipo;
    this.hp = hp;
    this.nivel = nivel;
}

public Pokemon(String nome, String tipo) {
    this(nome, tipo, 20, 1);
}

public Pokemon() {
    this("Pokémon", "Normal");
}
```

### Uso 4: Passar o Objeto como Argumento

```java
public class Batalha {
    private Pokemon pokemon1;
    private Pokemon pokemon2;
    
    // A classe Pokémon passa uma referência a si mesma
    public Batalha(Pokemon pokemon1, Pokemon pokemon2) {
        this.pokemon1 = pokemon1;  // Guarda a referência
        this.pokemon2 = pokemon2;
    }
}

Pokemon pikachu = new Pokemon("Pikachu", "Elétrico", 35, 5);
Pokemon charmander = new Pokemon("Charmander", "Fogo", 39, 5);
Batalha batalha = new Batalha(pikachu, charmander);
// Batalha tem acesso aos Pokémon via referência
```

### Regra: Quando usar `this`?

**Use `this` quando:**
- Há ambiguidade entre atributo e parâmetro (OBRIGATÓRIO em construtores)
- Quer passar o objeto para outro método
- Quer chamar outro construtor da classe

```java
public class Pokemon {
    private int hp;
    private int maxHp;
    
    public Pokemon(int hp) {
        this.hp = hp;      // NECESSÁRIO (sem this, ambíguo)
        this.maxHp = hp;   // BOM (claro que é atributo)
    }
    
    public void receber_dano(int dano) {
        this.hp -= dano;   // BOM (claro)
    }
    
    public void apresentar() {
        System.out.println(this.hp);  // BOM (deixa claro)
    }
}
```

---

## 3. Sobrecarga (Overloading)

### O que é?

Múltiplos métodos/construtores com o **mesmo nome**, mas **listas de parâmetros diferentes**.

**Analogia Pokémon:**
- Pokémon pode atacar de formas diferentes
- `atacar()` - sem parâmetro, ataque padrão
- `atacar(tipo)` - especifica o tipo de ataque
- `atacar(adversario, tipo)` - especifica adversário e tipo

### Sobrecarga de Construtores

```java
public class Pokemon {
    private String nome;
    private String tipo;
    private int hp;
    private int nivel;
    
    // Construtor 1: sem parâmetros
    public Pokemon() {
        this.nome = "Pokémon";
        this.tipo = "Normal";
        this.hp = 20;
        this.nivel = 1;
    }
    
    // Construtor 2: só nome
    public Pokemon(String nome) {
        this.nome = nome;
        this.tipo = "Normal";
        this.hp = 20;
        this.nivel = 1;
    }
    
    // Construtor 3: nome e tipo
    public Pokemon(String nome, String tipo) {
        this.nome = nome;
        this.tipo = tipo;
        this.hp = 20;
        this.nivel = 1;
    }
    
    // Construtor 4: completo
    public Pokemon(String nome, String tipo, int hp, int nivel) {
        this.nome = nome;
        this.tipo = tipo;
        this.hp = hp;
        this.nivel = nivel;
    }
}

Pokemon p1 = new Pokemon();
Pokemon p2 = new Pokemon("Pikachu");
Pokemon p3 = new Pokemon("Pikachu", "Elétrico");
Pokemon p4 = new Pokemon("Pikachu", "Elétrico", 35, 5);
```

**Problema:** Repetição de código

```java
// ? Ruim: código repetido
public Pokemon() {
    this.nome = "Pokémon";
    this.tipo = "Normal";
    this.hp = 20;
    this.nivel = 1;
}

public Pokemon(String nome) {
    this.nome = nome;
    this.tipo = "Normal";
    this.hp = 20;
    this.nivel = 1;
}

public Pokemon(String nome, String tipo) {
    this.nome = nome;
    this.tipo = tipo;
    this.hp = 20;
    this.nivel = 1;
}
```

**Solução:** Constructor Chaining com `this()`

```java
// ? Bom: chamadas encadeadas
public Pokemon() {
    this("Pokémon", "Normal", 20, 1);
}

public Pokemon(String nome) {
    this(nome, "Normal", 20, 1);
}

public Pokemon(String nome, String tipo) {
    this(nome, tipo, 20, 1);
}

public Pokemon(String nome, String tipo, int hp, int nivel) {
    this.nome = nome;
    this.tipo = tipo;
    this.hp = hp;
    this.nivel = nivel;
}
```

### Sobrecarga de Métodos

```java
public class Pokemon {
    private int ataque;
    
    // Método 1: ataque padrão
    public void atacar() {
        System.out.println("Ataque padrão de " + this.ataque);
    }
    
    // Método 2: ataque com multiplicador
    public void atacar(double multiplicador) {
        int dano = (int)(this.ataque * multiplicador);
        System.out.println("Ataque com " + dano + " de dano");
    }
    
    // Método 3: ataque contra adversário
    public void atacar(Pokemon adversario) {
        adversario.hp -= this.ataque;
        System.out.println("Atacou e causou dano!");
    }
    
    // Método 4: ataque customizado
    public void atacar(Pokemon adversario, double multiplicador) {
        int dano = (int)(this.ataque * multiplicador);
        adversario.hp -= dano;
        System.out.println("Ataque especial com " + dano + " de dano!");
    }
}

Pokemon pikachu = new Pokemon();
pikachu.ataque = 55;
Pokemon charmander = new Pokemon();

pikachu.atacar();                    // Tipo 1
pikachu.atacar(1.5);                 // Tipo 2
pikachu.atacar(charmander);          // Tipo 3
pikachu.atacar(charmander, 2.0);     // Tipo 4
```

### Regras da Sobrecarga

**Deve variar:**
- ? Número de parâmetros
- ? Tipo de parâmetros
- ? Ordem de parâmetros

**Não basta:**
- ? Tipo de retorno (não diferencia)

```java
// ? ERRO: mesmo número e tipo de parâmetros
public void transferir(double valor) { }
public double transferir(double valor) { }  // Ambíguo!

// ? OK: número diferente
public void transferir(double valor) { }
public void transferir(double valor, String descricao) { }

// ? OK: tipo diferente
public void transferir(double valor) { }
public void transferir(int valor) { }

// ? OK: ordem diferente
public void comparar(String nome, int idade) { }
public void comparar(int idade, String nome) { }
```

---

## 4. Encapsulamento

### O Princípio

**Esconder detalhes de implementação**, expondo apenas operações seguras que mantêm o objeto **consistente**.

**Analogia Pokémon:**
- Um Pokémon é um encapsulamento
- Você não pode mudar diretamente o DNA dele
- Você só pode fazer operações seguras: atacar, defender, ganhar experiência
- O Pokémon internamente controla sua consistência

### O Problema do Acesso Público

```java
// ? Atributos públicos (INSEGURO)
public class ContaBancaria {
    public int numero;
    public String titular;
    public double saldo;
}

ContaBancaria conta = new ContaBancaria();
conta.numero = 1234;
conta.titular = "João";
conta.saldo = 1000;

// Qualquer um pode fazer isso:
conta.saldo = -9999;  // ? INCONSISTENTE!
conta.numero = 9999;  // ? Mudar número da conta?!

// Objeto em estado inválido
```

### A Solução: Getters e Setters

```java
// ? Atributos privados com controle
public class ContaBancaria {
    private int numero;      // private = só acessa dentro da classe
    private String titular;
    private double saldo;
    
    // Getter: ler com segurança
    public int getNumero() {
        return this.numero;
    }
    
    // Getter: ler com segurança
    public String getTitular() {
        return this.titular;
    }
    
    // Getter: ler com segurança
    public double getSaldo() {
        return this.saldo;
    }
    
    // Setter: escrever com validação
    public void setTitular(String titular) {
        if (titular != null && !titular.isEmpty()) {
            this.titular = titular;
        }
    }
    
    // Setter: número não pode mudar após criação
    public void setNumero(int numero) {
        if (this.numero == 0) {  // Só permite mudar se ainda é zero
            this.numero = numero;
        }
    }
    
    // Operações seguras para modificar saldo
    public void depositar(double valor) {
        if (valor > 0) {
            this.saldo += valor;
        }
    }
    
    public void sacar(double valor) {
        if (valor > 0 && valor <= this.saldo) {
            this.saldo -= valor;
            this.saldo -= 5.0;  // Taxa de saque
        }
    }
}

ContaBancaria conta = new ContaBancaria();
conta.setNumero(1234);
conta.setTitular("João");
conta.depositar(1000);

// Agora é seguro
System.out.println(conta.getSaldo());  // 1000

conta.setNumero(9999);  // Não faz nada (número já foi definido)
conta.depositar(500);   // OK: 1500
conta.sacar(100);       // OK: 1395 (1500 - 100 - 5 de taxa)
```

### Padrão JavaBeans

Convenção de nomenclatura em Java:

```
Atributo          Getter         Setter
-----------       -----------    -----------
nome              getNome()      setNome()
idade             getIdade()     setIdade()
ativo             isAtivo()      setAtivo()
```

```java
public class Pokemon {
    private String nome;
    private int hp;
    private boolean capturado;
    
    public String getNome() {
        return this.nome;
    }
    
    public void setNome(String nome) {
        this.nome = nome;
    }
    
    public int getHp() {
        return this.hp;
    }
    
    public void setHp(int hp) {
        if (hp > 0) {
            this.hp = hp;
        }
    }
    
    public boolean isCapturado() {
        return this.capturado;
    }
    
    public void setCapturado(boolean capturado) {
        this.capturado = capturado;
    }
}
```

### Quando Não Permitir Setter

```java
public class Pokemon {
    private String nome;
    private String tipo;
    private int hpMaximo;  // Não deve mudar após criação
    
    public Pokemon(String nome, String tipo, int hpMaximo) {
        this.nome = nome;
        this.tipo = tipo;
        this.hpMaximo = hpMaximo;
    }
    
    public String getNome() {
        return this.nome;
    }
    
    public void setNome(String nome) {
        this.nome = nome;  // OK: pode mudar nome
    }
    
    public String getTipo() {
        return this.tipo;
    }
    
    // ? Tipo não tem setter (imutável)
    
    public int getHpMaximo() {
        return this.hpMaximo;
    }
    
    // ? HpMaximo não tem setter (imutável)
}

Pokemon p = new Pokemon("Pikachu", "Elétrico", 35);
p.setNome("Pikachu Gigante");  // OK
// p.setTipo("Fogo");          // Método não existe!
```

---

## 5. Modificadores de Acesso

### Hierarquia de Visibilidade

```
private  < (padrão) < protected < public
Mais restritivo ..................... Menos restritivo
```

### Tabela Completa

| Modificador | Mesma Classe | Mesmo Pacote | Subclasse (outro pacote) | Outro Pacote |
|-----------|--------------|-------------|------------------------|------------|
| `private` | ? | ? | ? | ? |
| (padrão) | ? | ? | ? | ? |
| `protected` | ? | ? | ? | ? |
| `public` | ? | ? | ? | ? |

### Private - Máxima Proteção

```java
// entities/Pokemon.java
package entities;

public class Pokemon {
    private String nome;        // Só Pokemon acessa
    private int hp;
    
    public Pokemon(String nome, int hp) {
        this.nome = nome;
        this.hp = hp;
    }
    
    public String getNome() {
        return this.nome;
    }
}

// application/Program.java
package application;
import entities.Pokemon;

public class Program {
    public static void main(String[] args) {
        Pokemon p = new Pokemon("Pikachu", 35);
        System.out.println(p.getNome());  // ? OK (getter público)
        System.out.println(p.nome);       // ? ERRO (private)
        p.hp = 50;                        // ? ERRO (private)
    }
}
```

### Protected - Herança

```java
// entities/Criatura.java
package entities;

public class Criatura {
    protected String nome;  // Subclasses acessam
    
    public Criatura(String nome) {
        this.nome = nome;
    }
}

// entities/Pokemon.java
package entities;

public class Pokemon extends Criatura {
    public void apresentar() {
        System.out.println(this.nome);  // ? Acessa protected da superclasse
    }
}
```

### Public - Acesso Irrestrito

```java
public class Pokemon {
    public String getNome() {  // Qualquer um acessa
        return this.nome;
    }
    
    public void atacar() {     // Qualquer um chama
        System.out.println("Atacou!");
    }
}
```

---

## 6. Exemplo Prático Completo: Conta Bancária

```java
package entities;

public class ContaBancaria {
    private int numero;
    private String titular;
    private double saldo;
    
    private static final double TAXA_SAQUE = 5.0;
    
    // Construtor: sem depósito inicial
    public ContaBancaria() {
    }
    
    // Construtor: com número e titular
    public ContaBancaria(int numero, String titular) {
        this.numero = numero;
        this.titular = titular;
    }
    
    // Construtor: com número, titular e depósito
    public ContaBancaria(int numero, String titular, double depositoInicial) {
        this(numero, titular);
        if (depositoInicial > 0) {
            this.saldo = depositoInicial;
        }
    }
    
    // Getters
    public int getNumero() {
        return this.numero;
    }
    
    public String getTitular() {
        return this.titular;
    }
    
    public double getSaldo() {
        return this.saldo;
    }
    
    // Setters com validação
    public void setTitular(String titular) {
        if (titular != null && !titular.isEmpty()) {
            this.titular = titular;
        }
    }
    
    // Operações seguras
    public void depositar(double valor) {
        if (valor > 0) {
            this.saldo += valor;
        }
    }
    
    public void sacar(double valor) {
        if (valor > 0) {
            double total = valor + TAXA_SAQUE;
            if (total <= this.saldo) {
                this.saldo -= total;
            }
        }
    }
    
    @Override
    public String toString() {
        return String.format(
            "Account %d, Holder: %s, Balance: $ %.2f",
            this.numero, this.titular, this.saldo
        );
    }
}

// ===== USO =====

package application;
import java.util.Locale;
import java.util.Scanner;
import entities.ContaBancaria;

public class Program {
    public static void main(String[] args) {
        Locale.setDefault(Locale.US);
        Scanner sc = new Scanner(System.in);
        
        System.out.print("Enter account number: ");
        int numero = sc.nextInt();
        
        System.out.print("Enter account holder: ");
        sc.nextLine();  // Limpa buffer
        String titular = sc.nextLine();
        
        System.out.print("Is there an initial deposit (y/n)? ");
        char resposta = sc.next().charAt(0);
        
        ContaBancaria conta;
        
        if (resposta == 'y') {
            System.out.print("Enter initial deposit value: ");
            double deposito = sc.nextDouble();
            conta = new ContaBancaria(numero, titular, deposito);
        } else {
            conta = new ContaBancaria(numero, titular);
        }
        
        System.out.println();
        System.out.println("Account data:");
        System.out.println(conta);
        
        System.out.println();
        System.out.print("Enter a deposit value: ");
        double deposito = sc.nextDouble();
        conta.depositar(deposito);
        System.out.println("Updated account data:");
        System.out.println(conta);
        
        System.out.println();
        System.out.print("Enter a withdraw value: ");
        double saque = sc.nextDouble();
        conta.sacar(saque);
        System.out.println("Updated account data:");
        System.out.println(conta);
        
        sc.close();
    }
}

// ===== SAÍDA =====
/*
Enter account number: 8532
Enter account holder: Alex Green
Is there an initial deposit (y/n)? y
Enter initial deposit value: 500.00

Account data:
Account 8532, Holder: Alex Green, Balance: $ 500.00

Enter a deposit value: 200.00
Updated account data:
Account 8532, Holder: Alex Green, Balance: $ 700.00

Enter a withdraw value: 300.00
Updated account data:
Account 8532, Holder: Alex Green, Balance: $ 395.00
*/
```

---

## 7. Exemplo com Pokémon: Pokedex

```java
package entities;

public class Pokemon {
    private int id;           // Imutável: não tem setter
    private String nome;
    private String tipo;
    private int hpMaximo;
    private int hp;
    private int ataque;
    private int defesa;
    private int nivel;
    
    // Construtor
    public Pokemon(int id, String nome, String tipo, int hpMaximo, 
                   int ataque, int defesa) {
        this.id = id;
        this.nome = nome;
        this.tipo = tipo;
        this.hpMaximo = hpMaximo;
        this.hp = hpMaximo;
        this.ataque = ataque;
        this.defesa = defesa;
        this.nivel = 1;
    }
    
    // Getters
    public int getId() {
        return this.id;
    }
    
    public String getNome() {
        return this.nome;
    }
    
    public String getTipo() {
        return this.tipo;
    }
    
    public int getHpMaximo() {
        return this.hpMaximo;
    }
    
    public int getHp() {
        return this.hp;
    }
    
    public int getAtaque() {
        return this.ataque;
    }
    
    public int getDefesa() {
        return this.defesa;
    }
    
    public int getNivel() {
        return this.nivel;
    }
    
    // Setters com validação
    public void setNome(String nome) {
        if (nome != null && !nome.isEmpty()) {
            this.nome = nome;
        }
    }
    
    // Operações seguras
    public void ganharExperiencia(int exp) {
        if (exp > 0) {
            this.nivel++;
            this.ataque += 2;
            this.defesa += 1;
        }
    }
    
    public void recuperarHP() {
        this.hp = this.hpMaximo;
    }
    
    public int calcularDano() {
        return this.ataque + (int)(Math.random() * 10);
    }
    
    public void receberDano(int dano) {
        int danoReduzido = Math.max(1, dano - this.defesa / 2);
        this.hp = Math.max(0, this.hp - danoReduzido);
    }
    
    public boolean estaVivo() {
        return this.hp > 0;
    }
    
    @Override
    public String toString() {
        return String.format(
            "#%d %s (%s) - Nível %d - HP: %d/%d - ATK: %d - DEF: %d",
            this.id, this.nome, this.tipo, this.nivel,
            this.hp, this.hpMaximo, this.ataque, this.defesa
        );
    }
}

// ===== USO =====

package application;
import entities.Pokemon;

public class Pokedex {
    public static void main(String[] args) {
        Pokemon pikachu = new Pokemon(25, "Pikachu", "Elétrico", 35, 55, 40);
        Pokemon charmander = new Pokemon(4, "Charmander", "Fogo", 39, 52, 43);
        
        System.out.println("=== POKEDEX ===");
        System.out.println(pikachu);
        System.out.println(charmander);
        
        System.out.println("\n=== GANHAR EXPERIÊNCIA ===");
        pikachu.ganharExperiencia(100);
        System.out.println(pikachu);
        
        System.out.println("\n=== BATALHA ===");
        while (pikachu.estaVivo() && charmander.estaVivo()) {
            int dano1 = pikachu.calcularDano();
            charmander.receberDano(dano1);
            System.out.printf("%s atacou %s! Dano: %d%n",
                pikachu.getNome(), charmander.getNome(), dano1);
            
            if (!charmander.estaVivo()) break;
            
            int dano2 = charmander.calcularDano();
            pikachu.receberDano(dano2);
            System.out.printf("%s atacou %s! Dano: %d%n",
                charmander.getNome(), pikachu.getNome(), dano2);
        }
        
        String vencedor = pikachu.estaVivo() ? "Pikachu" : "Charmander";
        System.out.println("\n" + vencedor + " venceu!");
    }
}
```

---

## 8. Checklist de Boas Práticas

### Construtores
- [ ] Sempre inicialize atributos no construtor
- [ ] Use `this()` para evitar repetição entre construtores
- [ ] Forneça múltiplas versões (padrão, simples, completa)
- [ ] Valide parâmetros no construtor se necessário

### Palavra `this`
- [ ] Use `this` quando há ambiguidade (especialmente em construtores)
- [ ] Use `this()` para chamar outro construtor
- [ ] Deixe claro que é um atributo, não uma variável local

### Sobrecarga
- [ ] Nomes iguais devem ter significado semelhante
- [ ] Varie número, tipo ou ordem de parâmetros
- [ ] Use constructor chaining para evitar repetição
- [ ] Mantenha interface consistente

### Encapsulamento
- [ ] Todos os atributos devem ser `private`
- [ ] Forneça getters/setters públicos quando necessário
- [ ] Valide dados no setter
- [ ] Exponha apenas operações seguras
- [ ] Atributos imutáveis não têm setter

### Modificadores de Acesso
- [ ] Use `private` por padrão
- [ ] Use `protected` apenas para herança
- [ ] Use `public` para interface da classe
- [ ] Documente a razão de cada modificador

---

## 9. Exercícios Práticos

### Exercício 1: Classe LibroBank (Conta Bancária Simples)

Implemente uma classe `ContaBancaria` com:
- Atributos: numero (imutável), titular, saldo
- Construtores: padrão, com número+titular, com número+titular+deposito
- Getters para todos
- Setter só para titular
- Métodos: `depositar(valor)`, `sacar(valor)` com taxa de 5
- toString() formatado

---

### Exercício 2: Classe Veiculo

Implemente uma classe `Veiculo` com:
- Atributos: marca, modelo, ano, velocidadeMaxima, velocidadeAtual
- Construtor completo com this()
- Getters para todos
- Setter: só para velocidadeAtual (não pode exceder velocidadeMaxima)
- Métodos: `acelerar(velocidade)`, `desacelerar()`, `parar()`
- toString() mostrand o estado

---

### Exercício 3: Classe Livro

Implemente uma classe `Livro` com:
- Atributos: isbn (imutável), titulo, autor, anoPublicacao, totalPaginas, paginasLidas
- Construtores: padrão, completo
- Getters para todos
- Setters: titulo, autor, paginasLidas (com validação)
- Métodos: `lerPaginas(n)`, `progresso()` retorna percentual
- toString()

---

### Exercício 4: Sistema de Funcionário

Implemente `Funcionario` com:
- Atributos: matricula (imutável), nome, departamento, salarioBruto, imposto
- Construtores: padrão, completo
- Método: `calcularSalarioLiquido()` = bruto - imposto
- Método: `aumentarSalario(porcentagem)` - afeta apenas bruto
- Getter/setter com validação
- toString() mostrando nome e líquido

---

### Exercício 5: Classe PokemonCard (Card Game)

Implemente `CartaPokemon` com:
- Atributos: numero (imutável), nome, tipo, raridade, condicao (nova/boa/desgastada)
- Múltiplos construtores (sem parâmetros, nome+tipo, completo)
- Getters para todos
- Setters: nome, condicao (com validação de raridade)
- Método: `obterValorMercado()` - varia com raridade e condição
- toString()

---

### Exercício 6: Classe Pessoa

Implemente `Pessoa` com:
- Atributos: cpf (imutável), nome, dataNascimento, email
- Construtores: padrão, completo
- Getters para todos
- Setters: nome, email (validar formato)
- Método: `calcularIdade()` baseado em data de nascimento
- toString()

---

### Exercício 7: Classe Departamento

Implemente `Departamento` com:
- Atributos: codigo (imutável), nome, descricao, orcamento
- Construtores
- Getters/setters com validação
- Método: `reduzirOrcamento(valor)` - não deixa negativo
- Método: `aumentarOrcamento(valor)`
- toString()

---

### Exercício 8: Classe Time de Esportes

Implemente `Time` com:
- Atributos: codigo (imutável), nome, cidade, totalJogadores
- Construtores
- Getters/setters
- Método: `adicionarJogador()` com limite
- Método: `removerJogador()`
- Método: `obterPorcentagemCadeiras()` = jogadores/limite * 100
- toString()

---

### Exercício 9: Classe Caixa (Caixa Eletrônico)

Implemente `CaixaEletronico` com:
- Atributos: codigo (imutável), banco, saldoDisponivel, limite
- Construtores
- Método: `sacar(valor)` - verifica saldo + limite
- Método: `depositar(valor)`
- Método: `solicitarEmprestimo(valor)` - se tiver espaço no limite
- toString()

---

### Exercício 10: Batalha Pokémon Completa

Implemente `Batalha` com:
- Classe `Pokemon` com encapsulamento completo
- Classe `Treinador` que tem um `Pokemon`
- Classe `Batalha` que recebe dois `Treinador` no construtor
- Simule turnos de batalha
- Exiba status após cada turno
- Declare vencedor quando um Pokémon cair

---

## 10. Referências Rápidas

### Constructor Chaining

```java
public class Exemplo {
    private String a;
    private String b;
    private String c;
    
    public Exemplo() {
        this("default", "values");
    }
    
    public Exemplo(String a, String b) {
        this(a, b, "c");
    }
    
    public Exemplo(String a, String b, String c) {
        this.a = a;
        this.b = b;
        this.c = c;
    }
}
```

### Padrão JavaBeans

```
Atributo private ? getter público getNome() ? setter público setNome()
Booleano private ? getter público isAtivo() ? setter público setAtivo()
```

### Validação em Setter

```java
public void setValor(int valor) {
    if (valor > 0 && valor <= MAX_VALOR) {
        this.valor = valor;
    }
}
```

### Método toString()

```java
@Override
public String toString() {
    return String.format(
        "Classe{attr1=%s, attr2=%d}",
        this.attr1, this.attr2
    );
}
```

---

