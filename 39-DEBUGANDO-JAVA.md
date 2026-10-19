# Debugando em Java 17 — Melhores Práticas

---

## Sumário

1. [Introdução: abordando Debugs em Java](#1-introdução-abordando-debugs-em-java)
2. [Configurando o ambiente de debug](#2-configurando-o-ambiente-de-debug)
3. [Breakpoints inteligentes](#3-breakpoints-inteligentes)
4. [Stack traces: como ler e interpretar](#4-stack-traces-como-ler-e-interpretar)
5. [Debugging com `System.out` vs logger](#5-debugging-com-systemout-vs-logger)
6. [Variáveis locais e expressões watch](#6-variáveis-locais-e-expressões-watch)
7. [Debugging de streams e optional](#7-debugging-de-streams-e-optional)
8. [Debugging de exceções customizadas](#8-debugging-de-exceções-customizadas)
9. [Avaliação de expressões em runtime](#9-avaliação-de-expressões-em-runtime)
10. [Debugging de padrão de patterns (switch com pattern matching)](#10-debugging-de-padrão-de-patterns-switch-com-pattern-matching)
11. [Debugging remoto em Java 17](#11-debugging-remoto-em-java-17)
12. [Profiling e identificação de gargalos](#12-profiling-e-identificação-de-gargalos)
13. [Armadilhas comuns e como evitá-las](#13-armadilhas-comuns-e-como-evitá-las)
14. [Cheat sheet de debug](#14-cheat-sheet-de-debug)

---

## 1. Introdução: abordando Debugs em Java

Debug é uma habilidade fundamental no desenvolvimento Java. Um erro silencioso pode causar impacto significativo em sistemas, independente do domínio:

- Dados processados incorretamente
- Estado da aplicação inconsistente
- Fluxo de execução desviado do esperado
- Comportamentos não-determinísticos em produção

O debug deve ser **preciso, rápido e rastreável**. Não há margem para especulação.

### Por que debugar?

Debug é como **auditoria de código em execução**: você segue cada etapa do programa — desde a entrada até a saída — e marca exatamente **onde** algo saiu do esperado e **por quê**.

### Contexto: exemplos em logística

Ao longo deste guia, usaremos exemplos de sistemas logísticos para ilustrar conceitos de debug. As mesmas técnicas aplicam-se a qualquer domínio: e-commerce, financeiro, saúde, redes sociais, etc.

Exemplo de erro silencioso em logística (mas que se aplica a qualquer sistema):
- Uma entrega desviada para o endereço errado
- Um rastreamento incorreto de peso e volume
- Uma coordenada GPS incorreta no mapa
- Um status de pedido congelado no "enviando"

As ferramentas e técnicas apresentadas aqui funcionam em **qualquer aplicação Java 17**.

---

## 2. Configurando o ambiente de debug

### VS Code (padrão para desenvolvimento em Java 17)

#### Iniciar aplicação em modo debug

1. **Instalar extensões necessárias:**
   - "Extension Pack for Java" (Microsoft)
   - "Debugger for Java" (Microsoft)

2. **Criar configuração de debug** (`.vscode/launch.json`):

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Java Debug",
            "type": "java",
            "name": "Debug Current File",
            "request": "launch",
            "mainClass": "${file}",
            "preLaunchTask": "java: build"
        },
        {
            "name": "Debug Main Class",
            "type": "java",
            "name": "Debug Main",
            "request": "launch",
            "mainClass": "com.logistica.SistemaRastreamento",
            "preLaunchTask": "java: build"
        }
    ]
}
```

3. **Pressionar F5** ou `Run → Start Debugging` para iniciar em modo debug

#### Em linha de comando com `jdwp`

```bash
# Compilar e rodar com debug ativo
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=*:5005 -cp target/classes Main

# Depois conectar com IDE:
# Run → Debug Configurations → + (Add New) → Remote → localhost:5005
```

#### Maven com debug

```bash
mvn clean compile exec:java@debug
```

#### Gradle

```gradle
// build.gradle
task runDebug {
    doFirst {
        System.setProperty('debug', 'true')
    }
    classpath = sourceSets.main.runtimeClasspath
    main = 'com.logistica.Main'
    jvmArgs = ['-Xdebug', '-Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005']
}
```

Execute com: `gradle runDebug`

### Exemplo: aplicação de logística iniciando com debug

```java
package com.logistica;

import java.time.LocalDateTime;

public class SistemaRastreamento {
    private static final int PORTA_DEBUG = 5005;
    
    public static void main(String[] args) {
        System.out.println("Sistema de rastreamento iniciando em DEBUG...");
        System.out.println("Conectar debugger em localhost:" + PORTA_DEBUG);
        
        var gerenciadorEntregas = new GerenciadorEntregas();
        gerenciadorEntregas.processarEntrega("LOGIS-001", "Rua das Flores, 123");
    }
}

class GerenciadorEntregas {
    void processarEntrega(String codigo, String endereco) {
        System.out.println("[" + LocalDateTime.now() + "] Processando: " + codigo);
        // Breakpoint aqui
        var resultado = validarEndereco(endereco);
        System.out.println("Validação: " + resultado);
    }
    
    private boolean validarEndereco(String endereco) {
        return endereco != null && !endereco.isBlank();
    }
}
```

---

## 3. Breakpoints inteligentes

### Breakpoint simples

Clique no margin esquerdo da linha desejada. Ícone vermelho = ativo, cinza = desabilitado.

### Breakpoint condicional

```java
// Quero parar apenas quando o peso for > 100kg
// Clicar com Shift+Click no breakpoint para adicionar condição

public class ProcessadorCarga {
    void distribuirCarga(Entrega entrega) {
        if (entrega.getPesoKg() > 100) { // BREAKPOINT AQUI com condição: entrega.getPesoKg() > 100
            System.out.println("Carga pesada detectada");
            registrarAviso(entrega);
        }
    }
    
    private void registrarAviso(Entrega entrega) {
        // ...
    }
}
```

**Em VS Code:**
1. Clique com botão direito no breakpoint (ponto vermelho)
2. Selecione "Edit Breakpoint"
3. Adicione a expressão: `entrega.getPesoKg() > 100`
4. Pressione Enter para confirmar

### Breakpoint de exceção

```java
// Parar toda vez que Exception de um tipo específico for lançada

public class CarregadorPacotes {
    void carregarDeLote(List<String> codigos) {
        try {
            for (String codigo : codigos) {
                processarCodigo(codigo);  // Pode lançar EnderecoInvalidoException
            }
        } catch (EnderecoInvalidoException e) {
            // Debug: quer saber em qual código foi?
            // Configure breakpoint de exceção em EnderecoInvalidoException
            System.err.println("Erro ao processar lote: " + e.getMessage());
        }
    }
    
    private void processarCodigo(String codigo) throws EnderecoInvalidoException {
        if (codigo == null || codigo.isBlank()) {
            throw new EnderecoInvalidoException("Código vazio para entrega");
        }
    }
}

class EnderecoInvalidoException extends Exception {
    EnderecoInvalidoException(String mensagem) { super(mensagem); }
}
```

**Em VS Code:**
1. Na paleta de comandos (`Ctrl+Shift+P`), digite "Debug: Set Exception Breakpoints"
2. Escolha o tipo de exceção (all, uncaught, user-uncaught, etc.)
3. Ou adicione manualmente em `.vscode/launch.json`:

```json
{
    "name": "Java Debug com Exception Breakpoint",
    "type": "java",
    "request": "launch",
    "mainClass": "com.logistica.Main",
    "exceptionBreakpoints": ["java.lang.Exception"]
}
```

### Breakpoint com ação (log point)

```java
// Logar valor sem pausar execução

public class RotaOtimizada {
    void calcularMelhorRota(String origem, String destino, double distancia) {
        // LOGPOINT aqui: "DEBUG: Calculando rota de {origem} para {destino}, distância: {distancia}km"
        var tempo = distancia / 80.0; // 80 km/h média
        System.out.println("Tempo estimado: " + tempo + "h");
    }
}
```

**Em VS Code:**
1. Clique com botão direito no breakpoint
2. Selecione "Edit Breakpoint"
3. Mude de "Breakpoint" para "Logpoint"
4. Digite a mensagem: `"Rota: " + origem + " → " + destino + ", dist=" + distancia + "km"`
5. Pressione Enter

O log point executará e exibirá a mensagem no console sem pausar a execução.

---

## 4. Stack traces: como ler e interpretar

### Estrutura de um stack trace

```
java.lang.NullPointerException: Cannot read field "status" because "entrega" is null
    at com.logistica.ProcessadorStatus.atualizarStatus(ProcessadorStatus.java:42)
    at com.logistica.GerenciadorEntregas.confirmarEntrega(GerenciadorEntregas.java:105)
    at com.logistica.api.EntregaController.processar(EntregaController.java:58)
    at com.logistica.Main.main(Main.java:15)
```

**Leitura de baixo para cima:**

1. `Main.java:15` — ponto de entrada
2. `EntregaController.java:58` — requisição chegou aqui
3. `GerenciadorEntregas.java:105` — serviço processou
4. `ProcessadorStatus.java:42` — erro ocorreu aqui ← **FOCO**

**Análise:**
- Linha 42 em `ProcessadorStatus.java` tenta acessar `entrega.getStatus()`
- A variável `entrega` é `null`
- Quem passou `null`? Rastreie para trás (linha 105 em `GerenciadorEntregas`)

### Debugar usando stack trace

```java
public class ProcessadorStatus {
    void atualizarStatus(Entrega entrega, String novoStatus) {
        if (entrega == null) {  // ← ADICIONAR VALIDAÇÃO APÓS LER STACK TRACE
            throw new IllegalArgumentException("Entrega não pode ser nula");
        }
        entrega.setStatus(novoStatus);
    }
}
```

### Reproduzindo o erro com informação do stack trace

```java
public class TesteRecuperacaoErro {
    public static void main(String[] args) {
        // Stack trace disse que erro foi em linha 42 de ProcessadorStatus
        // Repoduzir:
        
        Entrega entregaNula = null;
        var processador = new ProcessadorStatus();
        
        try {
            processador.atualizarStatus(entregaNula, "ENTREGUE");  // Vai falhar aqui
        } catch (NullPointerException e) {
            e.printStackTrace();  // Exibe stack trace
            System.out.println("\n=== ANÁLISE ===");
            System.out.println("Causa: entrega é nula na chamada de atualizarStatus");
            System.out.println("Local: linha 42 de ProcessadorStatus.java");
        }
    }
}
```

---

## 5. Debugging com `System.out` vs logger

### ❌ Evitar (legacy)

```java
public class RoteadoPacotes {
    void rotear(Pacote pacote) {
        System.out.println("DEBUG: Pacote recebido: " + pacote.getCodigo());
        System.out.println("DEBUG: Peso: " + pacote.getPeso());
        System.out.println("DEBUG: Destino: " + pacote.getDestino());
        
        // Problema: fica poluído em produção, difícil filtrar, sem contexto
    }
}
```

### ✅ Usar logger (Java 17)

```java
// Adicionar dependência:
// <dependency>
//     <groupId>org.slf4j</groupId>
//     <artifactId>slf4j-api</artifactId>
//     <version>2.0.7</version>
// </dependency>
// <dependency>
//     <groupId>ch.qos.logback</groupId>
//     <artifactId>logback-classic</artifactId>
//     <version>1.4.8</version>
// </dependency>

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class RoteadorPacotes {
    private static final Logger logger = LoggerFactory.getLogger(RoteadorPacotes.class);
    
    void rotear(Pacote pacote) {
        logger.debug("Pacote recebido: código={}, peso={}kg, destino={}",
                pacote.getCodigo(), pacote.getPeso(), pacote.getDestino());
        
        logger.info("Iniciando roteamento para {}", pacote.getDestino());
        
        var rota = calcularRota(pacote);
        
        logger.info("Rota calculada: {} km em {} minutos", 
                rota.getDistancia(), rota.getTempoMinutos());
    }
    
    private Rota calcularRota(Pacote pacote) {
        logger.trace("Iniciando cálculo de rota com algoritmo A*");
        return new Rota(150.5, 90);
    }
}

class Pacote {
    private String codigo;
    private double peso;
    private String destino;
    
    Pacote(String codigo, double peso, String destino) {
        this.codigo = codigo;
        this.peso = peso;
        this.destino = destino;
    }
    
    String getCodigo() { return codigo; }
    double getPeso() { return peso; }
    String getDestino() { return destino; }
}

class Rota {
    private double distancia;
    private int tempoMinutos;
    
    Rota(double distancia, int tempoMinutos) {
        this.distancia = distancia;
        this.tempoMinutos = tempoMinutos;
    }
    
    double getDistancia() { return distancia; }
    int getTempoMinutos() { return tempoMinutos; }
}
```

### Configurar nível de log (`logback.xml`)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Desenvolvimento: DEBUG -->
    <property name="LOG_LEVEL" value="DEBUG"/>
    
    <!-- Produção: INFO -->
    <!-- <property name="LOG_LEVEL" value="INFO"/> -->
    
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/logistica.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/logistica-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    </appender>
    
    <root level="${LOG_LEVEL}">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

### Comparativo

| Aspecto | `System.out.println` | Logger |
|---|---|---|
| Nível de severidade | Não | Sim (TRACE, DEBUG, INFO, WARN, ERROR) |
| Controle em runtime | Não | Sim (mudar sem recompilar) |
| Formatação | Manual | Automática |
| Performance | Sincronado (lento) | Assincronado (rápido) |
| Produção | Ruim (poluído) | Perfeito (nivéis filtráveis) |

---

## 6. Variáveis locais e expressões watch

### Inspetar variáveis no painel Debug

Durante a pausa em um breakpoint, em VS Code, o painel à esquerda ("Variables") mostra:

```java
public class CalibradorFrete {
    double calcularFrete(Entrega entrega) {
        var peso = entrega.getPeso();          // Variável: peso = 25.5
        var distancia = 150.0;                 // Variável: distancia = 150.0
        var tarifa = 2.50;                     // Variável: tarifa = 2.50
        
        var frete = peso * tarifa * (distancia / 100);  // BREAKPOINT AQUI
        
        return frete;
    }
}
```

**Painel Variables exibe:**
```
peso: 25.5
distancia: 150.0
tarifa: 2.50
entrega: Entrega@5647 (...)
```

### Adicionar Watch Expression customizado

```java
public class VerificadorRota {
    void analisarRota(Entrega entrega, List<String> paradas) {
        var pontoPartida = "SP";
        var pontoChegada = entrega.getDestino();
        
        // BREAKPOINT AQUI
        var quantidadeParadas = paradas.size();
        var distanciaTotal = paradas.stream().mapToDouble(p -> 15.0).sum();
        
        System.out.println("Pronto");
    }
}
```

**Adicionar Watch (clique direito na seção "Watch" ou use o +):**
- `paradas.size()` → mostra quantidade
- `paradas.stream().filter(p -> p.startsWith("SP")).count()` → filtra e conta
- `entrega.getPeso() * 2.50` → cálculo customizado
- `paradas.toString()` → conversão para string

### Avaliação de expressão complexa (Debug Console)

```java
public class OtimizadorEntregas {
    void otimizar(List<Entrega> entregas) {
        // BREAKPOINT AQUI
        var total = 0.0;
        for (Entrega e : entregas) {
            total += e.getFrete();
        }
        System.out.println("Total: " + total);
    }
}
```

Durante pausa no breakpoint:
1. Abra o painel "Debug Console" na parte inferior
2. Digite: `entregas.stream().mapToDouble(Entrega::getFrete).sum()`
3. Pressione Enter → resultado em tempo real

---

## 7. Debugging de streams e optional

### Streams são "preguiçosos" — pegar cada etapa

```java
public class FiltroEntregas {
    List<Entrega> filtrarEntregasAtrasadas(List<Entrega> todas) {
        // SEM DEBUG: difícil saber onde perdeu dados
        return todas.stream()
                .filter(e -> e.getStatus().equals("EM_TRANSITO"))
                .filter(e -> e.getDataPrazo().isBefore(LocalDate.now()))
                .map(e -> {
                    e.setStatus("ATRASADA");
                    return e;
                })
                .toList();
    }
}
```

**Estratégia 1: breakpoint dentro de lambda**

```java
public class FiltroEntregasDebug {
    List<Entrega> filtrarAtrasadas(List<Entrega> todas) {
        return todas.stream()
                .filter(e -> {
                    // BREAKPOINT AQUI — vê cada elemento
                    boolean resultado = e.getStatus().equals("EM_TRANSITO");
                    System.out.println("Filtro 1: " + e.getCodigo() + " = " + resultado);
                    return resultado;
                })
                .filter(e -> {
                    // BREAKPOINT AQUI
                    boolean atrasada = e.getDataPrazo().isBefore(LocalDate.now());
                    System.out.println("Filtro 2: " + e.getCodigo() + " = " + atrasada);
                    return atrasada;
                })
                .toList();
    }
}
```

**Estratégia 2: logging com peek()**

```java
public class FiltroComLogging {
    List<Entrega> filtrar(List<Entrega> todas) {
        var logger = LoggerFactory.getLogger(getClass());
        
        return todas.stream()
                .peek(e -> logger.debug("Recebido: {}", e.getCodigo()))
                .filter(e -> e.getStatus().equals("EM_TRANSITO"))
                .peek(e -> logger.debug("Passou filtro 1: {}", e.getCodigo()))
                .filter(e -> e.getDataPrazo().isBefore(LocalDate.now()))
                .peek(e -> logger.debug("Passou filtro 2: {}", e.getCodigo()))
                .toList();
    }
}
```

**Saída esperada:**
```
Recebido: LOG-001
Passou filtro 1: LOG-001
Recebido: LOG-002
Recebido: LOG-003
Passou filtro 1: LOG-003
Passou filtro 2: LOG-003
```

### Debug de Optional

```java
public class BuscadorEndereco {
    Optional<String> buscarEndereco(String codigo) {
        var logger = LoggerFactory.getLogger(getClass());
        
        var resultado = buscarNaBase(codigo);  // Retorna Optional
        
        logger.debug("Busca para {}: presente={}", codigo, resultado.isPresent());
        
        // Debug with ifPresentOrElse (Java 9+)
        resultado
                .ifPresentOrElse(
                        endereco -> logger.info("Endereço encontrado: {}", endereco),
                        () -> logger.warn("Endereço não encontrado para: {}", codigo)
                );
        
        return resultado;
    }
    
    private Optional<String> buscarNaBase(String codigo) {
        if ("LOG-001".equals(codigo)) {
            return Optional.of("Rua das Flores, 123");
        }
        return Optional.empty();
    }
}
```

**Usar `map()` com log:**

```java
public class TransformaOptional {
    String obterEnderecoPronto(String codigo) {
        var logger = LoggerFactory.getLogger(getClass());
        
        return buscarEndereco(codigo)
                .peek(e -> logger.debug("Encontrado: {}", e))
                .map(String::toUpperCase)
                .peek(e -> logger.debug("Transformado: {}", e))
                .or(() -> {
                    logger.warn("Usando endereço default para: {}", codigo);
                    return Optional.of("ENDEREÇO NÃO DISPONÍVEL");
                })
                .get();
    }
    
    private Optional<String> buscarEndereco(String codigo) {
        return Optional.of("Avenida Brasil, 1000");
    }
}
```

---

## 8. Debugging de exceções customizadas

### Criar exceção com contexto

```java
public class ErroEntregaException extends Exception {
    private final String codigoEntrega;
    private final String motivo;
    private final LocalDateTime instante;
    
    public ErroEntregaException(String codigoEntrega, String motivo) {
        super(formatarMensagem(codigoEntrega, motivo));
        this.codigoEntrega = codigoEntrega;
        this.motivo = motivo;
        this.instante = LocalDateTime.now();
    }
    
    private static String formatarMensagem(String codigo, String motivo) {
        return String.format("[%s] Erro na entrega: %s", codigo, motivo);
    }
    
    public String getCodigoEntrega() { return codigoEntrega; }
    public String getMotivo() { return motivo; }
    public LocalDateTime getInstante() { return instante; }
}
```

### Capturar e debugar

```java
public class ValidadorEntrega {
    void processar(Entrega entrega) {
        try {
            validarCamposObrigatorios(entrega);
            validarEndereco(entrega);
            validarPeso(entrega);
            System.out.println("✓ Entrega validada com sucesso");
        } catch (ErroEntregaException e) {
            // BREAKPOINT AQUI — vê detalhes no painel Variables
            System.err.println("Erro capturado: " + e.getMotivo());
            System.err.println("Código: " + e.getCodigoEntrega());
            System.err.println("Instante: " + e.getInstante());
            
            // Para reproduzir em testes
            throw new RuntimeException("Falha ao processar: " + e.getMessage(), e);
        }
    }
    
    private void validarCamposObrigatorios(Entrega entrega) throws ErroEntregaException {
        if (entrega.getCodigo() == null || entrega.getCodigo().isBlank()) {
            throw new ErroEntregaException(
                    "DESCONHECIDO",
                    "Código de entrega não preenchido"
            );
        }
    }
    
    private void validarEndereco(Entrega entrega) throws ErroEntregaException {
        if (entrega.getDestino() == null || entrega.getDestino().isBlank()) {
            throw new ErroEntregaException(
                    entrega.getCodigo(),
                    "Endereço de destino vazio"
            );
        }
    }
    
    private void validarPeso(Entrega entrega) throws ErroEntregaException {
        if (entrega.getPeso() <= 0 || entrega.getPeso() > 1000) {
            throw new ErroEntregaException(
                    entrega.getCodigo(),
                    "Peso inválido: " + entrega.getPeso() + "kg"
            );
        }
    }
}

class Entrega {
    private String codigo;
    private String destino;
    private double peso;
    
    Entrega(String codigo, String destino, double peso) {
        this.codigo = codigo;
        this.destino = destino;
        this.peso = peso;
    }
    
    String getCodigo() { return codigo; }
    String getDestino() { return destino; }
    double getPeso() { return peso; }
}
```

### Usar Cause Chain para rastrear origem

```java
public class ProcessadorEntrega {
    void processar(Entrega entrega) {
        try {
            var roteiro = montarRoteiro(entrega);
            executarRota(roteiro);
        } catch (ErroEntregaException e) {
            if (e.getCause() != null) {
                // Debugar causa raiz
                System.out.println("Causa raiz: " + e.getCause().getClass().getSimpleName());
                System.out.println("Mensagem: " + e.getCause().getMessage());
            }
        }
    }
    
    private Object montarRoteiro(Entrega entrega) throws ErroEntregaException {
        try {
            return new Object();  // Simplificado
        } catch (Exception e) {
            throw new ErroEntregaException(
                    entrega.getCodigo(),
                    "Falha ao montar roteiro"
            ).initCause(e);  // Encadear causa
        }
    }
    
    private void executarRota(Object roteiro) {
        // ...
    }
}
```

---

## 9. Avaliação de expressões em runtime

### Evaluate Expression para cálculos ad-hoc

```java
public class CalculadoraCustos {
    void calcularFrete(Entrega entrega, List<String> paradas) {
        var valorBase = 50.0;
        var taxaPorKm = 2.5;
        var taxaPorParada = 15.0;
        
        // BREAKPOINT AQUI
        var frete = 0.0;
    }
}
```

Durante pausa no breakpoint, abra **Evaluate Expression** (`Alt+F8`):

**Expressão 1:**
```
valorBase + (paradas.size() * taxaPorParada)
```
Resultado: `110.0` (50 + 3 paradas * 15)

**Expressão 2:**
```
entrega.getPeso() * taxaPorKm
```
Resultado: `62.5` (25kg * 2.5)

**Expressão 3:**
```
new java.util.ArrayList<>(paradas).stream()
    .map(p -> p.length())
    .reduce(0, Integer::sum)
```
Resultado: `45` (soma dos comprimentos)

### Alterar valor em runtime

```java
public class AjustadorPeso {
    void ajustar(Entrega entrega) {
        var pesoOriginal = entrega.getPeso();  // 150kg
        var frete = pesoOriginal * 2.50;
        
        // BREAKPOINT AQUI
        System.out.println("Frete: " + frete);
    }
}
```

Pausa em breakpoint, e **Set Value**:
1. Clique com botão direito na variável `pesoOriginal` no painel Variables
2. Selecione "Set Value"
3. Digite: `50.0`
4. Pressione Enter
5. Continuar execução (F5) → frete será recalculado com novo peso

---

## 10. Debugging de padrão de patterns (switch com pattern matching)

### Switch com padrões (Java 17)

```java
public class ClassificadorCarga {
    String classificar(Object obj) {
        // Pattern matching em switch — pode ser confuso se não debugar certo
        return switch (obj) {
            case Entrega e when e.getPeso() > 100 -> "PESADA";
            case Entrega e -> "NORMAL";
            case String s -> "CÓDIGO: " + s;
            case Integer i -> "ID: " + i;
            default -> "DESCONHECIDO";
        };
    }
}

record Entrega(String codigo, double peso) {}
```

**Debugar padrões:**

```java
public class DebugClassificador {
    String classificarComDebug(Object obj) {
        var logger = LoggerFactory.getLogger(getClass());
        
        logger.debug("Objeto recebido: tipo={}, valor={}", 
                obj.getClass().getSimpleName(), obj);
        
        // Teste cada padrão
        if (obj instanceof Entrega e) {
            logger.debug("Matched Entrega: peso={}", e.peso());
            if (e.peso() > 100) {
                logger.debug("Padrão: PESADA");
                return "PESADA";
            }
            logger.debug("Padrão: NORMAL");
            return "NORMAL";
        }
        
        if (obj instanceof String s) {
            logger.debug("Matched String: {}", s);
            return "CÓDIGO: " + s;
        }
        
        logger.debug("Nenhum padrão correspondeu");
        return "DESCONHECIDO";
    }
}
```

**Breakpoint em cada branch:**

```java
public class DebugComBreakpoints {
    String classificarDebug(Object obj) {
        return switch (obj) {
            case Entrega e when e.peso() > 100 -> {
                // BREAKPOINT AQUI (when = true)
                yield "PESADA";
            }
            case Entrega e -> {
                // BREAKPOINT AQUI (fallback)
                yield "NORMAL";
            }
            case String s -> {
                // BREAKPOINT AQUI
                yield "CÓDIGO: " + s;
            }
            default -> {
                // BREAKPOINT AQUI
                yield "DESCONHECIDO";
            }
        };
    }
}

record Entrega(String codigo, double peso) {}
```

---

## 11. Debugging remoto em Java 17

### Cenário: aplicação rodando em servidor de staging

```bash
# No servidor de staging, iniciar com debug listener
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=0.0.0.0:5005 \
     -jar logistica-app-1.0.jar
```

**Parâmetros:**
- `transport=dt_socket` — protocolo debug
- `server=y` — servidor aguarda IDE conectar
- `suspend=n` — não pausa na inicialização (suspend=y pausaria)
- `address=0.0.0.0:5005` — escuta em todas as interfaces, porta 5005

### Conectar VS Code ao debugger remoto

**VS Code:**
1. Abra `.vscode/launch.json`
2. Adicione configuração para debug remoto:

```json
{
    "name": "Debug Remoto",
    "type": "java",
    "request": "attach",
    "hostName": "192.168.1.100",
    "port": 5005,
    "preLaunchTask": "java: build"
}
```

3. Na paleta de comandos (`Ctrl+Shift+P`), selecione "Debug: Select and Start Debugging"
4. Escolha "Debug Remoto"

VS Code conecta e para em breakpoints como se fosse local.

### Exemplo: debugar erro em staging

```java
public class ProcessadorRemoto {
    void processarLote(List<String> codigos) {
        for (String codigo : codigos) {
            try {
                buscarEProcessar(codigo);
            } catch (Exception e) {
                // BREAKPOINT AQUI — mesmo em production, vê a causa
                System.err.println("Erro ao processar: " + codigo);
                e.printStackTrace();
            }
        }
    }
    
    private void buscarEProcessar(String codigo) throws Exception {
        var entrega = buscarDaBase(codigo);
        if (entrega == null) {
            throw new IllegalArgumentException("Entrega não encontrada: " + codigo);
        }
        validar(entrega);
    }
    
    private Object buscarDaBase(String codigo) {
        // Simula busca
        return "LOG-" + codigo;
    }
    
    private void validar(Object entrega) {
        // ...
    }
}
```

### Segurança em debug remoto

⚠️ **CUIDADO:** Debug remoto sem firewall = acesso total ao servidor

**Proteções:**
1. Sempre usar `suspend=n` em produção (evita pausa indefinida)
2. Abrir porta 5005 apenas para seu IP
3. Considerar SSH tunnel:

```bash
# De seu computador:
ssh -L 5005:localhost:5005 usuario@servidor.logistica.com

# Depois conectar a localhost:5005 na IDE
```

---

## 12. Profiling e identificação de gargalos

### Profiling com VS Code e ferramentas CLI

VS Code não possui profiler integrado como IntelliJ, mas você pode usar ferramentas de linha de comando:

#### JProfiler ou YourKit (integração externa)

Para profiling avançado, configure em `.vscode/launch.json`:

```json
{
    "name": "Debug com Profiler",
    "type": "java",
    "request": "launch",
    "mainClass": "com.logistica.CalibradorRota",
    "vmArgs": ["-Xmx4g", "-XX:+PrintGCDetails", "-XX:+PrintGCDateStamps"]
}
```

#### Monitorar em tempo real com `jstat`

```bash
# Em outro terminal, enquanto a aplicação roda:
# Monitorar heap usage
jstat -gc -h10 $(jps | grep CalibradorRota | awk '{print $1}') 1000

# Resultado:
#  S0C    S1C    S0U    S1U      EC        EU       OC         OU       MC     MU    CCSC   CCSU
# 25600  25600   0.0  25600 205824.0  205824.0  512000.0  102400.0 4864.0  60.0  384.0   0.0
```

#### Exemplo: debugar com thread dump

```java
public class CalibradorRota {
    void executar(List<Entrega> entregas) {
        for (Entrega entrega : entregas) {
            processarRoteiro(entrega);
            calcularTempo(entrega);
            salvarNaBase(entrega);
        }
    }
    
    private void processarRoteiro(Entrega entrega) {
        try { Thread.sleep(50); } catch (InterruptedException e) {}  // Simula IO
    }
    
    private void calcularTempo(Entrega entrega) {
        try { Thread.sleep(100); } catch (InterruptedException e) {}  // Simula cálculo
    }
    
    private void salvarNaBase(Entrega entrega) {
        try { Thread.sleep(30); } catch (InterruptedException e) {}  // Simula BD
    }
}
```

**Monitorar execução em tempo real:**
1. Execute o programa em debug mode
2. Em outro terminal, capture thread dump com `jstack`:

```bash
jstack $(jps | grep CalibradorRota | awk '{print $1}') > dump.txt
```

3. Analise `dump.txt` para ver em quais métodos as threads estão presas

**Exemplo de resultado esperado:**

```
calcularTempo:     100ms (50%)
processarRoteiro:  50ms  (25%)
salvarNaBase:      30ms  (15%)
Outro:             20ms  (10%)
```

### Identificar memory leaks com `jmap`

```java
public class VerificadorVazamento {
    static List<byte[]> cache = new ArrayList<>();  // Vazamento! Cresce infinito
    
    public void adicionarAoCache(Entrega entrega) {
        cache.add(new byte[1024 * 1024]);  // Adiciona 1MB toda vez
    }
}
```

**Debugar com jmap (ferramenta CLI):**
1. Execute a aplicação
2. Em outro terminal, gere heap dump:

```bash
# Capturar heap dump
jmap -dump:live,format=b,file=heap.bin $(jps | grep Main | awk '{print $1}')

# Analisar com jhat
jhat heap.bin
# Acesse http://localhost:7000 no navegador
```

3. Se o heap crescer indefinidamente → vazamento

```java
// CORRIGIR: usar cache com limite
import java.util.LinkedHashMap;
import java.util.Map;

public class VerificadorCorrigido {
    private static final LinkedHashMap<String, byte[]> cache = 
        new LinkedHashMap<String, byte[]>(16, 0.75f, true) {
            protected boolean removeEldestEntry(Map.Entry eldest) {
                return size() > 100;  // Máximo 100 entradas
            }
        };
    
    public void adicionarAoCache(Entrega entrega) {
        cache.put(entrega.getCodigo(), new byte[1024 * 1024]);
    }
}
```

---

## 13. Armadilhas comuns e como evitá-las

### Armadilha 1: Variáveis otimizadas pelo compilador

```java
public class OtimizacaoCompilador {
    void exemplo() {
        var x = 10;
        var y = 20;
        var z = x + y;  // BREAKPOINT AQUI
        // z pode ser otimizado para 30 direto, não ver x e y
    }
}
```

**Solução:** Compilar com `-g` flag (debug info):

```bash
javac -g -d bin src/OtimizacaoCompilador.java
```

### Armadilha 2: Código inline e linha errada

```java
public class LinhaErrada {
    double calcular(double a, double b) {
        return a + b + a * b;  // BREAKPOINT AQUI — qual parte falhou?
    }
}
```

**Solução:** Separar em linhas:

```java
public class LinhaCerta {
    double calcular(double a, double b) {
        var soma = a + b;          // BREAKPOINT 1
        var multiplicacao = a * b; // BREAKPOINT 2
        return soma + multiplicacao; // BREAKPOINT 3
    }
}
```

### Armadilha 3: Concorrência e race conditions

```java
public class ComCorrida {
    static int contador = 0;
    
    public static void incrementar() {
        contador++;  // Não é atômico! Pode perder incrementos
    }
}
```

**Debug com breakpoint:**
1. Breakpoint em `contador++`
2. Rodar com múltiplas threads
3. Um thread pausa em breakpoint
4. Outro thread modifica `contador`
5. Primeiro thread retoma com valor inconsistente

**Solução:**

```java
import java.util.concurrent.atomic.AtomicInteger;

public class ComSeguranca {
    static AtomicInteger contador = new AtomicInteger(0);
    
    public static void incrementar() {
        contador.incrementAndGet();  // Thread-safe
    }
}
```

### Armadilha 4: Exceção em thread secundária

```java
public class ErroEmThread {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            throw new RuntimeException("Erro silencioso");  // Quem vai ver?
        });
        t.start();
        System.out.println("Principal continuou");
    }
}
```

**Não há stack trace visível!**

**Solução: set UncaughtExceptionHandler**

```java
public class TreatamentoSeguro {
    public static void main(String[] args) {
        Thread.setDefaultUncaughtExceptionHandler((thread, throwable) -> {
            System.err.println("❌ Exceção em thread " + thread.getName());
            throwable.printStackTrace();
        });
        
        Thread t = new Thread(() -> {
            throw new RuntimeException("Agora vai aparecer");
        });
        t.start();
    }
}
```

---

## 14. Cheat sheet de debug

### Atalhos VS Code

| Ação | Linux/Windows | Mac |
|---|---|---|
### Atalhos VS Code

| Ação | Linux/Windows | Mac |
|---|---|---|
| Debug | `F5` | `F5` |
| Resumir execução | `F5` ou `Ctrl+F5` | `F5` ou `Cmd+F5` |
| Step over | `F10` | `F10` |
| Step into | `F11` | `F11` |
| Step out | `Shift+Ctrl+F11` | `Shift+Cmd+F11` |
| Toggle breakpoint | `Ctrl+K Ctrl+B` | `Cmd+K Cmd+B` |
| Debug Console | `Ctrl+Shift+Y` | `Cmd+Shift+Y` |
| Add Watch | Clique em "+" no painel Watch | Clique em "+" no painel Watch |
| Edit breakpoint | Clique direito no breakpoint | Clique direito no breakpoint |

### Comandos Java úteis

```bash
# Ver versão
java -version

# Compilar com debug info
javac -g Main.java

# Rodar com debug listener
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 Main

# Ver properties da JVM
jinfo -flags $(pidof java)

# Ver heap usage
jmap -heap $(pidof java)

# Gerar thread dump
jstack $(pidof java) > threads.txt
```

### Exemplo completo: debug de logística

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.time.LocalDate;
import java.util.List;
import java.util.Optional;

public class SistemaLogisticaCompleto {
    private static final Logger logger = LoggerFactory.getLogger(SistemaLogisticaCompleto.class);
    
    public static void main(String[] args) {
        var sistema = new SistemaLogisticaCompleto();
        
        var entregas = List.of(
                new Entrega("LOG-001", "SP", 50.0, "EM_TRANSITO", LocalDate.now().minusDays(2)),
                new Entrega("LOG-002", "RJ", 150.0, "PENDENTE", LocalDate.now()),
                new Entrega("LOG-003", "MG", 75.0, "ENTREGUE", LocalDate.now().plusDays(1))
        );
        
        sistema.processarLote(entregas);
    }
    
    void processarLote(List<Entrega> entregas) {
        logger.info("Iniciando processamento de {} entregas", entregas.size());
        
        entregas.stream()
                .peek(e -> logger.debug("Processando: {}", e.codigo()))
                .filter(this::estaAtrasada)
                .peek(e -> logger.warn("Entrega atrasada: {}", e.codigo()))
                .forEach(this::escalacionar);
        
        logger.info("Processamento concluído");
    }
    
    boolean estaAtrasada(Entrega entrega) {
        var hoje = LocalDate.now();
        // BREAKPOINT AQUI para ver qual entrega falha
        return "EM_TRANSITO".equals(entrega.status()) && entrega.dataPrazo().isBefore(hoje);
    }
    
    void escalacionar(Entrega entrega) {
        logger.error("ESCALAÇÃO: {} está {} dias atrasada", 
                entrega.codigo(),
                java.time.temporal.ChronoUnit.DAYS.between(entrega.dataPrazo(), LocalDate.now()));
    }
    
    record Entrega(String codigo, String destino, double peso, String status, LocalDate dataPrazo) {}
}
```

Execute em debug:
1. Breakpoint em `estaAtrasada()`
2. Step through cada entrega
3. Observe qual falha a condição
4. Ajuste lógica se necessário

---

## Conclusão

Debug em Java 17 é **arte e ciência**: é preciso conhecer as ferramentas (breakpoints, loggers, profilers) e saber **quando usá-las**. 

Em logística, onde dados incorretos causam danos reais, debug é essencial para manter a confiabilidade. Use logs em produção, breakpoints em desenvolvimento, e profilers para otimizar.

**Regra de ouro:** Se o erro é estranho e você não consegue reproduzir, **log com debug info** e **breakpoint condicional**.
