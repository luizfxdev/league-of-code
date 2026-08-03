# Operadores em PostgreSQL ? LoL Edition

## Exemplo Prático: Database de Game Server

```sql
-- Tabelas do jogo
CREATE TABLE champions (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,            -- "Garen", "Lux"
    health_base FLOAT NOT NULL,            -- HP base do campeão
    mana_base FLOAT NOT NULL,              -- Mana base
    attack_damage FLOAT NOT NULL,          -- AD base
    armor_resist FLOAT NOT NULL,           -- Armor base
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE players (
    id SERIAL PRIMARY KEY,
    summoner_name VARCHAR(50) NOT NULL UNIQUE,
    champion_id INT REFERENCES champions(id),
    level INT DEFAULT 1,                   -- 1-30
    experience INT DEFAULT 0,              -- XP acumulada
    gold FLOAT DEFAULT 0,                  -- Gold no jogo
    health_current FLOAT,
    mana_current FLOAT,
    is_alive BOOLEAN DEFAULT true,
    is_stunned BOOLEAN DEFAULT false,      -- CC aplicado?
    last_action_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE matches (
    id SERIAL PRIMARY KEY,
    player_id INT REFERENCES players(id),
    kills INT DEFAULT 0,
    deaths INT DEFAULT 0,
    assists INT DEFAULT 1,
    damage_dealt FLOAT DEFAULT 0,
    damage_taken FLOAT DEFAULT 0,
    gold_earned FLOAT DEFAULT 0,
    match_duration INT,                    -- em segundos
    match_date TIMESTAMP DEFAULT NOW()
);

-- Inserts de exemplo
INSERT INTO champions (nome, health_base, mana_base, attack_damage, armor_resist)
VALUES 
    ('Garen', 590, 0, 80, 40),
    ('Lux', 520, 380, 50, 30),
    ('Ahri', 510, 340, 53, 24);

INSERT INTO players (summoner_name, champion_id, level, experience, gold, health_current, mana_current, is_alive)
VALUES 
    ('ProGamer123', 1, 9, 8500, 2500, 590, 0, true),
    ('MidLaneKing', 2, 8, 7200, 1800, 380, 280, true);
```

---

## Operadores Comparativos em SQL

| Operador | Significado | LoL | Exemplo |
|----------|------------|-----|---------|
| `=` | Igual a | Mesmo nível | `WHERE level = 6` |
| `!=` ou `<>` | Diferente de | Campeões diferentes | `WHERE champion_id != 1` |
| `>` | Maior que | Mais HP | `WHERE health_current > 100` |
| `<` | Menor que | Menos ouro | `WHERE gold < 1300` |
| `>=` | Maior ou igual | HP >= necessário | `WHERE level >= 6` |
| `<=` | Menor ou igual | Mana <= cap | `WHERE experience <= 10000` |
| `BETWEEN` | Intervalo | Range de nível | `WHERE level BETWEEN 5 AND 9` |
| `IN` | Um dos valores | Múltiplos status | `WHERE champion_id IN (1, 2, 3)` |
| `LIKE` | Padrão de texto | Buscar nome | `WHERE summoner_name LIKE '%King%'` |
| `IS NULL` | Nulo/Vazio | Sem valor | `WHERE champion_id IS NULL` |

### Exemplos Práticos:

```sql
-- Players com level >= 6 (ulti destravada)
SELECT summoner_name, champion_id, level
FROM players
WHERE level >= 6;

-- Players que estão vivos
SELECT summoner_name, health_current
FROM players
WHERE is_alive = true;

-- Players em estado específico
SELECT summoner_name
FROM players
WHERE champion_id IN (1, 2)  -- Garen ou Lux
AND level BETWEEN 5 AND 10;

-- Matches onde o dano foi alto
SELECT summoner_name, damage_dealt
FROM players p
JOIN matches m ON p.id = m.player_id
WHERE m.damage_dealt > 1000;

-- Players com nome contendo "King"
SELECT summoner_name, champion_id
FROM players
WHERE summoner_name LIKE '%King%';
```

---

## Operadores Lógicos em SQL

| Operador | Significado | LoL | Quando Usar |
|----------|------------|-----|------------|
| `AND` | E (AND) | **Ambos** têm que ser verdadeiro | Múltiplas condições |
| `OR` | OU (OR) | **Um ou outro** vale | Alternativas |
| `NOT` | NÃO (NOT) | Inverte condição | Negar |

### AND ? Validações em Série

```sql
-- Players vivos AND com nível alto AND Com ouro suficiente
SELECT summoner_name, level, gold
FROM players
WHERE is_alive = true 
AND level >= 6
AND gold >= 1300
AND is_stunned = false;
-- Tudo tem que ser TRUE para aparecer na resultado
```

**Analogia:** Você precisa estar vivo **E** ter nível 6 **E** ter ouro **E** não estar stunado. Se um falhar, não aparece.

```sql
-- Exemplo: Players elegíveis para teamfight
SELECT p.summoner_name, c.nome AS champion, p.level, p.health_current, p.mana_current
FROM players p
JOIN champions c ON p.champion_id = c.id
WHERE p.is_alive = true                    -- Vivo
AND p.level >= 5                           -- Level suficiente
AND p.health_current > (c.health_base * 0.3)  -- Com HP razoável
AND p.mana_current > 50                    -- Com mana
AND p.is_stunned = false;                  -- Sem CC
```

### OR ? Alternativas Aceitáveis

```sql
-- Qualquer player que seja Garen OU Lux OU Ahri
SELECT summoner_name, champion_id
FROM players
WHERE champion_id = 1  -- Garen
OR champion_id = 2     -- Lux
OR champion_id = 3;    -- Ahri
-- Se UMA condição for TRUE, aparece no resultado
```

**Analogia:** Você ganha se destrói Nexo **OU** mata Baron **OU** destrui todas towers.

```sql
-- Players que entraram em matches RECENTEMENTE ou têm kills altas
SELECT p.summoner_name, m.kills, m.match_date
FROM players p
LEFT JOIN matches m ON p.id = m.player_id
WHERE m.match_date >= NOW() - INTERVAL '1 hour'  -- Última hora
OR m.kills >= 5;                                   -- Ou com 5+ kills
```

### NOT ? Negação

```sql
-- Players que NÃO estão stunados
SELECT summoner_name
FROM players
WHERE NOT is_stunned;
-- Equivalente a: WHERE is_stunned = false

-- Players que não são Garen
SELECT summoner_name, champion_id
FROM players
WHERE NOT champion_id = 1;
-- Equivalente a: WHERE champion_id != 1
```

**Analogia:** É o oposto. Se `is_stunned` é true, `NOT is_stunned` é false. Se é false, `NOT` vira true.

---

## JOINs ? Combinação de Tabelas (Operador de Relacionamento)

| Tipo | Significado | LoL | Uso |
|------|------------|-----|-----|
| `INNER JOIN` | Ambas existem | Jogador + Campeão | Dados que existem nos dois lados |
| `LEFT JOIN` | Mantém esquerda | Todos players (com/sem match) | Todos do lado esquerdo |
| `RIGHT JOIN` | Mantém direita | Todos matches (com/sem player) | Todos do lado direito |
| `FULL OUTER JOIN` | Ambos lados | Tudo que existe | Combinação completa |

### INNER JOIN ? "E ambos existem"

```sql
-- Players E seus campeões (só quem tem campeão atribuído)
SELECT p.summoner_name, c.nome, p.level
FROM players p
INNER JOIN champions c ON p.champion_id = c.id;

-- Resultado: só players que têm champion_id preenchido
```

**Analogia:** Só mostra jogadores que têm campeão selecionado. Se `champion_id` é NULL, não aparece.

### LEFT JOIN ? "Jogadores + seus matches (mesmo sem match)"

```sql
-- Todos os players, mas com dados de matches se existirem
SELECT p.summoner_name, m.kills, m.deaths, m.damage_dealt
FROM players p
LEFT JOIN matches m ON p.id = m.player_id
ORDER BY m.match_date DESC;

-- Resultado: todos os players, mas se não tem match, as colunas de match ficam NULL
```

**Analogia:** Mostra todos os jogadores. Se não tem partida registrada, as colunas de match aparecem em branco.

---

## Operadores Agregados ? Resumir Dados

| Operador | Significado | LoL | Exemplo |
|----------|------------|-----|---------|
| `COUNT()` | Contar linhas | Quantos players | `COUNT(*)` |
| `SUM()` | Somar valores | Total de kills | `SUM(kills)` |
| `AVG()` | Média | KDA médio | `AVG(kills)` |
| `MAX()` | Máximo | Maior HP | `MAX(health_current)` |
| `MIN()` | Mínimo | Menor ouro | `MIN(gold)` |
| `GROUP_CONCAT()` | Concatenar (MySQL) | Juntar nomes | `STRING_AGG()` em PostgreSQL |

### Exemplos Práticos:

```sql
-- Quantos players estão vivos agora?
SELECT COUNT(*) AS players_vivos
FROM players
WHERE is_alive = true;
-- Resultado: 2

-- KDA total por player
SELECT 
    p.summoner_name,
    SUM(m.kills) AS kills_total,
    SUM(m.deaths) AS deaths_total,
    SUM(m.assists) AS assists_total
FROM players p
LEFT JOIN matches m ON p.id = m.player_id
GROUP BY p.id, p.summoner_name
ORDER BY kills_total DESC;

-- Dano médio por campeão
SELECT 
    c.nome,
    COUNT(m.id) AS matches_played,
    AVG(m.damage_dealt) AS damage_medio,
    MAX(m.damage_dealt) AS damage_maximo
FROM champions c
LEFT JOIN players p ON c.id = p.champion_id
LEFT JOIN matches m ON p.id = m.player_id
GROUP BY c.id, c.nome
HAVING COUNT(m.id) > 0
ORDER BY damage_medio DESC;

-- Campeão mais jogado
SELECT 
    c.nome,
    COUNT(p.id) AS times_picked
FROM champions c
LEFT JOIN players p ON c.id = p.champion_id
GROUP BY c.id, c.nome
ORDER BY times_picked DESC
LIMIT 1;
```

---

## WHERE vs HAVING ? Diferença Crítica

| Comando | Usa | Quando | Exemplo |
|---------|-----|--------|---------|
| `WHERE` | Filtro simples | Antes de agrupar | `WHERE level > 5` |
| `HAVING` | Filtro agregado | Depois de GROUP BY | `HAVING SUM(kills) > 10` |

```sql
-- ? Errado: não pode usar agregado em WHERE
SELECT champion_id, SUM(kills)
FROM matches m
JOIN players p ON m.player_id = p.id
WHERE SUM(kills) > 5       -- ? ERRO!
GROUP BY champion_id;

-- ? Certo: usar HAVING para filtrar agregados
SELECT champion_id, SUM(kills) AS total_kills
FROM matches m
JOIN players p ON m.player_id = p.id
GROUP BY champion_id
HAVING SUM(kills) > 5;     -- ? Correto!

-- ? Também certo: WHERE antes de GROUP BY
SELECT champion_id, SUM(kills) AS total_kills
FROM matches m
JOIN players p ON m.player_id = p.id
WHERE level >= 5           -- ? Filtra antes de agrupar
GROUP BY champion_id;
```

---

## Boas Práticas ? Query Otimizada

```sql
-- ? Ruim: sem índices, sem constraints
SELECT p.summoner_name, c.nome, m.kills
FROM players p
LEFT JOIN champions c ON p.champion_id = c.id
LEFT JOIN matches m ON p.id = m.player_id
WHERE p.level > 5;
-- Pode ser LENTO se tabelas crescerem

-- ? Bom: com índices, com validação
CREATE INDEX idx_players_level ON players(level);
CREATE INDEX idx_matches_player ON matches(player_id);

SELECT p.summoner_name, c.nome, COUNT(m.id) as matches
FROM players p
INNER JOIN champions c ON p.champion_id = c.id
LEFT JOIN matches m ON p.id = m.player_id
WHERE p.level >= 6              -- Guard: nível válido
AND p.is_alive = true           -- Guard: vivo
AND m.match_date >= NOW() - INTERVAL '7 days'  -- Guard: semana recente
GROUP BY p.id, p.summoner_name, c.nome
HAVING COUNT(m.id) > 0          -- Guard: tem ao menos 1 match
ORDER BY matches DESC
LIMIT 10;
```

**Analogia:** Guard clauses em SQL = índices + WHERE + HAVING. Rejeita dados inúteis cedo.

---

## Case Expression ? Condicional em SELECT

```sql
-- Classificar players por performance
SELECT 
    p.summoner_name,
    CASE 
        WHEN p.level >= 15 THEN 'Pro'
        WHEN p.level >= 10 THEN 'Intermediate'
        WHEN p.level >= 5 THEN 'Beginner'
        ELSE 'Noob'
    END AS skill_tier,
    p.level
FROM players p
ORDER BY p.level DESC;

-- Resultado:
-- ProGamer123    | Pro           | 9      (não é realmente "Pro", mas está em 9)
-- MidLaneKing    | Intermediate  | 8
```

**Analogia:** CASE é como um if/else/else dentro do SELECT. Classifica resultado dinamicamente.

---

## Resumo: Quando Usar O Quê

| Situação | Operador | Exemplo |
|----------|----------|---------|
| Filtrar por um valor | `=` | `WHERE level = 6` |
| Filtrar por intervalo | `BETWEEN` | `WHERE level BETWEEN 5 AND 10` |
| Múltiplos valores específicos | `IN` | `WHERE champion_id IN (1, 2, 3)` |
| Padrão de texto | `LIKE` | `WHERE name LIKE '%King%'` |
| Múltiplas condições (todas true) | `AND` | `WHERE vivo AND level >= 6 AND ouro > 1000` |
| Múltiplas condições (uma true) | `OR` | `WHERE status = 'A' OR status = 'B'` |
| Negar condição | `NOT` ou `!=` | `WHERE NOT stunned` |
| Combinar tabelas (dados que existem em ambas) | `INNER JOIN` | Jogador + Campeão |
| Manter todos do lado esquerdo | `LEFT JOIN` | Todos players (com/sem match) |
| Resumir dados (count, sum, avg) | `GROUP BY + HAVING` | Kills por campeão |
| Condicional em resultado | `CASE` | Classificar por tier |