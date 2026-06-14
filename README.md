# Tenentis OS
> The Extensible SaaS Engine 

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Build](https://img.shields.io/badge/build-passing-brightgreen.svg)
![TypeScript](https://img.shields.io/badge/typescript-%23007ACC.svg?style=flat&logo=typescript&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/postgresql-%23316192.svg?style=flat&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=flat&logo=docker&logoColor=white)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)

*(Read the Portuguese version below / Leia a versão em português abaixo)*

**Tenentis OS** is an enterprise-grade, high-performance multi-tenant backend architecture. It is designed to provide absolute data isolation, extreme schema flexibility, and robust access control. Built as a foundational layer, it allows SaaS products to scale seamlessly without compromising on security or performance.

## The Meaning Behind the Name

*   **Tenentis:** Derived from the Latin root *tenere* (to hold or keep) and closely related to the word "tenant." It reflects the system's core purpose: securely holding and managing multiple distinct entities (tenants) within a unified environment.
*   **OS (Operating System):** It is not a traditional operating system, but rather an architectural foundation. Just as an OS manages hardware resources for different software applications, Tenentis OS manages database resources, routing, and schema validation for different clients, acting as the fundamental layer upon which your business logic is built.

## How It Works: The Core Mechanics

The system operates on a Zero-Trust architecture, ensuring that data leakage between tenants is mathematically impossible at the database level, regardless of application-level bugs.

### 1. The Request Lifecycle
1.  **Authentication & Context:** The client sends a request with a JWT.
2.  **Middleware Extraction:** The application middleware validates the token and extracts the `tenant_id` and the user's `permissions_scope`.
3.  **Database Context Injection:** Before executing any query, the application sets a strictly scoped session variable in PostgreSQL.
4.  **RLS Enforcement:** The PostgreSQL engine intercepts the query. The Row Level Security (RLS) policy reads the session variable and automatically filters the data. If the session variable is missing, the query returns an empty result set.

### 2. The Dynamic Schema Engine
1.  **Schema Registry:** An administrator or tenant defines the shape of their data (e.g., "Tenant A" needs a 'Tax ID' field for their customers). This definition is stored as a JSON Schema rule.
2.  **Runtime Validation:** When a user attempts to insert data, the API layer intercepts the payload and validates it against the tenant's specific rules registered in the system.
3.  **JSONB Persistence:** Validated custom data is stored within a `JSONB` column inside standard relational tables.
4.  **GIN Indexing:** To ensure lookups on these dynamic fields remain lightning-fast, Functional GIN Indexes are applied, allowing the database to search inside the JSON payload with the same performance as a standard column.

## Architectural Pillars

*   **Absolute Data Isolation:** Relying on PostgreSQL RLS instead of simple "WHERE tenant_id = X" clauses ensures that forgotten application filters never lead to data breaches.
*   **Attribute-Based Access Control (ABAC):** Beyond standard "Admin/User" roles, permissions are granular and context-aware, embedded directly into the authentication flow.
*   **Meta-Programming Flexibility:** The backend is agnostic to the specific business model. It adapts to the tenant's needs, solving the classic SaaS problem of "custom fields" without requiring complex database migrations.
*   **Performance at Scale:** Strategic use of Partial Indexes and Read Replicas (CQRS pattern) ensures that heavy reporting queries do not impact the transactional performance of the main application.

## Getting Started

1.  Clone the repository using: `git clone https://github.com/your-username/tenentis-os.git`
2.  Start the database and API via Docker using: `docker-compose up -d`
3.  Access the API documentation at `http://localhost:3000/docs`.

---
---

# Tenentis OS
> O Motor Extensível para SaaS

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Build](https://img.shields.io/badge/build-passing-brightgreen.svg)
![TypeScript](https://img.shields.io/badge/typescript-%23007ACC.svg?style=flat&logo=typescript&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/postgresql-%23316192.svg?style=flat&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=flat&logo=docker&logoColor=white)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)

O **Tenentis OS** é uma arquitetura de backend multi-tenant de alta performance e nível corporativo. Ele foi projetado para fornecer isolamento absoluto de dados, extrema flexibilidade de schema e controle de acesso robusto. Construído como uma camada fundamental, permite que produtos SaaS escalem de forma contínua sem comprometer a segurança ou a performance.

## O Significado do Nome

*   **Tenentis:** Derivado da raiz latina *tenere* (segurar, manter) e intimamente ligado à palavra "tenant" (inquilino/cliente). Reflete o propósito central do sistema: abrigar e gerenciar com segurança múltiplas entidades distintas dentro de um ambiente unificado.
*   **OS (Operating System):** Não é um sistema operacional tradicional, mas sim uma fundação arquitetural. Assim como um SO gerencia recursos de hardware para diferentes aplicações, o Tenentis OS gerencia recursos de banco de dados, roteamento e validação de schema para diferentes clientes, atuando como a camada fundamental sobre a qual a lógica de negócios é construída.

## Como Funciona: A Mecânica Central

O sistema opera em uma arquitetura Zero-Trust (Confiança Zero), garantindo que o vazamento de dados entre clientes seja matematicamente impossível no nível do banco de dados, independentemente de falhas na camada de aplicação.

### 1. O Ciclo de Vida da Requisição
1.  **Autenticação e Contexto:** O cliente envia uma requisição contendo um JWT.
2.  **Extração no Middleware:** O middleware da aplicação valida o token e extrai o `tenant_id` e o `permissions_scope` do usuário.
3.  **Injeção de Contexto no Banco:** Antes de executar qualquer consulta, a aplicação define uma variável de sessão estritamente isolada no PostgreSQL.
4.  **Execução do RLS:** O motor do PostgreSQL intercepta a consulta. A política de Row Level Security (Segurança em Nível de Linha) lê a variável de sessão e filtra os dados automaticamente. Se a variável de sessão não for definida, o banco retorna um resultado vazio.

### 2. O Motor de Schema Dinâmico
1.  **Registro de Schema:** Um administrador ou tenant define o formato de seus dados (ex: "Tenant A" precisa de um campo 'CNPJ' para seus clientes). Essa definição é salva como uma regra de JSON Schema.
2.  **Validação em Tempo de Execução:** Quando um usuário tenta inserir dados, a camada da API intercepta o payload e o valida contra as regras específicas daquele tenant registradas no sistema.
3.  **Persistência em JSONB:** Os dados customizados validados são armazenados dentro de uma coluna `JSONB` em tabelas relacionais padrão.
4.  **Indexação GIN:** Para garantir que as buscas nesses campos dinâmicos continuem extremamente rápidas, são aplicados Índices GIN Funcionais, permitindo que o banco de dados pesquise dentro do payload JSON com a mesma performance de uma coluna tradicional.

## Pilares Arquiteturais

*   **Isolamento Absoluto de Dados:** Depender do RLS do PostgreSQL em vez de simples cláusulas "WHERE tenant_id = X" garante que filtros esquecidos no código nunca resultem em vazamento de dados.
*   **Controle de Acesso Baseado em Atributos (ABAC):** Indo além de papéis simples como "Admin/Usuário", as permissões são granulares, sensíveis ao contexto e incorporadas diretamente no fluxo de autenticação.
*   **Flexibilidade por Meta-Programação:** O backend é agnóstico ao modelo de negócios específico. Ele se adapta às necessidades do cliente, resolvendo o problema clássico de "campos customizados" em SaaS sem exigir migrações complexas no banco de dados.
*   **Performance em Escala:** O uso estratégico de Índices Parciais e Réplicas de Leitura (padrão CQRS) garante que consultas pesadas de relatórios não impactem a performance transacional da aplicação principal.

## Como Começar

1.  Clone o repositório usando: `git clone https://github.com/seu-usuario/tenentis-os.git`
2.  Inicie o banco de dados e a API via Docker usando: `docker-compose up -d`
3.  Acesse a documentação da API em `http://localhost:3000/docs`.

---
*Desenvolvido com foco em segurança, escalabilidade e arquitetura limpa.*
