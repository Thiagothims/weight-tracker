# Design — Weight Tracker

## Decisões de design

| Decisão                      | Escolha                                                          |
|------------------------------|------------------------------------------------------------------|
| Build tool                   | Maven                                                            |
| Organização de pacotes       | Por camada (controller / service / repository / model)           |
| Migrations                   | Liquibase                                                        |
| Repositórios                 | Separados (`weight-tracker-backend` e `weight-tracker-frontend`) |
| Frontend linguagem           | TypeScript                                                       |
| CSS                          | CSS Modules                                                      |
| Estado global                | Zustand                                                          |
| Geração de código (backend)  | openapi-generator-maven-plugin                                   |
| Geração de código (frontend) | orval                                                            |
| HTTP client                  | Axios                                                            |
| Gráfico                      | Recharts                                                         |
| Roteamento                   | React Router v6                                                  |

---

## 1. Infraestrutura de desenvolvimento

### docker-compose.yml
Localizado na raiz de `weight-tracker-backend/`.
Provisiona apenas o PostgreSQL — Mailtrap é serviço cloud, não precisa de container.

```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: weight-tracker-db
    environment:
      POSTGRES_DB: weight_tracker
      POSTGRES_USER: wt_user
      POSTGRES_PASSWORD: wt_pass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Variáveis de ambiente — Backend (`application.properties` / `.env`)

```properties
# Banco de dados
spring.datasource.url=jdbc:postgresql://localhost:5432/weight_tracker
spring.datasource.username=wt_user
spring.datasource.password=wt_pass

# JPA
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

# JWT
jwt.secret=<chave-secreta-minimo-256bits>
jwt.access-token-expiration=3600000
jwt.refresh-token-expiration=604800000

# Password reset token (30 min em ms)
app.password-reset-token-expiration=1800000

# Mailtrap (desenvolvimento)
spring.mail.host=sandbox.smtp.mailtrap.io
spring.mail.port=2525
spring.mail.username=<mailtrap-username>
spring.mail.password=<mailtrap-password>
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true

# URL base do frontend (para links nos e-mails)
app.frontend-url=http://localhost:5173
```

### Variáveis de ambiente — Frontend (`.env.local`)

```env
VITE_API_BASE_URL=http://localhost:8080
```

---

## 2. Backend — Spring Boot

### Stack e versões

| Tecnologia                     | Versão                   |
|--------------------------------|--------------------------|
| Java                           | 21 (LTS)                 |
| Spring Boot                    | 3.3.x                    |
| Spring Security                | 6.x (incluso no Boot 3)  |
| Spring Data JPA                | 3.x (incluso no Boot 3)  |
| PostgreSQL Driver              | 42.x                     |
| Liquibase                      | 4.x (incluso no Boot 3)  |
| JJWT (JWT)                     | 0.12.x                   |
| MapStruct                      | 1.5.x                    |
| Lombok                         | 1.18.x                   |
| openapi-generator-maven-plugin | 7.x                      |
| JavaMailSender                 | (incluso no Spring Boot) |

### Estrutura de pastas

```
weight-tracker-backend/
├── src/
│   ├── main/
│   │   ├── java/com/weighttracker/
│   │   │   ├── WeightTrackerApplication.java
│   │   │   ├── config/
│   │   │   │   ├── SecurityConfig.java
│   │   │   │   ├── JwtConfig.java
│   │   │   │   ├── OpenApiConfig.java
│   │   │   │   └── MailConfig.java
│   │   │   ├── controller/
│   │   │   │   ├── AuthController.java
│   │   │   │   ├── ProfileController.java
│   │   │   │   └── WeightRecordController.java
│   │   │   ├── service/
│   │   │   │   ├── AuthService.java
│   │   │   │   ├── ProfileService.java
│   │   │   │   ├── WeightRecordService.java
│   │   │   │   ├── TokenService.java
│   │   │   │   ├── BmiService.java
│   │   │   │   └── EmailService.java
│   │   │   ├── repository/
│   │   │   │   ├── UserRepository.java
│   │   │   │   ├── WeightRecordRepository.java
│   │   │   │   ├── RefreshTokenRepository.java
│   │   │   │   └── PasswordResetTokenRepository.java
│   │   │   ├── model/
│   │   │   │   ├── entity/
│   │   │   │   │   ├── User.java
│   │   │   │   │   ├── WeightRecord.java
│   │   │   │   │   ├── RefreshToken.java
│   │   │   │   │   └── PasswordResetToken.java
│   │   │   │   └── enums/
│   │   │   │       └── BmiCategory.java
│   │   │   ├── security/
│   │   │   │   ├── JwtAuthenticationFilter.java
│   │   │   │   └── UserDetailsServiceImpl.java
│   │   │   ├── mapper/
│   │   │   │   ├── UserMapper.java
│   │   │   │   └── WeightRecordMapper.java
│   │   │   └── exception/
│   │   │       ├── GlobalExceptionHandler.java
│   │   │       ├── DuplicateRecordException.java
│   │   │       ├── InvalidTokenException.java
│   │   │       └── ResourceNotFoundException.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── application-dev.properties
│   │       └── db/changelog/
│   │           ├── db.changelog-master.yaml
│   │           ├── 001-create-users.yaml
│   │           ├── 002-create-weight-records.yaml
│   │           ├── 003-create-refresh-tokens.yaml
│   │           └── 004-create-password-reset-tokens.yaml
│   └── test/
│       └── java/com/weighttracker/
│           ├── controller/
│           ├── service/
│           └── repository/
├── spec/
│   └── openapi.yaml          ← cópia/link da spec (fonte para geração de código)
├── docker-compose.yml
└── pom.xml
```

### Arquitetura em camadas

```
HTTP Request
     │
     ▼
[Controller]          ← recebe DTO gerado, delega ao service
     │
     ▼
[Service]             ← lógica de negócio, validações BR-*
     │
     ▼
[Repository]          ← Spring Data JPA, queries ao banco
     │
     ▼
[Entity / DB]         ← PostgreSQL via Liquibase migrations
```

**Mapeamento DTO ↔ Entity:** feito pelo MapStruct nos mappers.
Os controllers nunca tocam nas entities diretamente.

### Geração de código com openapi-generator

O `pom.xml` configura o plugin para gerar:
- **Interfaces** de API (`AuthApi`, `ProfileApi`, `WeightRecordApi`)
- **DTOs** de request/response (gerados em `target/generated-sources`)

Os controllers implementam as interfaces geradas:
```java
@RestController
public class AuthController implements AuthApi {
    // implementa os métodos da interface gerada
}
```

Nenhum DTO é escrito à mão — todos vêm da spec.

### Estrutura do `pom.xml` (dependências principais)

```xml
<dependencies>
  <!-- Web -->
  <dependency>spring-boot-starter-web</dependency>

  <!-- Security -->
  <dependency>spring-boot-starter-security</dependency>

  <!-- JPA + PostgreSQL -->
  <dependency>spring-boot-starter-data-jpa</dependency>
  <dependency>postgresql</dependency>

  <!-- Liquibase -->
  <dependency>liquibase-core</dependency>

  <!-- Validation -->
  <dependency>spring-boot-starter-validation</dependency>

  <!-- JWT -->
  <dependency>jjwt-api (0.12.x)</dependency>
  <dependency>jjwt-impl</dependency>
  <dependency>jjwt-jackson</dependency>

  <!-- MapStruct -->
  <dependency>mapstruct</dependency>
  <dependency>mapstruct-processor (annotationProcessor)</dependency>

  <!-- Lombok -->
  <dependency>lombok</dependency>

  <!-- Mail -->
  <dependency>spring-boot-starter-mail</dependency>

  <!-- OpenAPI UI (Swagger) -->
  <dependency>springdoc-openapi-starter-webmvc-ui</dependency>

  <!-- Testes -->
  <dependency>spring-boot-starter-test</dependency>
  <dependency>spring-security-test</dependency>
</dependencies>

<plugins>
  <plugin>openapi-generator-maven-plugin</plugin>
</plugins>
```

### Liquibase — estrutura de migrations

```
resources/db/changelog/
├── db.changelog-master.yaml        ← inclui os demais em ordem
├── 001-create-users.yaml
├── 002-create-weight-records.yaml
├── 003-create-refresh-tokens.yaml
└── 004-create-password-reset-tokens.yaml
```

Cada arquivo corresponde exatamente ao schema definido em `spec/data-model.md`.
Novas alterações de schema sempre criam um **novo arquivo** de migration — nunca editam os existentes.

### Fluxo JWT

```
Login
  │
  ├─ valida email + senha (bcrypt)
  ├─ gera accessToken (JWT, 1h, assinalado com chave secreta)
  ├─ gera refreshToken (UUID aleatório)
  ├─ salva hash(refreshToken) em refresh_tokens
  └─ retorna AuthResponse { accessToken, refreshToken }

Requisição autenticada
  │
  ├─ JwtAuthenticationFilter extrai Bearer token do header
  ├─ valida assinatura + expiração do JWT
  ├─ carrega UserDetails via UserDetailsServiceImpl
  └─ injeta Authentication no SecurityContext

Renovação de token
  │
  ├─ recebe refreshToken
  ├─ busca hash no banco (revoked_at IS NULL AND expires_at > NOW())
  ├─ gera novo accessToken
  └─ retorna novo AuthResponse (refreshToken permanece o mesmo)
```

### CORS

Configurado em `SecurityConfig.java` para aceitar requisições de `http://localhost:5173` (Vite dev server) durante o desenvolvimento.

---

## 3. Frontend — React

### Stack e versões

| Tecnologia      | Versão         | Finalidade                                     |
|-----------------|----------------|------------------------------------------------|
| Node.js         | 20.x (LTS)     | Runtime                                        |
| React           | 18.x           | UI                                             |
| TypeScript      | 5.x            | Tipagem estática                               |
| Vite            | 5.x            | Build tool e dev server                        |
| React Router    | 6.x            | Roteamento SPA                                 |
| Axios           | 1.x            | HTTP client                                    |
| TanStack Query  | 5.x            | Cache, loading/error states, invalidação       |
| Zustand         | 4.x            | Estado global (auth + tema)                    |
| React Hook Form | 7.x            | Gerenciamento de formulários                   |
| Zod             | 3.x            | Validação de schemas + integração com RHF      |
| Recharts        | 2.x            | Gráfico de linha do dashboard                  |
| orval           | 7.x            | Geração de API client a partir do openapi.yaml |
| CSS Modules     | nativo no Vite | Estilos escopados por componente               |

### Estrutura de pastas

```
weight-tracker-frontend/
├── src/
│   ├── api/
│   │   └── generated/          ← gerado pelo orval (NUNCA editar manualmente)
│   │       ├── auth.ts
│   │       ├── profile.ts
│   │       ├── weightRecords.ts
│   │       └── models/         ← types gerados do openapi.yaml
│   ├── components/             ← componentes reutilizáveis
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   └── Button.module.css
│   │   ├── Input/
│   │   │   ├── Input.tsx
│   │   │   └── Input.module.css
│   │   ├── Modal/
│   │   ├── Toast/
│   │   ├── Badge/              ← badge de categoria IMC
│   │   ├── Sidebar/
│   │   └── PasswordStrength/   ← checklist de requisitos de senha
│   ├── pages/
│   │   ├── Login/
│   │   │   ├── LoginPage.tsx
│   │   │   └── LoginPage.module.css
│   │   ├── Register/
│   │   ├── ForgotPassword/
│   │   ├── ResetPassword/
│   │   ├── Dashboard/
│   │   │   ├── DashboardPage.tsx
│   │   │   ├── DashboardPage.module.css
│   │   │   ├── WeightCards.tsx
│   │   │   ├── WeightChart.tsx
│   │   │   └── MonthSelector.tsx
│   │   ├── Records/
│   │   │   ├── RecordsPage.tsx
│   │   │   ├── RecordsTable.tsx
│   │   │   └── InlineEditRow.tsx
│   │   └── Profile/
│   │       ├── ProfilePage.tsx
│   │       ├── PersonalInfoSection.tsx
│   │       ├── SecuritySection.tsx
│   │       └── DangerZoneSection.tsx
│   ├── store/
│   │   ├── authStore.ts        ← usuário autenticado + tokens
│   │   └── themeStore.ts       ← light / dark mode
│   ├── hooks/
│   │   ├── useAuth.ts          ← acesso simplificado ao authStore
│   │   └── useMonthFilter.ts   ← estado do seletor de mês/ano
│   ├── utils/
│   │   ├── bmi.ts              ← formatação de categoria IMC para exibição
│   │   ├── date.ts             ← formatação de datas em pt-BR
│   │   └── axios.ts            ← instância Axios + interceptors
│   ├── styles/
│   │   ├── variables.css       ← tokens de cor e tipografia (CSS custom properties)
│   │   └── global.css          ← reset, font-face, body
│   ├── router/
│   │   ├── AppRouter.tsx       ← definição das rotas
│   │   └── PrivateRoute.tsx    ← guarda de rota autenticada
│   ├── App.tsx
│   └── main.tsx
├── orval.config.ts              ← configuração de geração de código
├── vite.config.ts
├── tsconfig.json
└── .env.local
```

### Zustand — stores

**`authStore.ts`**
```typescript
interface AuthState {
  user: UserProfile | null
  accessToken: string | null
  refreshToken: string | null
  isAuthenticated: boolean
  setAuth: (tokens: AuthResponse, user: UserProfile) => void
  setUser: (user: UserProfile) => void
  clearAuth: () => void
}
```

**`themeStore.ts`**
```typescript
interface ThemeState {
  theme: 'light' | 'dark'
  toggle: () => void
  setTheme: (theme: 'light' | 'dark') => void
}
// Persiste no localStorage via middleware zustand/middleware persist
```

### Axios — instância e interceptors

**`utils/axios.ts`**
```
instância Axios
  ├── baseURL: VITE_API_BASE_URL
  ├── interceptor de request:
  │     └─ injeta Authorization: Bearer <accessToken> do authStore
  └── interceptor de response:
        ├─ sucesso: passa direto
        └─ erro 401:
              ├─ tenta POST /auth/refresh com refreshToken
              ├─ sucesso: atualiza accessToken no store, repete a requisição original
              └─ falha: clearAuth() + redirect /login
```

Esse interceptor implementa o **silent refresh** — o usuário nunca percebe a renovação do token.

### TanStack Query — padrão de uso

Todas as chamadas à API seguem dois padrões:

**Leitura de dados (`useQuery`):**
```typescript
// Dashboard — carrega resumo do mês selecionado
const { data, isLoading, error } = useQuery({
  queryKey: ['month-summary', year, month],  // cache por ano/mês
  queryFn: () => getMonthSummary({ year, month })
})
// Trocar de mês invalida a queryKey e refaz a requisição automaticamente
```

**Escrita de dados (`useMutation`):**
```typescript
// Criar novo registro de peso
const { mutate, isPending } = useMutation({
  mutationFn: createWeightRecord,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['month-summary'] })
    queryClient.invalidateQueries({ queryKey: ['weight-records'] })
    // Dashboard e tabela atualizam automaticamente
  }
})
```

**Query keys padronizadas:**
| Dado | Query key |
|---|---|
| Resumo mensal | `['month-summary', year, month]` |
| Lista de registros | `['weight-records', year, month]` |
| Perfil do usuário | `['profile']` |

### React Hook Form + Zod — padrão de uso

Cada formulário tem um schema Zod correspondente:

```typescript
// schemas/weightRecord.schema.ts
export const createWeightRecordSchema = z.object({
  recordTime: z.string().regex(/^([01]\d|2[0-3]):[0-5]\d$/, 'Horário inválido'),
  weightKg: z.number({ invalid_type_error: 'Informe o peso' })
             .min(1, 'Peso inválido').max(500, 'Peso inválido')
})

export type CreateWeightRecordForm = z.infer<typeof createWeightRecordSchema>
```

```typescript
// No componente
const { register, handleSubmit, formState: { errors } } = useForm<CreateWeightRecordForm>({
  resolver: zodResolver(createWeightRecordSchema)
})
```

Os schemas Zod são a tradução das regras de `business-rules.md` para código TypeScript — mesma fonte de verdade, aplicada no cliente.

### Geração de código com orval

`orval.config.ts` aponta para `../weight-tracker-backend/spec/openapi.yaml` (ou cópia local).
Gera **funções Axios tipadas** para cada endpoint — consumidas diretamente pelo TanStack Query.

Exemplo do que é gerado:
```typescript
// api/generated/weightRecords.ts  (NUNCA editar manualmente)
export const createWeightRecord = (body: CreateWeightRecordRequest) =>
  axiosInstance.post<WeightRecord>('/api/weight-records', body)

export const listWeightRecords = (params: { year: number; month: number }) =>
  axiosInstance.get<WeightRecord[]>('/api/weight-records', { params })
```

### CSS Modules — convenção

- Um arquivo `.module.css` por componente/página
- Tokens de cor via variáveis CSS (`var(--bg-surface)`) definidas em `styles/variables.css`
- O tema light/dark é aplicado via atributo `data-theme="dark"` no `<html>`, alternando os valores das variáveis

```css
/* styles/variables.css */
:root {
  --bg-base: #F7F7F7;
  --text-primary: #111111;
  /* ... */
}

[data-theme="dark"] {
  --bg-base: #111111;
  --text-primary: #F0F0F0;
  /* ... */
}
```

### Roteamento

```tsx
// router/AppRouter.tsx
<Routes>
  {/* Públicas */}
  <Route path="/login" element={<LoginPage />} />
  <Route path="/register" element={<RegisterPage />} />
  <Route path="/forgot-password" element={<ForgotPasswordPage />} />
  <Route path="/reset-password" element={<ResetPasswordPage />} />

  {/* Autenticadas — envolvidas por PrivateRoute */}
  <Route element={<PrivateRoute />}>
    <Route path="/" element={<Navigate to="/dashboard" />} />
    <Route path="/dashboard" element={<DashboardPage />} />
    <Route path="/records" element={<RecordsPage />} />
    <Route path="/profile" element={<ProfilePage />} />
  </Route>
</Routes>
```

`PrivateRoute` verifica `isAuthenticated` no `authStore`. Se falso, redireciona para `/login` preservando a rota de origem (`state.from`).

---

## 4. Resumo da integração

```
openapi.yaml (spec)
      │
      ├──► openapi-generator-maven-plugin
      │         └─ gera interfaces + DTOs no backend (Java)
      │
      └──► orval
                └─ gera funções de API + types no frontend (TypeScript)

Backend (porta 8080)  ◄──── Axios (com interceptor JWT) ──── Frontend (porta 5173)
        │
        ▼
  PostgreSQL (porta 5432)  ←── docker-compose
```

---

## 5. Repositórios

| Repositório               | Conteúdo                                                     |
|---------------------------|--------------------------------------------------------------|
| `weight-tracker-backend`  | Spring Boot + Maven + docker-compose + cópia do openapi.yaml |
| `weight-tracker-frontend` | React + Vite + TypeScript + cópia do openapi.yaml            |

> A `spec/` atual (`/root/my_projects/weight-tracker/spec/`) permanece como repositório de referência. O `openapi.yaml` é copiado para ambos os projetos e deve ser mantido sincronizado manualmente a cada alteração na spec.
