# Business Rules Spec — Weight Tracker

## BR-01 — Registros de peso

### BR-01.1 — Apenas o dia atual
- O usuário só pode criar registro para a **data de hoje** (data do servidor)
- A data **não é informada** pelo cliente — é preenchida automaticamente pelo backend
- Registros para datas passadas ou futuras são rejeitados com HTTP `400`

### BR-01.2 — Um registro ativo por dia
- Cada usuário pode ter no máximo **1 registro ativo por data**
- Tentativa de criar segundo registro para a mesma data retorna HTTP `400`
- Registros com `deleted_at IS NOT NULL` não contam para esse limite
  (permitindo recriar um registro após exclusão)

### BR-01.3 — Edição sem restrição de data
- O usuário pode editar **qualquer registro ativo** de qualquer data, sem limite de tempo
- Apenas os campos `recordTime` e `weightKg` são editáveis
- A data do registro (`record_date`) **não pode ser alterada** após a criação

### BR-01.4 — Campos do formulário de novo registro
- O cliente envia apenas: `recordTime` e `weightKg`
- O campo `recordDate` é preenchido pelo servidor com `LocalDate.now()`

---

## BR-02 — Dashboard e histórico

### BR-02.1 — Empty state
- Quando não há registros ativos no mês selecionado, a API retorna `MonthSummary`
  com `totalRecords: 0` e todos os campos numéricos como `null`
- O frontend exibe mensagem clara: _"Nenhum peso registrado neste mês"_

### BR-02.2 — Filtro de período permitido
- O filtro de ano/mês aceita apenas períodos entre:
  - **Mínimo:** mês/ano de criação da conta do usuário (`users.created_at`)
  - **Máximo:** mês/ano atual no momento do acesso
- Requisições fora desse intervalo retornam HTTP `400`

### BR-02.3 — Período padrão ao abrir o app
- O app abre sempre exibindo o **mês atual**
- O seletor de período exibe apenas opções dentro do intervalo permitido (BR-02.2)

### BR-02.4 — Meses futuros
- O filtro **não permite** selecionar meses futuros
- O mês máximo disponível é sempre o mês corrente

---

## BR-03 — IMC

### BR-03.1 — Cálculo em tempo real
- IMC é calculado na camada de serviço, **não armazenado no banco**
- Fórmula: `IMC = weightKg / (heightM)²`
- IMC é recalculado a cada requisição usando a altura atual do perfil

### BR-03.2 — Altura não cadastrada
- Se `users.height_cm IS NULL`, os campos `bmi` e `bmiCategory` retornam `null`
- O frontend exibe aviso: _"Cadastre sua altura para calcular o IMC"_
  com link direto para a tela de perfil

### BR-03.3 — Atualização da altura
- Ao cadastrar ou atualizar a altura, o IMC de **todos os registros exibidos**
  é recalculado imediatamente (pois é calculado em tempo real — sem operação adicional)

### BR-03.4 — Categorias OMS
| IMC         | Categoria (`bmiCategory`)  |
|-------------|----------------------------|
| < 18.5      | `ABAIXO_DO_PESO`           |
| 18.5 – 24.9 | `PESO_NORMAL`              |
| 25.0 – 29.9 | `SOBREPESO`                |
| 30.0 – 34.9 | `OBESIDADE_GRAU_1`         |
| 35.0 – 39.9 | `OBESIDADE_GRAU_2`         |
| ≥ 40.0      | `OBESIDADE_GRAU_3`         |

---

## BR-04 — Autenticação e sessões

### BR-04.1 — Política de senha
Aplicada no cadastro, alteração e redefinição de senha:
- Mínimo de **8 caracteres**
- Ao menos **1 letra maiúscula**
- Ao menos **1 número**
- Ao menos **1 caractere especial** (ex: `@`, `#`, `!`, `$`, `%`)
- Máximo de **64 caracteres**
- Validada na camada de serviço; nunca no banco

### BR-04.2 — Armazenamento de senha
- Senha armazenada exclusivamente como hash **bcrypt** com custo mínimo **10**
- Jamais armazenada em texto plano ou hash reversível

### BR-04.3 — Validade dos tokens
| Token                | Validade   |
|----------------------|------------|
| Access Token (JWT)   | 1 hora     |
| Refresh Token        | 7 dias     |
| Password Reset Token | 30 minutos |

### BR-04.4 — Logout
- Revoga apenas o `refresh_token` do **dispositivo atual**
- Outras sessões permanecem ativas

### BR-04.5 — Login bloqueado
- Usuários com `users.deleted_at IS NOT NULL` não podem autenticar
- Retorna HTTP `401` (sem diferenciar da mensagem de credenciais inválidas,
  por segurança)

---

## BR-05 — Redefinição de senha (esqueci minha senha)

### BR-05.1 — Fluxo
1. Usuário informa o e-mail em `POST /api/auth/forgot-password`
2. Se o e-mail existir: gera token seguro, armazena hash em `password_reset_tokens`, envia e-mail via Mailtrap
3. E-mail contém link com o token em texto claro: `http://localhost:5173/reset-password?token=<TOKEN>`
4. Usuário acessa o link → tela de nova senha → `POST /api/auth/reset-password`
5. Backend valida o token (hash + `used_at IS NULL` + `expires_at > NOW()`)
6. Senha atualizada → `used_at` preenchido → todos os `refresh_tokens` do usuário deletados

### BR-05.2 — Segurança
- A resposta de `forgot-password` é sempre HTTP `204`, **independente** de o e-mail existir
  (evita enumeração de usuários)
- O token enviado por e-mail é armazenado apenas como hash SHA-256 no banco
- Ao solicitar nova redefinição, tokens anteriores do mesmo usuário são **deletados**
- Token é de uso único: segunda tentativa com o mesmo token retorna HTTP `400`

---

## BR-06 — Alteração de senha (usuário autenticado)

### BR-06.1 — Fluxo
1. Usuário autenticado envia senha atual + nova senha em `PUT /api/auth/change-password`
2. Backend valida a senha atual via bcrypt
3. Nova senha validada pelas regras de BR-04.1
4. Senha atualizada → todos os `refresh_tokens` **exceto o da sessão atual** são revogados

### BR-06.2 — Erro
- Senha atual incorreta → HTTP `400` com mensagem genérica (não confirmar se a senha é "quase certa")

---

## BR-07 — Alteração de e-mail (usuário autenticado)

### BR-07.1 — Fluxo
1. Usuário autenticado envia novo e-mail + senha atual em `PUT /api/auth/change-email`
2. Backend valida a senha atual via bcrypt
3. Verifica se o novo e-mail já está em uso → HTTP `409` se sim
4. E-mail atualizado → todos os `refresh_tokens` **exceto o da sessão atual** são revogados
5. Confirmação no novo e-mail **não é exigida no MVP**

### BR-07.2 — Restrições
- Novo e-mail não pode ser igual ao e-mail atual → HTTP `400`
- Novo e-mail não pode estar em uso por outra conta → HTTP `409`

---

## BR-08 — Perfil

### BR-08.1 — Campos obrigatórios
- `name` e `email` são obrigatórios e não podem ser deixados em branco
- `height_cm` é opcional: pode ser preenchido no cadastro ou posteriormente

### BR-08.2 — Campos editáveis via `PUT /api/profile`
- `name` e `height_cm` apenas
- `email` é alterado exclusivamente via `PUT /api/auth/change-email`
- `password` é alterada exclusivamente via `PUT /api/auth/change-password`

---

## BR-09 — Exclusão de conta

### BR-09.1 — Anonimização (transação única)
1. `users`: `name → "Usuário Removido"`, `email → "deleted_<hash>@removed"`, `password_hash → NULL`, `height_cm → NULL`, `deleted_at → NOW()`
2. `weight_records`: soft delete em cascata (`deleted_at → NOW()`) em todos os registros ativos
3. `refresh_tokens`: hard delete de todos os tokens do usuário
4. `password_reset_tokens`: hard delete de todos os tokens do usuário

### BR-09.2 — Irreversibilidade
- A anonimização é **irreversível**
- O frontend deve exibir confirmação explícita antes de executar a ação

---

## BR-10 — Dependência externa: serviço de e-mail

| Ambiente        | Provedor     | Finalidade                                                       |
|-----------------|--------------|------------------------------------------------------------------|
| Desenvolvimento | **Mailtrap** | Captura e-mails sem envio real. Interface web para visualização. |
| Produção        | A definir    | Provedor real (ex: SendGrid, SES) — fora do escopo do MVP        |

- Configuração via variáveis de ambiente: `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD`
- Falha no envio de e-mail **não deve** retornar erro ao usuário (logar internamente)
