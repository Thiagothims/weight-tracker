# Spec Task Runner

Execute tasks do projeto seguindo o fluxo de spec driven development definido em `spec/tasks.md`.

## Como usar

`/spec-task <alvo>`

O argumento `<alvo>` é **obrigatório** e define o escopo de execução:
- `EP-01` — executa todas as tasks pendentes do épico
- `ST-01.1` — executa todas as tasks pendentes da story
- `TB-01.1.3` ou `TF-01.2.5` — executa uma task específica

Se o argumento não for fornecido, responda com: `Informe o alvo: um épico (EP-XX), uma story (ST-XX.Y) ou uma task (TB-XX.Y.Z / TF-XX.Y.Z).` e pare.

## O que fazer

### Fase 1 — Carregar contexto

1. Leia **todos** os arquivos de spec em `spec/`:
   - `spec/tasks.md` — backlog com status de cada task (checkboxes)
   - `spec/business-rules.md` — regras de negócio
   - `spec/data-model.md` — modelo de dados
   - `spec/design.md` — guia de design e estrutura de pastas
   - `spec/uiux-spec.md` — especificação de UX/UI
   - `spec/notes.md` — desvios e decisões registradas (se existir)

2. Inspecione a estrutura atual do projeto para entender o que já foi implementado:
   - Liste os arquivos existentes nas pastas relevantes (backend e/ou frontend)
   - Se houver código já escrito, leia os arquivos-chave para entender os padrões adotados

### Fase 2 — Identificar o alvo

- Localize o alvo informado em `spec/tasks.md`
- Para épico: colete todas as tasks com `[ ]` dentro de todas as stories do épico
- Para story: colete todas as tasks com `[ ]` dentro da story
- Para task individual: localize a task pelo ID exato
- Se o alvo não existir em `spec/tasks.md`, informe o erro e pare

### Fase 3 — Executar a(s) task(s)

Para cada task pendente no escopo identificado, execute na ordem:

#### 3.1 — Antes de implementar

- Releia a task e seus critérios de aceitação da story pai
- Verifique nos arquivos de spec qual é o comportamento esperado
- Inspecione arquivos relacionados que já existem para manter consistência de estilo e padrões

#### 3.2 — Implementar

Execute a implementação conforme descrito na task:

- **Tasks de backend (TB-):** Java/Spring Boot em `backend/src/`
  - Siga os padrões já existentes no projeto (se houver código)
  - Annotations JPA, Lombok, MapStruct, Spring conforme `spec/data-model.md`
  - Endpoints aderentes ao `openapi.yaml` em `src/main/resources/spec/`
  - Migrations Liquibase em `db/changelog/` para qualquer mudança de schema

- **Tasks de frontend (TF-):** React/TypeScript em `frontend/src/`
  - Siga a estrutura de pastas de `spec/design.md`
  - Use os tokens CSS de `styles/variables.css`
  - Componentes com React Hook Form + Zod quando há formulário
  - TanStack Query (`useQuery`/`useMutation`) para chamadas à API
  - Zustand para estado global

- **Tasks de infraestrutura:** arquivos de configuração na raiz do projeto

#### 3.3 — Marcar como concluída

Imediatamente após concluir cada task, atualize `spec/tasks.md`:
- Troque `- [ ] TB-XX.Y.Z` por `- [x] TB-XX.Y.Z` (preservando o texto exato)
- Faça isso **antes** de passar para a próxima task

#### 3.4 — Registrar desvios

Se durante a implementação houver qualquer decisão não óbvia em relação ao spec:
- Abra (ou crie) `spec/notes.md`
- Adicione uma entrada no formato:

```markdown
## [DATA] Task TB-XX.Y.Z / TF-XX.Y.Z

**Decisão:** [o que foi feito diferente ou por quê]
**Motivo:** [razão técnica ou de contexto]
```

### Fase 4 — Verificar critérios de aceitação

Ao final de cada story completamente implementada (todas as tasks `[x]`):

1. Releia os **Critérios de aceitação** da story em `spec/tasks.md`
2. Para cada critério, confirme que a implementação o satisfaz
3. Se algum critério não foi atendido, identifique qual task responsável e implemente o que falta

### Fase 5 — Reportar

Exiba um sumário ao final:

```
## Spec Task — Resultado

**Escopo executado:** [épico / story / task ID]
**Tasks concluídas:** N
**Tasks ignoradas (já feitas):** N

### Implementado
- [TX-XX.Y.Z] descrição breve do que foi criado/modificado

### Arquivos criados/modificados
- `caminho/do/arquivo.java` — [o que contém]

### Desvios registrados
- [se houver] referência ao spec/notes.md

### Próxima task pendente
- [TX-XX.Y.Z] — [descrição]
```

## Regras importantes

- **Nunca pule uma task** com `[ ]` sem implementar — a ordem importa (dependências entre tasks)
- **Nunca marque `[x]`** uma task que não foi implementada nesta execução
- **Sempre leia o spec** antes de codificar — não suponha, confirme
- **Mantenha consistência** com o código já existente: antes de criar um padrão, verifique se já existe um
- **Um arquivo de migration por mudança de schema** — nunca DDL manual
- **Segurança primeiro**: ownership de recursos sempre validado; dados sensíveis nunca em logs
