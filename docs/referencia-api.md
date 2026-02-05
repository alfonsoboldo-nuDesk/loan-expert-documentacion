# Referencia de API

Referencia completa para la API REST de Champions Loan Expert.

## URL Base

```
Producción: https://champions-backend-561975502517.us-central1.run.app/api
Local:      http://localhost:8080/api
```

## Autenticación

Todos los endpoints (excepto `/auth/login`) requieren autenticación JWT.

```http
Authorization: Bearer <jwt_token>
```

---

## Endpoints de Autenticación

### POST `/auth/login`

Autenticar usuario y recibir token JWT.

**Solicitud:**
```json
{
  "email": "usuario@championsfunding.com",
  "password": "contraseña_compartida"
}
```

**Respuesta (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "user": {
    "id": "uuid",
    "email": "usuario@championsfunding.com",
    "role": "rep"
  }
}
```

**Errores:**
| Código | Descripción |
|--------|-------------|
| 401 | Credenciales inválidas |
| 400 | Dominio de email inválido |

---

### GET `/auth/me`

Obtener usuario autenticado actual.

**Respuesta (200):**
```json
{
  "id": "uuid",
  "email": "usuario@championsfunding.com",
  "role": "rep"
}
```

---

### GET `/auth/users`

Listar todos los usuarios. Requiere rol `manager` o `admin`.

**Respuesta (200):**
```json
{
  "items": [
    {
      "id": "uuid",
      "email": "usuario@championsfunding.com",
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

Actualizar rol de usuario. **Solo Admin.**

**Solicitud:**
```json
{
  "role": "manager"
}
```

**Respuesta (200):**
```json
{
  "id": "uuid",
  "email": "usuario@championsfunding.com",
  "role": "manager"
}
```

---

## Endpoints de Chat

### POST `/chat`

Enviar mensaje y recibir respuesta en streaming.

**Solicitud:**
```json
{
  "message": "¿Cuáles son los requisitos de DSCR?",
  "conversation_id": "uuid-o-null",
  "stream": true,
  "metadata_filter": null
}
```

**Respuesta (Stream SSE):**
```
data: {"type":"session_init","conversation_id":"uuid","message_id":"uuid"}

data: {"type":"tool_use","tool":"file_search","status":"start"}

data: {"type":"delta","content":"Los requisitos de DSCR "}

data: {"type":"delta","content":"son..."}

data: {"type":"tool_use","tool":"file_search","status":"complete"}

data: {"type":"citations","citations":[...],"inline_citations":[...]}

data: {"type":"done","message_id":"uuid","tokens_used":1234}
```

**Tipos de Eventos SSE:**

| Evento | Campos | Descripción |
|--------|--------|-------------|
| `session_init` | `conversation_id`, `message_id` | Nueva sesión iniciada |
| `tool_use` | `tool`, `status` | Inicio/fin de búsqueda de archivos |
| `delta` | `content` | Fragmento de texto |
| `citations` | `citations`, `inline_citations` | Referencias de fuentes |
| `done` | `message_id`, `tokens_used` | Stream completado |
| `error` | `code`, `error` | Error ocurrido |

---

### GET `/suggestions`

Obtener sugerencias dinámicas de chat.

**Respuesta (200):**
```json
{
  "suggestions": [
    {
      "text": "¿Cuáles son los requisitos mínimos de DSCR?",
      "source": "feedback"
    },
    {
      "text": "Comparar programas Full Doc vs Alt Doc",
      "source": "frequent"
    },
    {
      "text": "¿Cuáles son los requisitos de reservas?",
      "source": "default"
    }
  ]
}
```

**Fuentes de Sugerencias:**
| Fuente | Descripción |
|--------|-------------|
| `feedback` | Preguntas con feedback positivo (thumbs_up) |
| `frequent` | Consultas exitosas frecuentes |
| `default` | Generadas de documentos indexados |

---

## Endpoints de Conversaciones

### GET `/conversations`

Listar conversaciones del usuario.

**Parámetros de Consulta:**
| Param | Tipo | Default | Descripción |
|-------|------|---------|-------------|
| `limit` | int | 20 | Máx resultados |
| `cursor` | string | null | Cursor de paginación |
| `include_archived` | bool | false | Incluir archivadas |

**Respuesta (200):**
```json
{
  "items": [
    {
      "id": "uuid",
      "title": "Requisitos DSCR",
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

Obtener conversación con mensajes y citas.

**Respuesta (200):**
```json
{
  "conversation": {
    "id": "uuid",
    "title": "Requisitos DSCR",
    "archived": false,
    "created_at": "2025-01-27T10:00:00Z",
    "updated_at": "2025-01-27T10:30:00Z"
  },
  "messages": [
    {
      "id": "uuid",
      "role": "user",
      "content": "¿Cuáles son los requisitos de DSCR?",
      "created_at": "2025-01-27T10:00:00Z",
      "inline_citations": null
    },
    {
      "id": "uuid",
      "role": "assistant",
      "content": "Los requisitos de DSCR son...",
      "model": "gemini-2.0-flash",
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
      "snippet": "DSCR mínimo: 1.0...",
      "confidence": null,
      "metadata": null
    }
  ]
}
```

---

### PATCH `/conversations/{id}`

Actualizar título de conversación.

**Solicitud:**
```json
{
  "title": "Nuevo Título"
}
```

**Respuesta (200):**
```json
{
  "id": "uuid",
  "title": "Nuevo Título",
  "archived": false
}
```

---

### POST `/conversations/{id}/archive`

Archivar o restaurar conversación.

**Solicitud:**
```json
{
  "archived": true
}
```

**Respuesta (200):**
```json
{
  "id": "uuid",
  "archived": true,
  "archived_at": "2025-01-27T10:00:00Z"
}
```

---

### POST `/conversations/feedback`

Enviar feedback sobre un mensaje.

**Solicitud:**
```json
{
  "message_id": "uuid",
  "rating": "thumbs_up",
  "comment": "¡Muy útil!"
}
```

**Respuesta (200):**
```json
{
  "id": "uuid",
  "rating": "thumbs_up",
  "comment": "¡Muy útil!",
  "created_at": "2025-01-27T10:00:00Z"
}
```

---

## Endpoints de Admin

Todos los endpoints de admin requieren rol `admin` o `manager`.

### GET `/admin/analytics`

Obtener analytics con datos de series temporales.

**Parámetros de Consulta:**
| Param | Tipo | Default | Descripción |
|-------|------|---------|-------------|
| `days` | int | 30 | Período en días |

**Respuesta (200):**
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

Obtener logs de ejecución con paginación.

**Parámetros de Consulta:**
| Param | Tipo | Default | Descripción |
|-------|------|---------|-------------|
| `limit` | int | 50 | Máx resultados |
| `offset` | int | 0 | Saltar resultados |
| `status` | string | null | Filtrar por estado |
| `user_id` | string | null | Filtrar por usuario |

**Respuesta (200):**
```json
{
  "items": [
    {
      "id": "uuid",
      "user_id": "uuid",
      "user_email": "usuario@championsfunding.com",
      "conversation_id": "uuid",
      "message_id": "uuid",
      "input_preview": "¿Cuáles son los...",
      "input_full": "¿Cuáles son los requisitos de DSCR?",
      "model": "gemini-2.0-flash",
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

### GET `/admin/users`

Listar todos los usuarios con detalles.

**Respuesta (200):**
```json
{
  "items": [
    {
      "id": "uuid",
      "email": "usuario@championsfunding.com",
      "role": "rep",
      "status": "active",
      "last_login": "2025-01-27T10:00:00Z",
      "created_at": "2025-01-01T00:00:00Z"
    }
  ]
}
```

---

### PATCH `/admin/users/{user_id}/status`

Actualizar estado de usuario.

**Solicitud:**
```json
{
  "status": "suspended"
}
```

**Respuesta (200):**
```json
{
  "id": "uuid",
  "status": "suspended"
}
```

---

### GET `/admin/conversations`

Listar todas las conversaciones (de todos los usuarios).

**Respuesta (200):**
```json
{
  "items": [
    {
      "id": "uuid",
      "user_email": "usuario@championsfunding.com",
      "title": "Requisitos DSCR",
      "message_count": 10,
      "created_at": "2025-01-27T10:00:00Z",
      "updated_at": "2025-01-27T10:30:00Z"
    }
  ]
}
```

---

### DELETE `/admin/conversations/{id}`

Eliminar una conversación. **Solo Admin.**

**Respuesta (200):**
```json
{
  "deleted": true
}
```

---

### GET `/admin/documents`

Listar documentos indexados en File Search Store.

**Respuesta (200):**
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

## Respuestas de Error

Todos los errores siguen este formato:

```json
{
  "detail": {
    "error": "Mensaje legible",
    "code": "CODIGO_ERROR"
  }
}
```

### Códigos de Error

| Código | HTTP | Descripción |
|--------|------|-------------|
| `NOT_FOUND` | 404 | Recurso no encontrado |
| `UNAUTHORIZED` | 401 | Token inválido o faltante |
| `FORBIDDEN` | 403 | Permisos insuficientes |
| `CONVERSATION_ARCHIVED` | 400 | No se puede modificar conversación archivada |
| `MESSAGE_LIMIT` | 400 | Conversación tiene 200 mensajes |
| `CONVERSATION_LIMIT` | 400 | Usuario tiene 100 conversaciones activas |
| `RATE_LIMIT_EXCEEDED` | 429 | Demasiadas solicitudes |
| `NO_FILE_SEARCH_STORE` | 500 | File Search no configurado |
| `CHAT_FAILED` | 500 | Respuesta de IA falló |
