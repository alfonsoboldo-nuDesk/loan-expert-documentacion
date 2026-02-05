# Database Documentation

The application uses PostgreSQL with SQLAlchemy as the ORM.

## Tables

### users

Stores user accounts.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| email | String | User email (unique) |
| hashed_password | String | Bcrypt hashed password |
| role | String | `rep`, `manager`, or `admin` |
| status | String | `active` or `suspended` |
| created_at | DateTime | Account creation time |
| last_login | DateTime | Last login time |

### conversations

Stores chat sessions.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| user_id | UUID | Foreign key to users |
| title | String | Conversation title |
| archived | Boolean | Whether archived |
| archived_at | DateTime | When archived |
| created_at | DateTime | Creation time |
| updated_at | DateTime | Last update time |

### messages

Stores individual chat messages.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| conversation_id | UUID | Foreign key to conversations |
| role | String | `user` or `assistant` |
| content | Text | Message content |
| model | String | AI model used (for assistant) |
| tokens_used | Integer | Token count (for assistant) |
| created_at | DateTime | Message time |

### citations

Stores document references for assistant messages.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| message_id | UUID | Foreign key to messages |
| source_index | Integer | Citation number (1, 2, 3...) |
| source_name | String | Document name |
| source_uri | String | Gemini file URI |
| page_number | Integer | Page number |
| snippet | Text | Quoted text |
| confidence | Float | Confidence score |
| text_start_index | Integer | Position in text where citation starts |
| text_end_index | Integer | Position in text where citation ends |
| inline_number | Integer | Superscript number shown to user |

### feedback

Stores user ratings on responses.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| message_id | UUID | Foreign key to messages |
| user_id | UUID | Foreign key to users |
| rating | String | `thumbs_up` or `thumbs_down` |
| comment | Text | Optional comment |
| created_at | DateTime | Feedback time |

### run_logs

Stores analytics data for each AI request.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| user_id | UUID | Foreign key to users |
| conversation_id | UUID | Foreign key to conversations |
| message_id | UUID | Foreign key to messages |
| model | String | AI model used |
| tokens_used | Integer | Total tokens |
| latency_ms | Integer | Response time in milliseconds |
| status | String | `success` or `error` |
| error_code | String | Error code if failed |
| created_at | DateTime | Request time |

---

## Relationships

```
users
  └── conversations (one-to-many)
        └── messages (one-to-many)
              ├── citations (one-to-many)
              └── feedback (one-to-one)
  └── run_logs (one-to-many)
  └── feedback (one-to-many)
```

---

## Migrations

Migrations are handled by `scripts/run_migration.py`.

### Run a migration
```bash
cd backend
python scripts/run_migration.py
```

### Verify migration status
```bash
python scripts/run_migration.py --verify
```

### Rollback last migration
```bash
python scripts/run_migration.py --rollback
```

---

## Connection

The database connection is configured via the `DATABASE_URL` environment variable:

```
postgresql+asyncpg://user:password@host:5432/database
```

For Cloud SQL, the connection is made through the Cloud SQL Proxy or direct private IP.

---

## Indexes

The following columns are indexed for performance:

- `users.email` (unique)
- `conversations.user_id`
- `conversations.archived`
- `messages.conversation_id`
- `citations.message_id`
- `feedback.message_id`
- `run_logs.user_id`
- `run_logs.created_at`

---

## Schema Diagram

```
┌─────────────┐       ┌─────────────────┐       ┌─────────────────┐
│   users     │       │  conversations  │       │    messages     │
├─────────────┤       ├─────────────────┤       ├─────────────────┤
│ id (PK)     │──┐    │ id (PK)         │──┐    │ id (PK)         │
│ email       │  │    │ user_id (FK) ◄──┘  │    │ conversation_id │◄──┘
│ role        │  │    │ title           │  │    │ role            │
│ status      │  │    │ archived        │  │    │ content         │
│ created_at  │  │    │ created_at      │  │    │ model           │
│ last_login  │  │    │ updated_at      │  │    │ tokens_used     │
└─────────────┘  │    └─────────────────┘  │    │ created_at      │
                 │                         │    └────────┬────────┘
                 │                         │             │
                 │    ┌─────────────────┐  │    ┌────────▼────────┐
                 │    │    feedback     │  │    │    citations    │
                 │    ├─────────────────┤  │    ├─────────────────┤
                 │    │ id (PK)         │  │    │ id (PK)         │
                 └───►│ user_id (FK)    │  │    │ message_id (FK) │
                      │ message_id (FK) │◄─┘    │ source_name     │
                      │ rating          │       │ source_uri      │
                      │ comment         │       │ page_number     │
                      │ created_at      │       │ snippet         │
                      └─────────────────┘       │ inline_number   │
                                                └─────────────────┘
```

---

## Sample Queries

### Get user's conversations
```sql
SELECT * FROM conversations
WHERE user_id = $1 AND archived = false
ORDER BY updated_at DESC;
```

### Get conversation with messages
```sql
SELECT m.*, c.*
FROM messages m
LEFT JOIN citations c ON c.message_id = m.id
WHERE m.conversation_id = $1
ORDER BY m.created_at;
```

### Get usage analytics
```sql
SELECT
  DATE(created_at) as date,
  COUNT(*) as runs,
  SUM(tokens_used) as tokens,
  COUNT(CASE WHEN status = 'error' THEN 1 END) as errors
FROM run_logs
WHERE created_at > NOW() - INTERVAL '30 days'
GROUP BY DATE(created_at)
ORDER BY date;
```

---

## Backup and Recovery

### Cloud SQL Automatic Backups
Cloud SQL provides automatic daily backups with 7-day retention.

### Manual Export
```bash
gcloud sql export sql INSTANCE_NAME gs://bucket/export.sql --database=loan_expert
```

### Point-in-Time Recovery
Cloud SQL supports point-in-time recovery for the last 7 days.
