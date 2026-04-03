# InnovaZap CRM — Agent Skills & Procedures

## Project Overview
InnovaZap CRM is a SaaS CRM with WhatsApp integration, chatbot engine, pipeline kanban, and analytics.
- **Stack:** .NET 10, React 18, PostgreSQL, Redis, RabbitMQ (MassTransit)
- **Repo:** https://github.com/gilihamu/InnovaZap-CRM (branch: main)
- **Company ID:** a1b2c3d4-1111-4000-a000-1aa07a2a0001

## Git Workflow
All agents MUST commit and push their work:
```bash
git add -A
git commit -m "feat(service-name): description of change [IZP-XX]"
git pull --rebase origin main
git push origin main
```
Use conventional commits: feat, fix, chore, docs, test.
Always reference the issue identifier (IZP-XX) in the commit message.

## Coding Standards
- .NET 10 minimal APIs or controller-based where appropriate
- Use `BackgroundService` for worker services
- EF Core with PostgreSQL (Npgsql)
- MassTransit for RabbitMQ messaging
- Serilog for structured logging
- All API responses use camelCase JSON
- JWT Bearer auth on all endpoints except health checks
- Multitenant: every query MUST filter by tenant_id

## Agent-Specific Skills

### CTO - Tech Lead (CEO role)
**Responsibilities:**
- Architecture decisions and code review
- Create and delegate issues to other agents
- Solution file (.sln) management
- Cross-service integration design
- Auto-delegation: when idle, scan backlog and create issues from ROADMAP

**When no issues assigned:**
1. GET /api/companies/{companyId}/issues?status=todo,backlog
2. Identify unassigned issues and assign to appropriate agents
3. Create new issues from project ROADMAP.md if backlog is empty

### Backend Gateway Engineer
**Responsibilities:**
- InnovaZap.Gateway project (src/backend/InnovaZap.Gateway/)
- JWT authentication (login, refresh, validation)
- Tenant registration and isolation middleware
- Rate limiting and CORS
- Swagger/OpenAPI documentation

**Key patterns:**
- TenantMiddleware extracts tenant_id from JWT claims
- Use IHttpContextAccessor for tenant context
- Register MassTransit for event publishing

### CRM Service Engineer
**Responsibilities:**
- InnovaZap.CRM project (src/backend/InnovaZap.CRM/)
- InnovaZap.Shared project (src/backend/InnovaZap.Shared/)
- Leads CRUD with pagination, filters, sorting
- Pipeline stages and deals (kanban backend)
- Tasks management
- Timeline/activity log per lead

**Key patterns:**
- Publish LeadCreated, LeadUpdated, DealWon events via MassTransit
- Filter all queries by tenant_id
- Lead scoring: hot/warm/cold based on activity

### Messaging & WhatsApp Engineer
**Responsibilities:**
- InnovaZap.Messaging project (src/backend/InnovaZap.Messaging/)
- WhatsApp Cloud API webhook (inbound messages)
- Send messages via WhatsApp Business API
- Conversation management and inbox
- Message status tracking (sent, delivered, read)

**Key patterns:**
- POST /wa/webhook verifies Meta webhook signature
- Publish MessageReceived event to RabbitMQ
- Store media URLs in S3-compatible storage
- SignalR hub for real-time inbox updates

### Chatbot Engine Engineer
**Responsibilities:**
- InnovaZap.Chatbot project (src/backend/InnovaZap.Chatbot/)
- Flow engine: FlowDispatcher, StepRunner (state machine)
- Step types: send_message, collect_input, branch, handoff, ai_classify
- Flow definition stored as JSONB
- Execution state persisted per lead

**Key patterns:**
- Consume MessageReceived from MassTransit
- FlowExecution.State is a JSONB column tracking current step + variables
- When step requires input, pause and wait for next message
- Publish ChatbotQualified event when lead is qualified

### Frontend React Engineer
**Responsibilities:**
- src/frontend/apps/web/ (React 18 + TypeScript + Vite)
- TailwindCSS + shadcn/ui component library
- Feature-based folder structure (features/crm, features/inbox, etc.)
- React Query for API calls, Zustand for state
- SignalR for real-time updates

**Key patterns:**
- Every API call uses React Query with proper queryKey
- SignalR invalidates queries on events (no polling)
- Lazy loading routes with React Router v6
- Responsive design (mobile-first)

### DevOps & Infrastructure
**Responsibilities:**
- Docker Compose (dev + prod)
- EF Core migrations
- CI/CD configuration (GitHub Actions)
- Database schema management
- Infrastructure documentation

**Key patterns:**
- docker-compose.yml: PostgreSQL 16, Redis 7, RabbitMQ 3.13
- Health checks on all containers
- Migration runner service
- Environment variables via .env files (never hardcode secrets)

## RabbitMQ Exchange Topology
```
Exchange: crm.events (topic)
  - crm.lead.created -> chatbot-service, automation-service, analytics-service
  - crm.lead.updated -> analytics-service
  - crm.deal.won -> analytics-service, notification-service

Exchange: whatsapp.inbound (fanout)
  - wa.message.received -> messaging-service, chatbot-service

Exchange: chatbot.flows (direct)
  - chatbot.step.completed -> crm-service
  - chatbot.qualified -> crm-service, analytics-service
```

## Database Tables Reference
tenants, users, leads, pipeline_stages, deals, tasks, conversations, messages,
chatbot_flows, flow_executions, automations, automation_logs, payments
