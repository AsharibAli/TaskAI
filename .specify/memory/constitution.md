# TaskAI Constitution

> A production-ready, event-driven, AI-powered task management platform featuring conversational AI (CUI) and traditional graphical UI (GUI) interfaces.

## Core Principles

### I. Dual Interface Design
TaskAI provides two distinct interaction paradigms that sync in real-time:
- **Chat Mode (CUI)**: Natural language interface powered by GPT-4.1 via MCP with 18 specialized tools
- **Tasks Mode (GUI)**: Traditional visual interface with filtering, sorting, and organization
- Both interfaces must maintain state consistency; changes in one reflect immediately in the other

### II. Microservices Architecture
Four independent, loosely-coupled services form the system:
- **Backend API** (FastAPI, port 8000): Core business logic, authentication, task CRUD, event publishing
- **Frontend** (Next.js, port 3000): User interface serving both Chat and Tasks modes
- **Recurring Service** (port 8002): Processes task.completed events, creates recurring task instances
- **Notification Service** (port 8001): Handles reminder notifications via Resend email API

Services communicate via Kafka pub/sub through Dapr sidecars. Each service must be:
- Independently deployable
- Stateless (state persisted in PostgreSQL or Kafka)
- Horizontally scalable

### III. Event-Driven Communication
All cross-service communication uses Kafka topics via Dapr:
- `task-events`: Task lifecycle events (created, updated, completed, deleted)
- `reminders`: Reminder notification triggers

Event flow:
```
Backend API → Kafka → Recurring Service (task completion → next instance)
                   → Notification Service (reminders → email)
```

Services must not directly call each other for async operations; use pub/sub.

### IV. User Isolation & Security
- All data scoped to authenticated user via JWT
- JWT-based auth with bcrypt password hashing (12 rounds)
- 24-hour token expiration
- Environment-based secrets (never committed to git)
- CORS configuration for cross-origin requests

### V. Cloud-Native Deployment
- Kubernetes-native with Helm charts
- Dapr sidecars for service mesh capabilities
- Health check endpoints for liveness/readiness probes
- Resource limits defined for all pods
- Support for DOK, AKS, GKE, EKS

### VI. Observability First
All services must implement:
- OpenTelemetry tracing with correlation IDs
- Prometheus metrics for request/response monitoring
- Structured JSON logging (structlog)
- Health check endpoints (`/health`, `/ready`)

## Technology Standards

### Backend Stack
| Component | Technology | Version |
|-----------|------------|---------|
| Framework | FastAPI | 0.125+ |
| ORM | SQLModel (SQLAlchemy + Pydantic) | - |
| Database | PostgreSQL | 16 |
| Auth | JWT (python-jose) + bcrypt | - |
| AI/LLM | OpenAI API | GPT-4.1-2025-04-14 |
| MCP | Model Context Protocol | - |
| Events | Kafka via Dapr | 3.6 |
| Email | Resend API | - |
| Scheduling | APScheduler | - |
| Runtime | Python | 3.11+ |

### Frontend Stack
| Component | Technology | Version |
|-----------|------------|---------|
| Framework | Next.js (App Router) | 16 |
| Language | TypeScript | 5.x |
| Styling | Tailwind CSS | 3.x |
| Components | Shadcn/ui (Radix primitives) | - |
| Package Manager | pnpm | 9.x |

### Infrastructure
| Component | Technology | Version |
|-----------|------------|---------|
| Containers | Docker | 20.10+ |
| Orchestration | Kubernetes | 1.28+ |
| Package Manager | Helm | 3.x |
| Service Mesh | Dapr | 1.12+ |
| Message Queue | Apache Kafka (KRaft) | 3.6 |

## Code Quality Standards

### API Design
- RESTful endpoints with consistent naming (`/api/{resource}`)
- OpenAPI/Swagger documentation auto-generated
- Proper HTTP status codes (200, 201, 400, 401, 404, 500)
- Request validation via Pydantic models
- Consistent error response format: `{"detail": "message"}`

### Database Design
- User isolation via `user_id` foreign key on all user-owned tables
- Proper indexing for common query patterns
- Soft deletes where audit trails are needed
- UUID primary keys for public-facing IDs

### MCP Tools
18 specialized tools organized by category:
- **Core Operations**: add_task, list_tasks, complete_task, update_task, delete_task
- **Priority Management**: set_priority, filter_by_priority
- **Tag Management**: add_tag, remove_tag, filter_by_tag
- **Due Date Management**: set_due_date, show_overdue
- **Search & Filter**: search_tasks, combined_filter, sort_tasks
- **Reminders**: set_reminder
- **Recurrence**: set_recurrence

Tools must handle natural language input (e.g., "tomorrow", "next Friday").

### Testing Requirements
- Backend: pytest with httpx for integration tests
- Minimum coverage for critical paths (auth, task CRUD, event publishing)
- Integration tests for API endpoints
- Event flow testing for pub/sub scenarios

### Commit Conventions
Follow [Conventional Commits](https://www.conventionalcommits.org/):
```
feat(backend): add task priority filtering
fix(frontend): resolve chat input focus issue
docs(readme): update deployment instructions
chore(deps): update dependencies
```

## Performance & Reliability

### Performance Targets
- API response time: p95 < 200ms for CRUD operations
- Chat response: p95 < 3s (includes OpenAI API latency)
- Frontend First Contentful Paint: < 1.5s

### Reliability Standards
- Health check endpoints respond within 100ms
- Graceful degradation when Dapr unavailable (DAPR_ENABLED flag)
- Retry logic for external API calls (OpenAI, Resend)
- Circuit breakers for downstream service failures

### Scalability
- Stateless services enable horizontal scaling
- Database connection pooling
- Kubernetes HPA for auto-scaling based on CPU (target: 70%)

## Development Workflow

### Local Development
```bash
# Full stack via Docker Compose
docker-compose up -d

# Individual service development
cd backend && fastapi dev src/main.py
cd frontend && pnpm dev
```

### Deployment Pipeline
1. Build Docker images locally
2. Push to container registry (DigitalOcean)
3. Update Kubernetes deployments via Helm
4. Wait for rollout completion
5. Verify Dapr sidecar status
6. Health check validation

### Production Checklist
- [ ] Strong JWT_SECRET (32+ characters)
- [ ] Managed PostgreSQL (Neon, RDS)
- [ ] Managed Kafka (DigitalOcean, Confluent)
- [ ] CORS_ORIGINS configured for production domain
- [ ] HTTPS with TLS certificates
- [ ] Monitoring (Prometheus + Grafana)
- [ ] Log aggregation
- [ ] Auto-scaling configured
- [ ] Database backups enabled
- [ ] Resource limits set

## Governance

This constitution defines the architectural and quality standards for TaskAI. All contributions must:

1. **Maintain dual-interface consistency** - Changes to task state must reflect in both CUI and GUI
2. **Preserve service boundaries** - No direct inter-service calls for async operations
3. **Follow event-driven patterns** - Use Kafka pub/sub for cross-service communication
4. **Ensure user isolation** - All data operations scoped to authenticated user
5. **Include observability** - Logging, metrics, and tracing for new features

Amendments to this constitution require:
- Clear rationale documented
- Impact assessment on existing services
- Migration plan if breaking changes

**Version**: 1.0.0 | **Ratified**: 2025-12-18 | **Last Amended**: 2025-12-25
