# Implementation Plan: TaskAI Core Platform

**Branch**: `001-taskai-core-platform` | **Date**: 2025-12-25 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-taskai-core-platform/spec.md`
**Status**: ✅ IMPLEMENTED (retroactive documentation)

## Summary

TaskAI is a production-ready, event-driven, AI-powered task management platform featuring dual interfaces: a conversational AI (CUI) powered by GPT-4.1 via MCP, and a traditional graphical UI (GUI). The system uses a microservices architecture with 4 independent services communicating via Kafka pub/sub through Dapr sidecars, deployed on Kubernetes.

## Technical Context

**Language/Version**: Python 3.11+ (backend), TypeScript 5.x (frontend)
**Primary Dependencies**:
- Backend: FastAPI 0.125+, SQLModel, OpenAI API, APScheduler
- Frontend: Next.js 16, Tailwind CSS 3.x, Shadcn/ui
- Infrastructure: Dapr 1.12+, Kafka 3.6, Kubernetes 1.28+

**Storage**: PostgreSQL 16 (Neon for cloud, local for dev)
**Testing**: pytest with httpx (backend), Jest (frontend - planned)
**Target Platform**: Kubernetes clusters (DOK, AKS, GKE, EKS), Docker Compose for local dev
**Project Type**: Web application (microservices)
**Performance Goals**:
- API p95 < 200ms for CRUD operations
- Chat response p95 < 3s (includes OpenAI latency)
- Frontend FCP < 1.5s
- Support 100+ concurrent users

**Constraints**:
- Stateless services for horizontal scaling
- Event-driven async communication (no direct service calls)
- User data isolation via JWT scoping
- Graceful degradation when Dapr unavailable

**Scale/Scope**: Production deployment at https://taskai.asharib.xyz

## Constitution Check

*GATE: ✅ PASSED - Implementation follows all constitutional principles*

| Principle | Status | Evidence |
|-----------|--------|----------|
| I. Dual Interface Design | ✅ | Chat Mode + Tasks Mode with real-time sync |
| II. Microservices Architecture | ✅ | 4 services: Backend, Frontend, Recurring, Notification |
| III. Event-Driven Communication | ✅ | Kafka pub/sub via Dapr for task-events and reminders |
| IV. User Isolation & Security | ✅ | JWT auth, bcrypt hashing, user_id scoping |
| V. Cloud-Native Deployment | ✅ | Kubernetes with Helm, Dapr sidecars, health probes |
| VI. Observability First | ✅ | OpenTelemetry, Prometheus, structlog, health endpoints |

## Project Structure

### Documentation (this feature)

```text
specs/001-taskai-core-platform/
├── spec.md              # Feature specification
├── plan.md              # This file (implementation plan)
├── research.md          # Technology research (N/A - already implemented)
├── data-model.md        # Database schema documentation
├── quickstart.md        # Developer quickstart guide
├── contracts/           # API contracts (OpenAPI)
│   └── openapi.yaml
└── checklists/
    └── requirements.md  # Spec quality checklist
```

### Source Code (repository root)

```text
backend/                          # FastAPI backend service (port 8000)
├── src/
│   ├── api/                      # REST API route handlers
│   │   ├── auth.py              # Authentication endpoints
│   │   ├── tasks.py             # Task CRUD endpoints
│   │   └── chat.py              # Chat/AI endpoints
│   ├── core/                     # Core configuration
│   │   ├── config.py            # Environment settings
│   │   ├── database.py          # SQLModel setup
│   │   └── security.py          # JWT & bcrypt
│   ├── models/                   # Database models
│   │   ├── user.py              # User model
│   │   ├── task.py              # Task model
│   │   ├── tag.py               # Tag model
│   │   └── conversation.py      # Chat models
│   ├── services/                 # Business logic
│   │   ├── tasks.py             # Task service
│   │   ├── chat.py              # Chat processing
│   │   ├── events/              # Event publishing
│   │   │   ├── publisher.py     # Dapr pub/sub client
│   │   │   └── schemas.py       # Event data models
│   │   └── reminder_scheduler.py # Background reminder polling
│   ├── mcp/                      # Model Context Protocol
│   │   ├── agent.py             # Chat agent with 18 tools
│   │   └── tools.py             # MCP tool implementations
│   ├── middleware/               # FastAPI middleware
│   │   ├── logging.py           # Structured logging
│   │   ├── correlation.py       # Distributed tracing IDs
│   │   ├── metrics.py           # Prometheus metrics
│   │   └── errors.py            # Global error handling
│   ├── utils/                    # Utility functions
│   │   └── date_parser.py       # Natural language dates
│   └── main.py                   # Application entry point
├── tests/                        # Backend tests
├── Dockerfile
└── pyproject.toml

frontend/                         # Next.js 16 frontend (port 3000)
├── src/
│   ├── app/                      # App Router pages
│   │   ├── layout.tsx           # Root layout
│   │   ├── page.tsx             # Home/landing page
│   │   ├── (auth)/              # Auth pages (login, register)
│   │   └── home/                # Authenticated home page
│   ├── components/               # React components
│   │   ├── chat/                # Chat interface components
│   │   ├── tasks/               # Task UI components
│   │   ├── navigation/          # Headers, sidebars
│   │   ├── notifications/       # Notification center
│   │   └── ui/                  # Shadcn/ui primitives
│   ├── lib/                      # Utilities
│   │   ├── api.ts               # API client
│   │   └── utils.ts             # Helper functions
│   └── types/                    # TypeScript types
│       ├── task.ts              # Task type definitions
│       └── chat.ts              # Chat type definitions
├── public/                       # Static assets
├── Dockerfile
└── package.json

recurring-service/                # Recurring tasks microservice (port 8002)
├── src/
│   ├── api/
│   │   └── events.py            # Dapr webhook handler
│   ├── services/
│   │   ├── recurrence.py        # Next occurrence calculator
│   │   └── backend_client.py    # Backend API client
│   ├── handlers/
│   │   └── task_completion_handler.py
│   ├── core/
│   └── main.py
├── Dockerfile
└── pyproject.toml

notification-service/             # Notification microservice (port 8001)
├── src/
│   ├── api/
│   │   └── reminders.py         # Dapr webhook handler
│   ├── services/
│   │   └── email_service.py     # Resend API integration
│   ├── handlers/
│   ├── core/
│   └── main.py
├── Dockerfile
└── pyproject.toml

helm/                             # Kubernetes deployment
└── todo-chatbot/
    ├── Chart.yaml
    ├── values.yaml
    ├── helm-values-dapr-enabled.yaml
    └── templates/
        ├── backend/
        ├── frontend/
        ├── recurring-service/
        └── notification-service/

dapr/                             # Dapr configuration
├── components/                   # Local Dapr components
│   ├── kafka-pubsub.yaml
│   └── statestore.yaml
└── deployment/                   # Production Dapr components
    ├── 01-kafka-pubsub-component.yaml
    ├── 02-subscription-task-events.yaml
    └── 03-subscription-reminders.yaml

scripts/                          # Deployment automation
├── deploy-all.sh
├── deploy-backend.sh
├── deploy-frontend.sh
├── deploy-recurring-service.sh
├── deploy-notification-service.sh
└── README.md

docker-compose.yml               # Local development setup
.env.example                     # Environment template
```

**Structure Decision**: Web application with 4 microservices following the constitution's microservices architecture principle. Each service is independently deployable, stateless, and communicates via Kafka pub/sub through Dapr.

## Architecture Decisions

### AD-001: Event-Driven Architecture with Kafka + Dapr

**Decision**: Use Apache Kafka for event streaming with Dapr as the abstraction layer.

**Rationale**:
- Loose coupling between services (recurring-service and notification-service don't call backend directly)
- Reliable event delivery with Kafka's durability guarantees
- Dapr provides cloud-native abstractions that work across environments
- Graceful degradation via DAPR_ENABLED flag

**Alternatives Considered**:
- Direct HTTP calls between services → Rejected (tight coupling, harder to scale)
- RabbitMQ → Rejected (Kafka better for event sourcing patterns)
- AWS SQS/SNS → Rejected (vendor lock-in)

### AD-002: MCP for AI Tool Integration

**Decision**: Use Model Context Protocol (MCP) to integrate GPT-4.1 with 18 specialized task management tools.

**Rationale**:
- Structured tool definitions enable precise AI-to-function mapping
- Tools can validate inputs before execution
- Conversation context preserved for multi-turn interactions
- Extensible pattern for adding new capabilities

**Alternatives Considered**:
- Raw function calling → Rejected (less structured, harder to maintain)
- LangChain → Rejected (unnecessary complexity for this use case)

### AD-003: JWT-Based Stateless Authentication

**Decision**: Use JWT tokens with 24-hour expiration for stateless authentication.

**Rationale**:
- Stateless design enables horizontal scaling
- No session store required
- User context travels with each request
- Standard approach compatible with frontend frameworks

**Alternatives Considered**:
- Session-based auth → Rejected (requires session store, harder to scale)
- OAuth2/OIDC → Deferred (can be added later for SSO)

## Complexity Tracking

> No violations requiring justification. Implementation follows all constitutional principles.

| Principle | Compliance | Notes |
|-----------|------------|-------|
| 4 microservices max | ✅ | Exactly 4: Backend, Frontend, Recurring, Notification |
| Event-driven communication | ✅ | Kafka pub/sub via Dapr |
| Stateless services | ✅ | State in PostgreSQL, no service-local state |
| User isolation | ✅ | All queries scoped by user_id |

## Implementation Status

### Phase 1: Core Infrastructure ✅

- [x] Project scaffolding (monorepo structure)
- [x] Docker Compose for local development
- [x] PostgreSQL database setup
- [x] Kafka + Dapr configuration

### Phase 2: Backend API ✅

- [x] FastAPI application setup
- [x] SQLModel database models (User, Task, Tag, Conversation, Message)
- [x] JWT authentication (register, login, logout)
- [x] Task CRUD endpoints with filtering/sorting
- [x] Tag management (add/remove tags)
- [x] Due date support with natural language parsing
- [x] Reminder scheduling with APScheduler
- [x] Event publishing to Kafka

### Phase 3: AI Chat Interface ✅

- [x] MCP agent implementation
- [x] 18 specialized tools for task operations
- [x] Conversation persistence
- [x] Natural language intent interpretation
- [x] OpenAI GPT-4.1 integration

### Phase 4: Frontend ✅

- [x] Next.js 16 application with App Router
- [x] Authentication UI (login, register)
- [x] Tasks Mode (GUI) with CRUD operations
- [x] Chat Mode (CUI) with AI responses
- [x] Mode switching with state sync
- [x] Filtering, sorting, search UI
- [x] Responsive design with Tailwind CSS

### Phase 5: Event-Driven Features ✅

- [x] Recurring Service (task completion → next instance)
- [x] Notification Service (reminder → email via Resend)
- [x] Dapr subscriptions configuration
- [x] CloudEvent parsing and handling

### Phase 6: Production Deployment ✅

- [x] Kubernetes Helm charts
- [x] Dapr sidecar injection
- [x] DigitalOcean Kubernetes deployment
- [x] TLS/HTTPS configuration
- [x] Health check endpoints
- [x] Deployment automation scripts

## API Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/auth/register` | POST | User registration |
| `/api/auth/login` | POST | User authentication |
| `/api/auth/logout` | POST | Session invalidation |
| `/api/tasks` | GET | List tasks (with filters) |
| `/api/tasks` | POST | Create task |
| `/api/tasks/{id}` | GET | Get single task |
| `/api/tasks/{id}` | PUT | Update task |
| `/api/tasks/{id}` | DELETE | Delete task |
| `/api/tasks/{id}/complete` | PATCH | Toggle completion |
| `/api/tasks/{id}/tags` | POST | Add tag |
| `/api/tasks/{id}/tags/{name}` | DELETE | Remove tag |
| `/api/tasks/{id}/reminder` | POST | Set reminder |
| `/api/tasks/search` | GET | Full-text search |
| `/api/chat/chat` | POST | Send message to AI |
| `/api/chat/conversations` | GET | List conversations |

## MCP Tools (18 total)

| Category | Tools |
|----------|-------|
| Core Operations | add_task, list_tasks, complete_task, update_task, delete_task |
| Priority Management | set_priority, filter_by_priority |
| Tag Management | add_tag, remove_tag, filter_by_tag |
| Due Date Management | set_due_date, show_overdue |
| Search & Filter | search_tasks, combined_filter, sort_tasks |
| Reminders | set_reminder |
| Recurrence | set_recurrence |

## Next Steps

This plan documents an already-implemented system. Future enhancements could include:

1. **Testing improvements**: Increase test coverage, add E2E tests
2. **Performance optimization**: Add caching layer (Redis)
3. **Feature additions**: Subtasks, attachments, sharing/collaboration
4. **OAuth integration**: Google, GitHub SSO
5. **Mobile apps**: iOS/Android native clients
