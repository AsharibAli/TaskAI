# Tasks: TaskAI Core Platform

**Input**: Design documents from `/specs/001-taskai-core-platform/`
**Prerequisites**: plan.md ✅, spec.md ✅, data-model.md ✅, contracts/ ✅, quickstart.md ✅
**Status**: ✅ ALL COMPLETE (retroactive documentation of implemented project)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Web app**: `backend/src/`, `frontend/src/`
- **Microservices**: `recurring-service/src/`, `notification-service/src/`
- **Infrastructure**: `helm/`, `dapr/`, `scripts/`

---

## Phase 1: Setup (Shared Infrastructure) ✅

**Purpose**: Project initialization and basic structure

- [x] T001 Create monorepo project structure with backend/, frontend/, recurring-service/, notification-service/
- [x] T002 [P] Initialize Python backend with FastAPI in backend/pyproject.toml
- [x] T003 [P] Initialize Next.js 16 frontend with TypeScript in frontend/package.json
- [x] T004 [P] Initialize recurring-service with FastAPI in recurring-service/pyproject.toml
- [x] T005 [P] Initialize notification-service with FastAPI in notification-service/pyproject.toml
- [x] T006 Create docker-compose.yml with PostgreSQL, Kafka, and service definitions
- [x] T007 [P] Create .env.example with all required environment variables
- [x] T008 [P] Configure pnpm workspace in pnpm-workspace.yaml

---

## Phase 2: Foundational (Blocking Prerequisites) ✅

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T009 Create database configuration in backend/src/core/database.py
- [x] T010 [P] Create environment settings in backend/src/core/config.py
- [x] T011 [P] Create security utilities (JWT, bcrypt) in backend/src/core/security.py
- [x] T012 Create User model in backend/src/models/user.py
- [x] T013 [P] Create structured logging middleware in backend/src/middleware/logging.py
- [x] T014 [P] Create correlation ID middleware in backend/src/middleware/correlation.py
- [x] T015 [P] Create Prometheus metrics middleware in backend/src/middleware/metrics.py
- [x] T016 [P] Create global error handling in backend/src/middleware/errors.py
- [x] T017 Create FastAPI application entry point in backend/src/main.py
- [x] T018 [P] Create API client utility in frontend/src/lib/api.ts
- [x] T019 [P] Create TypeScript type definitions in frontend/src/types/

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel ✅

---

## Phase 3: User Story 1 - User Registration and Authentication (Priority: P1) ✅ 🎯 MVP

**Goal**: Enable users to register, login, and access secure task workspace

**Independent Test**: Register account, login, verify session established, logout

### Implementation for User Story 1 ✅

- [x] T020 [US1] Implement auth endpoints (register, login, logout) in backend/src/api/auth.py
- [x] T021 [P] [US1] Create registration page in frontend/src/app/(auth)/register/page.tsx
- [x] T022 [P] [US1] Create login page in frontend/src/app/(auth)/login/page.tsx
- [x] T023 [US1] Create root layout with auth context in frontend/src/app/layout.tsx
- [x] T024 [US1] Create protected home page redirect in frontend/src/app/page.tsx
- [x] T025 [US1] Create authenticated dashboard layout in frontend/src/app/home/layout.tsx

**Checkpoint**: User Story 1 fully functional - users can register, login, and logout ✅

---

## Phase 4: User Story 2 - Task CRUD Operations via GUI (Priority: P1) ✅

**Goal**: Enable visual task management with create, read, update, delete operations

**Independent Test**: Create task, view in list, edit details, delete task, mark complete

### Implementation for User Story 2 ✅

- [x] T026 [US2] Create Task model in backend/src/models/task.py
- [x] T027 [P] [US2] Create Tag model in backend/src/models/tag.py
- [x] T028 [US2] Implement task service in backend/src/services/tasks.py
- [x] T029 [US2] Implement task CRUD endpoints in backend/src/api/tasks.py
- [x] T030 [P] [US2] Create Task TypeScript types in frontend/src/types/task.ts
- [x] T031 [P] [US2] Create task list component in frontend/src/components/tasks/TaskList.tsx
- [x] T032 [P] [US2] Create task item component in frontend/src/components/tasks/TaskItem.tsx
- [x] T033 [P] [US2] Create task form component in frontend/src/components/tasks/TaskForm.tsx
- [x] T034 [US2] Create Tasks Mode page in frontend/src/app/home/tasks/page.tsx
- [x] T035 [US2] Implement mode toggle in frontend/src/components/navigation/

**Checkpoint**: User Story 2 fully functional - full task CRUD via GUI ✅

---

## Phase 5: User Story 3 - Natural Language Task Management via Chat (Priority: P1) ✅

**Goal**: Enable conversational task management powered by AI

**Independent Test**: Send chat commands, verify AI creates/lists/completes tasks

### Implementation for User Story 3 ✅

- [x] T036 [US3] Create Conversation model in backend/src/models/conversation.py
- [x] T037 [P] [US3] Create Message model (within conversation.py)
- [x] T038 [US3] Create MCP agent with tool definitions in backend/src/mcp/agent.py
- [x] T039 [US3] Implement 18 MCP tools in backend/src/mcp/tools.py
- [x] T040 [US3] Implement chat service in backend/src/services/chat.py
- [x] T041 [US3] Implement chat endpoints in backend/src/api/chat.py
- [x] T042 [P] [US3] Create Chat TypeScript types in frontend/src/types/chat.ts
- [x] T043 [P] [US3] Create chat message component in frontend/src/components/chat/ChatMessage.tsx
- [x] T044 [P] [US3] Create chat input component in frontend/src/components/chat/ChatInput.tsx
- [x] T045 [US3] Create Chat Mode page in frontend/src/app/home/chat/page.tsx
- [x] T046 [US3] Implement real-time sync between Chat and Tasks modes

**Checkpoint**: User Story 3 fully functional - AI chat manages tasks ✅

---

## Phase 6: User Story 4 - Task Organization with Priority and Tags (Priority: P2) ✅

**Goal**: Enable task organization by priority levels and tags

**Independent Test**: Create tasks with priorities, add tags, filter by both

### Implementation for User Story 4 ✅

- [x] T047 [US4] Add priority enum and field to Task model in backend/src/models/task.py
- [x] T048 [US4] Implement tag management endpoints in backend/src/api/tasks.py
- [x] T049 [US4] Add priority filter_by_priority MCP tool in backend/src/mcp/tools.py
- [x] T050 [US4] Add tag management MCP tools (add_tag, remove_tag, filter_by_tag)
- [x] T051 [P] [US4] Create priority selector component in frontend/src/components/tasks/
- [x] T052 [P] [US4] Create tag input component in frontend/src/components/tasks/
- [x] T053 [US4] Add filter/sort controls to task list UI

**Checkpoint**: User Story 4 fully functional - priority and tag organization ✅

---

## Phase 7: User Story 5 - Due Dates and Reminders (Priority: P2) ✅

**Goal**: Enable due date tracking and email reminder notifications

**Independent Test**: Set due date with natural language, set reminder, receive email

### Implementation for User Story 5 ✅

- [x] T054 [US5] Create natural language date parser in backend/src/utils/date_parser.py
- [x] T055 [US5] Add due_date, remind_at, reminder_sent fields to Task model
- [x] T056 [US5] Implement reminder endpoint in backend/src/api/tasks.py
- [x] T057 [US5] Create reminder scheduler in backend/src/services/reminder_scheduler.py
- [x] T058 [US5] Add set_due_date, show_overdue MCP tools in backend/src/mcp/tools.py
- [x] T059 [US5] Add set_reminder MCP tool
- [x] T060 [P] [US5] Create date picker component in frontend/src/components/tasks/
- [x] T061 [US5] Add overdue visual indicator to task list

**Checkpoint**: User Story 5 fully functional - due dates and reminders working ✅

---

## Phase 8: User Story 6 - Recurring Tasks (Priority: P3) ✅

**Goal**: Enable automatic task regeneration for recurring patterns

**Independent Test**: Create weekly recurring task, complete it, verify new instance created

### Implementation for User Story 6 ✅

- [x] T062 [US6] Add recurrence enum and parent_task_id to Task model
- [x] T063 [US6] Create event publisher in backend/src/services/events/publisher.py
- [x] T064 [P] [US6] Create event schemas in backend/src/services/events/schemas.py
- [x] T065 [US6] Publish task.completed events on completion toggle
- [x] T066 [US6] Create recurring-service event handler in recurring-service/src/handlers/task_completion_handler.py
- [x] T067 [US6] Implement recurrence calculator in recurring-service/src/services/recurrence.py
- [x] T068 [US6] Create backend client in recurring-service/src/services/backend_client.py
- [x] T069 [US6] Create Dapr event webhook in recurring-service/src/api/events.py
- [x] T070 [US6] Add set_recurrence MCP tool in backend/src/mcp/tools.py
- [x] T071 [P] [US6] Create recurrence selector component in frontend/src/components/tasks/

**Checkpoint**: User Story 6 fully functional - recurring tasks auto-regenerate ✅

---

## Phase 9: User Story 7 - Task Search (Priority: P3) ✅

**Goal**: Enable full-text search across task titles and descriptions

**Independent Test**: Create multiple tasks, search by keyword, verify results

### Implementation for User Story 7 ✅

- [x] T072 [US7] Implement search endpoint in backend/src/api/tasks.py
- [x] T073 [US7] Add search_tasks MCP tool in backend/src/mcp/tools.py
- [x] T074 [P] [US7] Create search input component in frontend/src/components/tasks/
- [x] T075 [US7] Integrate search with task list page

**Checkpoint**: User Story 7 fully functional - search working ✅

---

## Phase 10: Event-Driven Infrastructure ✅

**Purpose**: Kafka pub/sub via Dapr for inter-service communication

- [x] T076 Create local Dapr Kafka pubsub component in dapr/components/kafka-pubsub.yaml
- [x] T077 [P] Create local Dapr statestore component in dapr/components/statestore.yaml
- [x] T078 Create production Kafka pubsub component in dapr/deployment/01-kafka-pubsub-component.yaml
- [x] T079 [P] Create task-events subscription in dapr/deployment/02-subscription-task-events.yaml
- [x] T080 [P] Create reminders subscription in dapr/deployment/03-subscription-reminders.yaml
- [x] T081 Create notification-service email handler in notification-service/src/services/email_service.py
- [x] T082 Create notification-service Dapr webhook in notification-service/src/api/reminders.py
- [x] T083 Create notification-service main entry in notification-service/src/main.py

**Checkpoint**: Event-driven architecture complete - services communicate via Kafka ✅

---

## Phase 11: Production Deployment ✅

**Purpose**: Kubernetes deployment with Helm and Dapr

- [x] T084 Create Helm chart structure in helm/todo-chatbot/
- [x] T085 [P] Create Chart.yaml in helm/todo-chatbot/Chart.yaml
- [x] T086 [P] Create values.yaml in helm/todo-chatbot/values.yaml
- [x] T087 Create Dapr-enabled values in helm/todo-chatbot/helm-values-dapr-enabled.yaml
- [x] T088 [P] Create backend deployment template in helm/todo-chatbot/templates/backend/
- [x] T089 [P] Create frontend deployment template in helm/todo-chatbot/templates/frontend/
- [x] T090 [P] Create recurring-service deployment template in helm/todo-chatbot/templates/recurring-service/
- [x] T091 [P] Create notification-service deployment template in helm/todo-chatbot/templates/notification-service/
- [x] T092 [P] Create backend Dockerfile in backend/Dockerfile
- [x] T093 [P] Create frontend Dockerfile in frontend/Dockerfile
- [x] T094 [P] Create recurring-service Dockerfile in recurring-service/Dockerfile
- [x] T095 [P] Create notification-service Dockerfile in notification-service/Dockerfile
- [x] T096 Create deploy-all.sh script in scripts/deploy-all.sh
- [x] T097 [P] Create deploy-backend.sh script in scripts/deploy-backend.sh
- [x] T098 [P] Create deploy-frontend.sh script in scripts/deploy-frontend.sh
- [x] T099 [P] Create deploy-recurring-service.sh script in scripts/deploy-recurring-service.sh
- [x] T100 [P] Create deploy-notification-service.sh script in scripts/deploy-notification-service.sh

**Checkpoint**: Production deployment complete - running on DigitalOcean Kubernetes ✅

---

## Phase 12: Polish & Cross-Cutting Concerns ✅

**Purpose**: Final improvements affecting multiple user stories

- [x] T101 [P] Create README.md with comprehensive documentation
- [x] T102 [P] Create scripts/README.md with deployment documentation
- [x] T103 Add health check endpoints to all services
- [x] T104 Configure CORS for production domain
- [x] T105 [P] Create UI components library in frontend/src/components/ui/
- [x] T106 Add responsive design with Tailwind CSS
- [x] T107 Configure OpenTelemetry tracing across services
- [x] T108 Deploy to production at https://taskai.asharib.xyz

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - start immediately ✅
- **Foundational (Phase 2)**: Depends on Setup - BLOCKS all user stories ✅
- **User Stories (Phases 3-9)**: All depend on Foundational ✅
  - User stories proceeded in priority order (P1 → P2 → P3)
- **Event-Driven (Phase 10)**: Depends on US6 recurring tasks ✅
- **Production (Phase 11)**: Depends on all user stories ✅
- **Polish (Phase 12)**: Depends on all phases ✅

### User Story Dependencies

| Story | Priority | Dependencies | Status |
|-------|----------|--------------|--------|
| US1: Authentication | P1 | Foundational only | ✅ |
| US2: Task CRUD | P1 | Foundational, US1 | ✅ |
| US3: AI Chat | P1 | Foundational, US1, US2 | ✅ |
| US4: Priority & Tags | P2 | US2 | ✅ |
| US5: Due Dates & Reminders | P2 | US2 | ✅ |
| US6: Recurring Tasks | P3 | US2, Event-Driven | ✅ |
| US7: Search | P3 | US2 | ✅ |

### Parallel Opportunities (Historical)

Within each user story, tasks marked [P] were parallelizable:
- All model definitions ran in parallel
- All frontend components ran in parallel
- All deployment scripts ran in parallel
- All Dapr configurations ran in parallel

---

## Summary

| Metric | Value |
|--------|-------|
| Total Tasks | 108 |
| Setup Phase | 8 tasks |
| Foundational Phase | 11 tasks |
| User Story 1 (Auth) | 6 tasks |
| User Story 2 (CRUD) | 10 tasks |
| User Story 3 (Chat) | 11 tasks |
| User Story 4 (Priority/Tags) | 7 tasks |
| User Story 5 (Due Dates) | 8 tasks |
| User Story 6 (Recurring) | 10 tasks |
| User Story 7 (Search) | 4 tasks |
| Event-Driven | 8 tasks |
| Production | 17 tasks |
| Polish | 8 tasks |
| **Completion** | **100%** |

---

## Notes

- All 108 tasks completed and verified
- Project deployed to production at https://taskai.asharib.xyz
- All 7 user stories independently functional
- Constitution compliance verified (6/6 principles)
- Event-driven architecture operational with Kafka + Dapr
