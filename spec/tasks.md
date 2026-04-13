# Tasks — Weight Tracker
## Estrutura: Epic → Story → Task

---

## Instruções de execução

- Leia todos os arquivos de spec em `spec/` antes de começar qualquer implementação
- Inspecione os arquivos já existentes no projeto para entender o estado atual
- Execute as tasks na ordem em que aparecem dentro de cada story
- Marque cada task com `[x]` assim que concluída, antes de passar para a próxima
- Se houver desvio ou decisão não óbvia em relação ao spec, registre em `spec/notes.md`
- Ao final do épico, confirme que todos os checkboxes do épico estão marcados

---

## Épicos

| ID    | Nome                    |
|-------|-------------------------|
| EP-01 | Infraestrutura e Setup  |
| EP-02 | Autenticação            |
| EP-03 | Redefinição de senha    |
| EP-04 | Perfil e conta          |
| EP-05 | Registros de peso       |
| EP-06 | Dashboard               |
| EP-07 | Qualidade e finalização |

---

## EP-01 — Infraestrutura e Setup

---

### ST-01.1 — Projeto backend configurado e funcionando localmente

> Como desenvolvedor, quero o projeto backend rodando localmente com banco de dados provisionado via Docker, para começar a implementar as funcionalidades.

**Critérios de aceitação:**
- [ ] Aplicação sobe sem erros com `mvn spring-boot:run`
- [ ] Banco PostgreSQL acessível via docker-compose
- [ ] Todas as migrations Liquibase executam com sucesso (4 tabelas criadas)
- [ ] Interfaces e DTOs gerados a partir do `openapi.yaml` disponíveis em `target/generated-sources`

**Tasks:**
- [ ] TB-01.1.1 — Criar projeto Spring Boot via Spring Initializr (Web, Security, Data JPA, Validation, Mail, Liquibase, PostgreSQL Driver, Lombok)
- [ ] TB-01.1.2 — Adicionar dependências ao `pom.xml`: JJWT 0.12.x, MapStruct 1.5.x, SpringDoc OpenAPI, openapi-generator-maven-plugin
- [ ] TB-01.1.3 — Criar `docker-compose.yml` com PostgreSQL 16 (banco `weight_tracker`, usuário `wt_user`)
- [ ] TB-01.1.4 — Configurar `application.properties` e `application-dev.properties` com variáveis de banco, JWT, mail e URL do frontend
- [ ] TB-01.1.5 — Copiar `openapi.yaml` para `src/main/resources/spec/` e configurar o plugin `openapi-generator-maven-plugin` no `pom.xml`
- [ ] TB-01.1.6 — Executar `mvn generate-sources` e validar geração das interfaces (`AuthApi`, `ProfileApi`, `WeightRecordApi`) e DTOs
- [ ] TB-01.1.7 — Criar arquivo `db/changelog/db.changelog-master.yaml` e os 4 changelogs de migration conforme `spec/data-model.md`
- [ ] TB-01.1.8 — Subir docker-compose, executar a aplicação e validar que as 4 tabelas foram criadas corretamente no banco
- [ ] TB-01.1.9 — Configurar CORS em `SecurityConfig.java` permitindo `http://localhost:5173`

---

### ST-01.2 — Projeto frontend configurado e funcionando localmente

> Como desenvolvedor, quero o projeto frontend com todas as dependências instaladas, geração de código do `openapi.yaml` funcionando e estrutura de pastas definida, para começar a implementar as telas.

**Critérios de aceitação:**
- [ ] `npm run dev` sobe o servidor Vite na porta 5173 sem erros
- [ ] Arquivos gerados pelo orval disponíveis em `src/api/generated/`
- [ ] Tokens de cor CSS aplicados corretamente no tema light e dark
- [ ] Rotas públicas e privadas configuradas — rota protegida redireciona para `/login`

**Tasks:**
- [ ] TF-01.2.1 — Criar projeto com `npm create vite@latest` usando template `react-ts`
- [ ] TF-01.2.2 — Instalar dependências: React Router, Axios, TanStack Query, Zustand, React Hook Form, Zod, Recharts, orval
- [ ] TF-01.2.3 — Criar estrutura de pastas conforme `spec/design.md` (`api/`, `components/`, `pages/`, `store/`, `hooks/`, `utils/`, `styles/`, `router/`)
- [ ] TF-01.2.4 — Criar `styles/variables.css` com todos os tokens de cor (light/dark) e `styles/global.css` com reset e importação da fonte Inter
- [ ] TF-01.2.5 — Configurar `orval.config.ts` apontando para o `openapi.yaml` e executar geração (`npx orval`) — validar arquivos em `src/api/generated/`
- [ ] TF-01.2.6 — Criar instância Axios em `utils/axios.ts` com `baseURL` via `VITE_API_BASE_URL`
- [ ] TF-01.2.7 — Criar `authStore.ts` (Zustand) com campos: `user`, `accessToken`, `refreshToken`, `isAuthenticated` e ações: `setAuth`, `setUser`, `clearAuth`
- [ ] TF-01.2.8 — Criar `themeStore.ts` (Zustand) com persist no localStorage e `useEffect` que aplica `data-theme` no `<html>`
- [ ] TF-01.2.9 — Configurar `QueryClientProvider` (TanStack Query) em `main.tsx`
- [ ] TF-01.2.10 — Criar `AppRouter.tsx` com rotas públicas e `PrivateRoute.tsx` que redireciona para `/login` se não autenticado

---

## EP-02 — Autenticação

---

### ST-02.1 — Cadastro de novo usuário

> Como visitante, quero criar uma conta informando nome, e-mail e senha, para acessar o sistema.

**Critérios de aceitação:**
- [ ] Cadastro bem-sucedido retorna tokens e redireciona para o dashboard
- [ ] E-mail duplicado exibe mensagem de erro clara
- [ ] Requisitos de senha são exibidos em tempo real conforme digitação
- [ ] Campos obrigatórios validados antes do envio

**Tasks Backend:**
- [ ] TB-02.1.1 — Criar entity `User` com Lombok + JPA annotations conforme `data-model.md`
- [ ] TB-02.1.2 — Criar `UserRepository` com métodos: `findByEmail`, `findByEmailAndDeletedAtIsNull`
- [ ] TB-02.1.3 — Criar `TokenService`: gerar access token JWT (JJWT), gerar refresh token (UUID), salvar hash SHA-256 no banco, validar JWT
- [ ] TB-02.1.4 — Criar entity `RefreshToken` e `RefreshTokenRepository`
- [ ] TB-02.1.5 — Implementar `AuthService.register`: validar política de senha, hash bcrypt (custo 10), salvar user, gerar e retornar tokens
- [ ] TB-02.1.6 — Criar `AuthController` implementando `AuthApi` — expor endpoint `POST /api/auth/register`

**Tasks Frontend:**
- [ ] TF-02.1.1 — Criar schema Zod `registerSchema` com regras de `business-rules.md` (BR-04.1)
- [ ] TF-02.1.2 — Criar componentes base reutilizáveis: `Button`, `Input` (com toggle de senha), `PasswordStrength` (checklist em tempo real)
- [ ] TF-02.1.3 — Criar `RegisterPage` com React Hook Form + `registerSchema`, `useMutation` para `register`, salvar tokens no `authStore` e redirecionar para `/dashboard`

---

### ST-02.2 — Login

> Como usuário cadastrado, quero fazer login com e-mail e senha, para acessar meus registros.

**Critérios de aceitação:**
- [ ] Login bem-sucedido redireciona para o dashboard
- [ ] Credenciais inválidas exibem mensagem genérica (sem indicar qual campo está errado)
- [ ] Conta deletada/anonimizada recebe o mesmo erro de credenciais inválidas
- [ ] Usuário autenticado que tenta acessar `/login` é redirecionado para `/dashboard`

**Tasks Backend:**
- [ ] TB-02.2.1 — Implementar `AuthService.login`: buscar por e-mail (`deleted_at IS NULL`), validar senha bcrypt, gerar tokens
- [ ] TB-02.2.2 — Expor endpoint `POST /api/auth/login` no `AuthController`

**Tasks Frontend:**
- [ ] TF-02.2.1 — Criar schema Zod `loginSchema`
- [ ] TF-02.2.2 — Criar `LoginPage` com formulário, `useMutation` para `login`, salvar no `authStore`, redirecionar para `/dashboard`
- [ ] TF-02.2.3 — Adicionar link "Esqueceu a senha?" e "Não tem conta? Cadastre-se" na página

---

### ST-02.3 — Renovação automática de sessão (silent refresh)

> Como usuário, quero que minha sessão seja renovada automaticamente quando o access token expirar, sem precisar fazer login novamente.

**Critérios de aceitação:**
- [ ] Requisição com access token expirado é retentada automaticamente com novo token
- [ ] Refresh token inválido ou expirado encerra a sessão e redireciona para `/login`
- [ ] O usuário não percebe a renovação acontecendo

**Tasks Backend:**
- [ ] TB-02.3.1 — Criar `JwtAuthenticationFilter`: extrair Bearer token, validar via `TokenService`, injetar `Authentication` no `SecurityContext`
- [ ] TB-02.3.2 — Configurar `SecurityConfig`: cadeia de filtros, endpoints públicos, adicionar `JwtAuthenticationFilter`
- [ ] TB-02.3.3 — Criar `UserDetailsServiceImpl` implementando `UserDetailsService`
- [ ] TB-02.3.4 — Implementar `AuthService.refreshToken`: validar hash no banco (`revoked_at IS NULL AND expires_at > NOW()`), gerar novo access token
- [ ] TB-02.3.5 — Expor endpoint `POST /api/auth/refresh` no `AuthController`

**Tasks Frontend:**
- [ ] TF-02.3.1 — Adicionar interceptor de request no Axios: injetar `Authorization: Bearer <accessToken>` do `authStore`
- [ ] TF-02.3.2 — Adicionar interceptor de response no Axios: em erro 401, chamar `POST /auth/refresh`, atualizar store e repetir requisição original; em falha do refresh, chamar `clearAuth()` e redirecionar para `/login`

---

### ST-02.4 — Logout

> Como usuário, quero encerrar minha sessão no dispositivo atual, para proteger minha conta.

**Critérios de aceitação:**
- [ ] Logout remove o refresh token do dispositivo atual no servidor
- [ ] Outras sessões (outros dispositivos) permanecem ativas
- [ ] Após logout, usuário é redirecionado para `/login`

**Tasks Backend:**
- [ ] TB-02.4.1 — Implementar `AuthService.logout`: revogar o refresh token do dispositivo atual (`revoked_at = NOW()`)
- [ ] TB-02.4.2 — Expor endpoint `POST /api/auth/logout` no `AuthController`

**Tasks Frontend:**
- [ ] TF-02.4.1 — Criar `Sidebar` com botão "Sair": chamar `useMutation` para `logout`, depois `clearAuth()` e redirecionar para `/login`

---

## EP-03 — Redefinição de senha

---

### ST-03.1 — Solicitar redefinição de senha por e-mail

> Como usuário que esqueceu a senha, quero informar meu e-mail e receber um link de redefinição, para recuperar o acesso à minha conta.

**Critérios de aceitação:**
- [ ] A resposta é sempre a mesma, independente de o e-mail existir (evita enumeração)
- [ ] E-mail recebido no Mailtrap contém link válido com token
- [ ] Solicitar novo link invalida o anterior
- [ ] Falha no envio de e-mail não retorna erro ao usuário (logado internamente)

**Tasks Backend:**
- [ ] TB-03.1.1 — Criar entity `PasswordResetToken` e `PasswordResetTokenRepository`
- [ ] TB-03.1.2 — Criar `EmailService`: configurar `JavaMailSender`, método `sendPasswordResetEmail(to, token)` com template de e-mail simples
- [ ] TB-03.1.3 — Implementar `AuthService.forgotPassword`: verificar e-mail, deletar tokens anteriores, gerar token seguro, salvar hash, chamar `EmailService` em try/catch (falha silenciosa)
- [ ] TB-03.1.4 — Expor endpoint `POST /api/auth/forgot-password` no `AuthController`

**Tasks Frontend:**
- [ ] TF-03.1.1 — Criar schema Zod `forgotPasswordSchema`
- [ ] TF-03.1.2 — Criar `ForgotPasswordPage`: formulário de e-mail, após submit substituir formulário por mensagem de sucesso genérica, link para voltar ao login

---

### ST-03.2 — Redefinir senha via link do e-mail

> Como usuário, quero acessar o link recebido por e-mail e definir uma nova senha, para recuperar o acesso à conta.

**Critérios de aceitação:**
- [ ] Token inválido ou expirado exibe mensagem de erro com link para solicitar novo
- [ ] Nova senha validada pelas regras de `business-rules.md` (BR-04.1)
- [ ] Após redefinição, todas as sessões ativas são encerradas
- [ ] Token não pode ser reutilizado após uso

**Tasks Backend:**
- [ ] TB-03.2.1 — Implementar `AuthService.resetPassword`: validar hash + `used_at IS NULL` + `expires_at > NOW()`, atualizar senha (bcrypt), marcar token como usado, deletar todos os `refresh_tokens` do usuário
- [ ] TB-03.2.2 — Expor endpoint `POST /api/auth/reset-password` no `AuthController`

**Tasks Frontend:**
- [ ] TF-03.2.1 — Criar schema Zod `resetPasswordSchema`
- [ ] TF-03.2.2 — Criar `ResetPasswordPage`: ler token da query string (`?token=...`), formulário com `PasswordStrength`, tratar token inválido exibindo mensagem inline com link para `/forgot-password`, redirecionar para `/login` com toast de sucesso após redefinição

---

## EP-04 — Perfil e conta

---

### ST-04.1 — Visualizar e editar informações pessoais

> Como usuário, quero visualizar e editar meu nome e altura, para manter meu perfil atualizado e garantir o cálculo correto do IMC.

**Critérios de aceitação:**
- [ ] Perfil exibe nome, e-mail e altura atual
- [ ] Edição de nome e altura salva com sucesso
- [ ] Se altura não cadastrada, exibe aviso "Cadastre sua altura" com destaque visual
- [ ] Campo `name` não pode ser salvo vazio

**Tasks Backend:**
- [ ] TB-04.1.1 — Criar `BmiService`: calcular IMC e retornar `BmiCategory` conforme tabela OMS (`business-rules.md` BR-03.4), retornar `null` se altura for `null`
- [ ] TB-04.1.2 — Criar `UserMapper` com MapStruct: `User` → `UserProfile` DTO
- [ ] TB-04.1.3 — Criar `ProfileService.getProfile` e `ProfileService.updateProfile` (nome + altura)
- [ ] TB-04.1.4 — Criar `ProfileController` implementando `ProfileApi` — expor `GET /api/profile` e `PUT /api/profile`

**Tasks Frontend:**
- [ ] TF-04.1.1 — Criar layout autenticado `AuthLayout` com `Sidebar` + área de conteúdo
- [ ] TF-04.1.2 — Criar schema Zod `updateProfileSchema`
- [ ] TF-04.1.3 — Criar `PersonalInfoSection`: `useQuery` para carregar perfil, formulário com React Hook Form, `useMutation` para `updateProfile`, invalidar query `['profile']` ao salvar
- [ ] TF-04.1.4 — Criar `ProfilePage` compondo as seções (pessoal, segurança, zona de perigo)

---

### ST-04.2 — Alterar e-mail

> Como usuário, quero alterar o meu e-mail de acesso, confirmando com minha senha atual.

**Critérios de aceitação:**
- [ ] Requer senha atual correta para confirmar identidade
- [ ] Novo e-mail não pode ser igual ao atual
- [ ] Novo e-mail não pode já estar em uso
- [ ] Sessões em outros dispositivos são encerradas após a troca

**Tasks Backend:**
- [ ] TB-04.2.1 — Implementar `ProfileService.changeEmail`: validar senha atual (bcrypt), validar novo e-mail (diferente do atual e não em uso), atualizar e-mail, revogar refresh tokens de outros dispositivos
- [ ] TB-04.2.2 — Expor endpoint `PUT /api/auth/change-email` no `AuthController`

**Tasks Frontend:**
- [ ] TF-04.2.1 — Criar schema Zod `changeEmailSchema`
- [ ] TF-04.2.2 — Criar `SecuritySection` com accordion "Alterar e-mail": formulário com novo e-mail + senha atual, `useMutation`, atualizar `authStore.user.email` ao salvar

---

### ST-04.3 — Alterar senha

> Como usuário autenticado, quero alterar minha senha informando a senha atual e a nova senha.

**Critérios de aceitação:**
- [ ] Requer senha atual correta
- [ ] Nova senha validada pelas regras de `business-rules.md` (BR-04.1)
- [ ] Sessões em outros dispositivos são encerradas; sessão atual permanece ativa
- [ ] Requisitos de senha exibidos em tempo real durante digitação

**Tasks Backend:**
- [ ] TB-04.3.1 — Implementar `ProfileService.changePassword`: validar senha atual (bcrypt), validar nova senha, atualizar hash, revogar refresh tokens de outros dispositivos (mantendo o atual)
- [ ] TB-04.3.2 — Expor endpoint `PUT /api/auth/change-password` no `AuthController`

**Tasks Frontend:**
- [ ] TF-04.3.1 — Criar schema Zod `changePasswordSchema`
- [ ] TF-04.3.2 — Adicionar accordion "Alterar senha" na `SecuritySection` com `PasswordStrength` e `useMutation`

---

### ST-04.4 — Excluir conta

> Como usuário, quero excluir minha conta permanentemente, sabendo que todos os meus dados serão anonimizados.

**Critérios de aceitação:**
- [ ] Confirmação exige digitação da senha atual
- [ ] Anonimização executada em transação única (conforme `business-rules.md` BR-09.1)
- [ ] Após exclusão, todas as sessões são encerradas e usuário redirecionado para `/login`
- [ ] Ação é irreversível — mensagem clara antes da confirmação

**Tasks Backend:**
- [ ] TB-04.4.1 — Implementar `ProfileService.deleteAccount`: validar senha, executar anonimização transacional (update users + soft delete weight_records + hard delete refresh_tokens + hard delete password_reset_tokens)
- [ ] TB-04.4.2 — Expor endpoint `DELETE /api/profile` no `ProfileController`

**Tasks Frontend:**
- [ ] TF-04.4.1 — Criar schema Zod `deleteAccountSchema` (campo senha)
- [ ] TF-04.4.2 — Criar `DangerZoneSection` com botão vermelho + `Modal` de confirmação com campo de senha, `useMutation`, após sucesso: `clearAuth()` + redirect `/login` + toast "Conta encerrada"

---

## EP-05 — Registros de peso

---

### ST-05.1 — Registrar peso do dia

> Como usuário, quero registrar meu peso de hoje informando o horário da aferição e o valor em kg, para acompanhar minha evolução.

**Critérios de aceitação:**
- [ ] Data preenchida automaticamente pelo servidor com a data atual
- [ ] Apenas um registro por dia — tentativa duplicada exibe mensagem clara
- [ ] Formulário exibe a data atual de forma descritiva (não editável)
- [ ] Horário pré-preenchido com o horário atual do navegador
- [ ] Após salvar, dashboard atualiza automaticamente

**Tasks Backend:**
- [ ] TB-05.1.1 — Criar entity `WeightRecord` e `WeightRecordRepository` com métodos: `existsByUserIdAndRecordDateAndDeletedAtIsNull`, `findByUserIdAndRecordDateBetweenAndDeletedAtIsNull`
- [ ] TB-05.1.2 — Criar `WeightRecordMapper` com MapStruct: `WeightRecord` + `height_cm` do usuário → `WeightRecord` DTO (com IMC calculado via `BmiService`)
- [ ] TB-05.1.3 — Implementar `WeightRecordService.create`: usar `LocalDate.now()` como data, verificar duplicata, salvar
- [ ] TB-05.1.4 — Expor endpoint `POST /api/weight-records` no `WeightRecordController`

**Tasks Frontend:**
- [ ] TF-05.1.1 — Criar componentes base: `Modal`, `Toast`, `Badge`
- [ ] TF-05.1.2 — Criar schema Zod `createWeightRecordSchema` (horário + peso)
- [ ] TF-05.1.3 — Criar `NewRecordModal`: exibir data atual formatada em pt-BR, horário pré-preenchido, React Hook Form + schema, `useMutation` para `createWeightRecord`, ao salvar: invalidar queries `['month-summary']` e `['weight-records']`, fechar modal e exibir toast de sucesso

---

### ST-05.2 — Editar registro de peso

> Como usuário, quero editar o horário ou o peso de um registro já existente, para corrigir um valor incorreto.

**Critérios de aceitação:**
- [ ] Qualquer registro ativo pode ser editado (sem restrição de data)
- [ ] Apenas `recordTime` e `recordDate` são editáveis (a data em si não muda)
- [ ] Edição feita inline na tabela, sem abrir nova página ou modal
- [ ] ESC cancela a edição sem salvar

**Tasks Backend:**
- [ ] TB-05.2.1 — Implementar `WeightRecordService.update`: validar que o registro pertence ao usuário autenticado, atualizar campos
- [ ] TB-05.2.2 — Expor endpoint `PUT /api/weight-records/{id}` no `WeightRecordController`

**Tasks Frontend:**
- [ ] TF-05.2.1 — Criar schema Zod `updateWeightRecordSchema`
- [ ] TF-05.2.2 — Criar `InlineEditRow`: ao clicar no ícone de editar, linha vira inputs de horário e peso; botões confirmar (✓) e cancelar (✗); ESC cancela; `useMutation` para `updateWeightRecord`, ao salvar invalidar queries e sair do modo edição

---

### ST-05.3 — Excluir registro de peso

> Como usuário, quero excluir um registro de peso, para remover uma entrada incorreta.

**Critérios de aceitação:**
- [ ] Confirmação exibida inline na própria linha antes de excluir
- [ ] Exclusão lógica (soft delete) — dado não é perdido permanentemente
- [ ] Tabela atualiza imediatamente após exclusão

**Tasks Backend:**
- [ ] TB-05.3.1 — Implementar `WeightRecordService.delete`: validar ownership, preencher `deleted_at`
- [ ] TB-05.3.2 — Expor endpoint `DELETE /api/weight-records/{id}` no `WeightRecordController`

**Tasks Frontend:**
- [ ] TF-05.3.1 — Adicionar confirmação inline na linha da tabela ao clicar no ícone de lixeira: exibir "Excluir registro de DD mmm?" com botões Confirmar/Cancelar; `useMutation` para `deleteWeightRecord`, invalidar queries ao confirmar

---

### ST-05.4 — Visualizar registros do mês em tabela

> Como usuário, quero ver todos os meus registros do mês em uma tabela com data, horário, peso e IMC, para acompanhar meu histórico.

**Critérios de aceitação:**
- [ ] Registros ordenados por data decrescente (mais recente no topo)
- [ ] Coluna IMC exibe valor + badge colorido com categoria
- [ ] Se altura não cadastrada, coluna IMC exibe `—`
- [ ] Empty state exibido quando não há registros no mês
- [ ] Skeleton loader durante carregamento

**Tasks Backend:**
- [ ] TB-05.4.1 — Implementar `WeightRecordService.listByMonth`: buscar registros ativos do usuário no mês/ano, calcular IMC via `BmiService`, ordenar por data desc
- [ ] TB-05.4.2 — Expor endpoint `GET /api/weight-records?year=&month=` no `WeightRecordController`

**Tasks Frontend:**
- [ ] TF-05.4.1 — Criar hook `useMonthFilter`: estado de ano/mês, navegação, limites de período (criação da conta até mês atual)
- [ ] TF-05.4.2 — Criar componente `MonthSelector` (setas + texto + dropdown com meses disponíveis)
- [ ] TF-05.4.3 — Criar `RecordsTable`: `useQuery` para `listWeightRecords`, skeleton loader, empty state, colunas com `Badge` de IMC, ações de editar e excluir
- [ ] TF-05.4.4 — Criar `RecordsPage` compondo `MonthSelector` + `RecordsTable`

---

## EP-06 — Dashboard

---

### ST-06.1 — Visualizar resumo do mês

> Como usuário, quero ver o primeiro e o último peso registrado no mês selecionado, para acompanhar minha evolução mensal.

**Critérios de aceitação:**
- [ ] Card "Primeiro peso" exibe peso + data + horário do primeiro registro do mês
- [ ] Card "Último peso" exibe peso + data + horário + IMC (badge) do último registro
- [ ] Se altura não cadastrada, badge laranja "Cadastre sua altura" no lugar do IMC com link para perfil
- [ ] Se nenhum registro no mês, exibe empty state com botão "Registrar peso hoje"

**Tasks Backend:**
- [ ] TB-06.1.1 — Implementar `WeightRecordService.getMonthSummary`: buscar primeiro e último registro, calcular IMC via `BmiService`, montar `MonthSummary` DTO
- [ ] TB-06.1.2 — Expor endpoint `GET /api/weight-records/summary?year=&month=` no `WeightRecordController`

**Tasks Frontend:**
- [ ] TF-06.1.1 — Criar componente `WeightCards`: dois cards lado a lado, exibir valores do `MonthSummary`, badge de IMC via componente `Badge`, aviso de altura ausente com link para `/profile`
- [ ] TF-06.1.2 — Criar empty state do dashboard com mensagem e botão "Registrar peso hoje"

---

### ST-06.2 — Visualizar gráfico de evolução do peso

> Como usuário, quero visualizar um gráfico de linha com a evolução do meu peso ao longo do mês, para identificar tendências.

**Critérios de aceitação:**
- [ ] Gráfico exibe linha contínua com pontos em cada registro
- [ ] Eixo X mostra os dias com registro; eixo Y escala automática com margem de ±2kg
- [ ] Tooltip ao hover exibe: data, peso e IMC (se disponível)
- [ ] Com apenas 1 ponto, exibe somente o ponto (sem linha)
- [ ] Sem dados, exibe empty state dentro do container do gráfico

**Tasks Backend:**
- [ ] TB-06.2.1 — Garantir que `MonthSummary.trendData` retorna a série temporal ordenada por data com `date`, `weightKg` e `bmi`

**Tasks Frontend:**
- [ ] TF-06.2.1 — Criar componente `WeightChart` com Recharts: `LineChart` + `XAxis` (dias) + `YAxis` (auto domain) + `Tooltip` customizado + `Line` (type linear, pontos circulares raio 4px, cor `--accent`)
- [ ] TF-06.2.2 — Implementar `CustomTooltip`: card com data formatada em pt-BR, peso e IMC + categoria

---

### ST-06.3 — Navegar entre meses no dashboard

> Como usuário, quero filtrar o dashboard por ano e mês, para visualizar meu histórico de períodos anteriores.

**Critérios de aceitação:**
- [ ] Navegação pelas setas atualiza dashboard e gráfico sem recarregar a página
- [ ] Meses futuros desabilitados
- [ ] Meses anteriores à criação da conta não disponíveis
- [ ] Dropdown exibe apenas meses dentro do intervalo permitido

**Tasks Frontend:**
- [ ] TF-06.3.1 — Integrar `MonthSelector` com `useMonthFilter` na `DashboardPage`
- [ ] TF-06.3.2 — Criar `DashboardPage`: `useQuery` para `getMonthSummary` com `queryKey: ['month-summary', year, month]`, compor `MonthSelector` + `WeightCards` + `WeightChart` + empty state + botão "Registrar peso hoje"

---

## EP-07 — Qualidade e finalização

---

### ST-07.1 — Tratamento de erros consistente na API

> Como desenvolvedor, quero que todos os erros da API retornem no formato definido no `openapi.yaml`, para que o frontend trate as respostas de forma previsível.

**Critérios de aceitação:**
- [ ] Erros de validação (422) retornam `ValidationErrorResponse` com mapa campo → mensagem
- [ ] Erros de negócio (400, 409) retornam `ErrorResponse` com mensagem clara
- [ ] Erros não esperados (500) retornam mensagem genérica (sem stack trace)

**Tasks Backend:**
- [ ] TB-07.1.1 — Criar exceções de negócio: `DuplicateRecordException`, `InvalidTokenException`, `ResourceNotFoundException`, `InvalidCredentialsException`
- [ ] TB-07.1.2 — Criar `GlobalExceptionHandler` com `@RestControllerAdvice`: mapear cada exceção para o status HTTP e schema correto do `openapi.yaml`
- [ ] TB-07.1.3 — Garantir que erros de validação Bean Validation (`@Valid`) retornam `ValidationErrorResponse`

**Tasks Frontend:**
- [ ] TF-07.1.1 — Adicionar tratamento global no interceptor Axios: erros 500 e de rede disparam toast genérico via `Toast`
- [ ] TF-07.1.2 — Garantir que erros 422 exibem mensagens por campo no formulário correspondente

---

### ST-07.2 — Testes automatizados

> Como desenvolvedor, quero testes automatizados nos serviços e controllers principais, para garantir que as regras de negócio estão corretas.

**Critérios de aceitação:**
- [ ] `BmiService` coberto com testes unitários para todas as categorias OMS
- [ ] `WeightRecordService` com testes para: criar registro, tentar duplicar, editar, excluir
- [ ] `AuthService` com testes para: register, login, refresh token, logout
- [ ] Controller de weight records com teste de integração validando contrato com `openapi.yaml`

**Tasks Backend:**
- [ ] TB-07.2.1 — Escrever testes unitários para `BmiService` (todos os ranges de categoria)
- [ ] TB-07.2.2 — Escrever testes unitários para `WeightRecordService` (criar, duplicar, editar, soft delete)
- [ ] TB-07.2.3 — Escrever testes unitários para `AuthService` (register, login com senha errada, refresh, logout)
- [ ] TB-07.2.4 — Escrever teste de integração para `WeightRecordController` com MockMvc

---

### ST-07.3 — Tema e experiência visual

> Como usuário, quero que o tema claro e escuro funcione em todas as telas, para usar o app no modo que preferir.

**Critérios de aceitação:**
- [ ] Toggle de tema na sidebar alterna entre light e dark
- [ ] Preferência persiste após fechar e reabrir o navegador
- [ ] Todos os tokens CSS são aplicados corretamente nas duas variantes
- [ ] Sem flicker de tema ao carregar a página

**Tasks Frontend:**
- [ ] TF-07.3.1 — Garantir que `themeStore` aplica `data-theme` no `<html>` antes do primeiro render (evitar flicker)
- [ ] TF-07.3.2 — Revisar todas as telas em modo dark: verificar contraste, bordas e cores de badge
- [ ] TF-07.3.3 — Testar navegação por teclado nos modais (foco gerenciado ao abrir/fechar) e labels nos inputs para acessibilidade básica

---

## Resumo do backlog

| Epic                         | Stories        | Tasks Backend   | Tasks Frontend   |
|------------------------------|----------------|-----------------|------------------|
| EP-01 — Setup                | 2              | 9               | 10               |
| EP-02 — Autenticação         | 4              | 9               | 7                |
| EP-03 — Redefinição de senha | 2              | 4               | 4                |
| EP-04 — Perfil e conta       | 4              | 8               | 8                |
| EP-05 — Registros de peso    | 4              | 8               | 9                |
| EP-06 — Dashboard            | 3              | 3               | 6                |
| EP-07 — Qualidade            | 3              | 6               | 3                |
| **Total**                    | **22 stories** | **47 tasks**    | **47 tasks**     |
