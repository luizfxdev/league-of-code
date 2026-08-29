# Encapsulamento em Java

## O que é Encapsulamento?

Encapsulamento é o primeiro pilar da Programação Orientada a Objetos. É a prática de esconder os detalhes internos de um objeto e expor apenas o necessário através de uma interface controlada. Pense em um Pokémon: você não vê como o sistema interno dele processa um ataque ? você apenas chama o método `atacar()` e ele executa.

Sem encapsulamento, qualquer classe poderia mexer diretamente nos atributos internos de um Pokémon (reduzir HP fraudulentamente, mudar tipo, etc.). Com encapsulamento, você controla como esses dados são modificados.

---

## Por que Encapsulamento é Crítico

**Controle de estado**: Garante que um objeto nunca entre em estado inválido.  
**Manutenibilidade**: Mudanças internas não quebram código externo.  
**Reusabilidade**: Interfaces bem definidas facilitam reutilização.  
**Segurança lógica**: Previne operações ilógicas (ex: Pokémon com HP negativo).

### Problema sem Encapsulamento

```java
public class Pokemon {
    public int hp;
    public int ataque;
    public String tipo;
}

// Código externo pode fazer qualquer coisa:
Pokemon pikachu = new Pokemon();
pikachu.hp = -9999;  // ? Inválido, mas o Java não impede
pikachu.ataque = Integer.MAX_VALUE;  // ? Quebra lógica
```

Resultado: sistema frágil, difícil de debugar, impossível manter.

---

## Modificadores de Acesso

Java oferece quatro níveis de encapsulamento:

| Modificador | Mesma Classe | Mesmo Package | Subclass | Mundo | Uso |
|---|---|---|---|---|---|
| `private` | ? | ? | ? | ? | Dados sensíveis, métodos internos |
| `package-private` (sem modificador) | ? | ? | ? | ? | Colaboradores no mesmo pacote |
| `protected` | ? | ? | ? | ? | Dados/métodos para subclasses |
| `public` | ? | ? | ? | ? | Interface oficial da classe |

### Exemplo: Pokémon com Níveis de Acesso

```java
public class Pokemon {
    // Atributo privado ? só a classe acessa
    private int hp;
    private int ataque;
    private int defesa;
    
    // Visível apenas dentro do pacote
    int experienciaInterna;
    
    // Protegido ? subclasses (ex: PokemonLendario) acessam
    protected String habilidadeEspecial;
    
    // Público ? parte da API oficial
    public String nome;
    public String tipo;
}
```

---

## Getters e Setters: A Barreira de Controle

Getters e setters são os guardiões do encapsulamento. Permitem acesso leitura/escrita com **validação**.

### Getters (Leitura Controlada)

```java
public class Pokemon {
    private int hp;
    private int hpMax;
    
    // Getter simples
    public int getHp() {
        return this.hp;
    }
    
    // Getter com lógica
    public double getHpPercentual() {
        return (double) this.hp / this.hpMax * 100;
    }
    
    // Getter que mascara interno
    public String getStatusVida() {
        if (this.hp > hpMax * 0.75) return "Saudável";
        if (this.hp > hpMax * 0.5) return "Levemente ferido";
        if (this.hp > 0) return "Crítico";
        return "Desmaiado";
    }
}
```

**Por que não expor `hp` diretamente?**
- Se você trocar a implementação interna (ex: armazenar como percentual), código externo quebra.
- Com getter, você controla o que é retornado.

### Setters (Escrita Validada)

```java
public class Pokemon {
    private int hp;
    private int hpMax;
    private int nivel;
    
    // Setter com validação
    public void setHp(int novoHp) {
        if (novoHp < 0) {
            this.hp = 0;  // Garante que HP nunca fica negativo
        } else if (novoHp > this.hpMax) {
            this.hp = this.hpMax;  // Não ultrapassa máximo
        } else {
            this.hp = novoHp;
        }
    }
    
    // Setter que aceita lógica de negócio
    public void setNivel(int novoNivel) {
        if (novoNivel < 1 || novoNivel > 100) {
            throw new IllegalArgumentException("Nível deve estar entre 1 e 100");
        }
        this.nivel = novoNivel;
        recalcularStats();  // Atualiza atributos quando nível muda
    }
    
    // Setter "somente leitura" ? não oferece setter
    public String getNome() {
        return nome;  // Sem setName() ? nome não pode mudar
    }
}
```

---

## Encapsulamento de Comportamento (Métodos Privados)

Nem todo método deve ser público. Métodos internos (auxiliares) ficam `private`.

### Exemplo: Ataque Pokémon

```java
public class Pokemon {
    private int ataque;
    private int velocidade;
    
    // Método público ? API oficial
    public int executarAtaque() {
        int dano = calcularDano();  // Chama método privado
        return dano;
    }
    
    // Método privado ? detalhes internos
    private int calcularDano() {
        int dano = ataque * 2;
        if (temVantagemTipo()) {
            dano = (int) (dano * 1.5);
        }
        return dano + random(1, 10);  // Variação aleatória
    }
    
    // Método privado ? lógica interna
    private boolean temVantagemTipo() {
        // Lógica complexa de tipo do Pokémon
        return false;
    }
}
```

**Vantagem:** Se você precisar mudar como `calcularDano()` funciona, não quebra código que chama `executarAtaque()`.

---

## Imutabilidade: Encapsulamento ao Máximo

Objetos imutáveis não podem ser modificados após criação. Encapsulamento extremo.

### Pokémon Imutável

```java
public final class PokemonStats {  // final = não pode ser herdado
    private final int hp;
    private final int ataque;
    private final int defesa;
    private final String tipo;
    
    // Construtor obrigatório
    public PokemonStats(int hp, int ataque, int defesa, String tipo) {
        this.hp = hp;
        this.ataque = ataque;
        this.defesa = defesa;
        this.tipo = tipo;
    }
    
    // Apenas getters ? SEM SETTERS
    public int getHp() { return hp; }
    public int getAtaque() { return ataque; }
    public int getDefesa() { return defesa; }
    public String getTipo() { return tipo; }
    
    // Retorna novo objeto em vez de modificar
    public PokemonStats comBonus(int bonusAtaque) {
        return new PokemonStats(
            this.hp,
            this.ataque + bonusAtaque,
            this.defesa,
            this.tipo
        );
    }
}
```

**Vantagens:**
- Thread-safe (sem sincronização necessária)
- Seguro usar como chave de HashMap
- Predictável ? estado nunca muda

**Trade-off:** Menos flexível, mais memória (cria novo objeto a cada mudança).

---

## Encapsulamento de Coleções

Coleções expostas diretamente são um furo de segurança.

### ? Errado

```java
public class Pokedex {
    public List<Pokemon> pokemons = new ArrayList<>();
}

Pokedex pokedex = new Pokedex();
pokedex.pokemons.clear();  // ? Qualquer um limpa toda a Pokédex!
```

### ? Correto (Opção 1: Defensiva)

```java
public class Pokedex {
    private List<Pokemon> pokemons = new ArrayList<>();
    
    // Retorna cópia ? não expõe original
    public List<Pokemon> getPokemons() {
        return new ArrayList<>(this.pokemons);
    }
    
    // Métodos controlados
    public void adicionarPokemon(Pokemon pokemon) {
        if (pokemon != null && !pokemons.contains(pokemon)) {
            pokemons.add(pokemon);
        }
    }
}
```

### ? Correto (Opção 2: Imutável)

```java
public class Pokedex {
    private List<Pokemon> pokemons = new ArrayList<>();
    
    // Retorna lista imutável ? não pode modificar
    public List<Pokemon> getPokemons() {
        return Collections.unmodifiableList(this.pokemons);
    }
}

Pokedex pokedex = new Pokedex();
pokedex.getPokemons().clear();  // ? Lança UnsupportedOperationException
```

---

## Padrão Builder com Encapsulamento

Para objetos complexos com muitos atributos, Builder garante estado válido.

```java
public class Pokemon {
    private final String nome;
    private final String tipo;
    private final int hp;
    private final int ataque;
    private final int defesa;
    private final int velocidade;
    private final List<String> ataques;
    
    // Construtor privado ? só Builder cria
    private Pokemon(Builder builder) {
        this.nome = builder.nome;
        this.tipo = builder.tipo;
        this.hp = builder.hp;
        this.ataque = builder.ataque;
        this.defesa = builder.defesa;
        this.velocidade = builder.velocidade;
        this.ataques = builder.ataques;
    }
    
    // Builder público
    public static class Builder {
        private String nome;
        private String tipo;
        private int hp = 50;
        private int ataque = 50;
        private int defesa = 50;
        private int velocidade = 50;
        private List<String> ataques = new ArrayList<>();
        
        public Builder nome(String nome) {
            this.nome = nome;
            return this;
        }
        
        public Builder tipo(String tipo) {
            this.tipo = tipo;
            return this;
        }
        
        public Builder hp(int hp) {
            if (hp < 1 || hp > 255) {
                throw new IllegalArgumentException("HP deve estar entre 1 e 255");
            }
            this.hp = hp;
            return this;
        }
        
        public Builder adicionarAtaque(String ataque) {
            if (this.ataques.size() < 4) {
                this.ataques.add(ataque);
            }
            return this;
        }
        
        public Pokemon build() {
            if (nome == null || tipo == null) {
                throw new IllegalStateException("Nome e tipo são obrigatórios");
            }
            return new Pokemon(this);
        }
    }
}

// Uso
Pokemon pikachu = new Pokemon.Builder()
    .nome("Pikachu")
    .tipo("Elétrico")
    .hp(35)
    .ataque(55)
    .defesa(40)
    .adicionarAtaque("Choque do Trovão")
    .adicionarAtaque("Cabeçada")
    .build();
```

**Vantagem:** Validação em cada setter, estado sempre válido ao construir.

---

## Encapsulamento em Hierarquias (Herança)

Subclasses precisam de acesso a alguns atributos, mas não ao mundo.

```java
// Classe base encapsulada
public class Pokemon {
    private int hp;
    private int mana;
    protected int nivelEvoluido;  // Subclasses acessam
    
    // Subclass acessa protected
    protected void ativarEvolucao() {
        this.nivelEvoluido = 2;
    }
    
    // Método protegido para subclasses fazerem override
    protected void calcularBonusEvoluido() {
        hp *= 1.2;
        mana *= 1.1;
    }
}

public class Charizard extends Pokemon {
    @Override
    protected void calcularBonusEvoluido() {
        super.calcularBonusEvoluido();
        // Lógica adicional de Charizard
    }
}
```

**Regra de ouro:** Use `protected` com moderação. Se muitas subclasses acessam, talvez o atributo deveria ser `private` com métodos `protected`.

---

## Record: Encapsulamento Moderno (Java 14+)

Records são classes imutáveis de alto nível, ideais para DTOs e value objects.

```java
// Record auto-gera getters, equals(), hashCode(), toString()
public record PokemonRecord(
    String nome,
    String tipo,
    int hp,
    int ataque,
    int defesa
) {}

// Uso
PokemonRecord pikachu = new PokemonRecord("Pikachu", "Elétrico", 35, 55, 40);
System.out.println(pikachu.nome());  // Getter automático
```

**Vantagens:**
- Menos boilerplate
- Imutável por padrão
- Perfeito para encapsular dados puros

**Limitações:**
- Todos os campos são públicos (via getters)
- Não permite métodos sem lógica adicional fácil
- Não compatível com Java < 14

---

## Anti-padrões: Erros Comuns de Encapsulamento

### 1. Getters/Setters Triviais (Sem Validação)

```java
// ? Por que não apenas deixar público?
public class Pokemon {
    private int hp;
    
    public int getHp() {
        return hp;  // Nenhuma lógica
    }
    
    public void setHp(int hp) {
        this.hp = hp;  // Nenhuma validação
    }
}
```

**Solução:** Remova getter/setter trivial ou adicione lógica.

### 2. Expor Referências Mutáveis

```java
// ? Errado
private List<String> ataques;
public List<String> getAtaques() {
    return ataques;  // Retorna referência direta!
}

// Externo pode fazer:
pokemon.getAtaques().clear();  // ?? Limpa tudo

// ? Correto
public List<String> getAtaques() {
    return new ArrayList<>(ataques);  // Cópia
}
```

### 3. Lógica de Negócio no Setter

```java
// ? Errado ? setters devem ser simples
public void setNivel(int nivel) {
    this.nivel = nivel;
    recalcularStats();
    salvarNoBancoDados();
    enviarNotificacao();
}

// ? Correto ? método separado
public void subirNivel() {
    this.nivel++;
    recalcularStats();
}

// Lógica externa trata persistência
```

### 4. Bloat de Getters/Setters

```java
// ? Classe com 50 atributos = 50 getters + 50 setters
// Sinal de que a classe faz muito

// ? Refatore em múltiplas classes
public class Pokemon {
    private Stats stats;  // Agrupa hp, ataque, etc
    private Inventario inventario;
}
```

---

## Trade-offs: Quando Flexibilizar Encapsulamento

| Situação | Solução | Motivo |
|---|---|---|
| Performance crítica (milhões de objetos) | `public final` ao invés de getter | Acesso direto é mais rápido |
| Prototyping rápido | Sem encapsulamento rigoroso | Ganha velocidade inicial |
| Framework (Hibernate, Jackson) | Aceitar acesso package-private | Framework precisa acessar |
| Dados puramente imutáveis | Record ou final fields | Simplicidade |
| API pública de bilioteca | Máximo encapsulamento | Compatibilidade de versões |

---

## Checklist: Encapsulamento Correto

- [ ] Todos os atributos são `private`?
- [ ] Apenas getters/setters necessários são públicos?
- [ ] Setters validam entrada?
- [ ] Coleções retornam cópias ou imutáveis?
- [ ] Métodos auxiliares são `private`?
- [ ] `protected` é usado apenas quando subclasses precisam?
- [ ] Estado do objeto nunca fica inválido?
- [ ] Documentei o contrato público (Javadoc)?

---

## Resumo em Analógia Pokémon

Um Pokémon é uma "caixa preta" com controles:
- **Atributos privados**: HP, ataque, evolução interna ? ninguém mexe
- **Getters**: `getHp()` retorna estado, mas não deixa fraudar
- **Métodos públicos**: `atacar()`, `defender()`, `evoluir()` ? interface oficial
- **Validação**: Pokémon nunca entra em estado inválido
- **Mudanças internas**: Se trocar como `calcularDano()` funciona, API pública não quebra

Sem encapsulamento: alguém do lado de fora muda o HP para -9999, o tipo para "Sagrado" (não existe), e quebra tudo.

Com encapsulamento: tudo funciona como esperado.

---

## Leitura Recomendada

- **Effective Java** (Joshua Bloch) ? Item 15 (Minimize Mutability)
- **Clean Code** (Robert Martin) ? Capítulo 6 (Objects and Data Structures)
- **Java Concurrency in Practice** ? Capitulo 3 (Visibility)
- Documentação oficial: [Java Language Specification § 6.6 (Access Control)](https://docs.oracle.com/javase/specs/jls/se21/html/jls-6.html#jls-6.6)