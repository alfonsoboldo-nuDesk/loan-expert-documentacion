# Arquitectura en Profundidad

Este documento proporciona un recorrido detallado de la arquitectura del sistema Champions Loan Expert.

## Visión General del Sistema

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              Google Cloud Platform                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │                    Cloud Run: Frontend (Next.js 14)                     │   │
│   │                  champions-frontend-561975502517                        │   │
│   │                                                                         │   │
│   │   Navegador ──► Assets Estáticos ──► App React ──► Llamadas API         │   │
│   │                                                                         │   │
│   └───────────────────────────────────┬─────────────────────────────────────┘   │
│                                       │                                         │
│                          REST API + Server-Sent Events                          │
│                                       │                                         │
│   ┌───────────────────────────────────▼─────────────────────────────────────┐   │
│   │                    Cloud Run: Backend (FastAPI)                         │   │
│   │                   champions-backend-561975502517                        │   │
│   │                                                                         │   │
│   │   Router API ──► Servicios ──► Base de Datos ──► APIs Externas          │   │
│   │                                                                         │   │
│   └───────────────────────────────────┬─────────────────────────────────────┘   │
│                                       │                                         │
└───────────────────────────────────────┼─────────────────────────────────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
                    ▼                   ▼                   ▼
            ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
            │  Cloud SQL    │   │ Gemini 2.0    │   │    Secret     │
            │  PostgreSQL   │   │ File Search   │   │    Manager    │
            └───────────────┘   └───────────────┘   └───────────────┘
```

---

## Arquitectura del Frontend

### Stack Tecnológico
- **Framework:** Next.js 14 (App Router)
- **Lenguaje:** TypeScript
- **Estilos:** Tailwind CSS
- **Estado:** Zustand (chat, auth, theme, notifications)
- **Íconos:** Lucide React
- **Markdown:** react-markdown + remark-gfm

### Estructura Completa de Archivos

```
frontend/src/
├── app/                           # Next.js App Router
│   ├── layout.tsx                 # Layout raíz (providers, fuentes)
│   ├── page.tsx                   # Landing → redirige a /chat
│   ├── chat/
│   │   └── page.tsx               # Interfaz principal de chat
│   ├── admin/
│   │   └── page.tsx               # Panel admin (5 pestañas)
│   └── settings/
│       └── page.tsx               # Preferencias de usuario
│
├── components/
│   ├── auth/
│   │   ├── LoginForm.tsx          # Formulario email/contraseña
│   │   └── index.ts
│   │
│   ├── chat/
│   │   ├── ChatLayout.tsx         # Layout principal (sidebar + chat)
│   │   ├── ChatInput.tsx          # Input de mensaje con enviar
│   │   ├── MessageList.tsx        # Mensajes con citas inline
│   │   ├── Sidebar.tsx            # Lista de conversaciones + header
│   │   ├── SourcesPanel.tsx       # Sidebar de citas
│   │   └── index.ts
│   │
│   ├── providers/
│   │   ├── ThemeProvider.tsx      # Contexto de modo oscuro
│   │   └── ChunkErrorHandler.tsx  # Error boundary de Next.js
│   │
│   └── ui/
│       ├── Button.tsx             # Botón reutilizable
│       ├── Input.tsx              # Input reutilizable
│       ├── Toast.tsx              # Notificaciones toast
│       └── index.ts
│
├── hooks/
│   └── useTranslation.ts          # Hook de i18n
│
├── lib/
│   ├── api.ts                     # Cliente API (authApi, chatApi, adminApi)
│   ├── i18n.ts                    # Strings de traducción (es/en)
│   └── utils.ts                   # cn() mezclador de clases
│
├── store/
│   ├── auth.ts                    # Estado auth (usuario, token, login/logout)
│   ├── chat.ts                    # Estado chat (mensajes, streaming, citas)
│   └── notifications.ts           # Notificaciones toast
│
└── types/
    └── index.ts                   # Interfaces TypeScript
```

### Detalle de Componentes de Página

| Página | Ruta | Características |
|------|-------|----------|
| **Chat** | `/chat` | Lista de mensajes, input, panel de citas, sidebar de conversaciones |
| **Admin** | `/admin` | 5 pestañas: Analytics, Logs, Usuarios, Conversaciones, Documentos |
| **Settings** | `/settings` | Toggle de tema, selector de idioma |

### Componentes Principales

| Componente | Archivo | Propósito |
|-----------|------|---------|
| `ChatLayout` | `components/chat/ChatLayout.tsx` | Layout grid: sidebar + main + sources |
| `MessageList` | `components/chat/MessageList.tsx` | Renderiza mensajes con markdown, citas inline, botones de feedback |
| `ChatInput` | `components/chat/ChatInput.tsx` | Textarea con botón enviar, Enter para enviar |
| `Sidebar` | `components/chat/Sidebar.tsx` | Lista de conversaciones, nuevo chat, enlace a settings |
| `SourcesPanel` | `components/chat/SourcesPanel.tsx` | Panel de citas colapsable, números de página |
| `LoginForm` | `components/auth/LoginForm.tsx` | Email + contraseña, manejo de errores |
| `Toast` | `components/ui/Toast.tsx` | Notificaciones success/error/warning |
| `ThemeProvider` | `components/providers/ThemeProvider.tsx` | Contexto de modo oscuro con detección de sistema |

### Gestión de Estado (Stores Zustand)

```typescript
// Auth Store - Estado de autenticación
interface AuthStore {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;

  // Acciones
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  checkAuth: () => Promise<boolean>;
}

// Chat Store - Estado principal para funcionalidad de chat
interface ChatStore {
  // Conversación
  conversationId: string | null;
  messages: Message[];
  citations: Citation[];
  conversations: Conversation[];

  // Streaming
  isStreaming: boolean;
  streamingContent: string;
  toolStatus: string | null;

  // Citas Inline (para marcadores persistentes)
  inlineCitations: InlineCitation[];
  inlineCitationsMessageId: string | null;

  // Estado UI
  highlightedCitation: number | null;
  isLoadingMessages: boolean;

  // Acciones
  sendMessage: (message: string) => Promise<void>;
  loadConversation: (id: string) => Promise<void>;
  loadConversations: () => Promise<void>;
  createNewConversation: () => void;
  archiveConversation: (id: string) => Promise<void>;
  highlightCitation: (index: number) => void;
}

// Notification Store - Notificaciones toast
interface NotificationStore {
  notifications: Notification[];

  // Acciones
  addNotification: (type: 'success' | 'error' | 'warning' | 'info', message: string) => void;
  removeNotification: (id: string) => void;
}
```

### Estructura del Cliente API

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

### Procesamiento de Stream SSE

```typescript
// En chat store - acción sendMessage
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
          set({ toolStatus: event.status === 'start' ? 'Buscando documentos...' : null });
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
          // Finalizar mensaje y agregar al array de mensajes
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

## Arquitectura del Backend

### Stack Tecnológico
- **Framework:** FastAPI (Python 3.11)
- **ORM:** SQLAlchemy (async)
- **Validación:** Pydantic v2
- **IA:** SDK google-genai

### Estructura de Módulos

```
backend/app/
├── main.py              # Entrada de aplicación FastAPI
├── config.py            # Configuración de entorno
│
├── api/                 # Endpoints REST
│   ├── auth.py          # Endpoints /api/auth/*
│   ├── chat.py          # /api/chat, /api/suggestions
│   ├── conversations.py # /api/conversations/*
│   ├── admin.py         # /api/admin/*
│   └── deps.py          # Inyección de dependencias
│
├── models/
│   ├── database.py      # Modelos ORM SQLAlchemy
│   ├── schemas.py       # Request/response Pydantic
│   └── errors.py        # Códigos de error y clasificación
│
├── services/
│   ├── chat_service.py        # Gemini + File Search RAG
│   ├── conversation_service.py # Operaciones CRUD
│   └── auth_service.py        # Manejo de JWT
│
└── db/
    └── connection.py    # Gestión de sesión de BD
```

### Flujo de Request

```
Request de Usuario
     │
     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Capa API (FastAPI Router)                                                   │
│  ─────────────────────────────────────────────────────────────────────────── │
│                                                                              │
│  1. Recibir HTTP Request                                                     │
│  2. Validar con schema Pydantic                                              │
│  3. Verificar autenticación JWT                                              │
│  4. Enrutar al endpoint apropiado                                            │
│                                                                              │
└────────────────────────────────────┬────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Capa de Servicio                                                            │
│  ─────────────────────────────────────────────────────────────────────────── │
│                                                                              │
│  ChatService:                                                                │
│    • Construir contexto de conversación                                      │
│    • Detectar programa de préstamo del mensaje                               │
│    • Llamar a Gemini con File Search                                         │
│    • Extraer citas de grounding_metadata                                     │
│    • Transmitir respuesta vía SSE                                            │
│                                                                              │
│  ConversationService:                                                        │
│    • Operaciones CRUD para conversaciones                                    │
│    • Gestión de mensajes                                                     │
│    • Almacenamiento de citas                                                 │
│                                                                              │
└────────────────────────────────────┬────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Capa de Datos                                                               │
│  ─────────────────────────────────────────────────────────────────────────── │
│                                                                              │
│  PostgreSQL (Cloud SQL):                                                     │
│    • users          - Cuentas de usuario                                     │
│    • conversations  - Sesiones de chat                                       │
│    • messages       - Mensajes individuales                                  │
│    • citations      - Referencias a fuentes                                  │
│    • run_logs       - Datos de analytics                                     │
│    • feedback       - Valoraciones de usuarios                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Esquema de Base de Datos

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           ESQUEMA DE BASE DE DATOS                           │
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

## Manejo de Errores

### Clasificación de Códigos de Error

```python
class ErrorCode(str, Enum):
    # Errores de configuración
    NO_FILE_SEARCH_STORE = "NO_FILE_SEARCH_STORE"
    MODEL_NOT_FOUND = "MODEL_NOT_FOUND"

    # Límite de tasa
    RATE_LIMIT_EXCEEDED = "RATE_LIMIT_EXCEEDED"
    QUOTA_EXCEEDED = "QUOTA_EXCEEDED"

    # Errores de servicio
    SERVICE_UNAVAILABLE = "SERVICE_UNAVAILABLE"
    CONNECTION_ERROR = "CONNECTION_ERROR"
    TIMEOUT = "TIMEOUT"

    # Errores de respuesta
    NO_RESPONSE = "NO_RESPONSE"
    NO_CITATIONS = "NO_CITATIONS"
    INVALID_RESPONSE = "INVALID_RESPONSE"
    CONTENT_BLOCKED = "CONTENT_BLOCKED"

    # Genérico
    CHAT_FAILED = "CHAT_FAILED"
```

Todos los errores son:
1. Clasificados por `classify_gemini_error()`
2. Registrados con código específico
3. Rastreados en tabla `run_logs`
4. Retornados al cliente con mensaje amigable
