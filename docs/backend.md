# Backend Documentation

The backend is a FastAPI application with Python 3.11, SQLAlchemy, and the Google GenAI SDK.

## Technology Stack

| Technology | Purpose |
|------------|---------|
| FastAPI | High-performance Python web framework |
| Python 3.11 | Runtime |
| SQLAlchemy | Async ORM |
| Pydantic v2 | Data validation |
| google-genai | Gemini SDK |
| asyncpg | Async PostgreSQL driver |

---

## Project Structure

```
backend/
├── app/
│   ├── main.py             # FastAPI application entry point
│   ├── config.py           # Environment configuration
│   │
│   ├── api/                # REST endpoints
│   │   ├── auth.py         # Login, user management
│   │   ├── chat.py         # Chat with AI, suggestions
│   │   ├── conversations.py # Conversation CRUD
│   │   ├── admin.py        # Admin dashboard endpoints
│   │   ├── health.py       # Health check
│   │   └── deps.py         # Dependency injection
│   │
│   ├── models/
│   │   ├── database.py     # SQLAlchemy ORM models
│   │   ├── schemas.py      # Pydantic request/response models
│   │   └── errors.py       # Error codes
│   │
│   ├── services/
│   │   ├── chat_service.py        # Gemini AI integration
│   │   ├── conversation_service.py # Conversation operations
│   │   └── auth_service.py        # JWT and authentication
│   │
│   └── db/
│       └── connection.py   # Database session management
│
├── scripts/                # Utility scripts
│   ├── setup_db.py         # Initialize database
│   ├── run_migration.py    # Run migrations
│   ├── create_admin.py     # Create admin user
│   └── setup_file_search.py # Upload documents to Gemini
│
├── file_search_name_map.json # Maps Gemini file IDs to names
├── Dockerfile
├── requirements.txt
└── cloudbuild.yaml         # Cloud Build configuration
```

---

## Services

### ChatService (`services/chat_service.py`)

Handles all AI interactions with Gemini.

**Key methods:**

| Method | Purpose |
|--------|---------|
| `chat_stream()` | Send message to Gemini and stream response |
| `_extract_citations()` | Extract document references from response |
| `_resolve_source_name()` | Convert file IDs to readable names |
| `_detect_program()` | Identify which loan program user is asking about |

**Features:**
- Uses Gemini 3 Flash with File Search
- Streams responses via Server-Sent Events (SSE)
- Extracts and formats citations from grounding metadata
- Caches responses for repeated questions

### ConversationService (`services/conversation_service.py`)

Manages conversations and messages.

| Method | Purpose |
|--------|---------|
| `create_conversation()` | Start a new conversation |
| `get_conversation()` | Get conversation with messages |
| `add_message()` | Save a message |
| `add_citations()` | Save citation data |
| `archive_conversation()` | Archive/unarchive |
| `submit_feedback()` | Save user rating |

### AuthService (`services/auth_service.py`)

Handles authentication.

| Method | Purpose |
|--------|---------|
| `authenticate_user()` | Validate email and password |
| `create_access_token()` | Generate JWT |
| `get_current_user()` | Decode JWT and get user |

---

## API Endpoints

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/login` | Login and get JWT |
| GET | `/api/auth/me` | Get current user |
| GET | `/api/auth/users` | List users (manager+) |
| PATCH | `/api/auth/users/{id}/role` | Change user role (admin) |

### Chat

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/chat` | Send message, get streaming response |
| GET | `/api/suggestions` | Get suggested questions |

### Conversations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/conversations` | List user's conversations |
| GET | `/api/conversations/{id}` | Get conversation with messages |
| PATCH | `/api/conversations/{id}` | Update title |
| POST | `/api/conversations/{id}/archive` | Archive/unarchive |
| POST | `/api/conversations/feedback` | Submit feedback |

### Admin

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/admin/analytics` | Usage analytics |
| GET | `/api/admin/users` | All users |
| GET | `/api/admin/runs` | Run logs |
| GET | `/api/admin/conversations` | All conversations |
| GET | `/api/admin/documents` | Indexed documents |
| DELETE | `/api/admin/conversations/{id}` | Delete conversation |

---

## File Routing System

Gemini returns file IDs like `files/my6heaulw7lm`. The file routing system converts these to readable names using `file_search_name_map.json`:

```json
{
  "my6heaulw7lm": "Accelerator-Activator_Alt_Doc_Matrix.md",
  "6m7zk1jwfroi": "Accelerator-Activator_Full_Doc_Matrix.md"
}
```

The `_resolve_source_name()` function in ChatService handles this conversion.

---

## Error Handling

All errors are classified with specific codes:

| Code | Meaning |
|------|---------|
| `NO_FILE_SEARCH_STORE` | Gemini File Search not configured |
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `QUOTA_EXCEEDED` | Billing limit reached |
| `SERVICE_UNAVAILABLE` | Gemini temporarily down |
| `NO_CITATIONS` | No documents matched query |
| `CHAT_FAILED` | Generic AI error |

Errors are logged to the `run_logs` table for monitoring.

---

## SSE Event Types

The chat endpoint returns Server-Sent Events:

```
event: session_init
data: {"type":"session_init","conversation_id":"...","message_id":"..."}

event: tool_use
data: {"type":"tool_use","tool":"file_search","status":"start"}

event: delta
data: {"type":"delta","content":"The DSCR requirements are..."}

event: citations
data: {"type":"citations","citations":[...],"inline_citations":[...]}

event: done
data: {"type":"done","message_id":"...","tokens_used":500}
```

---

## Scripts

### setup_db.py

Initialize database tables:
```bash
python scripts/setup_db.py
python scripts/setup_db.py --reset  # Warning: deletes all data
```

### run_migration.py

Run database migrations:
```bash
python scripts/run_migration.py
python scripts/run_migration.py --verify
python scripts/run_migration.py --rollback
```

### create_admin.py

Create admin users:
```bash
python scripts/create_admin.py admin@championsfunding.com
python scripts/create_admin.py user@championsfunding.com --role manager
```

### setup_file_search.py

Upload documents to Gemini File Search:
```bash
python scripts/setup_file_search.py
python scripts/setup_file_search.py --clear  # Recreate store
```

---

## Environment Variables

### Required

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | JWT signing key (min 32 chars) |
| `SHARED_PASSWORD` | Team shared password |
| `GOOGLE_API_KEY` | Gemini API key |
| `GEMINI_FILE_SEARCH_STORE` | File Search Store ID |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `GEMINI_MODEL` | `gemini-3-flash` | Model to use |
| `LOG_LEVEL` | `INFO` | Logging level |
| `CORS_ORIGINS` | `*` | Allowed origins |

---

## Running Locally

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set environment variables (see .env.example)

# Initialize database
python scripts/setup_db.py

# Run server
uvicorn app.main:app --reload --port 8080
```

Backend will be at: http://localhost:8080
API docs at: http://localhost:8080/docs
