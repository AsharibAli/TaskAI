# Data Model: TaskAI Core Platform

**Feature**: 001-taskai-core-platform
**Date**: 2025-12-25
**Status**: ✅ IMPLEMENTED

## Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                          USERS                                   │
│  id (UUID, PK)                                                  │
│  email (VARCHAR, UNIQUE)                                        │
│  hashed_password (VARCHAR)                                      │
│  full_name (VARCHAR)                                            │
│  profile_picture (VARCHAR, nullable)                            │
│  created_at (TIMESTAMP)                                         │
│  updated_at (TIMESTAMP)                                         │
└───────────────────────────┬─────────────────────────────────────┘
                            │ 1:N
            ┌───────────────┼───────────────┬───────────────┐
            │               │               │               │
            ▼               ▼               ▼               ▼
┌───────────────────┐ ┌───────────────┐ ┌─────────────────────────┐
│      TASKS        │ │     TAGS      │ │     CONVERSATIONS       │
│  id (UUID, PK)    │ │  id (UUID,PK) │ │  id (UUID, PK)          │
│  user_id (FK)     │ │  user_id (FK) │ │  user_id (FK)           │
│  title (VARCHAR)  │ │  name (VAR)   │ │  title (VARCHAR)        │
│  description      │ │  created_at   │ │  created_at             │
│  is_completed     │ └───────┬───────┘ │  updated_at             │
│  priority         │         │         └───────────┬─────────────┘
│  due_date         │         │                     │ 1:N
│  remind_at        │         │                     ▼
│  reminder_sent    │         │         ┌─────────────────────────┐
│  recurrence       │         │         │       MESSAGES          │
│  parent_task_id   │         │         │  id (UUID, PK)          │
│  created_at       │         │         │  conversation_id (FK)   │
│  updated_at       │         │         │  role (VARCHAR)         │
└─────────┬─────────┘         │         │  content (TEXT)         │
          │                   │         │  created_at             │
          │                   │         └─────────────────────────┘
          │ N:M               │
          ▼                   │
┌─────────────────────────────┴───────────────────────────────────┐
│                        TASK_TAGS (junction)                      │
│  task_id (FK, PK)                                               │
│  tag_id (FK, PK)                                                │
└─────────────────────────────────────────────────────────────────┘
```

## Entities

### User

Represents a registered account holder.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, auto-gen | Unique identifier |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Login credential |
| hashed_password | VARCHAR(255) | NOT NULL | bcrypt hash (12 rounds) |
| full_name | VARCHAR(255) | NOT NULL | Display name |
| profile_picture | VARCHAR(500) | nullable | URL to avatar image |
| created_at | TIMESTAMP | NOT NULL, default NOW | Account creation time |
| updated_at | TIMESTAMP | NOT NULL, auto-update | Last modification time |

**Indexes**:
- `users_email_idx` UNIQUE on (email)

**Relationships**:
- One User → Many Tasks
- One User → Many Tags
- One User → Many Conversations

---

### Task

Represents a unit of work to be completed.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, auto-gen | Unique identifier |
| user_id | UUID | FK → users.id, NOT NULL | Owner reference |
| title | VARCHAR(500) | NOT NULL | Task title |
| description | TEXT | nullable | Detailed description |
| is_completed | BOOLEAN | NOT NULL, default FALSE | Completion status |
| priority | ENUM | NOT NULL, default 'medium' | low, medium, high |
| due_date | TIMESTAMP | nullable | Deadline |
| remind_at | TIMESTAMP | nullable | Reminder notification time |
| reminder_sent | BOOLEAN | NOT NULL, default FALSE | Reminder delivery flag |
| recurrence | ENUM | NOT NULL, default 'none' | none, daily, weekly, monthly |
| parent_task_id | UUID | FK → tasks.id, nullable | Parent for recurring instances |
| created_at | TIMESTAMP | NOT NULL, default NOW | Task creation time |
| updated_at | TIMESTAMP | NOT NULL, auto-update | Last modification time |

**Indexes**:
- `tasks_user_completed_idx` on (user_id, is_completed)
- `tasks_user_due_date_idx` on (user_id, due_date)
- `tasks_user_priority_idx` on (user_id, priority)
- `tasks_parent_task_idx` on (parent_task_id)

**Relationships**:
- Many Tasks → One User
- Many Tasks ↔ Many Tags (via task_tags)
- One Task → Many Tasks (parent → children for recurring)

**Validation Rules**:
- title: 1-500 characters, non-empty after trim
- priority: must be one of [low, medium, high]
- recurrence: must be one of [none, daily, weekly, monthly]
- due_date: must be in future when setting reminder
- remind_at: must be before or equal to due_date

---

### Tag

Represents a category label for organizing tasks.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, auto-gen | Unique identifier |
| user_id | UUID | FK → users.id, NOT NULL | Owner reference |
| name | VARCHAR(100) | NOT NULL | Tag name (case-insensitive) |
| created_at | TIMESTAMP | NOT NULL, default NOW | Tag creation time |

**Indexes**:
- `tags_user_name_idx` UNIQUE on (user_id, LOWER(name))

**Relationships**:
- Many Tags → One User
- Many Tags ↔ Many Tasks (via task_tags)

**Validation Rules**:
- name: 1-100 characters, non-empty after trim
- name uniqueness enforced per user (case-insensitive)

---

### TaskTags (Junction Table)

Many-to-many relationship between tasks and tags.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| task_id | UUID | FK → tasks.id, PK | Task reference |
| tag_id | UUID | FK → tags.id, PK | Tag reference |

**Indexes**:
- Composite PK on (task_id, tag_id)
- `task_tags_tag_idx` on (tag_id)

**Cascade Rules**:
- DELETE task → DELETE task_tags entries
- DELETE tag → DELETE task_tags entries

---

### Conversation

Represents a chat session with the AI.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, auto-gen | Unique identifier |
| user_id | UUID | FK → users.id, NOT NULL | Owner reference |
| title | VARCHAR(255) | nullable | Conversation title (auto-generated) |
| created_at | TIMESTAMP | NOT NULL, default NOW | Session start time |
| updated_at | TIMESTAMP | NOT NULL, auto-update | Last activity time |

**Indexes**:
- `conversations_user_idx` on (user_id)
- `conversations_user_updated_idx` on (user_id, updated_at DESC)

**Relationships**:
- Many Conversations → One User
- One Conversation → Many Messages

---

### Message

Represents a single exchange in a conversation.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, auto-gen | Unique identifier |
| conversation_id | UUID | FK → conversations.id, NOT NULL | Parent conversation |
| role | ENUM | NOT NULL | user, assistant |
| content | TEXT | NOT NULL | Message content |
| created_at | TIMESTAMP | NOT NULL, default NOW | Message timestamp |

**Indexes**:
- `messages_conversation_idx` on (conversation_id)
- `messages_conversation_created_idx` on (conversation_id, created_at)

**Relationships**:
- Many Messages → One Conversation

**Cascade Rules**:
- DELETE conversation → DELETE messages

---

## Enumerations

### Priority
```python
class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
```

### Recurrence
```python
class Recurrence(str, Enum):
    NONE = "none"
    DAILY = "daily"
    WEEKLY = "weekly"
    MONTHLY = "monthly"
```

### MessageRole
```python
class MessageRole(str, Enum):
    USER = "user"
    ASSISTANT = "assistant"
```

## State Transitions

### Task Completion State Machine

```
┌──────────────┐     mark_complete     ┌──────────────┐
│  INCOMPLETE  │ ──────────────────▶  │   COMPLETE   │
│is_completed= │                       │is_completed= │
│    false     │ ◀──────────────────  │    true      │
└──────────────┘     mark_incomplete   └──────────────┘
                                              │
                                              │ (if recurring)
                                              ▼
                                    ┌──────────────────┐
                                    │  NEW TASK CREATED │
                                    │  (next occurrence)│
                                    └──────────────────┘
```

### Reminder State Machine

```
┌──────────────┐     set_reminder     ┌──────────────┐     time reached    ┌──────────────┐
│  NO REMINDER │ ──────────────────▶  │   PENDING    │ ──────────────────▶ │    SENT      │
│ remind_at=   │                      │ remind_at=   │                     │ remind_at=   │
│   NULL       │                      │  <datetime>  │                     │  <datetime>  │
│reminder_sent │                      │reminder_sent │                     │reminder_sent │
│   = false    │                      │   = false    │                     │   = true     │
└──────────────┘                      └──────────────┘                     └──────────────┘
```

## Query Patterns

### Common Queries (with index support)

1. **List user's incomplete tasks**
   ```sql
   SELECT * FROM tasks
   WHERE user_id = ? AND is_completed = false
   ORDER BY created_at DESC
   ```

2. **List overdue tasks**
   ```sql
   SELECT * FROM tasks
   WHERE user_id = ? AND due_date < NOW() AND is_completed = false
   ORDER BY due_date ASC
   ```

3. **Filter by priority**
   ```sql
   SELECT * FROM tasks
   WHERE user_id = ? AND priority = ?
   ORDER BY due_date ASC
   ```

4. **Filter by tag**
   ```sql
   SELECT t.* FROM tasks t
   JOIN task_tags tt ON t.id = tt.task_id
   JOIN tags tg ON tt.tag_id = tg.id
   WHERE t.user_id = ? AND LOWER(tg.name) = LOWER(?)
   ```

5. **Get pending reminders**
   ```sql
   SELECT * FROM tasks
   WHERE remind_at <= NOW() AND reminder_sent = false
   ORDER BY remind_at ASC
   ```

6. **Full-text search**
   ```sql
   SELECT * FROM tasks
   WHERE user_id = ? AND (
     title ILIKE '%search%' OR
     description ILIKE '%search%'
   )
   ```
