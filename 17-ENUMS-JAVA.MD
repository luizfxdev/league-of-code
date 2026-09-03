# Enumerações em Java

## O que é uma Enumeração?

Enumeração (`enum`) é um tipo especial em Java que define um conjunto literal de constantes relacionadas. É a forma mais semântica e segura de representar valores com opções fixas e pré-definidas.

### Por que usar?

- ✅ **Semântica clara**: código legível e auto-explicativo
- ✅ **Segurança em tempo de compilação**: o compilador valida valores
- ✅ **Evita strings mágicas**: nada de `"PENDING_PAYMENT"` solto no código
- ✅ **Type-safety**: não aceita valores inválidos

---

## Syntax Básica

```java
public enum OrderStatus {
    PENDING_PAYMENT,
    PROCESSING,
    SHIPPED,
    DELIVERED
}
```

**Uso:**

```java
OrderStatus status = OrderStatus.DELIVERED;

// Conversão de String para enum
OrderStatus converted = OrderStatus.valueOf("DELIVERED");
```

---

## Exemplo Prático: Ciclo de Vida de um Pedido

### Definição da Enum

```java
package entities.enums;

public enum OrderStatus {
    PENDING_PAYMENT,    // int 0
    PROCESSING,         // int 1
    SHIPPED,            // int 2
    DELIVERED          // int 3
}
```

### Uso na Entidade

```java
package entities;

import java.util.Date;
import entities.enums.OrderStatus;

public class Order {
    private Integer id;
    private Date moment;
    private OrderStatus status;
    
    // getters e setters
    public OrderStatus getStatus() {
        return status;
    }
    
    public void setStatus(OrderStatus status) {
        this.status = status;
    }
}
```

### Diagrama UML

```
┌──────────────────┐
│    <<enum>>      │
│   OrderStatus    │
├──────────────────┤
│ PENDING_PAYMENT  │
│ PROCESSING       │
│ SHIPPED          │
│ DELIVERED        │
└──────────────────┘
         ▲
         │ usa
         │
┌────────────────┐
│     Order      │
├────────────────┤
│ - id: Integer  │
│ - moment: Date │
│ - status: OrderStatus
└────────────────┘
```

---

## Conversão de String para Enum

```java
// Direto (já conhecendo o valor)
OrderStatus os1 = OrderStatus.DELIVERED;

// Via valueOf() (quando vem de formulário, BD, API)
OrderStatus os2 = OrderStatus.valueOf("DELIVERED");

// Com tratamento de erro
try {
    OrderStatus os3 = OrderStatus.valueOf("INVALIDO");
} catch (IllegalArgumentException e) {
    System.out.println("Status inválido!");
}
```

---

## Enums com Propriedades e Métodos

Enums em Java são mais poderosas: podem ter atributos, construtores e métodos.

### Exemplo: Status com Descrição

```java
public enum OrderStatus {
    PENDING_PAYMENT(0, "Aguardando Pagamento"),
    PROCESSING(1, "Processando"),
    SHIPPED(2, "Enviado"),
    DELIVERED(3, "Entregue");
    
    private final int code;
    private final String description;
    
    OrderStatus(int code, String description) {
        this.code = code;
        this.description = description;
    }
    
    public int getCode() {
        return code;
    }
    
    public String getDescription() {
        return description;
    }
}
```

**Uso:**

```java
OrderStatus status = OrderStatus.DELIVERED;
System.out.println(status.getCode());          // 3
System.out.println(status.getDescription());   // Entregue
```

---

## Boas Práticas

### 1. **Nomeação**
```java
// ✅ BOM - nome descritivo, plural ou singular?
public enum PaymentMethod {
    CREDIT_CARD,
    DEBIT_CARD,
    BANK_TRANSFER
}

// ❌ RUIM - muito genérico
public enum Status {
    S1, S2, S3
}
```

### 2. **Localização no Projeto**
```
src/
├── main/
│   └── java/
│       └── com/app/
│           ├── entities/
│           │   ├── Order.java
│           │   └── Client.java
│           └── enums/          ← Pasta dedicada
│               ├── OrderStatus.java
│               ├── PaymentMethod.java
│               └── WorkerLevel.java
```

### 3. **Quando NÃO Usar**
```java
// ❌ Evite: apenas 2 valores (use boolean)
public enum BooleanEnum {
    YES, NO
}

// ✅ Melhor:
private boolean active;

// ❌ Evite: valores que mudam frequentemente
public enum Country {
    BRAZIL, USA, ...  // E quando adicionar novo país?
}

// ✅ Melhor: use tabela no banco de dados
```

### 4. **Tratamento de Valores Desconhecidos**
```java
public static OrderStatus safeValueOf(String value) {
    try {
        return OrderStatus.valueOf(value.toUpperCase());
    } catch (IllegalArgumentException e) {
        return OrderStatus.PENDING_PAYMENT;  // valor padrão
    }
}
```

---

## Composição: Enums + Objetos

Enums frequentemente trabalham com composição para criar estruturas mais complexas.

### Exemplo: Worker com WorkerLevel (Enum)

```java
public enum WorkerLevel {
    JUNIOR,      // int 0
    MID_LEVEL,   // int 1
    SENIOR       // int 2
}
```

```java
public class Worker {
    private String name;
    private WorkerLevel level;
    private Double baseSalary;
    
    public Worker(String name, WorkerLevel level, Double baseSalary) {
        this.name = name;
        this.level = level;
        this.baseSalary = baseSalary;
    }
    
    public void addContract(HourContract contract) {
        // lógica
    }
    
    public Double income(int year, int month) {
        // cálculo
    }
}
```

---

## Iterando sobre Enums

```java
// Obter todos os valores
OrderStatus[] statuses = OrderStatus.values();

for (OrderStatus status : statuses) {
    System.out.println(status.name());  // nome como String
    System.out.println(status.ordinal()); // posição (0, 1, 2, 3)
}

// Verificar se existe
boolean exists = Arrays.asList(OrderStatus.values())
                        .contains(OrderStatus.DELIVERED);
```

---

## Padrão de Design: Categorias de Classes

Enums geralmente fazem parte de um domínio maior com **Entities**, **Services** e **Repositories**.

```
┌──────────────────────────────────────────┐
│         ESTRUTURA DO PROJETO             │
├──────────────────────────────────────────┤
│                                          │
│  📦 Entities                             │
│  ├── Order                               │
│  ├── OrderItem                           │
│  ├── Product                             │
│  └── Client                              │
│                                          │
│  📦 Enums                                │
│  └── OrderStatus                         │
│                                          │
│  📦 Services                             │
│  ├── OrderService                        │
│  └── PaymentService                      │
│                                          │
│  📦 Repositories                         │
│  └── OrderRepository                     │
│                                          │
│  📦 Controllers (Web)                    │
│  └── OrderController                     │
│                                          │
└──────────────────────────────────────────┘
```

---

# Exercício Resolvido: Sistema de Pedidos com Composição

## Enunciado

**Criar um sistema de pedidos onde:**
- Um `Order` possui vários `OrderItem`s
- Cada `OrderItem` referencia um `Product`
- Um `Order` pertence a um `Client`
- Cada `Order` tem um `OrderStatus`
- Calcular o total e exibir sumário

---

## Solução Passo a Passo

### 1️⃣ Criar a Enum

```java
package entities.enums;

public enum OrderStatus {
    PENDING_PAYMENT,
    PROCESSING,
    SHIPPED,
    DELIVERED
}
```

**O que faz:** Define os 4 estados possíveis de um pedido. O compilador garante que nenhum outro valor será aceito.

---

### 2️⃣ Criar a Classe Product

```java
package entities;

public class Product {
    private String name;
    private Double price;
    
    public Product(String name, Double price) {
        this.name = name;
        this.price = price;
    }
    
    public String getName() {
        return name;
    }
    
    public Double getPrice() {
        return price;
    }
}
```

**O que faz:** Representa um produto com nome e preço. Simples e sem estado mutável (boa prática).

---

### 3️⃣ Criar a Classe OrderItem

```java
package entities;

public class OrderItem {
    private Integer quantity;
    private Double price;
    private Product product;
    
    public OrderItem(Integer quantity, Double price, Product product) {
        this.quantity = quantity;
        this.price = price;
        this.product = product;
    }
    
    public Integer getQuantity() {
        return quantity;
    }
    
    public Double getPrice() {
        return price;
    }
    
    public Product getProduct() {
        return product;
    }
    
    // Subtotal do item
    public Double getSubTotal() {
        return quantity * price;
    }
}
```

**O que faz:**
- Quantidade do produto
- Preço unitário (pode ser diferente do preço atual do produto)
- Referência ao produto
- `getSubTotal()` calcula quantidade × preço

---

### 4️⃣ Criar a Classe Client

```java
package entities;

import java.text.SimpleDateFormat;
import java.util.Date;

public class Client {
    private String name;
    private String email;
    private Date birthDate;
    
    private static SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");
    
    public Client(String name, String email, Date birthDate) {
        this.name = name;
        this.email = email;
        this.birthDate = birthDate;
    }
    
    public String getName() {
        return name;
    }
    
    public String getEmail() {
        return email;
    }
    
    public Date getBirthDate() {
        return birthDate;
    }
    
    @Override
    public String toString() {
        return name + " (" + sdf.format(birthDate) + ") - " + email;
    }
}
```

**O que faz:** Representa um cliente com nome, email e data de nascimento. `toString()` formata a exibição.

---

### 5️⃣ Criar a Classe Order

```java
package entities;

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import entities.enums.OrderStatus;

public class Order {
    private Date moment;
    private OrderStatus status;
    private Client client;
    private List<OrderItem> items;
    
    private static SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy HH:mm:ss");
    
    public Order(Date moment, OrderStatus status, Client client) {
        this.moment = moment;
        this.status = status;
        this.client = client;
        this.items = new ArrayList<>();
    }
    
    public Date getMoment() {
        return moment;
    }
    
    public OrderStatus getStatus() {
        return status;
    }
    
    public void setStatus(OrderStatus status) {
        this.status = status;
    }
    
    public Client getClient() {
        return client;
    }
    
    public List<OrderItem> getItems() {
        return items;
    }
    
    // Adicionar item ao pedido
    public void addItem(OrderItem item) {
        items.add(item);
    }
    
    // Remover item do pedido
    public void removeItem(OrderItem item) {
        items.remove(item);
    }
    
    // Calcular total
    public Double total() {
        double sum = 0.0;
        for (OrderItem item : items) {
            sum += item.getSubTotal();
        }
        return sum;
    }
    
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("Order moment: ");
        sb.append(sdf.format(moment));
        sb.append("\nOrder status: ");
        sb.append(status);
        sb.append("\nClient: ");
        sb.append(client);
        sb.append("\nOrder items:\n");
        for (OrderItem item : items) {
            sb.append(item.getProduct().getName());
            sb.append(", $");
            sb.append(String.format("%.2f", item.getPrice()));
            sb.append(", Quantity: ");
            sb.append(item.getQuantity());
            sb.append(", Subtotal: $");
            sb.append(String.format("%.2f", item.getSubTotal()));
            sb.append("\n");
        }
        sb.append("Total price: $");
        sb.append(String.format("%.2f", total()));
        return sb.toString();
    }
}
```

**O que faz:**
- **Composição com List**: `Order` "tem" vários `OrderItem`s
- **Enum**: `status` é tipo `OrderStatus` (seguro)
- **Composição com Client**: `Order` "tem" um `Client`
- `addItem()` e `removeItem()`: gerenciam itens
- `total()`: soma todos os subtotais
- `toString()`: exibe o pedido formatado

---

### 6️⃣ Programa Principal (Main)

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Scanner;
import entities.Client;
import entities.Order;
import entities.OrderItem;
import entities.Product;
import entities.enums.OrderStatus;

public class Program {
    public static void main(String[] args) throws ParseException {
        Scanner sc = new Scanner(System.in);
        SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");
        
        // 1. LER DADOS DO CLIENTE
        System.out.println("Enter cliente data:");
        System.out.print("Name: ");
        String clientName = sc.nextLine();
        
        System.out.print("Email: ");
        String clientEmail = sc.nextLine();
        
        System.out.print("Birth date (DD/MM/YYYY): ");
        Date clientBirthDate = sdf.parse(sc.nextLine());
        
        Client client = new Client(clientName, clientEmail, clientBirthDate);
        
        // 2. LER DADOS DO PEDIDO
        System.out.println("Enter order data:");
        System.out.print("Status: ");
        OrderStatus orderStatus = OrderStatus.valueOf(sc.nextLine());
        
        Order order = new Order(new Date(), orderStatus, client);
        
        // 3. LER QUANTIDADE DE ITENS
        System.out.print("How many items to this order? ");
        int n = sc.nextInt();
        sc.nextLine(); // limpar buffer
        
        // 4. LER CADA ITEM
        for (int i = 1; i <= n; i++) {
            System.out.println("Enter #" + i + " item data:");
            
            System.out.print("Product name: ");
            String productName = sc.nextLine();
            
            System.out.print("Product price: ");
            Double productPrice = sc.nextDouble();
            
            System.out.print("Quantity: ");
            Integer quantity = sc.nextInt();
            sc.nextLine(); // limpar buffer
            
            Product product = new Product(productName, productPrice);
            OrderItem item = new OrderItem(quantity, productPrice, product);
            
            order.addItem(item);
        }
        
        // 5. EXIBIR SUMÁRIO
        System.out.println("\n" + order);
        
        sc.close();
    }
}
```

**Fluxo linha por linha:**

| Linha | O que faz |
|-------|-----------|
| `System.out.println("Enter cliente data:")` | Aviso ao usuário |
| `String clientName = sc.nextLine()` | Lê o nome como String |
| `clientBirthDate = sdf.parse(...)` | Converte "DD/MM/YYYY" para objeto `Date` |
| `Client client = new Client(...)` | Cria instância do cliente |
| `OrderStatus.valueOf(sc.nextLine())` | Converte a string "PROCESSING" para enum |
| `Order order = new Order(new Date(), ...)` | Cria pedido com data/hora atual e status |
| `for (int i = 1; i <= n; i++)` | Loop para ler N itens |
| `Product product = new Product(...)` | Cria o produto |
| `OrderItem item = new OrderItem(...)` | Cria o item (composição) |
| `order.addItem(item)` | Adiciona item à lista do pedido |
| `System.out.println("\n" + order)` | Exibe usando `toString()` customizado |

---

### 📊 Resultado da Execução

```
Enter cliente data:
Name: Alex Green
Email: alex@gmail.com
Birth date (DD/MM/YYYY): 15/03/1985
Enter order data:
Status: PROCESSING
How many items to this order? 2
Enter #1 item data:
Product name: TV
Product price: 1000.00
Quantity: 1
Enter #2 item data:
Product name: Mouse
Product price: 40.00
Quantity: 2

Order moment: 20/04/2018 11:25:09
Order status: PROCESSING
Client: Alex Green (15/03/1985) - alex@gmail.com
Order items:
TV, $1000.00, Quantity: 1, Subtotal: $1000.00
Mouse, $40.00, Quantity: 2, Subtotal: $80.00
Total price: $1080.00
```

---

### 🎯 Conceitos Aplicados

| Conceito | Onde foi usado |
|----------|----------------|
| **Enum** | `OrderStatus` (segurança de tipo) |
| **Composição "tem-um"** | `Order` → `Client` |
| **Composição "tem-vários"** | `Order` → `List<OrderItem>` |
| **Encapsulamento** | Getters e setters |
| **Polimorfismo de método** | `toString()` customizado |
| **ArrayList** | Armazenar items dinamicamente |
| **SimpleDateFormat** | Parse e formatação de datas |

---

## Diagrama em Memória

```
Cliente:
┌─────────────────┐
│     Client      │
├─────────────────┤
│ name: "Alex"    │
│ email: "a@...   │
│ birthDate: 1985 │
└────────┬────────┘
         │
         │ client
         ▼
┌──────────────────────┐
│      Order           │
├──────────────────────┤
│ moment: 20/04/2018   │
│ status: PROCESSING   │ ← OrderStatus (Enum)
│ items: [ ]           │
└──────────┬───────────┘
           │
           ├─ items[0]: OrderItem
           │   ├─ quantity: 1
           │   ├─ price: 1000.00
           │   └─ product → Product("TV", 1000.00)
           │
           └─ items[1]: OrderItem
               ├─ quantity: 2
               ├─ price: 40.00
               └─ product → Product("Mouse", 40.00)
```

---

## Checklist Final

- ✅ Enum para estados imutáveis
- ✅ Composição com `List<>` para múltiplos itens
- ✅ Encapsulamento com getters/setters
- ✅ Métodos de cálculo (`total()`)
- ✅ `toString()` customizado para exibição
- ✅ Tratamento de entrada com `Scanner`
- ✅ Conversão de tipos (`valueOf()`, `parse()`)