# Architecture Deep Dive

This document provides a detailed walkthrough of the Champions Loan Expert system architecture.

## System Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              Google Cloud Platform                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │                    Cloud Run: Frontend (Next.js 14)                     │   │
│   │                  champions-frontend-561975502517                        │   │
│   │                                                                         │   │
│   │   Browser ──► Static Assets ──► React App ──► API Calls                 │   │
│   │                                                                         │   │
│   └───────────────────────────────────┬─────────────────────────────────────┘   │
│                                       │                                         │
│                          REST API + Server-Sent Events                          │
│                                       │                                         │
│   ┌───────────────────────────────────▼─────────────────────────────────────┐   │
│   │                    Cloud Run: Backend (FastAPI)                         │   │
│   │                   champions-backend-561975502517                        │   │
│   │                                                                         │   │
│   │   API Router ──► Services ──► Database ──► External APIs                │   │
│   │                                                                         │   │
│   └───────────────────────────────────┬─────────────────────────────────────┘   │
│                                       │                                         │
└───────────────────────────────────────┼─────────────────────────────────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
                    ▼                   ▼                   ▼
            ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
            │  Cloud SQL    │   │ Gemini 3      │   │    Secret     │
            │  PostgreSQL   │   │ File Search   │   │    Manager    │
            └───────────────┘   └───────────────┘   └───────────────┘
```

---

## Frontend Architecture

### Technology Stack
- **Framework:** Next.js 14 (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **State:** Zustand (chat, auth, theme, notifications)
- **Icons:** Lucide React
- **Markdown:** react-markdown + remark-gfm

### Complete File Structure

```
frontend/src/
├── app/                           # Next.js App Router
│   ├── layout.tsx                 # Root layout (providers, fonts)
│   ├── page.tsx                   # Landing → redirects to /chat
│   ├── chat/
│   │   └── page.tsx               # Main chat interface
│   ├── admin/
│   │   └── page.tsx               # Admin panel (5 tabs)
│   └── settings/
│       └── page.tsx               # User preferences
│
├── components/
│   ├── auth/
│   │   ├── LoginForm.tsx          # Email/password form
│   │   └── index.ts
│   │
│   ├── chat/
│   │   ├── ChatLayout.tsx         # Main layout (sidebar + chat)
│   │   ├── ChatInput.tsx          # Message input with send
│   │   ├── MessageList.tsx        # Messages with inline citations
│   │   ├── Sidebar.tsx            # Conversation list + header
│   │   ├── SourcesPanel.tsx       # Citations sidebar
│   │   └── index.ts
│   │
│   ├── providers/
│   │   ├── ThemeProvider.tsx      # Dark mode context
│   │   └── ChunkErrorHandler.tsx  # Next.js error boundary
│   │
│   └── ui/
│       ├── Button.tsx             # Reusable button
│       ├── Input.tsx              # Reusable input
│       ├── Toast.tsx              # Toast notifications
│       └── index.ts
│
├── hooks/
│   └── useTranslation.ts          # i18n hook
│
├── lib/
│   ├── api.ts                     # API client (authApi, chatApi, adminApi)
│   ├── i18n.ts                    # Translation strings (es/en)
│   └── utils.ts                   # cn() class merger
│
├── store/
│   ├── auth.ts                    # Auth state (user, token, login/logout)
│   ├── chat.ts                    # Chat state (messages, streaming, citations)
│   └── notifications.ts           # Toast notifications
│
└── types/
    └── index.ts                   # TypeScript interfaces
```

### Page Component Details

| Page | Route | Features |
|------|-------|----------|
| **Chat** | `/chat` | Message list, input, sources panel, conversation sidebar |
| **Admin** | `/admin` | 5 tabs: Analytics, Logs, Users, Conversations, Documents |
| **Settings** | `/settings` | Theme toggle, language selector |

### Key Components

| Component | File | Purpose |
|-----------|------|---------|
| `ChatLayout` | `components/chat/ChatLayout.tsx` | Grid layout: sidebar + main + sources |
| `MessageList` | `components/chat/MessageList.tsx` | Renders messages with markdown, inline citations, feedback buttons |
| `ChatInput` | `components/chat/ChatInput.tsx` | Textarea with send button, Enter to submit |
| `Sidebar` | `components/chat/Sidebar.tsx` | Conversation list, new chat, settings link |
| `SourcesPanel` | `components/chat/SourcesPanel.tsx` | Collapsible citations panel, page numbers |
| `LoginForm` | `components/auth/LoginForm.tsx` | Email + password, error handling |
| `Toast` | `components/ui/Toast.tsx` | Success/error/warning notifications |
| `ThemeProvider` | `components/providers/ThemeProvider.tsx` | Dark mode context with system detection |

### State Management (Zustand Stores)

```typescript
// Auth Store - Authentication state
interface AuthStore {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;

  // Actions
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  checkAuth: () => Promise<boolean>;
}

// Chat Store - Main state for chat functionality
interface ChatStore {
  // Conversation
  conversationId: string | null;
  messages: Message[];
  citations: Citation[];
  conversations: Conversation[];

  // Streaming
  isStreaming: boolean;
  streamingContent: string;
  toolStatus: string | null;

  // Inline Citations (for persistent markers)
  inlineCitations: InlineCitation[];
  inlineCitationsMessageId: string | null;

  // UI State
  highlightedCitation: number | null;
  isLoadingMessages: boolean;

  // Actions
  sendMessage: (message: string) => Promise<void>;
  loadConversation: (id: string) => Promise<void>;
  loadConversations: () => Promise<void>;
  createNewConversation: () => void;
  archiveConversation: (id: string) => Promise<void>;
  highlightCitation: (index: number) => void;
}

// Notification Store - Toast notifications
interface NotificationStore {
  notifications: Notification[];

  // Actions
  addNotification: (type: 'success' | 'error' | 'warning' | 'info', message: string) => void;
  removeNotification: (id: string) => void;
}
```

### API Client Structure

```typescript
// lib/api.ts
export const authApi = {
  login: (email, password) => POST('/auth/login'),
  me: () => GET('/auth/me'),
  users: () => GET('/auth/users'),
  updateRole: (userId, role) => PATCH(`/auth/users/${userId}/role`),
};

export const chatApi = {
  sendMessage: (message, conversationId) => POST('/chat', { stream: true }),
  getSuggestions: () => GET('/suggestions'),
};

export const conversationsApi = {
  list: (cursor?) => GET('/conversations'),
  get: (id) => GET(`/conversations/${id}`),
  update: (id, title) => PATCH(`/conversations/${id}`),
  archive: (id, archived) => POST(`/conversations/${id}/archive`),
  submitFeedback: (data) => POST('/conversations/feedback'),
};

export const adminApi = {
  analytics: (days) => GET(`/admin/analytics?days=${days}`),
  stats: () => GET('/admin/stats'),
  runs: (params) => GET('/admin/runs'),
  users: () => GET('/admin/users'),
  conversations: () => GET('/admin/conversations'),
  documents: () => GET('/admin/documents'),
  deleteConversation: (id) => DELETE(`/admin/conversations/${id}`),
};
```

### SSE Stream Processing

```typescript
// In chat store - sendMessage action
const response = await fetch('/api/chat', { method: 'POST', body: JSON.stringify(request) });
const reader = response.body.getReader();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  const text = new TextDecoder().decode(value);
  const lines = text.split('\n');

  for (const line of lines) {
    if (line.startsWith('data: ')) {
      const event = JSON.parse(line.slice(6));

      switch (event.type) {
        case 'session_init':
          set({ conversationId: event.conversation_id });
          break;
        case 'tool_use':
          set({ toolStatus: event.status === 'start' ? 'Searching documents...' : null });
          break;
        case 'delta':
          set(state => ({ streamingContent: state.streamingContent + event.content }));
          break;
        case 'citations':
          set({
            citations: event.citations,
            inlineCitations: event.inline_citations || [],
            inlineCitationsMessageId: currentMessageId
          });
          break;
        case 'done':
          // Finalize message and add to messages array
          break;
        case 'error':
          notify.error(event.message);
          break;
      }
    }
  }
}
```

---

## Backend Architecture

### Technology Stack
- **Framework:** FastAPI (Python 3.11)
- **ORM:** SQLAlchemy (async)
- **Validation:** Pydantic v2
- **AI:** google-genai SDK

### Module Structure

```
backend/app/
├── main.py              # FastAPI application entry
├── config.py            # Environment configuration
│
├── api/                 # REST endpoints
│   ├── auth.py          # /api/auth/* endpoints
│   ├── chat.py          # /api/chat, /api/suggestions
│   ├── conversations.py # /api/conversations/*
│   ├── admin.py         # /api/admin/*
│   └── deps.py          # Dependency injection
│
├── models/
│   ├── database.py      # SQLAlchemy ORM models
│   ├── schemas.py       # Pydantic request/response
│   └── errors.py        # Error codes and classification
│
├── services/
│   ├── chat_service.py        # Gemini + File Search RAG
│   ├── conversation_service.py # CRUD operations
│   └── auth_service.py        # JWT handling
│
└── db/
    └── connection.py    # Database session management
```

### Request Flow

```
User Request
     │
     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  API Layer (FastAPI Router)                                                  │
│  ─────────────────────────────────────────────────────────────────────────── │
│                                                                              │
│  1. Receive HTTP Request                                                     │
│  2. Validate with Pydantic schema                                            │
│  3. Check JWT authentication                                                 │
│  4. Route to appropriate endpoint                                            │
│                                                                              │
└────────────────────────────────────┬────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Service Layer                                                               │
│  ─────────────────────────────────────────────────────────────────────────── │
│                                                                              │
│  ChatService:                                                                │
│    • Build conversation context                                              │
│    • Detect loan program from message                                        │
│    • Call Gemini with File Search                                            │
│    • Extract citations from grounding_metadata                               │
│    • Stream response via SSE                                                 │
│                                                                              │
│  ConversationService:                                                        │
│    • CRUD operations for conversations                                       │
│    • Message management                                                      │
│    • Citation storage                                                        │
│                                                                              │
└────────────────────────────────────┬────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Data Layer                                                                  │
│  ─────────────────────────────────────────────────────────────────────────── │
│                                                                              │
│  PostgreSQL (Cloud SQL):                                                     │
│    • users          - User accounts                                          │
│    • conversations  - Chat sessions                                          │
│    • messages       - Individual messages                                    │
│    • citations      - Source references                                      │
│    • run_logs       - Analytics data                                         │
│    • feedback       - User ratings                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Database Schema

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATABASE SCHEMA                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐       ┌─────────────────┐       ┌─────────────────┐        │
│  │   users     │       │  conversations  │       │    messages     │        │
│  ├─────────────┤       ├─────────────────┤       ├─────────────────┤        │
│  │ id (PK)     │──────►│ id (PK)         │──────►│ id (PK)         │        │
│  │ email       │       │ user_id (FK)    │       │ conversation_id │        │
│  │ role        │       │ title           │       │ role            │        │
│  │ status      │       │ archived        │       │ content         │        │
│  │ created_at  │       │ created_at      │       │ model           │        │
│  │ last_login  │       │ updated_at      │       │ tokens_used     │        │
│  └─────────────┘       └─────────────────┘       │ created_at      │        │
│                                                  └────────┬────────┘        │
│                                                           │                 │
│                                                           ▼                 │
│  ┌─────────────┐       ┌─────────────────┐       ┌─────────────────┐        │
│  │  run_logs   │       │    feedback     │       │    citations    │        │
│  ├─────────────┤       ├─────────────────┤       ├─────────────────┤        │
│  │ id (PK)     │       │ id (PK)         │       │ id (PK)         │        │
│  │ user_id     │       │ message_id (FK) │       │ message_id (FK) │        │
│  │ conv_id     │       │ user_id (FK)    │       │ source_index    │        │
│  │ message_id  │       │ rating          │       │ source_name     │        │
│  │ model       │       │ comment         │       │ source_uri      │        │
│  │ tokens_used │       │ created_at      │       │ page_number     │        │
│  │ latency_ms  │       └─────────────────┘       │ snippet         │        │
│  │ status      │                                 │ confidence      │        │
│  │ error_code  │                                 │ text_start_index│        │
│  │ created_at  │                                 │ text_end_index  │        │
│  └─────────────┘                                 │ inline_number   │        │
│                                                  └─────────────────┘        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Error Handling

### Error Code Classification

```python
class ErrorCode(str, Enum):
    # Configuration errors
    NO_FILE_SEARCH_STORE = "NO_FILE_SEARCH_STORE"
    MODEL_NOT_FOUND = "MODEL_NOT_FOUND"

    # Rate limiting
    RATE_LIMIT_EXCEEDED = "RATE_LIMIT_EXCEEDED"
    QUOTA_EXCEEDED = "QUOTA_EXCEEDED"

    # Service errors
    SERVICE_UNAVAILABLE = "SERVICE_UNAVAILABLE"
    CONNECTION_ERROR = "CONNECTION_ERROR"
    TIMEOUT = "TIMEOUT"

    # Response errors
    NO_RESPONSE = "NO_RESPONSE"
    NO_CITATIONS = "NO_CITATIONS"
    INVALID_RESPONSE = "INVALID_RESPONSE"
    CONTENT_BLOCKED = "CONTENT_BLOCKED"

    # Generic
    CHAT_FAILED = "CHAT_FAILED"
```

All errors are:
1. Classified by `classify_gemini_error()`
2. Logged with specific code
3. Tracked in `run_logs` table
4. Returned to client with friendly message
