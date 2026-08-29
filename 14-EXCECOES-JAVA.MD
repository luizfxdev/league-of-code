# Exceções em Java - LoL Edition

Exceções são **eventos que interrompem o fluxo normal** do programa. Sem tratá-las, seu código quebra.

---

## 1. O Conceito

### O Problema

```java
public class Jogo {
    public static void main(String[] args) {
        String[] campeoes = {"Garen", "Lux", "Ahri"};
        
        // ❌ Acesso fora do limite
        System.out.println(campeoes[5]);  // Array tem 3 elementos!
        
        // ArrayIndexOutOfBoundsException!
        // Programa QUEBRA aqui
        
        // Essa linha nunca executa
        System.out.println("Jogo finalizado");
    }
}

// Output:
// Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 5 out of bounds for length 3
```

### A Solução: Try-Catch

```java
public class Jogo {
    public static void main(String[] args) {
        String[] campeoes = {"Garen", "Lux", "Ahri"};
        
        try {
            // Código que pode gerar exceção
            System.out.println(campeoes[5]);
        } catch (ArrayIndexOutOfBoundsException e) {
            // Tratamento da exceção
            System.out.println("Índice inválido! Tamanho do array: " + campeoes.length);
        }
        
        // Programa CONTINUA normalmente
        System.out.println("Jogo finalizado");
    }
}

// Output:
// Índice inválido! Tamanho do array: 3
// Jogo finalizado
```

---

## 2. Estrutura Try-Catch

### Sintaxe Básica

```java
try {
    // Código que pode gerar exceção
} catch (TipoDaExcecao e) {
    // Tratamento: o que fazer quando erro acontece
}
```

### Exemplo Prático

```java
public class Campeao {
    public void atacar(Campeao alvo) {
        try {
            // Pode gerar NullPointerException
            int dano = calcularDano();
            alvo.levarDano(dano);
            System.out.println("Ataque bem-sucedido!");
        } catch (NullPointerException e) {
            // Alvo é null
            System.out.println("Erro: alvo inválido!");
        }
    }
}
```

---

## 3. Múltiplos Catches

### Capturar Exceções Diferentes

```java
try {
    // Vários cenários de erro
    String[] campeoes = {"Garen", "Lux"};
    int indice = Integer.parseInt("abc");  // NumberFormatException
    String nome = campeoes[indice];         // ArrayIndexOutOfBoundsException
} catch (NumberFormatException e) {
    System.out.println("Número inválido!");
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Índice inválido!");
} catch (Exception e) {
    // Captura QUALQUER exceção
    System.out.println("Erro desconhecido: " + e.getMessage());
}
```

### Ordem Importa!

```java
try {
    // código
} catch (Exception e) {
    // ❌ Ordem errada: Exception é o pai de tudo!
} catch (NullPointerException e) {
    // Nunca chega aqui
}

// ✓ Ordem correta: filhos antes, pai depois
try {
    // código
} catch (NullPointerException e) {
    // Específico
} catch (Exception e) {
    // Genérico
}
```

---

## 4. Finally: Limpeza Garantida

### Executar Independentemente

```java
try {
    int[] numeros = {1, 2, 3};
    System.out.println(numeros[10]);
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Erro no array!");
} finally {
    // Sempre executa, INDEPENDENTEMENTE
    System.out.println("Limpando recursos...");
}

// Output:
// Erro no array!
// Limpando recursos...
```

### Caso Prático: Fechar Arquivo

```java
FileReader reader = null;
try {
    reader = new FileReader("campeoes.txt");
    // Lê arquivo
} catch (IOException e) {
    System.out.println("Erro ao ler arquivo!");
} finally {
    // Garante que fecha, mesmo com exceção
    if (reader != null) {
        reader.close();
    }
}
```

### Try-With-Resources (Java 7+)

Mais simples e automático:

```java
// Recurso é fechado automaticamente
try (FileReader reader = new FileReader("campeoes.txt")) {
    // Lê arquivo
} catch (IOException e) {
    System.out.println("Erro!");
}
// reader é fechado automaticamente aqui
```

---

## 5. Throws: Passar Exceção Adiante

### Não Tratar Localmente

```java
public class Jogo {
    // Método "lança" exceção para quem chamar tratar
    public static void abrirArquivo(String arquivo) throws IOException {
        FileReader reader = new FileReader(arquivo);
        // Se der erro, quem chamou trata
    }
    
    public static void main(String[] args) {
        try {
            abrirArquivo("inexistente.txt");
        } catch (IOException e) {
            System.out.println("Erro ao abrir arquivo!");
        }
    }
}
```

### Quando Usar?

```java
// ✓ Use throws quando o método não sabe como tratar
public void salvarCampeao(Campeao c) throws IOException {
    // Não sabe salvar em arquivo, passa adiante
}

// ✓ Use try-catch quando SABE como tratar
public void lerCampeao() {
    try {
        salvarCampeao(garen);
    } catch (IOException e) {
        // Sabe como tratar
    }
}
```

---

## 6. Tipos de Exceções

### Checked vs. Unchecked

```java
// CHECKED: Você DEVE tratar (IOException, SQLException)
try {
    FileReader reader = new FileReader("arquivo.txt");
} catch (FileNotFoundException e) {
    // Obrigado a tratar
}

// UNCHECKED: Não é obrigado tratar (NullPointerException, ArrayIndexOutOfBoundsException)
int[] array = {1, 2, 3};
System.out.println(array[10]);  // ArrayIndexOutOfBoundsException (não precisa catch)
```

### Hierarquia

```
Throwable
  ├── Error (muito grave, não trate)
  │   └── OutOfMemoryError
  └── Exception
      ├── Checked (obrigatório tratar)
      │   └── IOException, SQLException
      └── Unchecked / RuntimeException
          ├── NullPointerException
          ├── ArrayIndexOutOfBoundsException
          └── NumberFormatException
```

---

## 7. Criando Exceções Personalizadas

### Extend RuntimeException

```java
// Exceção customizada
public class CampeaoInvalidoException extends RuntimeException {
    public CampeaoInvalidoException(String mensagem) {
        super(mensagem);
    }
}

// Usando:
public class Jogo {
    public void criarCampeao(String nome, int hp) {
        if (nome == null || nome.isEmpty()) {
            throw new CampeaoInvalidoException("Nome do campeão não pode ser vazio!");
        }
        
        if (hp <= 0) {
            throw new CampeaoInvalidoException("HP deve ser maior que 0!");
        }
        
        // Criar campeão
    }
}

// Tratando:
try {
    jogo.criarCampeao("", 100);
} catch (CampeaoInvalidoException e) {
    System.out.println("Erro: " + e.getMessage());
}
```

### Extend Exception (Checked)

```java
// Exceção que OBRIGA tratamento
public class CampeaoNaoEncontradoException extends Exception {
    public CampeaoNaoEncontradoException(String mensagem) {
        super(mensagem);
    }
}

// Método OBRIGA tratamento
public Campeao buscarCampeao(String nome) throws CampeaoNaoEncontradoException {
    // Se não encontra, lança exceção
    if (!existe(nome)) {
        throw new CampeaoNaoEncontradoException("Campeão " + nome + " não existe!");
    }
    return getBD(nome);
}

// Obrigado a tratar
try {
    Campeao c = buscarCampeao("Pokemon");
} catch (CampeaoNaoEncontradoException e) {
    System.out.println(e.getMessage());
}
```

---

## 8. Exemplo Prático Completo

```java
// Exceção personalizada
public class JogoException extends Exception {
    public JogoException(String mensagem) {
        super(mensagem);
    }
}

// Classe de Jogo
public class Jogo {
    private Map<String, Integer> campeoes = new HashMap<>();
    
    // Método que valida e lança exceção
    public void registrarCampeao(String nome, int hp) throws JogoException {
        if (nome == null || nome.trim().isEmpty()) {
            throw new JogoException("Nome não pode ser vazio!");
        }
        
        if (hp <= 0 || hp > 1000) {
            throw new JogoException("HP deve estar entre 1 e 1000!");
        }
        
        if (campeoes.containsKey(nome)) {
            throw new JogoException("Campeão já registrado!");
        }
        
        campeoes.put(nome, hp);
        System.out.println(nome + " registrado com " + hp + " HP!");
    }
    
    // Método que usa o anterior
    public Integer obterHP(String nome) throws JogoException {
        if (!campeoes.containsKey(nome)) {
            throw new JogoException("Campeão não encontrado!");
        }
        return campeoes.get(nome);
    }
    
    // Simulação
    public static void main(String[] args) {
        Jogo jogo = new Jogo();
        
        try {
            jogo.registrarCampeao("Garen", 650);
            jogo.registrarCampeao("Lux", 450);
            jogo.registrarCampeao("Ahri", 495);
            
            System.out.println("HP do Garen: " + jogo.obterHP("Garen"));
            
            // ❌ Vai lançar exceção
            jogo.registrarCampeao("", 500);
            
        } catch (JogoException e) {
            System.out.println("ERRO NO JOGO: " + e.getMessage());
        } finally {
            System.out.println("Jogo finalizado!");
        }
    }
}

// Output:
// Garen registrado com 650 HP!
// Lux registrado com 450 HP!
// Ahri registrado com 495 HP!
// HP do Garen: 650
// ERRO NO JOGO: Nome não pode ser vazio!
// Jogo finalizado!
```

---

## 9. Boas Práticas

### Sempre Mostre Informações Úteis

```java
// ❌ Ruim: não informa nada
try {
    int valor = Integer.parseInt(entrada);
} catch (NumberFormatException e) {
    System.out.println("Erro!");
}

// ✓ Bom: explica o que aconteceu
try {
    int valor = Integer.parseInt(entrada);
} catch (NumberFormatException e) {
    System.out.println("Erro: valor '" + entrada + "' não é um número válido!");
    System.out.println("Por favor, digite um número inteiro.");
}
```

### Log da Exception

```java
try {
    // código
} catch (Exception e) {
    // Registra o erro para debug
    e.printStackTrace();  // Mostra stack trace
    
    // Ou com log:
    System.err.println("Erro: " + e.getMessage());
}
```

### Não Engula Exceções

```java
// ❌ NÃO faça isso (silencia erros!)
try {
    int valor = Integer.parseInt(entrada);
} catch (Exception e) {
    // Ignorar completamente
}

// ✓ Sempre reaja de forma apropriada
try {
    int valor = Integer.parseInt(entrada);
} catch (NumberFormatException e) {
    System.out.println("Entrada inválida!");
    // Pedir nova entrada, logar erro, etc.
}
```

---

## 10. Padrões Comuns

### Validação com Exceção

```java
public class Campeao {
    private String nome;
    
    public void setNome(String nome) throws IllegalArgumentException {
        if (nome == null || nome.isEmpty()) {
            throw new IllegalArgumentException("Nome inválido!");
        }
        this.nome = nome;
    }
}

// Uso:
try {
    campeao.setNome("");
} catch (IllegalArgumentException e) {
    System.out.println(e.getMessage());
}
```

### Retry Pattern

```java
public int tentarConectar(int tentativas) throws JogoException {
    for (int i = 0; i < tentativas; i++) {
        try {
            return conectar();
        } catch (ConnectionException e) {
            if (i == tentativas - 1) {
                throw new JogoException("Falha ao conectar após " + tentativas + " tentativas");
            }
            System.out.println("Tentativa " + (i + 1) + " falhou, tentando novamente...");
        }
    }
    return -1;
}
```

### Chain of Exceptions

```java
try {
    salvarDados();
} catch (IOException e) {
    // Envolver em exceção customizada
    throw new JogoException("Falha ao salvar jogo", e);
}
```

---

## 11. Erros Comuns

### Capturar Exception Genérica

```java
// ❌ Ruim: ignora tipo específico
try {
    int x = Integer.parseInt("abc");
} catch (Exception e) {
    System.out.println("Erro!");
}

// ✓ Melhor: específico
try {
    int x = Integer.parseInt("abc");
} catch (NumberFormatException e) {
    System.out.println("Número inválido!");
}
```

### Lançar Sem Informações

```java
// ❌ Ruim: não explica
throw new RuntimeException("erro");

// ✓ Melhor: explica
throw new RuntimeException("Falha ao carregar campeão com ID: " + id);
```

### Usar Exceção para Controle de Fluxo

```java
// ❌ Muito ruim: exceção como if-else
try {
    int index = 0;
    while (true) {
        processar(array[index]);
        index++;
    }
} catch (ArrayIndexOutOfBoundsException e) {
    // FIM do array
}

// ✓ Correto: usar lógica normal
for (int i = 0; i < array.length; i++) {
    processar(array[i]);
}
```

---

## Resumo

| Conceito | Uso |
|----------|-----|
| **try** | Bloco com código que pode gerar erro |
| **catch** | Trata exceção específica |
| **finally** | Executa sempre |
| **throw** | Lança exceção |
| **throws** | Método lança exceção para quem chamar |

---

## Dicas Finais

✓ **Sempre trate exceções** (não deixe quebrado)  
✓ Seja **específico** (não pegue Exception)  
✓ Use **throws** para passar adiante  
✓ Use **finally** para limpar recursos  
✓ Crie **exceções customizadas** quando necessário  
✓ Nunca **engula exceções** silenciosamente  

Exceções bem tratadas = código robusto! 🎮
