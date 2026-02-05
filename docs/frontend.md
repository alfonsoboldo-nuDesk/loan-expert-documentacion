# Frontend Documentation

The frontend is a Next.js 14 application using TypeScript, Tailwind CSS, and Zustand for state management.

## Technology Stack

| Technology | Purpose |
|------------|---------|
| Next.js 14 | React framework with App Router |
| TypeScript | Type safety |
| Tailwind CSS | Utility-first styling |
| Zustand | Lightweight state management |
| Lucide React | Icon library |
| react-markdown | Markdown rendering |

---

## Project Structure

```
frontend/src/
├── app/                    # Next.js App Router pages
│   ├── layout.tsx          # Root layout with providers
│   ├── page.tsx            # Landing (redirects to /chat)
│   ├── chat/page.tsx       # Main chat interface
│   ├── admin/page.tsx      # Admin dashboard
│   └── settings/page.tsx   # User settings
│
├── components/
│   ├── auth/               # Authentication components
│   │   └── LoginForm.tsx   # Login form
│   │
│   ├── chat/               # Chat components
│   │   ├── ChatLayout.tsx  # Main layout container
│   │   ├── ChatInput.tsx   # Message input
│   │   ├── MessageList.tsx # Message rendering
│   │   ├── Sidebar.tsx     # Conversation list
│   │   └── SourcesPanel.tsx # Citations panel
│   │
│   ├── providers/          # React providers
│   │   └── ThemeProvider.tsx
│   │
│   └── ui/                 # Reusable UI components
│       ├── Button.tsx
│       ├── Input.tsx
│       └── Toast.tsx
│
├── lib/
│   ├── api.ts              # API client
│   ├── i18n.ts             # Translations
│   └── utils.ts            # Utilities
│
├── store/
│   ├── auth.ts             # Auth state
│   ├── chat.ts             # Chat state
│   └── notifications.ts    # Toast state
│
└── types/
    └── index.ts            # TypeScript types
```

---

## Pages

### Chat Page (`/chat`)

The main interface where users interact with the AI assistant.

**Features:**
- Conversation list in sidebar
- Message history with inline citations
- Real-time streaming responses
- Citation panel showing sources
- Feedback buttons (thumbs up/down)

**Components used:**
- `ChatLayout` - Grid container
- `Sidebar` - Conversation list
- `MessageList` - Messages
- `ChatInput` - Input field
- `SourcesPanel` - Citations

### Admin Page (`/admin`)

Dashboard for managers and admins to monitor usage.

**Tabs:**
1. **Analytics** - Usage charts and metrics
2. **Logs** - Request history with filtering
3. **Users** - User management
4. **Conversations** - All conversations
5. **Documents** - Indexed documents

**Required role:** `manager` or `admin`

### Settings Page (`/settings`)

User preferences.

**Options:**
- Theme (Light / Dark / System)
- Language (English / Spanish)

---

## Components

### ChatLayout

Main layout component that arranges the chat interface.

```tsx
<ChatLayout>
  ├── Sidebar (left)
  ├── Main content (center)
  │   ├── MessageList
  │   └── ChatInput
  └── SourcesPanel (right, collapsible)
</ChatLayout>
```

### MessageList

Renders conversation messages with:
- Markdown formatting
- Inline citation markers `[¹]`
- Feedback buttons
- Copy message button
- Timestamp

**Key feature:** Citation markers are clickable and highlight the corresponding source in the panel.

### SourcesPanel

Shows citations for the current response:
- Document name
- Page numbers (if available)
- Relevant snippet
- Click to expand full text

### Sidebar

Left panel containing:
- New conversation button
- Conversation list (sorted by date)
- Archive/unarchive actions
- Settings link
- Logout button

---

## State Management

### Auth Store (`store/auth.ts`)

```typescript
interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
}

interface AuthActions {
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  checkAuth: () => Promise<boolean>;
  setUser: (user: User) => void;
}
```

### Chat Store (`store/chat.ts`)

```typescript
interface ChatState {
  // Conversation data
  conversationId: string | null;
  messages: Message[];
  citations: Citation[];
  conversations: Conversation[];

  // Streaming state
  isStreaming: boolean;
  streamingContent: string;
  toolStatus: string | null;

  // Inline citations
  inlineCitations: InlineCitation[];

  // UI state
  highlightedCitation: number | null;
}

interface ChatActions {
  sendMessage: (message: string) => Promise<void>;
  loadConversation: (id: string) => Promise<void>;
  loadConversations: () => Promise<void>;
  createNewConversation: () => void;
  archiveConversation: (id: string) => Promise<void>;
  submitFeedback: (messageId: string, rating: string) => Promise<void>;
  highlightCitation: (index: number | null) => void;
}
```

### Notification Store (`store/notifications.ts`)

```typescript
interface Notification {
  id: string;
  type: 'success' | 'error' | 'warning' | 'info';
  message: string;
}

interface NotificationActions {
  addNotification: (type: string, message: string) => void;
  removeNotification: (id: string) => void;
}
```

---

## API Client

Located in `lib/api.ts`, provides typed methods for all API calls.

### Authentication

```typescript
authApi.login(email, password)  // POST /api/auth/login
authApi.me()                    // GET /api/auth/me
authApi.users()                 // GET /api/auth/users
authApi.updateRole(userId, role) // PATCH /api/auth/users/{id}/role
```

### Chat

```typescript
chatApi.sendMessage(message, conversationId)  // POST /api/chat (SSE)
chatApi.getSuggestions()                      // GET /api/suggestions
```

### Conversations

```typescript
conversationsApi.list(cursor?)      // GET /api/conversations
conversationsApi.get(id)            // GET /api/conversations/{id}
conversationsApi.update(id, title)  // PATCH /api/conversations/{id}
conversationsApi.archive(id, bool)  // POST /api/conversations/{id}/archive
conversationsApi.submitFeedback(data) // POST /api/conversations/feedback
```

### Admin

```typescript
adminApi.analytics(days)        // GET /api/admin/analytics
adminApi.stats()                // GET /api/admin/stats
adminApi.runs(params)           // GET /api/admin/runs
adminApi.users()                // GET /api/admin/users
adminApi.conversations()        // GET /api/admin/conversations
adminApi.documents()            // GET /api/admin/documents
adminApi.deleteConversation(id) // DELETE /api/admin/conversations/{id}
```

---

## Streaming (SSE)

The chat endpoint returns Server-Sent Events for real-time responses.

### Event Types

| Event | Purpose |
|-------|---------|
| `session_init` | Returns conversation and message IDs |
| `tool_use` | File search started/completed |
| `delta` | Text chunk of response |
| `citations` | Citation data with inline positions |
| `done` | Stream completed |
| `error` | Error occurred |

### Processing Flow

```typescript
const response = await fetch('/api/chat', { method: 'POST', body });
const reader = response.body.getReader();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  // Parse SSE events
  const text = new TextDecoder().decode(value);
  for (const line of text.split('\n')) {
    if (line.startsWith('data: ')) {
      const event = JSON.parse(line.slice(6));
      handleEvent(event);
    }
  }
}
```

---

## Theming

### Dark Mode

Implemented via CSS classes and localStorage persistence.

```typescript
// ThemeProvider manages theme state
const [theme, setTheme] = useState<'light' | 'dark' | 'system'>('system');

// Apply theme class to document
useEffect(() => {
  document.documentElement.classList.toggle('dark', isDark);
}, [isDark]);
```

### Tailwind Configuration

Dark mode is enabled via the `class` strategy:

```javascript
// tailwind.config.js
module.exports = {
  darkMode: 'class',
  // ...
}
```

---

## Internationalization (i18n)

Simple key-based translation system supporting English and Spanish.

```typescript
// lib/i18n.ts
const translations = {
  en: {
    'chat.placeholder': 'Ask a question about loan programs...',
    'chat.send': 'Send',
    // ...
  },
  es: {
    'chat.placeholder': 'Haz una pregunta sobre programas de préstamo...',
    'chat.send': 'Enviar',
    // ...
  }
};

// Usage with hook
const { t } = useTranslation();
<input placeholder={t('chat.placeholder')} />
```

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_API_URL` | Backend API URL |

For local development:
```bash
NEXT_PUBLIC_API_URL=http://localhost:8082
```

For production:
```bash
NEXT_PUBLIC_API_URL=https://champions-backend-561975502517.us-central1.run.app
```
