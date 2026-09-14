# Estrutura try-catch em Java

## 1. Ideia central

`try-catch` separa dois mundos no código:

- **try** → "aqui é o fluxo normal, o que eu espero que aconteça"
- **catch** → "aqui é o plano B, caso algo dê errado no try"

Analogia de logística: você **tenta** despachar um caminhão numa rota. Se a rota estiver bloqueada, você **captura** esse problema e decide o que fazer (rota alternativa, aviso ao cliente, etc.) — em vez do sistema simplesmente travar.

---

## 2. Sintaxe básica

```java
try {
    // código que pode falhar
}
catch (TipoDeExcecao e) {
    // código que trata a falha
}
```

Regras de ouro:
- O `catch` só é acionado se o tipo da exceção lançada **for compatível** com o tipo declarado nele (a própria classe ou uma superclasse).
- Se nada der errado dentro do `try`, o(s) `catch` são **ignorados completamente**.
- Assim que uma exceção é lançada dentro do `try`, o restante do `try` é **abandonado na hora** e o controle pula direto pro `catch` compatível.

---

## 3. Exemplo mínimo

```java
package application;

import java.util.Scanner;

public class ConsultaFrota {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String[] frota = {"Caminhão-01", "Van-02", "Moto-03"};

        System.out.print("Digite o índice do veículo: ");

        try {
            int indice = sc.nextInt();
            System.out.println("Veículo: " + frota[indice]);
        }
        catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Não existe veículo nesse índice");
        }

        System.out.println("Consulta finalizada");
        sc.close();
    }
}
```

**O que acontece:**

| Entrada | Resultado |
|---|---|
| `1` | `Veículo: Van-02` — try termina normal, catch é ignorado |
| `9` | `Não existe veículo nesse índice` — exceção lançada, catch assume |

Em ambos os casos, `"Consulta finalizada"` é impresso — o programa **não trava**.

---

## 4. Múltiplos `catch` — tratando erros diferentes

Um mesmo bloco `try` pode gerar tipos diferentes de erro. Cada `catch` cuida de um tipo específico.

```java
package application;

import java.util.InputMismatchException;
import java.util.Scanner;

public class DespachoEntrega {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String[] destinos = {"São Paulo", "Rio de Janeiro", "Curitiba"};

        try {
            System.out.print("Índice do destino: ");
            int indice = sc.nextInt();
            System.out.println("Despachando para: " + destinos[indice]);
        }
        catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Destino não cadastrado");
        }
        catch (InputMismatchException e) {
            System.out.println("Digite um número válido");
        }

        sc.close();
    }
}
```

**Como o Java escolhe o catch:** ele testa **na ordem em que estão escritos**, de cima pra baixo, e entra no primeiro que for compatível com o tipo da exceção lançada.

| Entrada digitada | Exceção lançada | Catch acionado |
|---|---|---|
| `5` (índice inexistente) | `ArrayIndexOutOfBoundsException` | 1º catch |
| `abc` (não é número) | `InputMismatchException` | 2º catch |

---

## 5. Ordem importa: específico primeiro, genérico depois

`Exception` é superclasse de praticamente todo erro comum. Se você colocar `catch (Exception e)` **antes** de um catch mais específico, o específico nunca será alcançado — e o Java nem deixa compilar.

```java
// ERRADO — não compila
try {
    // ...
}
catch (Exception e) {
    System.out.println("Erro genérico");
}
catch (ArrayIndexOutOfBoundsException e) { // inalcançável
    System.out.println("Índice inválido");
}
```

```java
// CERTO — específico primeiro, genérico por último
try {
    // ...
}
catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Índice inválido");
}
catch (Exception e) {
    System.out.println("Erro inesperado: " + e.getMessage());
}
```

**Regra prática:** do mais específico (subclasse) para o mais genérico (superclasse).

---

## 6. Capturando por hierarquia (upcasting é permitido)

Se você tem várias exceções customizadas que compartilham uma superclasse, um único `catch` da superclasse resolve todas — útil quando o tratamento é o mesmo.

```java
public class LogisticaException extends RuntimeException {
    public LogisticaException(String msg) { super(msg); }
}

public class CapacidadeExcedidaException extends LogisticaException {
    public CapacidadeExcedidaException(String msg) { super(msg); }
}

public class RotaInvalidaException extends LogisticaException {
    public RotaInvalidaException(String msg) { super(msg); }
}
```

```java
try {
    servico.despachar(entrega);
}
catch (LogisticaException e) { // pega CapacidadeExcedida OU RotaInvalida
    System.out.println("Falha no despacho: " + e.getMessage());
}
```

Se precisar de mensagens diferentes por tipo, use catches separados (específico antes do genérico).

---

## 7. O objeto da exceção (`e`)

O `catch` recebe um objeto — ele carrega informação, não é só um "aviso vazio".

```java
catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Mensagem: " + e.getMessage());   // texto do erro
    e.printStackTrace();                                  // caminho completo até o erro
}
```

- `e.getMessage()` → texto explicando o problema.
- `e.printStackTrace()` → mostra toda a pilha de chamadas até o ponto do erro (bom pra debug, ruim pra mostrar ao usuário final).

---

## 8. try-catch dentro de método (propagando ou não)

Você decide: **tratar ali mesmo** ou **deixar subir** pra quem chamou o método.

```java
// Trata localmente — quem chama não precisa saber que algo deu errado
public void calcularRota(int[] coordenadas, int indice) {
    try {
        System.out.println(coordenadas[indice]);
    }
    catch (ArrayIndexOutOfBoundsException e) {
        System.out.println("Coordenada inválida, usando rota padrão");
    }
}
```

```java
// Não trata — deixa a exceção subir (sem try-catch aqui)
public void calcularRota(int[] coordenadas, int indice) {
    System.out.println(coordenadas[indice]); // se der erro, sobe pra quem chamou
}
```

Use **tratar localmente** quando o método sabe como se recuperar. Use **deixar propagar** quando só quem chamou tem contexto suficiente pra decidir o que fazer.

---

## 9. Erro comum: `catch` vazio (nunca faça isso)

```java
// PÉSSIMA PRÁTICA
try {
    servico.despachar(entrega);
}
catch (Exception e) {
    // nada aqui
}
```

Isso **engole o erro silenciosamente** — o programa continua como se nada tivesse acontecido, e você nunca vai saber que uma entrega falhou. No mínimo, logue ou informe algo:

```java
catch (Exception e) {
    System.out.println("Falha ao despachar entrega " + entrega.getId() + ": " + e.getMessage());
}
```

---

## 10. Exemplo completo juntando tudo

```java
package application;

import java.util.InputMismatchException;
import java.util.Scanner;

public class PainelLogistica {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        Veiculo[] frota = {
            new Veiculo("Caminhão-01", 5000.0),
            new Veiculo("Van-02", 1200.0)
        };

        try {
            System.out.print("Índice do veículo: ");
            int indice = sc.nextInt();

            System.out.print("Peso da carga (kg): ");
            double peso = sc.nextDouble();

            frota[indice].carregar(peso);
            System.out.println("Carga aceita em " + frota[indice].getNome());
        }
        catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Veículo não encontrado");
        }
        catch (InputMismatchException e) {
            System.out.println("Digite valores numéricos válidos");
        }
        catch (CapacidadeExcedidaException e) {
            System.out.println("Não foi possível carregar: " + e.getMessage());
        }
        finally {
            sc.close();
            System.out.println("Painel encerrado");
        }
    }
}

class Veiculo {
    private String nome;
    private double capacidade;

    public Veiculo(String nome, double capacidade) {
        this.nome = nome;
        this.capacidade = capacidade;
    }

    public void carregar(double peso) {
        if (peso > capacidade) {
            throw new CapacidadeExcedidaException(
                "Peso " + peso + "kg excede capacidade de " + capacidade + "kg");
        }
    }

    public String getNome() { return nome; }
}

class CapacidadeExcedidaException extends RuntimeException {
    public CapacidadeExcedidaException(String msg) { super(msg); }
}
```

Esse exemplo mostra na prática: múltiplos catches, cada um pra um tipo de erro diferente, mais `finally` garantindo que o `Scanner` sempre fecha — dando ou não erro.

---

## 11. try-catch vs if/else

A diferença central: **quem detecta o problema**.

- **if/else** → você mesmo verifica a condição *antes* de agir. Precisa prever e checar cada caso manualmente.
- **try/catch** → você tenta executar, e só reage *se* algo lançar uma exceção. Não precisa prever tudo antecipadamente — o erro "avisa sozinho".

```java
// if/else — você precisa lembrar de checar tudo, sempre
if (indice >= 0 && indice < frota.length) {
    System.out.println(frota[indice]);
} else {
    System.out.println("Veículo não encontrado");
}
```

```java
// try/catch — você tenta, a JVM avisa se der ruim
try {
    System.out.println(frota[indice]);
}
catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Veículo não encontrado");
}
```

| | if/else | try/catch |
|---|---|---|
| Uso ideal | Regras de negócio previsíveis (ex: `if (peso > capacidade)`) | Erros inesperados/externos (ex: arquivo não existe, índice inválido, falha de rede) |
| Quem "lança" o problema | Você mesmo, via condição | A JVM ou código de terceiros |
| Custo em performance | Baixo, sempre executa | Baixo no caminho normal, mas lançar exceção tem custo maior |
| Múltiplas causas de erro | Vira `if/else if` aninhado, fica poluído | Um `catch` por tipo, mais organizado |
| Interrompe o fluxo? | Não, você controla manualmente | Sim, automaticamente pula pro catch |

**Regra prática:** use `if` para validar coisas que você **controla e prevê** (peso, saldo, formato de dado). Use `try/catch` para coisas que **fogem do seu controle** ou vêm de operações arriscadas (parsing, I/O, acesso a índice, chamadas externas).

---

## 12. Resumo rápido

| Situação | O que fazer |
|---|---|
| Um tipo de erro possível | Um `try` + um `catch` |
| Vários tipos de erro possíveis | Vários `catch`, do mais específico pro mais genérico |
| Erros relacionados com mesmo tratamento | Capturar pela superclasse comum |
| Preciso liberar recurso sempre | Usar `finally` |
| Não sei tratar aqui, quem chama sabe | Não usar try-catch, deixar propagar (ou usar `throws`) |
| Não faço ideia do que fazer com o erro | **Nunca** deixe o catch vazio — pelo menos logue |