# Data Model Spec — Weight Tracker

## Decisões de design

| Decisão           | Escolha                                                                                                         |
|-------------------|-----------------------------------------------------------------------------------------------------------------|
| Perfil do usuário | Colunas dentro de `users` (relação 1:1, sem join desnecessário)                                                 |
| IMC               | Calculado em tempo real no serviço: `weight_kg / (height_m)²`                                                   |
| Exclusão de peso  | Soft delete (`deleted_at`)                                                                                      |
| Refresh tokens    | Múltiplos por usuário (suporte multi-device)                                                                    |
| Exclusão de conta | Anonimização + soft delete em `users`, cascade soft delete em `weight_records`, hard delete em `refresh_tokens` |

---

## Entidades

### `users`
Armazena autenticação e perfil do usuário na mesma tabela.

| Coluna          | Tipo           | Nullable   | Descrição                                    |
|-----------------|----------------|------------|----------------------------------------------|
| `id`            | `BIGSERIAL`    | NOT NULL   | Chave primária                               |
| `name`          | `VARCHAR(100)` | NOT NULL   | Nome de exibição                             |
| `email`         | `VARCHAR(255)` | NOT NULL   | E-mail único para login                      |
| `password_hash` | `VARCHAR(255)` | NULL       | Hash bcrypt da senha. NULL após anonimização |
| `height_cm`     | `NUMERIC(5,2)` | NULL       | Altura em cm. Necessária para cálculo do IMC |
| `deleted_at`    | `TIMESTAMP`    | NULL       | Preenchido na deleção/anonimização da conta  |
| `created_at`    | `TIMESTAMP`    | NOT NULL   | Data de criação. Default: `NOW()`            |
| `updated_at`    | `TIMESTAMP`    | NOT NULL   | Última atualização. Default: `NOW()`         |

**Constraints:**
- `UNIQUE (email)` — e-mail é identificador único de login
- `CHECK (height_cm > 50 AND height_cm < 300)` — validação de range

**Regra de negócio:** Usuários com `deleted_at IS NOT NULL` devem ter login bloqueado, independente de terem senha válida.

---

### `weight_records`
Um registro por usuário por data. Soft delete para preservar histórico anonimizável.

| Coluna        | Tipo           | Nullable   | Descrição                     |
|---------------|----------------|------------|-------------------------------|
| `id`          | `BIGSERIAL`    | NOT NULL   | Chave primária                |
| `user_id`     | `BIGINT`       | NOT NULL   | FK para `users.id`            |
| `record_date` | `DATE`         | NOT NULL   | Data da aferição              |
| `record_time` | `TIME`         | NOT NULL   | Horário da aferição           |
| `weight_kg`   | `NUMERIC(5,2)` | NOT NULL   | Peso em kg. Ex: `70.50`       |
| `deleted_at`  | `TIMESTAMP`    | NULL       | Preenchido na exclusão lógica |
| `created_at`  | `TIMESTAMP`    | NOT NULL   | Default: `NOW()`              |
| `updated_at`  | `TIMESTAMP`    | NOT NULL   | Default: `NOW()`              |

**Constraints:**
- `UNIQUE (user_id, record_date) WHERE deleted_at IS NULL` — índice parcial: apenas um registro ativo por data por usuário
- `CHECK (weight_kg > 1 AND weight_kg < 500)` — validação de range
- `FOREIGN KEY (user_id) REFERENCES users(id)`

**Regra de negócio:** Se o usuário tentar criar um segundo registro para a mesma data, a API retorna `400` (tratado na camada de serviço antes de chegar ao banco).

---

### `refresh_tokens`
Múltiplos tokens por usuário para suportar multi-device. Hard delete no logout e na deleção de conta.

| Coluna       | Tipo           | Nullable   | Descrição                              |
|--------------|----------------|------------|----------------------------------------|
| `id`         | `BIGSERIAL`    | NOT NULL   | Chave primária                         |
| `user_id`    | `BIGINT`       | NOT NULL   | FK para `users.id`                     |
| `token_hash` | `VARCHAR(255)` | NOT NULL   | Hash SHA-256 do refresh token          |
| `expires_at` | `TIMESTAMP`    | NOT NULL   | Expiração: criação + 7 dias            |
| `revoked_at` | `TIMESTAMP`    | NULL       | Preenchido no logout desse dispositivo |
| `created_at` | `TIMESTAMP`    | NOT NULL   | Default: `NOW()`                       |

**Constraints:**
- `UNIQUE (token_hash)` — hash deve ser único globalmente
- `FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE`

**Regra de negócio:** Token é considerado válido apenas se `revoked_at IS NULL AND expires_at > NOW()`.

---

### `password_reset_tokens`
Tokens de uso único para o fluxo de redefinição de senha via e-mail. Hard delete após uso ou expiração.

| Coluna       | Tipo           | Nullable   | Descrição                                |
|--------------|----------------|------------|------------------------------------------|
| `id`         | `BIGSERIAL`    | NOT NULL   | Chave primária                           |
| `user_id`    | `BIGINT`       | NOT NULL   | FK para `users.id`                       |
| `token_hash` | `VARCHAR(255)` | NOT NULL   | Hash SHA-256 do token enviado por e-mail |
| `expires_at` | `TIMESTAMP`    | NOT NULL   | Expiração: criação + 30 minutos          |
| `used_at`    | `TIMESTAMP`    | NULL       | Preenchido quando o token é consumido    |
| `created_at` | `TIMESTAMP`    | NOT NULL   | Default: `NOW()`                         |

**Constraints:**
- `UNIQUE (token_hash)` — hash deve ser único globalmente
- `FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE`

**Regras de negócio:**
- Token válido apenas se `used_at IS NULL AND expires_at > NOW()`
- Ao solicitar nova redefinição, tokens anteriores do mesmo usuário são deletados
- Após uso bem-sucedido: `used_at` é preenchido e todos os `refresh_tokens` do usuário são deletados

---

## Schema SQL (PostgreSQL)

```sql
-- ============================================================
-- USERS
-- ============================================================
CREATE TABLE users (
    id           BIGSERIAL       PRIMARY KEY,
    name         VARCHAR(100)    NOT NULL,
    email        VARCHAR(255)    NOT NULL,
    password_hash VARCHAR(255)   NULL,
    height_cm    NUMERIC(5,2)    NULL,
    deleted_at   TIMESTAMP       NULL,
    created_at   TIMESTAMP       NOT NULL DEFAULT NOW(),
    updated_at   TIMESTAMP       NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_users_email    UNIQUE (email),
    CONSTRAINT chk_users_height  CHECK (height_cm IS NULL OR (height_cm > 50 AND height_cm < 300))
);

-- ============================================================
-- WEIGHT RECORDS
-- ============================================================
CREATE TABLE weight_records (
    id          BIGSERIAL     PRIMARY KEY,
    user_id     BIGINT        NOT NULL,
    record_date DATE          NOT NULL,
    record_time TIME          NOT NULL,
    weight_kg   NUMERIC(5,2)  NOT NULL,
    deleted_at  TIMESTAMP     NULL,
    created_at  TIMESTAMP     NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMP     NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_weight_records_user
        FOREIGN KEY (user_id) REFERENCES users(id),

    CONSTRAINT chk_weight_records_kg
        CHECK (weight_kg > 1 AND weight_kg < 500)
);

-- Índice parcial: garante unicidade apenas entre registros ativos
CREATE UNIQUE INDEX uq_weight_records_user_date
    ON weight_records (user_id, record_date)
    WHERE deleted_at IS NULL;

-- ============================================================
-- REFRESH TOKENS
-- ============================================================
CREATE TABLE refresh_tokens (
    id          BIGSERIAL     PRIMARY KEY,
    user_id     BIGINT        NOT NULL,
    token_hash  VARCHAR(255)  NOT NULL,
    expires_at  TIMESTAMP     NOT NULL,
    revoked_at  TIMESTAMP     NULL,
    created_at  TIMESTAMP     NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_refresh_tokens_hash  UNIQUE (token_hash),

    CONSTRAINT fk_refresh_tokens_user
        FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- ============================================================
-- PASSWORD RESET TOKENS
-- ============================================================
CREATE TABLE password_reset_tokens (
    id          BIGSERIAL     PRIMARY KEY,
    user_id     BIGINT        NOT NULL,
    token_hash  VARCHAR(255)  NOT NULL,
    expires_at  TIMESTAMP     NOT NULL,
    used_at     TIMESTAMP     NULL,
    created_at  TIMESTAMP     NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_password_reset_tokens_hash  UNIQUE (token_hash),

    CONSTRAINT fk_password_reset_tokens_user
        FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

---

## Índices de performance

```sql
-- Busca de usuário por email no login
CREATE INDEX idx_users_email         ON users (email) WHERE deleted_at IS NULL;

-- Listagem de registros por usuário/mês (query principal do dashboard)
CREATE INDEX idx_weight_records_user_date
    ON weight_records (user_id, record_date)
    WHERE deleted_at IS NULL;

-- Validação de refresh token na renovação de sessão
CREATE INDEX idx_refresh_tokens_hash ON refresh_tokens (token_hash);

-- Limpeza periódica de tokens expirados
CREATE INDEX idx_refresh_tokens_expires ON refresh_tokens (expires_at);

-- Busca do token de redefinição de senha
CREATE INDEX idx_password_reset_tokens_hash    ON password_reset_tokens (token_hash);
CREATE INDEX idx_password_reset_tokens_user    ON password_reset_tokens (user_id);
CREATE INDEX idx_password_reset_tokens_expires ON password_reset_tokens (expires_at);
```

---

## Estratégia de anonimização (deleção de conta)

Executada em uma única transação:

```sql
-- 1. Anonimizar dados pessoais do usuário
UPDATE users SET
    name          = 'Usuário Removido',
    email         = 'deleted_' || encode(sha256(email::bytea), 'hex') || '@removed',
    password_hash = NULL,
    height_cm     = NULL,
    deleted_at    = NOW(),
    updated_at    = NOW()
WHERE id = :userId;

-- 2. Soft delete em cascata nos registros de peso
UPDATE weight_records SET
    deleted_at = NOW()
WHERE user_id = :userId
  AND deleted_at IS NULL;

-- 3. Hard delete nos refresh tokens (encerrar todas as sessões)
DELETE FROM refresh_tokens WHERE user_id = :userId;
```

---

## Regras de negócio (Business Rules)

Estas regras são aplicadas na camada de serviço (Spring Boot), não no banco:

### Autenticação
- Senha deve ter no mínimo 8 caracteres, ao menos 1 letra maiúscula, 1 número e 1 caractere especial
- Senha armazenada exclusivamente como hash bcrypt (custo mínimo: 10)
- Login bloqueado se `users.deleted_at IS NOT NULL`
- Refresh token válido somente se `revoked_at IS NULL AND expires_at > NOW()`
- Access token JWT expira em **1 hora**
- Refresh token expira em **7 dias**

### Registros de peso
- Apenas 1 registro ativo por usuário por data
- Tentativa de criar segundo registro para a mesma data retorna HTTP 400
- Edição e exclusão permitidas apenas para registros pertencentes ao usuário autenticado
- Peso armazenado com 2 casas decimais (`NUMERIC(5,2)`)

### IMC
- Calculado em tempo real: `weight_kg / (height_m)²`
- Se `height_cm IS NULL`, retornar `bmi: null` e `bmiCategory: null` na resposta
- Categorias OMS:
  | IMC | Categoria |
  |---|---|
  | < 18.5 | `ABAIXO_DO_PESO` |
  | 18.5 – 24.9 | `PESO_NORMAL` |
  | 25.0 – 29.9 | `SOBREPESO` |
  | 30.0 – 34.9 | `OBESIDADE_GRAU_1` |
  | 35.0 – 39.9 | `OBESIDADE_GRAU_2` |
  | ≥ 40.0 | `OBESIDADE_GRAU_3` |

### Perfil
- `height_cm` é opcional no cadastro, pode ser preenchido posteriormente
- Atualizar a altura não recalcula IMCs históricos (calculados sempre em tempo real)
