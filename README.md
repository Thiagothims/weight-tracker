# Weight Tracker

Aplicação web para registro e acompanhamento de peso corporal. O usuário registra seu peso diariamente e visualiza sua evolução através de gráficos e resumos mensais, com cálculo automático de IMC baseado na altura cadastrada no perfil.

## Stack

**Backend**
- Java 21 + Spring Boot 3
- Spring Security + JWT (JJWT)
- Spring Data JPA + PostgreSQL
- Liquibase (migrations)
- MapStruct (mapeamento de objetos)
- OpenAPI Generator (geração de interfaces a partir do contrato)
- Docker Compose (banco local)

**Frontend**
- React 18 + TypeScript
- Vite
- React Router v6
- TanStack Query + Axios
- React Hook Form + Zod
- Zustand (estado global)
- Recharts (gráficos)
- orval (geração de cliente HTTP a partir do contrato)
- CSS Modules + tokens CSS (tema light/dark)

## Funcionalidades

- Cadastro e autenticação com JWT + silent refresh
- Redefinição de senha por e-mail
- Perfil com nome e altura; alteração de e-mail e senha
- Registro diário de peso (um por dia, data definida pelo servidor)
- Tabela de registros do mês com edição inline e exclusão
- Dashboard com gráfico de evolução e resumo mensal
- Cálculo de IMC com categoria OMS
- Tema claro e escuro com persistência
