# Data-Hora em Java - Conceitos Fundamentais

## 1. Tipos de Data-Hora

### Local (sem fuso horário)
- **LocalDate**: apenas data (ano-mês-dia)
- **LocalDateTime**: data + hora, sem timezone
- Uso: sistemas de região única, dados não sensíveis a timezone

### Global (com fuso horário)
- **Instant**: representação UTC de um momento específico
- Uso: sistemas multi-região, APIs, bancos de dados

### Duração
- **Duration**: tempo decorrido entre dois instantes
- **ChronoUnit**: unidades de tempo (dias, horas, etc)

---

## 2. Timezone (Fuso Horário)

### Conceitos
- **UTC/GMT**: padrão universal (Greenwich Mean Time)
- **Horários relativos ao UTC**: São Paulo é UTC-3, Portugal é UTC+1
- **Representação em Java**: `ZoneId` (ex: `"America/Sao_Paulo"`)

### Conversão
```
Dado global 2022-07-23T14:30:00Z (UTC)
  ? Brasil (UTC-3)
Exibição local: 23/07/22 11:30
```

---

## 3. Padrão ISO 8601

### Data-hora local
```
2022-07-21                    // apenas data
2022-07-21T14:52              // data + hora
2022-07-22T14:52:09           // data + hora + segundo
2022-07-22T14:52:09.4073      // com milissegundos
```

### Data-hora global
```
2022-07-23T14:52:09Z          // UTC (Z = Zulu time)
2022-07-23T14:52:09-03:00     // com offset explícito
```

---

## 4. Boas Práticas

### ? Armazene UTC, Mostre Local
- **Banco/API**: sempre salve em UTC
- **Exibição**: converta para timezone do usuário no frontend/cliente
- Evita inconsistências e facilita operações multi-região

### Quando usar cada tipo
| Tipo | Quando |
|------|--------|
| LocalDate/DateTime | Data de nascimento, hora local de uma venda regional |
| Instant | Timestamps de API, auditoria, registros globais |
| Duration | Diferenças entre eventos, intervalos de tempo |

---

## 5. Operações Comuns

### Instanciação
```java
Instant.now()                                    // agora em UTC
LocalDateTime.now()                             // agora local
LocalDate.of(2022, 7, 23)                      // data específica
Instant.parse("2022-07-23T14:30:00Z")          // parse ISO 8601
```

### Formatação
```java
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");
String formatted = LocalDateTime.now().format(fmt);
```

### Cálculos
```java
LocalDateTime data = LocalDateTime.now();
LocalDateTime futuro = data.plusDays(5).minusHours(2);

Instant i1 = Instant.now();
Instant i2 = i1.plus(Duration.ofHours(3));
Duration duracao = Duration.between(i1, i2);
```

### Conversão entre tipos
```java
Instant inst = Instant.now();
ZoneId z = ZoneId.of("America/Sao_Paulo");
LocalDateTime local = LocalDateTime.ofInstant(inst, z);
```

---

## 6. Legado (Java < 8)

### Date (evitar em código novo)
```java
Date d = new Date();                           // agora
SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");
String str = sdf.format(d);
Date parsed = sdf.parse("23/07/2022");
```

### Calendar (evitar em código novo)
```java
Calendar cal = Calendar.getInstance();
cal.setTime(date);
cal.add(Calendar.HOUR_OF_DAY, 4);
int mes = 1 + cal.get(Calendar.MONTH);         // mês começa em 0
```

?? **Date e Calendar são mutáveis e não thread-safe. Use Java 8+ (LocalDateTime, Instant).**

---

## 7. Checklist de Implementação

- [ ] Banco de dados: usar tipo TIMESTAMP WITH TIMEZONE ou arquivar em UTC
- [ ] API: retornar Instant ou string ISO 8601
- [ ] Frontend: receber UTC e converter com `new Date(iso)` ou library (Day.js, etc)
- [ ] Exibição: mostrar timezone junto com a hora para o usuário
- [ ] Testes: cobrir DST (horário de verão) se aplicável