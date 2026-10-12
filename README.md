# NetAdmin – gestão dos links de intranet da PMESP

Sistema web que gerencia os links de intranet de cerca de 2.100 unidades da
Polícia Militar do Estado de São Paulo. Criei o sistema por iniciativa própria
na Seção de Redes. Em produção desde jul/2026.

> Sistema interno da PMESP: código, telas e dados são privados. Este repositório
> apresenta só o problema, a arquitetura e as decisões técnicas.

## O problema

```mermaid
flowchart LR
    subgraph Antes
        direction TB
        S1[5 sistemas separados<br/>entre eles uma planilha Excel<br/>e um sistema legado]
    end
    subgraph Depois
        direction TB
        N[NetAdmin<br/>fonte única de dados]
    end
    Antes -->|unificação| Depois
```

O fluxo de pedidos de link estava dividido em 5 sistemas. Isso gerava retrabalho
e erros de digitação. O NetAdmin virou a fonte única de dados de todas as seções.

## Arquitetura

Três públicos num mesmo deploy, cada um com o próprio host no Nginx:

```mermaid
flowchart LR
    A([Administradores]) --> H1[Painel administrativo]
    F([Unidades]) --> H2[Formulários públicos<br/>com rate limit]
    C([Consulta pública]) --> H3[Consulta por IP]
    H1 & H2 & H3 --> NG[Nginx<br/>TLS, um host por público]
    NG --> API[API FastAPI]
    API --> PG[(PostgreSQL<br/>pg_trgm)]
    API --> AD[Active Directory<br/>LDAP]
```

- **Painel administrativo:** cadastro e aprovação de solicitações, com login
  pelo Active Directory, tokens JWT e privilégios por pessoa validados no servidor.
- **Formulários públicos:** novo link, upgrade, mudança de endereço, desativação
  e atualização cadastral, com validações contra erro de digitação.
- **Consulta pública** por IP.
- Geração de modelos de e-mail a partir das solicitações.

## Backend em camadas

```mermaid
flowchart LR
    R[Rotas] --> SV[Serviços] --> M[SQLAlchemy async] --> DB[(PostgreSQL)]
    R -.-> SC[Schemas Pydantic]
    MG[Migrações Alembic] --> DB
```

## Qualidade

- 257 testes automatizados (172 de backend, 85 de frontend), contagem de 2026-09-18.
- Lint sem avisos e CI no GitHub Actions.
- Deploy com Docker Compose e Nginx (HTTPS) em servidor Linux no data center da
  corporação, com operação por SSH.

## Stack

Python · FastAPI · SQLAlchemy (async) · Alembic · PostgreSQL · React 19 ·
TypeScript · Vite · styled-components · React Hook Form · Zod · Docker · Nginx ·
GitHub Actions · Linux

---

[Leonardo Richeto](https://github.com/Leonardo-Richeto) ·
[LinkedIn](https://www.linkedin.com/in/leonardo-richeto-183a37215)
