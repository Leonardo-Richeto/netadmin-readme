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
        a1([Unidade]) --> a2[0800] --> a3[Planilha Excel<br/>de solicitação] --> a4[E-mail] --> a5[Chamado no Cherwell]
        a5 --> a6[Planilha Excel<br/>de controle] --> a7[E-mail ao órgão<br/>público fiscalizador] --> a8[Ferramenta de intermediação<br/>com a operadora] --> a9([Operadora instala o link])
        a6 --> a10[E-mails de aviso<br/>ao NOC]
    end
    subgraph Depois
        direction TB
        d1([Unidade]) --> d2[NetAdmin<br/>solicitação e controle] --> d3[E-mail ao órgão<br/>público fiscalizador<br/>montado pelo NetAdmin] --> d4[Ferramenta de intermediação<br/>com a operadora] --> d5([Operadora instala o link])
        d2 --> d6[E-mails de aviso ao NOC<br/>montados pelo NetAdmin]
    end
    Antes ~~~ Depois

    classDef removido stroke:#f85149,stroke-width:2px,stroke-dasharray:5 3
    classDef netadmin stroke:#3fb950,stroke-width:3px
    class a2,a3,a4,a5,a6 removido
    class d2 netadmin
```

Em vermelho tracejado, as etapas que o NetAdmin eliminou: o 0800, as planilhas
Excel, o e-mail da planilha e o chamado no Cherwell. O NetAdmin virou a fonte
única de dados das seções.

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
