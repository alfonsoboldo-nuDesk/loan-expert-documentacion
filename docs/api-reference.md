# API Reference

Complete reference for the Champions Loan Expert REST API.

## Base URL

```
Production: https://champions-backend-561975502517.us-central1.run.app/api
Local:      http://localhost:8080/api
```

## Authentication

All endpoints (except `/auth/login`) require JWT authentication.

```http
Authorization: Bearer <jwt_token>
```

---

## Authentication Endpoints

### POST `/auth/login`

Authenticate user and receive JWT token.

**Request:**
```json
{
  "email": "user@championsfunding.com",
  "password": "shared_password"
}
```

**Response (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "user": {
    "id": "uuid",
    "email": "user@championsfunding.com",
    "role": "rep"
  }
}
```

**Errors:**
| Code | Description |
|------|-------------|
| 401 | Invalid credentials |
| 400 | Invalid email domain |

---

### GET `/auth/me`

Get current authenticated user.

**Response (200):**
```json
{
  "id": "uuid",
  "email": "user@championsfunding.com",
  "role": "rep"
}
```

---

### GET `/auth/users`

List all users. Requires `manager` or `admin` role.

**Response (200):**
```json
{
  "items": [
    {
      "id": "uuid",
      "email": "user@championsfunding.com",
      "role": "rep",
      "status": "active",
      "last_login": "2025-01-27T10:00:00Z",
      "created_at": "2025-01-01T00:00:00Z"
    }
  ]
}
```

---

### PATCH `/auth/users/{user_id}/role`

Update user role. **Admin only.**

**Request:**
```json
{
  "role": "manager"
}
```

**Response (200):**
```json
{
  "id": "uuid",
  "email": "user@championsfunding.com",
  "role": "manager"
}
```

---

## Chat Endpoints

### POST `/chat`

Send message and receive streaming response.

**Request:**
```json
{
  "message": "What are the DSCR requirements?",
  "conversation_id": "uuid-or-null",
  "stream": true,
  "metadata_filter": null
}
```

**Response (SSE Stream):**
```
data: {"type":"session_init","conversation_id":"uuid","message_id":"uuid"}

data: {"type":"tool_use","tool":"file_search","status":"start"}

data: {"type":"delta","content":"The DSCR requirements "}

data: {"type":"delta","content":"are..."}

data: {"type":"tool_use","tool":"file_search","status":"complete"}

data: {"type":"citations","citations":[...],"inline_citations":[...]}

data: {"type":"done","message_id":"uuid","tokens_used":1234}
```

**SSE Event Types:**

| Event | Fields | Description |
|-------|--------|-------------|
| `session_init` | `conversation_id`, `message_id` | New session started |
| `tool_use` | `tool`, `status` | File search start/end |
| `delta` | `content` | Text chunk |
| `citations` | `citations`, `inline_citations` | Source references |
| `done` | `message_id`, `tokens_used` | Stream completed |
| `error` | `code`, `error` | Error occurred |

---

### GET `/suggestions`

Get dynamic chat suggestions.

**Response (200):**
```json
{
  "suggestions": [
    {
      "text": "What are the minimum DSCR requirements?",
      "source": "feedback"
    },
    {
      "text": "Compare Full Doc vs Alt Doc programs",
      "source": "frequent"
    },
    {
      "text": "What are the reserve requirements?",
      "source": "default"
    }
  ]
}
```

**Suggestion Sources:**
| Source | Description |
|--------|-------------|
| `feedback` | Questions with positive feedback (thumbs_up) |
| `frequent` | Frequently successful queries |
| `default` | Generated from indexed documents |

---

## Conversation Endpoints

### GET `/conversations`

List user's conversations.

**Query Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `limit` | int | 20 | Max results |
| `cursor` | string | null | Pagination cursor |
| `include_archived` | bool | false | Include archived |

**Response (200):**
```json
{
  "items": [
    {
      "id": "uuid",
      "title": "DSCR Requirements",
      "archived": false,
      "created_at": "2025-01-27T10:00:00Z",
      "updated_at": "2025-01-27T10:30:00Z"
    }
  ],
  "next_cursor": "2025-01-27T10:00:00Z"
}
```

---

### GET `/conversations/{id}`

Get conversation with messages and citations.

**Response (200):**
```json
{
  "conversation": {
    "id": "uuid",
    "title": "DSCR Requirements",
    "archived": false,
    "created_at": "2025-01-27T10:00:00Z",
    "updated_at": "2025-01-27T10:30:00Z"
  },
  "messages": [
    {
      "id": "uuid",
      "role": "user",
      "content": "What are the DSCR requirements?",
      "created_at": "2025-01-27T10:00:00Z",
      "inline_citations": null
    },
    {
      "id": "uuid",
      "role": "assistant",
      "content": "The DSCR requirements are...",
      "model": "gemini-3-flash",
      "tokens_used": 500,
      "created_at": "2025-01-27T10:00:05Z",
      "inline_citations": [
        {
          "citation_number": 1,
          "start_index": 0,
          "end_index": 25,
          "source_name": "DSCR_Matrix.md",
          "page_number": 2
        }
      ]
    }
  ],
  "citations": [
    {
      "index": 1,
      "source_name": "DSCR_Matrix.md",
      "source_uri": "files/abc123",
      "page_number": 2,
      "page_numbers": [2, 3],
      "snippet": "Minimum DSCR: 1.0...",
      "confidence": null,
      "metadata": null
    }
  ]
}
```

---

### PATCH `/conversations/{id}`

Update conversation title.

**Request:**
```json
{
  "title": "New Title"
}
```

**Response (200):**
```json
{
  "id": "uuid",
  "title": "New Title",
  "archived": false
}
```

---

### POST `/conversations/{id}/archive`

Archive or restore conversation.

**Request:**
```json
{
  "archived": true
}
```

**Response (200):**
```json
{
  "id": "uuid",
  "archived": true,
  "archived_at": "2025-01-27T10:00:00Z"
}
```

---

### POST `/conversations/feedback`

Submit feedback on a message.

**Request:**
```json
{
  "message_id": "uuid",
  "rating": "thumbs_up",
  "comment": "Very helpful!"
}
```

**Response (200):**
```json
{
  "id": "uuid",
  "rating": "thumbs_up",
  "comment": "Very helpful!",
  "created_at": "2025-01-27T10:00:00Z"
}
```

---

## Admin Endpoints

All admin endpoints require `admin` or `manager` role.

### GET `/admin/analytics`

Get analytics with time series data.

**Query Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `days` | int | 30 | Period in days |

**Response (200):**
```json
{
  "summary": {
    "total_users": 25,
    "total_conversations": 150,
    "total_messages": 1200,
    "total_tokens": 500000,
    "total_runs": 600,
    "total_errors": 15
  },
  "daily_stats": [
    {
      "date": "2025-01-27",
      "runs": 50,
      "tokens": 25000,
      "errors": 2,
      "active_users": 10
    }
  ],
  "period_days": 30
}
```

---

### GET `/admin/runs`

Get run logs with pagination.

**Query Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `limit` | int | 50 | Max results |
| `offset` | int | 0 | Skip results |
| `status` | string | null | Filter by status |
| `user_id` | string | null | Filter by user |

**Response (200):**
```json
{
  "items": [
    {
      "id": "uuid",
      "user_id": "uuid",
      "user_email": "user@championsfunding.com",
      "conversation_id": "uuid",
      "message_id": "uuid",
      "input_preview": "What are the...",
      "input_full": "What are the DSCR requirements?",
      "model": "gemini-3-flash",
      "tokens_used": 500,
      "latency_ms": 2500,
      "status": "success",
      "error_code": null,
      "feedback": {
        "rating": "thumbs_up",
        "comment": null
      },
      "created_at": "2025-01-27T10:00:00Z"
    }
  ],
  "total": 600,
  "next_cursor": "uuid"
}
```

---

### DELETE `/admin/conversations/{id}`

Delete a conversation. **Admin only.**

**Response (200):**
```json
{
  "deleted": true
}
```

---

### GET `/admin/documents`

List indexed documents in File Search Store.

**Response (200):**
```json
{
  "items": [
    {
      "id": "uuid",
      "file_search_doc_id": "abc123",
      "filename": "DSCR_Matrix.md",
      "program": "DSCR",
      "is_active": true,
      "created_at": "2025-01-01T00:00:00Z"
    }
  ]
}
```

---

## Error Responses

All errors follow this format:

```json
{
  "detail": {
    "error": "Human readable message",
    "code": "ERROR_CODE"
  }
}
```

### Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| `NOT_FOUND` | 404 | Resource not found |
| `UNAUTHORIZED` | 401 | Invalid or missing token |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `CONVERSATION_ARCHIVED` | 400 | Cannot modify archived conversation |
| `MESSAGE_LIMIT` | 400 | Conversation has 200 messages |
| `CONVERSATION_LIMIT` | 400 | User has 100 active conversations |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `NO_FILE_SEARCH_STORE` | 500 | File Search not configured |
| `CHAT_FAILED` | 500 | AI response failed |
