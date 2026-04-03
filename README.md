# InnovaZap CRM

SaaS CRM com integração WhatsApp, chatbot engine, pipeline de vendas e analytics.

## Stack
- **Backend:** .NET 10 (ASP.NET Core) - microserviços
- **Frontend:** React 18 + TypeScript + Vite + TailwindCSS + shadcn/ui
- **Banco:** PostgreSQL (EF Core) + Redis (cache/sessões)
- **Mensageria:** RabbitMQ (MassTransit)
- **Storage:** S3-compatible (Cloudflare R2)

## Serviços
| Serviço | Path | Responsabilidade |
|---------|------|-----------------|
| Gateway | `src/backend/InnovaZap.Gateway` | Auth JWT, multitenant, rate limiting |
| CRM | `src/backend/InnovaZap.CRM` | Leads, pipeline kanban, tasks, deals |
| Messaging | `src/backend/InnovaZap.Messaging` | WhatsApp API, inbox, conversations |
| Chatbot | `src/backend/InnovaZap.Chatbot` | Flow engine, state machine, AI |
| Automation | `src/backend/InnovaZap.Automation` | Cron jobs, follow-ups |
| Analytics | `src/backend/InnovaZap.Analytics` | Dashboard, KPIs, funnel |
| Shared | `src/backend/InnovaZap.Shared` | DTOs, events, contracts |
| Frontend | `src/frontend/apps/web` | React SPA |

## Desenvolvimento
```bash
docker compose up -d   # PostgreSQL, Redis, RabbitMQ
cd src/backend && dotnet run --project InnovaZap.Gateway
cd src/frontend/apps/web && npm run dev
```

Gerenciado por agentes AI via [Paperclip](https://paperclip.co).
