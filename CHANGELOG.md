# Registro de Cambios

Todos los cambios notables de Champions Loan Expert v2.

## [2.5.1] - 2025-02

### Corregido - Citas Inline Persistentes
- **Nuevo Sistema de Persistencia**: Las citas inline [¹] ahora persisten después de recargar la página
- **Migración de Base de Datos**: Agregadas columnas `text_start_index`, `text_end_index`, `inline_number` a tabla citations
- **Backend Actualizado**: Modificados conversation_service.py, chat_service.py, conversations.py
- **Frontend Actualizado**: MessageList ahora usa `message.inline_citations` de la base de datos

### Archivos Cambiados
- `backend/app/models/database.py` - Nuevas columnas en modelo Citation
- `backend/app/services/conversation_service.py` - Guardar datos de citas inline
- `backend/app/services/chat_service.py` - Incluir inline_citations en respuesta
- `backend/app/api/conversations.py` - Retornar inline_citations por mensaje
- `backend/app/models/schemas.py` - Campo inline_citations en MessageResponse
- `frontend/src/types/index.ts` - inline_citations en interfaz Message
- `frontend/src/components/chat/MessageList.tsx` - Usar citas inline desde DB

---

## [2.5.0] - 2025-01-27

### Agregado - Manejo Completo de Errores
- **Nuevo Módulo de Códigos de Error**: `backend/app/models/errors.py` con códigos estandarizados
- **Clasificación de Errores**: Clasificación automática de errores de API de Gemini
- **Seguimiento de Errores en Run Log**: Todas las ejecuciones ahora rastrean códigos de error específicos

### Códigos de Error Implementados
| Código | Descripción |
|--------|-------------|
| `NO_FILE_SEARCH_STORE` | File Search Store no configurado |
| `MODEL_NOT_FOUND` | Modelo de IA no disponible |
| `RATE_LIMIT_EXCEEDED` | 429 - Demasiadas solicitudes |
| `QUOTA_EXCEEDED` | Límite de facturación/cuota alcanzado |
| `SERVICE_UNAVAILABLE` | 503 - Servicio temporalmente caído |
| `CONNECTION_ERROR` | Conexión abortada/reseteada |
| `TIMEOUT` | Solicitud agotó tiempo |
| `NO_RESPONSE` | Sin respuesta del modelo |
| `NO_CITATIONS` | Respuesta sin citas de documentos |
| `INVALID_RESPONSE` | Formato de respuesta inesperado |
| `CONTENT_BLOCKED` | Filtro de seguridad bloqueó respuesta |
| `CHAT_FAILED` | Error genérico no clasificado |

### Archivos Cambiados
- `backend/app/models/errors.py` - Nuevo archivo con enum ErrorCode y classify_gemini_error()
- `backend/app/services/chat_service.py` - Clasificación de errores integrada
- `backend/app/api/chat.py` - Logging de ejecución actualizado con códigos de error

---

## [2.4.0] - 2025-01-27

### Correcciones de Seguridad
- **Endpoints de Debug Eliminados**: Eliminados `/bootstrap/admin` y `/debug/users` que exponían datos sensibles sin autenticación
- **Cambios de Rol Solo Admin**: Endpoint `/users/{user_id}/role` ahora requiere rol admin (antes permitía cualquier usuario autenticado)

### Archivos Cambiados
- `backend/app/api/auth.py` - Endpoints debug eliminados, verificación admin agregada a actualización de rol

---

## [2.3.0] - 2025-01-27

### Agregado - Sugerencias Dinámicas
- **Backend**: Nuevo endpoint `/api/suggestions` que retorna sugerencias dinámicas de chat
  - Prioridad 1: Preguntas con feedback positivo (thumbs_up)
  - Prioridad 2: Consultas exitosas frecuentes de run logs
  - Prioridad 3: Sugerencias basadas en documentos generadas de programas de préstamos indexados
- **Frontend**: Componente EmptyState ahora obtiene sugerencias dinámicas
  - Caché SessionStorage con TTL de 5 minutos
  - Fallback automático a sugerencias estáticas i18n cuando no hay datos
- Sugerencias basadas en programas indexados: DSCR, Full Doc, Alt Doc, Ally, Super Jumbo, Foreign National, ITIN

### Archivos Cambiados
- `backend/app/api/chat.py` - Endpoint `/api/suggestions` y función `_generate_program_suggestions()`
- `backend/app/models/schemas.py` - Schemas `SuggestionItem` y `SuggestionsResponse`
- `frontend/src/components/chat/MessageList.tsx` - EmptyState con sugerencias dinámicas
- `frontend/src/lib/api.ts` - Método `chatApi.getSuggestions()`
- `frontend/src/types/index.ts` - Tipos de sugerencias

---

## [2.2.0] - 2025-01-27

### Agregado - Mejoras en Panel Admin
- **Ver Conversaciones Completas**: Botón de ojo para ver conversación completa en modal
  - Muestra todos los mensajes con timestamps
  - Etiquetas User/Assistant con código de colores
  - Lista de mensajes desplazable
- **Mensajes de Log Expandibles**: Clic para expandir contenido completo del mensaje en pestaña Logs
  - Vista previa muestra primeros 100 caracteres
  - Clic para revelar mensaje completo
  - Campo `input_full` agregado a respuesta de API

### Archivos Cambiados
- `backend/app/api/admin.py` - `input_full` agregado a respuesta RunLog
- `backend/app/models/schemas.py` - Campo `input_full` en `RunLogEntry`
- `frontend/src/app/admin/page.tsx` - `ConversationModal`, inputs expandibles, handler ver conversación
- `frontend/src/types/index.ts` - `input_full` en interfaz `RunLogEntry`

---

## [2.1.0] - 2025-01-26

### Agregado - Modo Oscuro
- Detección de preferencia del sistema con toggle manual
- Preferencia persistente en localStorage
- Tema oscuro completo para todos los componentes
- Página de configuración para control de tema

### Agregado - Internacionalización (i18n)
- Soporte para Español (es) e Inglés (en)
- Selector de idioma en configuración
- Todo el texto de UI traducible
- Preferencia de idioma persistente

### Agregado - Sistema de Notificaciones
- Notificaciones toast (success, error, warning, info)
- Auto-cierre con duración configurable
- Store de notificaciones basado en Zustand
- Usado en toda la app para feedback al usuario

### Agregado - Exportar Conversación
- Botón de descarga en header del chat
- Exporta como archivo Markdown (.md)
- Incluye todos los mensajes con timestamps
- Citas incluidas cuando están disponibles

### Archivos Cambiados
- `frontend/src/store/theme.ts` - Nuevo store de tema
- `frontend/src/store/language.ts` - Nuevo store de idioma
- `frontend/src/store/notifications.ts` - Nuevo store de notificaciones
- `frontend/src/hooks/useTranslation.ts` - Hook de traducción
- `frontend/src/lib/translations.ts` - Strings de traducción
- `frontend/src/app/settings/page.tsx` - Página de configuración con tema/idioma
- `frontend/src/components/chat/ChatHeader.tsx` - Botón de exportación
- `frontend/src/components/ui/Notifications.tsx` - Componente Toast
- `frontend/src/app/layout.tsx` - Integración de proveedor de tema

---

## [2.0.0] - 2025-01-25

### Lanzamiento Inicial
- Chat potenciado por RAG con Gemini 2.0 Flash File Search
- Gestión de conversaciones (crear, archivar, renombrar)
- Respuestas en streaming SSE en tiempo real
- Extracción y visualización de citas
- Sistema de feedback (pulgar arriba/abajo)
- Acceso basado en roles (rep, manager, admin)
- Dashboard de admin con analytics
- Autenticación JWT con expiración de 30 días

---

## Resumen de Endpoints de API

| Método | Endpoint | Descripción | Auth |
|--------|----------|-------------|------|
| POST | `/api/auth/login` | Login de usuario | No |
| POST | `/api/chat` | Enviar mensaje (stream SSE) | Sí |
| GET | `/api/suggestions` | Obtener sugerencias dinámicas | Sí |
| GET | `/api/conversations` | Listar conversaciones del usuario | Sí |
| GET | `/api/conversations/:id` | Obtener detalle de conversación | Sí |
| PATCH | `/api/conversations/:id` | Actualizar título | Sí |
| POST | `/api/conversations/:id/archive` | Archivar/restaurar | Sí |
| POST | `/api/conversations/feedback` | Enviar feedback | Sí |
| GET | `/api/admin/analytics` | Datos de analytics | Admin |
| GET | `/api/admin/stats` | Estadísticas del sistema | Admin |
| GET | `/api/admin/users` | Listar usuarios | Admin |
| PATCH | `/api/admin/users/:id/role` | Actualizar rol | Admin |
| PATCH | `/api/admin/users/:id/status` | Actualizar estado | Admin |
| GET | `/api/admin/runs` | Logs de ejecución | Admin |
| GET | `/api/admin/conversations` | Todas las conversaciones | Admin |
| DELETE | `/api/admin/conversations/:id` | Eliminar conversación | Admin |
| GET | `/api/admin/documents` | Listar documentos | Admin |
| DELETE | `/api/admin/reset-data` | Resetear todos los datos | Admin |

---

## URLs de Despliegue

- **Frontend**: https://champions-frontend-561975502517.us-central1.run.app
- **Backend**: https://champions-backend-561975502517.us-central1.run.app
- **Docs de API**: https://champions-backend-561975502517.us-central1.run.app/docs
