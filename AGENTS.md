Vou gerar um `agents.md` **extremamente detalhado**, bilíngue (Português → English), com guias de configuração, padrões de código, estrutura de diretórios expandida e instruções claras para agentes de IA e desenvolvedores. O documento será muito maior e cobrirá todos os aspectos que você mencionou.

---

```markdown
<!-- agents.md – Tenentis OS -->
<div align="center">
  <h1>🤖 Tenentis OS – Guia do Agente / Agent Guide</h1>
  <p><strong>Documentação essencial para agentes de IA e desenvolvedores</strong></p>
  <p>
    <a href="#-português">🇧🇷 Português</a> •
    <a href="#-english">🇬🇧 English</a>
  </p>
</div>

---

## 🇧🇷 Português

### 1. Visão Geral do Projeto

**Tenentis OS** é um motor SaaS multi‑tenant completo, ainda em estágio de scaffold. Atualmente, o repositório contém apenas a estrutura de diretórios e um `README.md`. Nenhum código fonte real foi implementado – todos os arquivos são stubs vazios. Cabe ao agente compreender a arquitetura pretendida e começar a implementar os componentes, respeitando as decisões de design já documentadas.

### 2. Arquitetura Planejada

A arquitetura é distribuída em quatro grandes domínios:

#### a) Backend (Node.js + TypeScript)
- **API Gateway**: ponto de entrada único, autenticação JWT, rate limiting.
- **Schema Engine**: entidades customizáveis por tenant, validadas via JSON Schema.
- **Segurança**: RLS (Row Level Security) no PostgreSQL, políticas ABAC.
- **Comunicação assíncrona**: publicação de eventos no RabbitMQ/Kafka para workers.

#### b) Workers (Java 21 + Spring Boot)
Processamento pesado em background com Virtual Threads (Project Loom):
- **audit-worker**: consome `tenant-events` e persiste logs de auditoria.
- **notification-worker**: consome `notification-events` e envia e‑mails.
- **report-worker**: consome `report-events`, gera CSVs com dados de tenants.
- **llm-worker**: consome `llm-events`, sugere schemas JSON via OpenAI e executa RAG.

#### c) Plataforma de IA (Python)
Serviços independentes, containerizados:
- **rag-engine**: motor de Retrieval‑Augmented Generation com Qdrant.
- **embeddings**: geração de embeddings (OpenAI, SentenceTransformers, etc.).
- **model-hub**: catálogo e versionamento de modelos fine‑tunados.

#### d) Frontends (Next.js 14 + Turborepo)
- **admin-portal**: gestão de tenants, usuários, configurações globais.
- **tenant-portal**: interface white‑label para cada tenant.
- **shared-ui**: biblioteca de componentes reutilizáveis (shadcn/ui + Tailwind).

Toda a comunicação entre serviços é assíncrona (via broker) e o isolamento de dados é garantido pelo PostgreSQL RLS, acionado por `SET app.current_tenant_id` a cada requisição.

### 3. Estrutura de Diretórios Expandida

```
tenentis-os/
├── src/                              # Backend TypeScript
│   ├── api/
│   │   ├── controllers/              # Lógica das rotas
│   │   ├── middlewares/              # auth, tenant, release-client, error handler
│   │   └── routes/                   # Definição de endpoints
│   ├── services/                     # Casos de uso
│   ├── repositories/                 # Acesso a dados (com RLS)
│   ├── db/
│   │   ├── connection.ts             # Pool de conexões com PostgreSQL
│   │   ├── migrations/               # Scripts de migração (node-pg-migrate)
│   │   └── seeds/                    # Dados iniciais
│   ├── schema/                       # Schema Engine (validação, versionamento)
│   ├── security/                     # JWT, bcrypt, RBAC/ABAC
│   ├── events/                       # Publicação de eventos (RabbitMQ/Kafka)
│   └── utils/                        # Helpers, logger, constantes
│
├── workers/                          # Workers Java 21 + Spring Boot
│   ├── audit-worker/
│   │   ├── pom.xml
│   │   ├── Dockerfile
│   │   └── src/main/java/com/tenentis/audit/
│   ├── notification-worker/
│   ├── report-worker/
│   └── llm-worker/
│       ├── agents/                   # SchemaSuggester (OpenAI)
│       └── rag/                      # TenantRagEngine (Qdrant)
│
├── frontend/                         # Monorepo Turborepo
│   ├── admin-portal/                 # Next.js 14
│   ├── tenant-portal/
│   └── shared-ui/                    # Componentes compartilhados
│
├── ai-platform/                      # Serviços Python
│   ├── rag-engine/
│   ├── embeddings/
│   └── model-hub/
│
├── shared/                           # Tipos, constantes, API client
│   ├── types/                        # Interfaces TypeScript compartilhadas
│   ├── constants/
│   └── utils/
│
├── tests/                            # Testes automatizados
│   ├── unit/
│   ├── integration/
│   ├── e2e/                          # Playwright
│   └── performance/                  # k6
│
├── infrastructure/                   # IaC & CI/CD
│   ├── docker/                       # Docker Compose, Dockerfiles
│   ├── kubernetes/                   # Manifests K8s
│   ├── terraform/                    # Provisionamento cloud
│   ├── monitoring/                   # Prometheus, Grafana, alertas
│   └── ci-cd/                        # GitHub Actions, GitLab CI
│
└── docs/                             # Documentação adicional
    ├── agents.md                     # Este arquivo
    ├── architecture.md
    └── rls-guide.md
```

### 4. Estado Atual do Repositório

- **Commit inicial**: apenas scaffold de diretórios e `README.md`.
- **Arquivos de configuração vazios**: `package.json`, `tsconfig.json`, `turbo.json`, `pnpm-workspace.yaml`, etc.
- **Nenhum código** existe em qualquer arquivo fonte.

### 5. O que um Agente Deve Fazer

#### a) Configuração Inicial do Ambiente
1. Criar `package.json` raiz (ou por workspace) com scripts de `dev`, `build`, `test`, `lint`.
2. Configurar `tsconfig.json` com paths para os módulos.
3. Se usar pnpm workspaces, criar `pnpm-workspace.yaml`.
4. Configurar Turborepo (`turbo.json`) para orquestrar builds e testes.
5. Criar `docker-compose.yml` para subir PostgreSQL, RabbitMQ, Redis, Qdrant.
6. Definir `Dockerfile` para cada serviço (já existem stubs vazios).

#### b) Implementação Gradual
- Começar pelo backend: conexão com banco, middleware de tenant, autenticação.
- Depois workers: criar classes Java com listeners RabbitMQ e lógica de negócio.
- Frontends: inicializar projetos Next.js com Tailwind e shadcn/ui.
- IA: criar serviços Python com FastAPI, integração Qdrant.
- Testes: estruturar suites com Jest, Playwright, k6.
- Infraestrutura: escrever manifests Kubernetes, Terraform, CI/CD.

#### c) Padrões de Código Obrigatórios
- **Isolamento de tenant**: todo acesso a dados deve ser precedido por `SET app.current_tenant_id`.
- **Virtual Threads**: workers Java devem usar `spring.threads.virtual.enabled: true`.
- **Multi‑estágio no Docker**: builds com `maven:3.9-eclipse-temurin-21` e runtime com `eclipse-temurin:21-jre-alpine`.
- **Idempotência nos workers**: mensagens podem ser reprocessadas; implementar idempotência.
- **Logs estruturados**: JSON ou formato padrão com nível, tenant, traceId.
- **Testes**: seguir pirâmide de testes (muitos unitários, alguns de integração, poucos e2e).

### 6. Configuração de Desenvolvimento (Passo a Passo)

#### Pré‑requisitos
- Docker & Docker Compose
- Node.js 20+
- pnpm
- JDK 21 (opcional, pode usar contêiner)
- Python 3.11 (opcional, idem)

#### Inicializando o Projeto
```bash
# 1. Clonar repositório
git clone <repo-url>
cd tenentis-os

# 2. Iniciar infraestrutura
docker-compose up -d

# 3. Instalar dependências do backend
cd src
pnpm init                    # se package.json não existir
pnpm add express typescript ...
pnpm install
cd ..

# 4. Rodar migrações (assim que implementadas)
pnpm run migrate

# 5. Iniciar API em modo dev
pnpm run dev

# 6. Build e execução dos workers (via Docker)
cd workers/audit-worker
docker build -t audit-worker .
docker run --rm audit-worker
```

### 7. Exemplos de Implementação Essenciais

#### Middleware de Tenant (TypeScript)
```typescript
// src/api/middlewares/tenant.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { pool } from '../../db/connection';

export async function tenantContext(req: Request, res: Response, next: NextFunction) {
  const tenantId = req.headers['x-tenant-id'] as string;
  if (!tenantId) return res.status(400).json({ error: 'Tenant ID obrigatório' });
  
  const client = await pool.connect();
  try {
    await client.query('SET app.current_tenant_id = $1', [tenantId]);
    // Armazena client para release posterior (ver release-client middleware)
    (req as any).dbClient = client;
    next();
  } catch (err) {
    client.release();
    next(err);
  }
}
```

#### Worker Java (Audit Processor simplificado)
```java
@RabbitListener(queues = "${app.queue}")
public void process(Map<String, Object> event) {
    String tenantId = (String) event.get("tenantId");
    jdbcTemplate.execute("SET app.current_tenant_id = '" + tenantId + "'");
    jdbcTemplate.update("INSERT INTO audit_logs (...) VALUES (...)", ...);
}
```

### 8. Políticas de Contribuição
- Código deve ser escrito em inglês (nomes de variáveis, comentários podem ser em português).
- Commits seguem Conventional Commits (`feat:`, `fix:`, `docs:`, etc.).
- Pull Requests exigem revisão e passagem nos testes automatizados.
- Toda nova funcionalidade deve incluir testes unitários e, quando aplicável, de integração.

---

## 🇬🇧 English

### 1. Project Overview

**Tenentis OS** is a complete multi‑tenant SaaS engine, currently in scaffold stage. The repository contains only the directory skeleton and a `README.md`. No actual source code has been written – every file is an empty stub. The agent's mission is to understand the intended architecture and begin implementing components while adhering to documented design decisions.

### 2. Planned Architecture

Distributed across four major domains:

#### a) Backend (Node.js + TypeScript)
- **API Gateway**: single entry point, JWT auth, rate limiting.
- **Schema Engine**: per‑tenant customizable entities validated via JSON Schema.
- **Security**: RLS (Row Level Security) in PostgreSQL, ABAC policies.
- **Async communication**: publishes events to RabbitMQ/Kafka for workers.

#### b) Workers (Java 21 + Spring Boot)
Heavy background processing with Virtual Threads (Project Loom):
- **audit-worker**: consumes `tenant-events` and persists audit logs.
- **notification-worker**: consumes `notification-events` and sends emails.
- **report-worker**: consumes `report-events`, generates CSVs from tenant data.
- **llm-worker**: consumes `llm-events`, suggests JSON schemas via OpenAI and executes RAG.

#### c) AI Platform (Python)
Independent containerized services:
- **rag-engine**: Retrieval‑Augmented Generation engine using Qdrant.
- **embeddings**: embedding generation (OpenAI, SentenceTransformers, etc.).
- **model-hub**: cataloging and versioning of fine‑tuned models.

#### d) Frontends (Next.js 14 + Turborepo)
- **admin-portal**: tenant, user, and global settings management.
- **tenant-portal**: white‑label interface per tenant.
- **shared-ui**: reusable component library (shadcn/ui + Tailwind).

All inter‑service communication is asynchronous (via broker) and data isolation is enforced by PostgreSQL RLS, activated per‑request with `SET app.current_tenant_id`.

### 3. Expanded Directory Structure

```
tenentis-os/
├── src/                              # TypeScript backend
│   ├── api/
│   │   ├── controllers/
│   │   ├── middlewares/
│   │   └── routes/
│   ├── services/
│   ├── repositories/
│   ├── db/
│   │   ├── connection.ts
│   │   ├── migrations/
│   │   └── seeds/
│   ├── schema/
│   ├── security/
│   ├── events/
│   └── utils/
│
├── workers/                          # Java 21 + Spring Boot
│   ├── audit-worker/
│   ├── notification-worker/
│   ├── report-worker/
│   └── llm-worker/
│
├── frontend/                         # Turborepo monorepo
│   ├── admin-portal/
│   ├── tenant-portal/
│   └── shared-ui/
│
├── ai-platform/                      # Python services
│   ├── rag-engine/
│   ├── embeddings/
│   └── model-hub/
│
├── shared/                           # Shared types, constants, API client
├── tests/                            # Unit, integration, e2e, performance
├── infrastructure/                   # Docker, K8s, Terraform, monitoring, CI/CD
└── docs/
```

### 4. Current Repository State

- **Initial commit**: directory scaffold and `README.md` only.
- **Empty configuration files**: `package.json`, `tsconfig.json`, `turbo.json`, `pnpm-workspace.yaml`, etc.
- **No source code** exists in any file.

### 5. Agent Tasks

#### a) Initial Environment Setup
1. Create root `package.json` (or per‑workspace) with `dev`, `build`, `test`, `lint` scripts.
2. Configure `tsconfig.json` with path aliases.
3. If using pnpm workspaces, create `pnpm-workspace.yaml`.
4. Set up Turborepo (`turbo.json`) to orchestrate builds and tests.
5. Create `docker-compose.yml` for PostgreSQL, RabbitMQ, Redis, Qdrant.
6. Write `Dockerfile` for each service (existing stubs are empty).

#### b) Incremental Implementation
- Start with backend: DB connection, tenant middleware, authentication.
- Then workers: Java classes with RabbitMQ listeners and business logic.
- Frontends: scaffold Next.js projects with Tailwind and shadcn/ui.
- AI: create Python services with FastAPI and Qdrant integration.
- Tests: structure suites with Jest, Playwright, k6.
- Infrastructure: write Kubernetes manifests, Terraform, CI/CD pipelines.

#### c) Mandatory Coding Standards
- **Tenant isolation**: every data access must be preceded by `SET app.current_tenant_id`.
- **Virtual Threads**: Java workers must enable `spring.threads.virtual.enabled: true`.
- **Multi‑stage Docker**: build with `maven:3.9-eclipse-temurin-21`, runtime with `eclipse-temurin:21-jre-alpine`.
- **Idempotency in workers**: messages may be redelivered; implement idempotency.
- **Structured logging**: JSON or standard format with level, tenant, traceId.
- **Testing**: follow the test pyramid (many unit, some integration, few e2e).

### 6. Development Setup (Step by Step)

#### Prerequisites
- Docker & Docker Compose
- Node.js 20+
- pnpm
- JDK 21 (optional, use container)
- Python 3.11 (optional, use container)

#### Project Initialization
```bash
# 1. Clone repo
git clone <repo-url>
cd tenentis-os

# 2. Start infrastructure
docker-compose up -d

# 3. Install backend dependencies
cd src
pnpm init                    # if package.json missing
pnpm add express typescript ...
pnpm install
cd ..

# 4. Run migrations (once implemented)
pnpm run migrate

# 5. Start API in dev mode
pnpm run dev

# 6. Build & run workers (via Docker)
cd workers/audit-worker
docker build -t audit-worker .
docker run --rm audit-worker
```

### 7. Essential Implementation Examples

#### Tenant Middleware (TypeScript)
```typescript
// src/api/middlewares/tenant.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { pool } from '../../db/connection';

export async function tenantContext(req: Request, res: Response, next: NextFunction) {
  const tenantId = req.headers['x-tenant-id'] as string;
  if (!tenantId) return res.status(400).json({ error: 'Tenant ID required' });
  
  const client = await pool.connect();
  try {
    await client.query('SET app.current_tenant_id = $1', [tenantId]);
    (req as any).dbClient = client;
    next();
  } catch (err) {
    client.release();
    next(err);
  }
}
```

#### Java Worker (simplified Audit Processor)
```java
@RabbitListener(queues = "${app.queue}")
public void process(Map<String, Object> event) {
    String tenantId = (String) event.get("tenantId");
    jdbcTemplate.execute("SET app.current_tenant_id = '" + tenantId + "'");
    jdbcTemplate.update("INSERT INTO audit_logs (...) VALUES (...)", ...);
}
```

### 8. Contribution Policies
- Code must be written in English (variable names, comments may be in Portuguese).
- Commits follow Conventional Commits (`feat:`, `fix:`, `docs:`, etc.).
- Pull Requests require review and passing automated tests.
- Every new feature must include unit tests and, where applicable, integration tests.

---

<div align="center">
  <p>📘 <strong>Tenentis OS</strong> — Construído com isolamento real, escalabilidade e IA generativa.</p>
  <p>Built with true tenant isolation, scalability, and generative AI.</p>
</div>
```

---

