# UI/UX Spec — Weight Tracker

## 1. Princípios visuais

### Estilo
Minimalista. Poucos elementos, espaço em branco generoso, tipografia como elemento visual principal.
Sem gradientes, sem sombras pesadas, sem excessos decorativos.

### Paleta de cores

| Token              | Light     | Dark      | Uso                                |
|--------------------|-----------|-----------|------------------------------------|
| `--bg-base`        | `#F7F7F7` | `#111111` | Fundo da aplicação                 |
| `--bg-surface`     | `#FFFFFF` | `#1C1C1C` | Cards, modais, sidebar             |
| `--bg-subtle`      | `#EFEFEF` | `#252525` | Hover, inputs, linhas zebradas     |
| `--border`         | `#E0E0E0` | `#2E2E2E` | Divisores, bordas de input         |
| `--text-primary`   | `#111111` | `#F0F0F0` | Títulos, valores principais        |
| `--text-secondary` | `#6B6B6B` | `#9A9A9A` | Labels, datas, metadados           |
| `--text-disabled`  | `#BCBCBC` | `#444444` | Campos desabilitados               |
| `--accent`         | `#222222` | `#F0F0F0` | Botões primários, links ativos     |
| `--accent-fg`      | `#FFFFFF` | `#111111` | Texto sobre botão primário         |
| `--danger`         | `#C0392B` | `#E74C3C` | Excluir conta, erros críticos      |
| `--success`        | `#27AE60` | `#2ECC71` | Feedback de sucesso                |
| `--warning`        | `#E67E22` | `#F39C12` | Avisos (ex: altura não cadastrada) |

### Tipografia
- **Família:** Inter (Google Fonts) — sans-serif, legível em tamanhos pequenos
- **Escala:**
  | Uso | Tamanho | Peso |
  |---|---|---|
  | Título de página | 24px | 600 |
  | Subtítulo / label de seção | 13px | 500, uppercase, letter-spacing |
  | Valor numérico principal (peso, IMC) | 32px | 700 |
  | Corpo / texto corrido | 14px | 400 |
  | Legenda / metadado | 12px | 400 |

### Arredondamento e espaçamento
- Border radius padrão: `8px` (inputs, cards, badges)
- Border radius modal: `12px`
- Grid de espaçamento: múltiplos de `8px`

---

## 2. Layout geral

```
┌──────────────────────────────────────────────────────┐
│  SIDEBAR (240px fixo)  │  CONTENT AREA (flex)        │
│                        │                             │
│  [Logo / App name]     │  [Conteúdo da rota atual]   │
│                        │                             │
│  • Dashboard           │                             │
│  • Registros           │                             │
│  • Perfil              │                             │
│                        │                             │
│  ─────────────────     │                             │
│  [Toggle dark/light]   │                             │
│  [Avatar + nome]       │                             │
│  [Sair]                │                             │
└──────────────────────────────────────────────────────┘
```

### Sidebar — detalhes
- Largura fixa: `240px` no desktop
- Background: `--bg-surface`
- Borda direita: `1px solid --border`
- Item ativo: fundo `--bg-subtle`, texto `--text-primary`, barra lateral esquerda de `3px` na cor `--accent`
- Item inativo: texto `--text-secondary`
- **Toggle de tema** (ícone sol/lua) no rodapé da sidebar
- **Avatar** exibe inicial do nome do usuário em círculo com fundo `--bg-subtle`
- Botão **"Sair"** no rodapé com ícone, texto `--text-secondary`, sem destaque

### Responsividade
- Desktop (≥ 1024px): sidebar visível e fixa
- Tablet/Mobile (< 1024px): sidebar recolhida, abre via hamburguer menu no topo
  _(layout mobile é simplificado para MVP; foco no desktop)_

---

## 3. Telas públicas (sem autenticação)

### 3.1 — Login

```
┌────────────────────────────────────┐
│                                    │
│         Weight Tracker             │  ← nome do app, 24px, centrado
│                                    │
│  E-mail                            │
│  ┌──────────────────────────────┐  │
│  │ joao@email.com               │  │
│  └──────────────────────────────┘  │
│                                    │
│  Senha                             │
│  ┌──────────────────────────────┐  │
│  │ ••••••••           [👁]      │  │
│  └──────────────────────────────┘  │
│                                    │
│  [           Entrar             ]  │  ← botão primário, largura total
│                                    │
│  Esqueceu a senha?                 │  ← link texto, alinhado à esquerda
│                                    │
│  Não tem conta? Cadastre-se        │  ← link para /register
│                                    │
└────────────────────────────────────┘
```

- Formulário centralizado vertical e horizontalmente na tela
- Largura máxima do card: `400px`
- Background da página: `--bg-base`
- Card com `--bg-surface` e border `1px solid --border`
- Erro de credenciais: mensagem em `--danger` abaixo do botão, sem destacar campo específico
  (ex: _"E-mail ou senha inválidos"_)

---

### 3.2 — Cadastro

Mesma estrutura visual do Login, com campos adicionais:

```
  Nome
  ┌──────────────────────────────┐
  │ João Silva                   │
  └──────────────────────────────┘

  E-mail
  ┌──────────────────────────────┐
  │ joao@email.com               │
  └──────────────────────────────┘

  Senha
  ┌──────────────────────────────┐
  │ ••••••••           [👁]      │
  └──────────────────────────────┘

  [         Criar conta         ]

  Já tem conta? Entrar
```

- Erros de validação: exibidos abaixo de cada campo individualmente
- Requisitos de senha: exibidos como checklist abaixo do campo ao focar
  ```
  ✓ Mínimo 8 caracteres
  ✗ Letra maiúscula
  ✗ Número
  ✗ Caractere especial
  ```
  (ícone `✓` verde quando satisfeito, `✗` cinza quando não)

---

### 3.3 — Esqueci minha senha

```
  Redefinir senha

  Informe seu e-mail cadastrado e enviaremos
  um link para criar uma nova senha.

  E-mail
  ┌──────────────────────────────┐
  │                              │
  └──────────────────────────────┘

  [      Enviar link de acesso   ]

  ← Voltar para o login
```

- Após envio: substitui o formulário por mensagem de sucesso:
  _"Se este e-mail estiver cadastrado, você receberá um link em instantes."_
- Não indica se o e-mail existe ou não (segurança BR-05.2)

---

### 3.4 — Redefinir senha (via link do e-mail)

```
  Nova senha

  Crie uma senha forte para sua conta.

  Nova senha
  ┌──────────────────────────────┐
  │ ••••••••           [👁]      │
  └──────────────────────────────┘
  ✓ Mínimo 8 caracteres
  ✗ Letra maiúscula  ...

  [        Salvar nova senha     ]
```

- Se o token for inválido/expirado: exibe mensagem de erro com link para solicitar novo
  _"Este link expirou ou já foi utilizado. Solicitar novo link →"_
- Após sucesso: redireciona para `/login` com mensagem toast:
  _"Senha redefinida com sucesso. Faça login."_

---

## 4. Telas autenticadas

### 4.1 — Dashboard

```
┌─────────────────────────────────────────────────────────┐
│  Dashboard                    [+ Registrar peso hoje]   │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  ◀  Abril 2026  ▶                                │   │  ← seletor mês/ano
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌────────────────────┐  ┌───────────────────────────┐  │
│  │ PRIMEIRO PESO      │  │ ÚLTIMO PESO               │  │
│  │                    │  │                           │  │
│  │  70,20 kg          │  │  69,80 kg                 │  │
│  │  01 abr · 07:30    │  │  12 abr · 08:00           │  │
│  │                    │  │  [IMC 22,8 · Peso Normal] │  │  ← badge/tag
│  └────────────────────┘  └───────────────────────────┘  │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │                                                  │   │
│  │   Evolução do peso — Abril 2026                  │   │
│  │                                                  │   │
│  │  71 ─ ·                                          │   │
│  │  70 ─   · · ·  ·                                 │   │
│  │  69 ─         · · ·                              │   │
│  │       1  5  10  15  20  25  30                   │   │
│  │                                                  │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

**Seletor de mês/ano:**
- Setas `◀ ▶` para navegar entre meses
- Clique no texto abre dropdown com lista de meses disponíveis
- Meses futuros desabilitados; meses anteriores ao cadastro ocultos

**Cards de peso:**
- Dois cards lado a lado, largura igual
- Label em uppercase, 12px, `--text-secondary`
- Valor em 32px bold, `--text-primary`
- Data e horário em 12px, `--text-secondary`
- Badge de IMC no card "Último Peso": fundo `--bg-subtle`, borda `--border`, texto 12px
  - Se altura não cadastrada: badge laranja _"Cadastre sua altura"_ com link para perfil

**Gráfico de linha:**
- Biblioteca: **Recharts**
- Linha contínua na cor `--accent`
- Pontos de dados: círculo preenchido, raio 4px, cor `--accent`
- Tooltip ao hover: card com data + peso + IMC (se disponível)
- Eixo X: dias do mês (apenas dias com registro)
- Eixo Y: escala automática com margem de ±2kg
- Grid horizontal: linhas tracejadas em `--border`
- Sem linha de tendência suavizada
- Se apenas 1 ponto: exibe o ponto sem linha
- Se sem dados: exibe empty state dentro do container do gráfico

**Empty state (sem registros no mês):**
```
  ┌──────────────────────────────────────────┐
  │                                          │
  │   Nenhum peso registrado em Abril 2026   │
  │   Comece registrando seu peso hoje.      │
  │                                          │
  │   [+ Registrar peso hoje]                │
  │                                          │
  └──────────────────────────────────────────┘
```
- Exibido no lugar dos cards e do gráfico

**Botão "Registrar peso hoje":**
- Visível no topo direito da página
- Se já existe registro para hoje: botão não aparece (usuário vai à tabela de registros para editar)

---

### 4.2 — Registros

```
┌─────────────────────────────────────────────────────────┐
│  Registros                                              │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  ◀  Abril 2026  ▶                                │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌────────┬──────────┬──────────┬─────────┬──────────┐  │
│  │ Data   │ Horário  │ Peso     │ IMC     │ Ações    │  │
│  ├────────┼──────────┼──────────┼─────────┼──────────┤  │
│  │12 abr  │ 08:00    │ 69,80 kg │22,8 ·🟢 │ ✏  🗑     │  │
│  │11 abr  │ 07:45    │ 70,00 kg │22,9 ·🟢 │ ✏  🗑     │  │
│  │ ...    │          │          │         │          │  │
│  └────────┴──────────┴──────────┴─────────┴──────────┘  │
└─────────────────────────────────────────────────────────┘
```

**Tabela:**
- Ordenada por data decrescente (mais recente primeiro)
- Coluna IMC: valor numérico + badge colorido com categoria abreviada
  - `PESO_NORMAL` → badge verde sutil
  - `SOBREPESO` → badge amarelo sutil
  - `OBESIDADE_*` → badge vermelho sutil
  - `ABAIXO_DO_PESO` → badge azul sutil
  - Sem altura cadastrada: `—`
- Coluna Ações: ícone de lápis (editar) e ícone de lixeira (excluir)
- Hover na linha: fundo `--bg-subtle`

**Edição inline:**
- Ao clicar no ícone de lápis, a linha entra em modo de edição:
```
│  12 abr │ [08:00  ▼] │ [69,80  ] kg │ 22,8 ·🟢 │ ✓  ✗  │
```
- Campo de horário: input time
- Campo de peso: input numérico com step `0.01`
- `✓` salva, `✗` cancela
- Apenas uma linha pode estar em edição por vez
- ESC também cancela a edição

**Exclusão:**
- Ao clicar na lixeira: abre diálogo de confirmação inline acima da linha:
```
  Excluir registro de 12 abr? [Confirmar] [Cancelar]
```
- Sem modal — confirmação discreta e contextual

**Empty state:**
```
  Nenhum peso registrado em Abril 2026.
```

---

### 4.3 — Modal: Registrar novo peso

Abre a partir do botão "Registrar peso hoje" no Dashboard.

```
┌──────────────────────────────────────────┐
│  Registrar peso de hoje            [ ✕ ] │
│  Sábado, 12 de abril de 2026             │  ← data exibida, não editável
│                                          │
│  Horário da aferição                     │
│  ┌───────────────────────────────────┐   │
│  │ 07:30                             │   │
│  └───────────────────────────────────┘   │
│                                          │
│  Peso (kg)                               │
│  ┌───────────────────────────────────┐   │
│  │ 70,50                             │   │
│  └───────────────────────────────────┘   │
│                                          │
│  [Cancelar]          [Salvar registro]   │
└──────────────────────────────────────────┘
```

- Data exibida como texto descritivo (não é campo editável)
- Horário pré-preenchido com a hora atual do navegador
- Campo de peso: placeholder `0,00`, aceita vírgula e ponto como separador decimal
- Botão "Salvar": desabilitado até os dois campos serem preenchidos
- Após salvar: modal fecha, dashboard atualiza, toast de sucesso:
  _"Peso registrado com sucesso."_
- Fechar com `✕`, `Cancelar` ou `ESC`

---

### 4.4 — Perfil

```
┌─────────────────────────────────────────────────────────┐
│  Perfil                                                 │
│                                                         │
│  ── INFORMAÇÕES PESSOAIS ─────────────────────────────  │
│                                                         │
│  Nome                                                   │
│  ┌──────────────────────────────────────────────────┐   │
│  │ João Silva                                       │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  Altura (cm)                                            │
│  ┌──────────────────────────────────────────────────┐   │
│  │ 175                                              │   │
│  └──────────────────────────────────────────────────┘   │
│  Utilizada para o cálculo do IMC.                       │
│                                                         │
│  [           Salvar alterações               ]          │
│                                                         │
│                                                         │
│  ── CONTA E SEGURANÇA ────────────────────────────────  │
│                                                         │
│  E-mail atual: joao@email.com          [Alterar e-mail] │
│                                                         │
│  Senha                                  [Alterar senha] │
│                                                         │
│                                                         │
│  ── ZONA DE PERIGO ───────────────────────────────────  │
│                                                         │
│  Excluir conta                                          │
│  Todos os seus dados serão anonimizados                 │
│  permanentemente. Esta ação é irreversível.             │
│                                                         │
│  [          Excluir minha conta              ]          │  ← botão --danger
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Seção "Conta e Segurança":**
- "Alterar e-mail" e "Alterar senha": abrem painéis expansíveis (accordion) inline
- Não navega para outra página

**Painel expandido — Alterar e-mail:**
```
  E-mail atual: joao@email.com          [▲ Fechar]

  Novo e-mail
  ┌──────────────────────────────────────┐
  │                                      │
  └──────────────────────────────────────┘

  Confirme sua senha
  ┌──────────────────────────────────────┐
  │ ••••••••                   [👁]      │
  └──────────────────────────────────────┘

  [Cancelar]           [Salvar novo e-mail]
```

**Painel expandido — Alterar senha:**
```
  Senha atual
  ┌──────────────────────────────────────┐
  │ ••••••••                   [👁]      │
  └──────────────────────────────────────┘

  Nova senha
  ┌──────────────────────────────────────┐
  │ ••••••••                   [👁]      │
  └──────────────────────────────────────┘
  ✓ Mínimo 8 caracteres  ✗ Maiúscula ...

  [Cancelar]              [Salvar senha]
```

**Modal — Confirmar exclusão de conta:**
```
┌──────────────────────────────────────────┐
│  Excluir conta                    [ ✕ ] │
│                                          │
│  Esta ação é irreversível. Todos os seus │
│  dados serão anonimizados.               │
│                                          │
│  Para confirmar, digite sua senha:       │
│  ┌───────────────────────────────────┐   │
│  │ ••••••••               [👁]       │   │
│  └───────────────────────────────────┘   │
│                                          │
│  [Cancelar]       [Excluir minha conta]  │
│                                [danger]  │
└──────────────────────────────────────────┘
```
- Botão "Excluir minha conta" desabilitado até a senha ser digitada
- Após confirmação: logout automático + redirect para `/login` com toast:
  _"Conta encerrada. Até logo."_

---

## 5. Componentes globais

### Toast / Notificações
- Posição: canto inferior direito
- Duração: 4 segundos (auto-dismiss)
- Variantes: sucesso (borda `--success`), erro (borda `--danger`), neutro (borda `--border`)
- Empilhados se múltiplos

### Feedback de carregamento
- Botões: substituir texto por spinner inline ao submeter
- Tabela / gráfico: skeleton loader (retângulos animados em `--bg-subtle`)
- Sem spinners de página inteira

### Estados de erro de API
- Erros `422` de validação: mensagem abaixo do campo específico
- Erros `400`/`409`: mensagem acima do botão de submit, dentro do formulário
- Erros `500`: toast de erro genérico — _"Ocorreu um erro inesperado. Tente novamente."_
- Erro de rede/offline: toast — _"Sem conexão. Verifique sua internet."_

### Inputs
- Border: `1px solid --border`
- Focus: border `--accent`, sem glow/shadow
- Erro: border `--danger`, mensagem em `--danger` abaixo
- Height padrão: `40px`
- Padding: `0 12px`

### Botões
| Variante   | Fundo        | Texto              | Border         | Uso              |
|------------|--------------|--------------------|----------------|------------------|
| Primary    | `--accent`   | `--accent-fg`      | nenhum         | Ação principal   |
| Secondary  | transparente | `--text-primary`   | `1px --border` | Ação secundária  |
| Ghost      | transparente | `--text-secondary` | nenhum         | Cancelar, fechar |
| Danger     | `--danger`   | `#fff`             | nenhum         | Excluir conta    |

---

## 6. Navegação e rotas

| Rota                | Tela                | Acesso                          |
|---------------------|---------------------|---------------------------------|
| `/login`            | Login               | Público                         |
| `/register`         | Cadastro            | Público                         |
| `/forgot-password`  | Esqueci minha senha | Público                         |
| `/reset-password`   | Redefinir senha     | Público (token via query param) |
| `/` ou `/dashboard` | Dashboard           | Autenticado                     |
| `/records`          | Registros           | Autenticado                     |
| `/profile`          | Perfil              | Autenticado                     |

**Regras de redirecionamento:**
- Usuário não autenticado tentando acessar rota protegida → redirect `/login`
- Usuário autenticado acessando `/login` ou `/register` → redirect `/dashboard`
- Token de reset inválido/expirado em `/reset-password` → mensagem de erro inline (sem redirect)
