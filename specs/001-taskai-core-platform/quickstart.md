# Quickstart Guide: TaskAI

**Feature**: 001-taskai-core-platform
**Date**: 2025-12-25

## Prerequisites

- **Docker** 20.10+ & **Docker Compose** 2.x
- **Node.js** 18+ with **pnpm** 9+
- **Python** 3.11+
- **OpenAI API Key** (for Chat Mode)

## Local Development Setup

### 1. Clone Repository

```bash
git clone https://github.com/AsharibAli/TaskAI.git
cd TaskAI
```

### 2. Environment Configuration

```bash
# Copy example environment file
cp .env.example .env
```

Edit `.env` with your values:

```bash
# Database (auto-configured for docker-compose)
POSTGRES_PASSWORD=taskai_secret

# Authentication
JWT_SECRET=your-super-secret-jwt-key-min-32-chars

# OpenAI (for Chat Mode)
OPENAI_API_KEY=sk-your-openai-api-key
OPENAI_MODEL=gpt-4.1-2025-04-14

# Email (Optional - for reminders)
RESEND_API_KEY=re_your_resend_api_key
RESEND_FROM_EMAIL=onboarding@resend.dev
```

Generate a secure JWT secret:
```bash
openssl rand -hex 32
```

### 3. Start All Services

```bash
# Start all services (Backend, Frontend, PostgreSQL, Kafka, Dapr)
docker-compose up -d

# Check service health
docker-compose ps

# View logs
docker-compose logs -f backend
docker-compose logs -f frontend
```

### 4. Access the Application

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:8000 |
| API Docs (Swagger) | http://localhost:8000/docs |
| PostgreSQL | localhost:9432 |
| Kafka | localhost:9092 |

### 5. Create Your First User

**Option 1: Via UI**
1. Go to http://localhost:3000/register
2. Fill in email, password, and full name
3. Click Register

**Option 2: Via API**
```bash
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "securepassword",
    "full_name": "John Doe"
  }'
```

### 6. Start Using TaskAI

1. **Login** at http://localhost:3000/login
2. **Switch to Chat Mode** (toggle at top)
3. **Try AI commands**:
   - "Create a task to buy groceries"
   - "Show me all high priority tasks"
   - "Mark task 'buy groceries' as done"
4. **Switch to Tasks Mode** (toggle at top)
5. **Use GUI** to filter, sort, and manage tasks visually

---

## Individual Service Development

### Backend Development

```bash
cd backend

# Create virtual environment
uv venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
uv sync

# Run development server
fastapi dev src/main.py
```

Backend runs at http://localhost:8000

### Frontend Development

```bash
cd frontend

# Install dependencies
pnpm install

# Run development server
pnpm dev
```

Frontend runs at http://localhost:3000

### Recurring Service Development

```bash
cd recurring-service

# Create virtual environment and install
uv sync

# Run development server
fastapi dev src/main.py --port 8002
```

### Notification Service Development

```bash
cd notification-service

# Create virtual environment and install
uv sync

# Run development server
fastapi dev src/main.py --port 8001
```

---

## Running with Dapr (Event-Driven Features)

For recurring tasks and email reminders, you need Dapr sidecars.

### Initialize Dapr

```bash
dapr init
```

### Run Backend with Dapr

```bash
dapr run --app-id backend --app-port 8000 --dapr-http-port 3500 \
  --components-path ./dapr/components \
  -- uvicorn src.main:app --host 0.0.0.0 --port 8000
```

### Run Recurring Service with Dapr

```bash
dapr run --app-id recurring-service --app-port 8002 --dapr-http-port 3502 \
  --components-path ./dapr/components \
  -- uvicorn src.main:app --host 0.0.0.0 --port 8002
```

### Run Notification Service with Dapr

```bash
dapr run --app-id notification-service --app-port 8001 --dapr-http-port 3501 \
  --components-path ./dapr/components \
  -- uvicorn src.main:app --host 0.0.0.0 --port 8001
```

---

## Testing

### Backend Tests

```bash
cd backend

# Run all tests
pytest

# Run with coverage
pytest --cov=src --cov-report=html

# Run specific test file
pytest tests/test_tasks.py
```

### API Testing with curl

```bash
# Login and get token
TOKEN=$(curl -s -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"securepassword"}' \
  | jq -r '.access_token')

# Create a task
curl -X POST http://localhost:8000/api/tasks \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Test task",
    "priority": "high",
    "due_date": "2026-02-01T17:00:00Z"
  }'

# List tasks
curl -X GET "http://localhost:8000/api/tasks" \
  -H "Authorization: Bearer $TOKEN"

# Chat with AI
curl -X POST http://localhost:8000/api/chat/chat \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "Show me all my tasks"}'
```

---

## Database Management

### Connect to PostgreSQL

```bash
docker exec -it TaskAI-postgres-1 psql -U postgres -d taskai
```

### Useful SQL Commands

```sql
-- View tables
\dt

-- View tasks table schema
\d tasks

-- Query tasks
SELECT id, title, priority, due_date, is_completed FROM tasks;

-- View users
SELECT id, email, full_name, created_at FROM users;
```

### Backup and Restore

```bash
# Backup
docker exec TaskAI-postgres-1 pg_dump -U postgres taskai > backup.sql

# Restore
docker exec -i TaskAI-postgres-1 psql -U postgres taskai < backup.sql
```

---

## Troubleshooting

### Common Issues

**Port already in use**
```bash
# Find process using port
lsof -i :8000
# Kill process
kill -9 <PID>
```

**Docker containers not starting**
```bash
# Reset Docker Compose
docker-compose down -v
docker-compose up -d
```

**Database connection issues**
```bash
# Check PostgreSQL logs
docker-compose logs postgres

# Verify connection
docker exec -it TaskAI-postgres-1 pg_isready
```

**Chat mode not working**
- Verify OPENAI_API_KEY is set in .env
- Check backend logs for API errors
- Ensure OpenAI API has sufficient credits

---

## Next Steps

- Review [spec.md](./spec.md) for feature requirements
- Review [plan.md](./plan.md) for architecture details
- Review [data-model.md](./data-model.md) for database schema
- Explore the API at http://localhost:8000/docs
