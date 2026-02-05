# Roadmap - Champions Loan Expert

This document tracks planned features, improvements, and technical debt.

---

## Next Up

### Priority 1: Critical

#### Persistent Inline Citations (COMPLETED)
**Status:** Implemented in v2.5.1

Citation markers [¹] now persist after page reload.

**Changes made:**
- Added `text_start_index`, `text_end_index`, `inline_number` columns to citations table
- Modified `conversation_service.py` to save inline citation data
- Modified `chat_service.py` to include inline_citations in response
- Modified `conversations.py` to return inline_citations per message
- Updated frontend to use `message.inline_citations`

---

### Priority 2: High

#### User Invitation System
**Status:** Planned

Allow admins to invite new users by email directly from the admin panel.

**Proposed Implementation:**

```
┌──────────────────────────────────────────┐
│  Admin Panel - Users Tab                  │
│  ┌────────────────────────────────────┐  │
│  │ [+ Invite User]          [Refresh] │  │
│  └────────────────────────────────────┘  │
│                                          │
│  Click [+ Invite User]:                  │
│  ┌────────────────────────────────────┐  │
│  │  Email: [________________________] │  │
│  │  Role:  [Rep ▼]                    │  │
│  │                                    │  │
│  │     [Cancel]    [Send Invitation]  │  │
│  └────────────────────────────────────┘  │
│                                          │
│  Result:                                 │
│  ┌────────────────────────────────────┐  │
│  │  ✓ User Created                    │  │
│  │  Email: new@championsfunding.com   │  │
│  │  Password: Abc123XyZ!@#            │  │
│  │                                    │  │
│  │  [Copy Credentials]       [Close]  │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

**Files to modify:**

| File | Change |
|------|--------|
| `backend/app/api/admin.py` | Add endpoint `POST /admin/users/invite` |
| `backend/app/models/schemas.py` | Add `UserInvite`, `UserInviteResponse` |
| `frontend/src/lib/api.ts` | Add `adminApi.inviteUser()` |
| `frontend/src/app/admin/page.tsx` | Add invitation modal UI |

---

### Priority 3: Medium

#### Password Reset Flow
**Status:** Planned

Allow users to reset their password.

**Options:**
1. **Admin-initiated:** Admin generates new password for user
2. **Self-service (future):** Email-based password reset

---

## Technical Debt

### Database

#### Consider Alembic for Migrations
**Priority:** Medium

Currently using manual SQL scripts. Consider adding Alembic for:
- Version tracking
- Automatic rollback
- Multiple developers

**Current approach:**
```bash
python scripts/run_migration.py
```

**Alembic approach:**
```bash
alembic revision --autogenerate -m "description"
alembic upgrade head
```

---

### Backend

#### Add Unit Tests
**Priority:** High

Current test coverage: Low

**Areas needing tests:**
- `chat_service.py` - Citation extraction
- `conversation_service.py` - CRUD operations
- `auth_service.py` - JWT validation

---

#### Add Request Validation
**Priority:** Medium

Some endpoints lack proper input validation:
- String length limits
- Email format validation
- UUID format validation

---

### Frontend

#### Add E2E Tests
**Priority:** Medium

Consider Playwright or Cypress for:
- Login flow
- Chat flow
- Admin panel

---

## Version History

| Version | Date | Highlights |
|---------|------|------------|
| 2.5.1 | 2025-02 | Persistent inline citations |
| 2.5.0 | 2025-01 | Error handling, error codes |
| 2.4.0 | 2025-01 | Security hardening |
| 2.3.0 | 2025-01 | Dynamic suggestions |
| 2.2.0 | 2025-01 | Admin improvements |
| 2.1.0 | 2025-01 | Dark mode, i18n |
| 2.0.0 | 2025-01 | Initial release |

---

## Success Criteria

### Short term (Q1 2025)
- [x] Inline citations persist correctly
- [ ] User invitation flow working
- [ ] 95% uptime

### Medium term (Q2 2025)
- [ ] Test coverage > 60%
- [ ] Mobile-friendly UI
- [ ] Usage analytics dashboard

### Long term (2025)
- [ ] Multi-tenant support
- [ ] Custom document upload
- [ ] API for external integrations
