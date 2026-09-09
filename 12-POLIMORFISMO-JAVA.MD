# Polimorfismo em Java

Um dos pilares da POO. Permite que um mesmo método se comporte de forma diferente dependendo do objeto que o executa.

---

## 1. O Problema Sem Polimorfismo

```java
Veiculo veiculo = obterVeiculo();

// ❌ Precisa checar o tipo manualmente
if (veiculo instanceof Caminhao) {
    ((Caminhao) veiculo).calcularFrete();
} else if (veiculo instanceof Moto) {
    ((Moto) veiculo).calcularFrete();
} else if (veiculo instanceof Van) {
    ((Van) veiculo).calcularFrete();
}
// Código repetitivo e difícil de manter
```

---

## 2. A Solução: Polimorfismo

```java
public class Veiculo {
    protected String placa;
    
    public double calcularFrete(double distancia) {
        return distancia * 1.0;
    }
}

public class Caminhao extends Veiculo {
    @Override
    public double calcularFrete(double distancia) {
        return distancia * 2.5; // Mais caro: carga pesada
    }
}

public class Moto extends Veiculo {
    @Override
    public double calcularFrete(double distancia) {
        return distancia * 0.8; // Mais barato: entrega leve
    }
}

public class Van extends Veiculo {
    @Override
    public double calcularFrete(double distancia) {
        return distancia * 1.5;
    }
}
```

```java
// ✓ Sem checagem de tipo — cada um calcula à sua maneira
Veiculo veiculo = obterVeiculo();
double frete = veiculo.calcularFrete(100);
```

---

## 2.1. Pré-requisito: Herança

Polimorfismo depende de herança (ou implementação de interface) para existir. É a herança que permite que uma variável do tipo `Veiculo` referencie objetos de subtipos diferentes (`Caminhao`, `Moto`, `Van`). O polimorfismo é o comportamento que aparece quando cada subtipo executa sua própria versão do método herdado.

---

## 3. Uma Referência, Vários Tipos

```java
Veiculo v1 = new Caminhao();  // ✓ Permitido
Veiculo v2 = new Moto();      // ✓ Permitido
Veiculo v3 = new Van();       // ✓ Permitido

// O contrário NÃO é permitido
Caminhao c = new Veiculo();   // ❌ Erro de compilação
```

Cada `v1`, `v2`, `v3` é do tipo declarado `Veiculo`, mas o comportamento em tempo de execução depende do objeto real que cada um guarda.

---

## 4. Aplicação Prática: Frota Mista

```java
public class Frota {
    private List<Veiculo> veiculos = new ArrayList<>();
    
    public void adicionar(Veiculo v) {
        veiculos.add(v);
    }
    
    // Funciona com QUALQUER subtipo de Veiculo
    public double calcularFreteTotal(double distancia) {
        double total = 0;
        for (Veiculo v : veiculos) {
            total += v.calcularFrete(distancia); // Polimorfismo em ação
        }
        return total;
    }
}
```

```java
Frota frota = new Frota();
frota.adicionar(new Caminhao());
frota.adicionar(new Moto());
frota.adicionar(new Van());

double total = frota.calcularFreteTotal(100);
// Cada veículo contribui com seu próprio cálculo de frete
```

---

## 5. Override: A Base do Polimorfismo em Tempo de Execução

O método na subclasse precisa ter a **mesma assinatura** do método na superclasse.

```java
public class Veiculo {
    public void iniciarEntrega() {
        System.out.println("Iniciando entrega padrão");
    }
}

public class Moto extends Veiculo {
    @Override
    public void iniciarEntrega() {
        System.out.println("Moto: entrega expressa iniciada");
    }
}

public class Caminhao extends Veiculo {
    @Override
    public void iniciarEntrega() {
        System.out.println("Caminhão: carregando docas antes de sair");
    }
}
```

```java
Veiculo v = new Moto();
v.iniciarEntrega(); // "Moto: entrega expressa iniciada"

v = new Caminhao();
v.iniciarEntrega(); // "Caminhão: carregando docas antes de sair"
```

O método chamado é sempre definido pelo **tipo real do objeto**, nunca pelo tipo da variável.

---

## 6. Polimorfismo com Interfaces

Interfaces também permitem polimorfismo — sem precisar de herança de classe.

```java
public interface Rastreavel {
    String obterLocalizacao();
}

public class Caminhao implements Rastreavel {
    @Override
    public String obterLocalizacao() {
        return "GPS do caminhão: -8.05, -34.9";
    }
}

public class Moto implements Rastreavel {
    @Override
    public String obterLocalizacao() {
        return "GPS da moto: -8.01, -34.85";
    }
}
```

```java
List<Rastreavel> rastreaveis = new ArrayList<>();
rastreaveis.add(new Caminhao());
rastreaveis.add(new Moto());

for (Rastreavel r : rastreaveis) {
    System.out.println(r.obterLocalizacao()); // Cada um responde à sua forma
}
```

---

## 7. Polimorfismo em Parâmetros de Métodos

Um método pode aceitar a superclasse e funcionar com qualquer subtipo.

```java
public class ServicoEntrega {
    public void despachar(Veiculo veiculo, double distancia) {
        System.out.println("Frete: " + veiculo.calcularFrete(distancia));
        veiculo.iniciarEntrega();
    }
}
```

```java
ServicoEntrega servico = new ServicoEntrega();
servico.despachar(new Caminhao(), 200); // Funciona
servico.despachar(new Moto(), 50);      // Funciona
servico.despachar(new Van(), 120);      // Funciona
```

Sem polimorfismo, seria necessário um método `despachar` sobrecarregado para cada tipo de veículo.

---

## 8. Downcasting: Quando Precisar do Tipo Específico

Às vezes é necessário acessar um método que só existe na subclasse. Use `instanceof` antes de converter.

```java
Veiculo veiculo = new Caminhao();

if (veiculo instanceof Caminhao) {
    Caminhao caminhao = (Caminhao) veiculo;
    caminhao.carregar(5000); // Método exclusivo do Caminhao
}
```

```java
// ❌ Sem verificação: risco de ClassCastException
Moto moto = (Moto) veiculo; // Erro em runtime, pois veiculo é um Caminhao
```

Usar downcasting com frequência é sinal de que o design pode ser repensado — o polimorfismo existe justamente para evitar essas checagens.

---

## 9. Exemplo Completo: Central de Despacho

```java
public class Veiculo {
    protected String placa;
    
    public Veiculo(String placa) {
        this.placa = placa;
    }
    
    public double calcularFrete(double distancia) {
        return distancia * 1.0;
    }
    
    public void iniciarEntrega() {
        System.out.println(placa + ": entrega padrão iniciada");
    }
}

public class Caminhao extends Veiculo {
    public Caminhao(String placa) {
        super(placa);
    }
    
    @Override
    public double calcularFrete(double distancia) {
        return distancia * 2.5;
    }
    
    @Override
    public void iniciarEntrega() {
        System.out.println(placa + ": carregando docas antes de sair");
    }
}

public class Moto extends Veiculo {
    public Moto(String placa) {
        super(placa);
    }
    
    @Override
    public double calcularFrete(double distancia) {
        return distancia * 0.8;
    }
    
    @Override
    public void iniciarEntrega() {
        System.out.println(placa + ": entrega expressa iniciada");
    }
}

public class Van extends Veiculo {
    public Van(String placa) {
        super(placa);
    }
    
    @Override
    public double calcularFrete(double distancia) {
        return distancia * 1.5;
    }
}

public class CentralDespacho {
    private List<Veiculo> veiculos = new ArrayList<>();
    
    public void registrar(Veiculo v) {
        veiculos.add(v);
    }
    
    public void despacharTodos(double distancia) {
        for (Veiculo v : veiculos) {
            v.iniciarEntrega();
            System.out.println("Frete: R$" + v.calcularFrete(distancia));
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        CentralDespacho central = new CentralDespacho();
        central.registrar(new Caminhao("ABC1234"));
        central.registrar(new Moto("XYZ5678"));
        central.registrar(new Van("DEF9012"));
        
        central.despacharTodos(100);
    }
}

// Output:
// ABC1234: carregando docas antes de sair
// Frete: R$250.0
// XYZ5678: entrega expressa iniciada
// Frete: R$80.0
// DEF9012: entrega padrão iniciada
// Frete: R$150.0
```

---

## 10. Erros Comuns

### Sobrecarregar em vez de sobrescrever

```java
public class Veiculo {
    public double calcularFrete(double distancia) {
        return distancia;
    }
}

public class Caminhao extends Veiculo {
    // ❌ Assinatura diferente = overload, não override
    // O polimorfismo de tempo de execução NÃO acontece aqui
    public double calcularFrete(double distancia, double peso) {
        return distancia * 2.5 + peso;
    }
}
```

Sempre usar `@Override` ajuda o compilador a avisar quando a assinatura não bate.

### Downcasting sem verificação

```java
Veiculo v = new Van();
Caminhao c = (Caminhao) v; // ❌ ClassCastException em runtime
```

### Esperar polimorfismo em atributos

```java
public class Veiculo {
    protected double capacidade = 10;
}

public class Caminhao extends Veiculo {
    protected double capacidade = 100; // Isso é OCULTAÇÃO, não override
}

Veiculo v = new Caminhao();
System.out.println(v.capacidade); // 10 — usa o tipo da variável, não do objeto
```

Polimorfismo funciona para **métodos**, não para atributos.

---

## Resumo

| Conceito | Significado | Exemplo |
|----------|-------------|---------|
| **Polimorfismo** | Mesmo método, comportamentos diferentes | `veiculo.calcularFrete()` |
| **Override** | Mecanismo que viabiliza o polimorfismo | `@Override` na subclasse |
| **Upcasting** | Atribuir subtipo a uma referência do supertipo | `Veiculo v = new Caminhao();` |
| **Downcasting** | Converter de volta ao tipo específico | `(Caminhao) v` |
| **Polimorfismo por interface** | Múltiplos tipos implementando o mesmo contrato | `implements Rastreavel` |

---

## Dicas Finais

✓ Sempre use `@Override` para garantir que a assinatura está correta  
✓ Programe pensando na superclasse ou interface, não no tipo concreto  
✓ Evite `instanceof` e downcasting sempre que possível — se estiver checando tipos com frequência, repense o design  
✓ Lembre-se: polimorfismo funciona em métodos, não em atributos  
✓ Sem herança ou interface não há polimorfismo de subtipo em Java