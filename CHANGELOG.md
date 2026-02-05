# Changelog

All notable changes to Champions Loan Expert v2.

## [2.5.1] - 2025-02

### Fixed - Persistent Inline Citations
- **New Persistence System**: Inline citations [¹] now persist after page reload
- **Database Migration**: Added `text_start_index`, `text_end_index`, `inline_number` columns to citations table
- **Backend Updated**: Modified conversation_service.py, chat_service.py, conversations.py
- **Frontend Updated**: MessageList now uses `message.inline_citations` from database

### Files Changed
- `backend/app/models/database.py` - New columns in Citation model
- `backend/app/services/conversation_service.py` - Save inline citation data
- `backend/app/services/chat_service.py` - Include inline_citations in response
- `backend/app/api/conversations.py` - Return inline_citations per message
- `backend/app/models/schemas.py` - inline_citations field in MessageResponse
- `frontend/src/types/index.ts` - inline_citations in Message interface
- `frontend/src/components/chat/MessageList.tsx` - Use inline citations from DB

---

## [2.5.0] - 2025-01-27

### Added - Complete Error Handling
- **New Error Codes Module**: `backend/app/models/errors.py` with standardized codes
- **Error Classification**: Automatic classification of Gemini API errors
- **Run Log Error Tracking**: All runs now track specific error codes

### Error Codes Implemented
| Code | Description |
|------|-------------|
| `NO_FILE_SEARCH_STORE` | File Search Store not configured |
| `MODEL_NOT_FOUND` | AI model not available |
| `RATE_LIMIT_EXCEEDED` | 429 - Too many requests |
| `QUOTA_EXCEEDED` | Billing/quota limit reached |
| `SERVICE_UNAVAILABLE` | 503 - Service temporarily down |
| `CONNECTION_ERROR` | Connection aborted/reset |
| `TIMEOUT` | Request timed out |
| `NO_RESPONSE` | No response from model |
| `NO_CITATIONS` | Response without document citations |
| `INVALID_RESPONSE` | Unexpected response format |
| `CONTENT_BLOCKED` | Safety filter blocked response |
| `CHAT_FAILED` | Generic unclassified error |

### Files Changed
- `backend/app/models/errors.py` - New file with ErrorCode enum and classify_gemini_error()
- `backend/app/services/chat_service.py` - Error classification integrated
- `backend/app/api/chat.py` - Run logging updated with error codes

---

## [2.4.0] - 2025-01-27

### Security Fixes
- **Debug Endpoints Removed**: Removed `/bootstrap/admin` and `/debug/users` that exposed sensitive data without authentication
- **Admin-Only Role Changes**: `/users/{user_id}/role` endpoint now requires admin role (previously allowed any authenticated user)

### Files Changed
- `backend/app/api/auth.py` - Debug endpoints removed, admin check added to role update

---

## [2.3.0] - 2025-01-27

### Added - Dynamic Suggestions
- **Backend**: New `/api/suggestions` endpoint that returns dynamic chat suggestions
  - Priority 1: Questions with positive feedback (thumbs_up)
  - Priority 2: Frequent successful queries from run logs
  - Priority 3: Document-based suggestions generated from indexed loan programs
- **Frontend**: EmptyState component now fetches dynamic suggestions
  - SessionStorage cache with 5-minute TTL
  - Automatic fallback to static i18n suggestions when no data
- Program-based suggestions for: DSCR, Full Doc, Alt Doc, Ally, Super Jumbo, Foreign National, ITIN

### Files Changed
- `backend/app/api/chat.py` - `/api/suggestions` endpoint and `_generate_program_suggestions()`
- `backend/app/models/schemas.py` - `SuggestionItem` and `SuggestionsResponse` schemas
- `frontend/src/components/chat/MessageList.tsx` - EmptyState with dynamic suggestions
- `frontend/src/lib/api.ts` - `chatApi.getSuggestions()` method
- `frontend/src/types/index.ts` - Suggestion types

---

## [2.2.0] - 2025-01-27

### Added - Admin Panel Improvements
- **View Full Conversations**: Eye button to view complete conversation in modal
  - Shows all messages with timestamps
  - Color-coded User/Assistant labels
  - Scrollable message list
- **Expandable Log Messages**: Click to expand full message content in Logs tab
  - Preview shows first 100 characters
  - Click to reveal full message
  - `input_full` field added to API response

### Files Changed
- `backend/app/api/admin.py` - `input_full` added to RunLog response
- `backend/app/models/schemas.py` - `input_full` field in `RunLogEntry`
- `frontend/src/app/admin/page.tsx` - `ConversationModal`, expandable inputs, view conversation handler
- `frontend/src/types/index.ts` - `input_full` in `RunLogEntry` interface

---

## [2.1.0] - 2025-01-26

### Added - Dark Mode
- System preference detection with manual toggle
- Persistent preference in localStorage
- Complete dark theme for all components
- Settings page for theme control

### Added - Internationalization (i18n)
- Support for Spanish (es) and English (en)
- Language selector in settings
- All UI text translatable
- Persistent language preference

### Added - Notification System
- Toast notifications (success, error, warning, info)
- Auto-dismiss with configurable duration
- Zustand-based notification store
- Used throughout app for user feedback

### Added - Export Conversation
- Download button in chat header
- Exports as Markdown file (.md)
- Includes all messages with timestamps
- Citations included when available

### Files Changed
- `frontend/src/store/theme.ts` - New theme store
- `frontend/src/store/language.ts` - New language store
- `frontend/src/store/notifications.ts` - New notifications store
- `frontend/src/hooks/useTranslation.ts` - Translation hook
- `frontend/src/lib/translations.ts` - Translation strings
- `frontend/src/app/settings/page.tsx` - Settings page with theme/language
- `frontend/src/components/chat/ChatHeader.tsx` - Export button
- `frontend/src/components/ui/Notifications.tsx` - Toast component
- `frontend/src/app/layout.tsx` - Theme provider integration

---

## [2.0.0] - 2025-01-25

### Initial Release
- RAG-powered chat with Gemini 3 Flash File Search
- Conversation management (create, archive, rename)
- Real-time SSE streaming responses
- Citation extraction and display
- Feedback system (thumbs up/down)
- Role-based access (rep, manager, admin)
- Admin dashboard with analytics
- JWT authentication with 30-day expiry

---

## API Endpoints Summary

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/api/auth/login` | User login | No |
| POST | `/api/chat` | Send message (SSE stream) | Yes |
| GET | `/api/suggestions` | Get dynamic suggestions | Yes |
| GET | `/api/conversations` | List user's conversations | Yes |
| GET | `/api/conversations/:id` | Get conversation detail | Yes |
| PATCH | `/api/conversations/:id` | Update title | Yes |
| POST | `/api/conversations/:id/archive` | Archive/restore | Yes |
| POST | `/api/conversations/feedback` | Submit feedback | Yes |
| GET | `/api/admin/analytics` | Analytics data | Admin |
| GET | `/api/admin/stats` | System stats | Admin |
| GET | `/api/admin/users` | List users | Admin |
| PATCH | `/api/admin/users/:id/role` | Update role | Admin |
| PATCH | `/api/admin/users/:id/status` | Update status | Admin |
| GET | `/api/admin/runs` | Run logs | Admin |
| GET | `/api/admin/conversations` | All conversations | Admin |
| DELETE | `/api/admin/conversations/:id` | Delete conversation | Admin |
| GET | `/api/admin/documents` | List documents | Admin |
| DELETE | `/api/admin/reset-data` | Reset all data | Admin |

---

## Deployment URLs

- **Frontend**: https://champions-frontend-561975502517.us-central1.run.app
- **Backend**: https://champions-backend-561975502517.us-central1.run.app
- **API Docs**: https://champions-backend-561975502517.us-central1.run.app/docs
