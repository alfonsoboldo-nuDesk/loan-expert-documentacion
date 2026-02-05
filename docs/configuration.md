# Configuration Reference

All environment variables and configuration values for Champions Loan Expert.

## Environment Variables

### Backend (Required)

| Variable | Description | Example |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@host:5432/db` |
| `JWT_SECRET` | JWT signing key (min 32 chars) | `your-super-secret-key-here-32chars` |
| `SHARED_PASSWORD` | Team shared password | `secure_password` |
| `GOOGLE_API_KEY` | Gemini API key | `AIza...` |
| `GEMINI_FILE_SEARCH_STORE` | File Search Store ID | `stores/abc123...` |

### Backend (Optional)

| Variable | Description | Default |
|----------|-------------|---------|
| `GEMINI_MODEL` | Model name | `gemini-3-flash` |
| `LOG_LEVEL` | Logging level | `INFO` |
| `CORS_ORIGINS` | Allowed origins (comma-separated) | `*` |

### Frontend

| Variable | Description | Example |
|----------|-------------|---------|
| `NEXT_PUBLIC_API_URL` | Backend API URL | `https://champions-backend-xxx.run.app` |

---

## .env.example

```bash
# Backend Configuration
DATABASE_URL=postgresql+asyncpg://postgres:password@localhost:5432/loan_expert
JWT_SECRET=your-jwt-secret-key-at-least-32-characters
SHARED_PASSWORD=champions2024

# Gemini Configuration
GOOGLE_API_KEY=AIzaSy...
GEMINI_MODEL=gemini-3-flash
GEMINI_FILE_SEARCH_STORE=stores/abc123def456

# Frontend (for production)
NEXT_PUBLIC_API_URL=https://champions-backend-561975502517.us-central1.run.app
```

---

## Google Cloud Resources

### Cloud Run Services

| Service | URL | Region |
|---------|-----|--------|
| Frontend | `champions-frontend-561975502517.us-central1.run.app` | us-central1 |
| Backend | `champions-backend-561975502517.us-central1.run.app` | us-central1 |

### Cloud SQL

| Property | Value |
|----------|-------|
| Instance | PostgreSQL 14 |
| Region | us-central1 |
| Connection | Via Cloud SQL Proxy or private IP |

### Secret Manager

| Secret | Purpose |
|--------|---------|
| `jwt-secret` | JWT signing key |
| `google-api-key` | Gemini API key |
| `db-password` | Database password |
| `shared-password` | User authentication |

### Gemini File Search Store

| Property | Value |
|----------|-------|
| Store ID | `stores/...` |
| Model | `gemini-3-flash` |
| Documents | 16 indexed loan documents |

---

## File Search Name Map

The `file_search_name_map.json` maps Gemini file IDs to readable names.

**Location:** `backend/file_search_name_map.json`

```json
{
  "my6heaulw7lm": "Accelerator-Activator_Alt_Doc_Matrix.md",
  "6m7zk1jwfroi": "Accelerator-Activator_Full_Doc_Matrix.md",
  "y5t14gqb2gqf": "Accelerator_DSCR_1-4_Units_Matrix.md",
  "z0p5iyqywsqc": "Accelerator_DSCR_5-8_Units_Matrix.md",
  "m5cvwcrzgk4s": "Ally_Consumer_No_Ratio_Matrix.md",
  "82njhpyj0iq6": "CF_Underwriting-Guidelines.md",
  "qmhjgpakpzqh": "CF_Underwriting-Guidelines_Ally.md",
  "5r6xsv94vmkp": "CF_Underwriting-Guidelines_Super_Jumbo.md",
  "fh10d6v0nlhi": "Champions-Funding_Non-QM-Wholesale_Product-Catalog.md",
  "b2ykwzwzlv3r": "Foreign-National-Ambassador_DSCR_Matrix.md",
  "0o4wqv6dlx3f": "Foreign-National-Ambassador_Full-Alt-Doc_Matrix.md",
  "0hkwdxz6a4nw": "ITIN_Accelerator_DSCR_Matrix.md",
  "mvrqmqudz0qt": "ITIN_Accelerator_Full-Alt_Matrix.md",
  "mw2rdrtfnlzl": "State-Licensing-Survey.md",
  "gbrnm6c0xv4f": "Super-Jumbo_Matrix.md",
  "s53hx36r2ey8": "super-jumbo-guidelines-extracted.md"
}
```

---

## Program Detection Patterns

The system auto-detects loan programs from user messages.

```python
PROGRAM_PATTERNS = {
    "full_doc": {
        "patterns": [r"\bfull\s*doc\b", r"\bfull\s+documentation\b"],
        "name": "Full Doc Accelerator",
        "matrix": "Accelerator-Activator Full Doc Matrix",
        "asset_formula": "60 months",
    },
    "alt_doc": {
        "patterns": [r"\balt\s*doc\b", r"\balternative\s+doc\b"],
        "name": "Alt Doc Activator",
        "matrix": "Accelerator-Activator Alt Doc Matrix",
        "asset_formula": "60 months",
    },
    "dscr": {
        "patterns": [r"\bdscr\b", r"\bdebt\s+service\s+coverage\b"],
        "name": "DSCR",
        "matrix": "Accelerator DSCR Matrix",
    },
    "super_jumbo": {
        "patterns": [r"\bsuper\s*jumbo\b", r"\bjumbo\b"],
        "name": "Super Jumbo",
        "matrix": "Super Jumbo Matrix",
        "asset_formula": "84 months",
    },
    "ally": {
        "patterns": [r"\bally\b", r"\bno\s+ratio\b"],
        "name": "Ally No Ratio",
        "matrix": "Ally Consumer No Ratio Matrix",
    },
    "foreign_national": {
        "patterns": [r"\bforeign\s+national\b", r"\bambassador\b"],
        "name": "Foreign National Ambassador",
        "matrix": "Foreign National Ambassador Matrix",
    },
    "itin": {
        "patterns": [r"\bitin\b"],
        "name": "ITIN",
        "matrix": "ITIN Matrix",
    }
}
```

---

## Indexed Documents

| Document | Program | Type |
|----------|---------|------|
| `Accelerator-Activator_Alt_Doc_Matrix.md` | Alt Doc | Matrix |
| `Accelerator-Activator_Full_Doc_Matrix.md` | Full Doc | Matrix |
| `Accelerator_DSCR_1-4_Units_Matrix.md` | DSCR | Matrix |
| `Accelerator_DSCR_5-8_Units_Matrix.md` | DSCR | Matrix |
| `Ally_Consumer_No_Ratio_Matrix.md` | Ally | Matrix |
| `CF_Underwriting-Guidelines.md` | General | Guidelines |
| `CF_Underwriting-Guidelines_Ally.md` | Ally | Guidelines |
| `CF_Underwriting-Guidelines_Super_Jumbo.md` | Super Jumbo | Guidelines |
| `Champions-Funding_Non-QM-Wholesale_Product-Catalog.md` | All | Catalog |
| `Foreign-National-Ambassador_DSCR_Matrix.md` | Foreign National | Matrix |
| `Foreign-National-Ambassador_Full-Alt-Doc_Matrix.md` | Foreign National | Matrix |
| `ITIN_Accelerator_DSCR_Matrix.md` | ITIN | Matrix |
| `ITIN_Accelerator_Full-Alt_Matrix.md` | ITIN | Matrix |
| `State-Licensing-Survey.md` | General | Reference |
| `Super-Jumbo_Matrix.md` | Super Jumbo | Matrix |
| `super-jumbo-guidelines-extracted.md` | Super Jumbo | Guidelines |

---

## Docker Configuration

### docker-compose.yml

```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8082:8080"
    environment:
      - DATABASE_URL=postgresql+asyncpg://postgres:password@db:5432/loan_expert
      - JWT_SECRET=${JWT_SECRET}
      - SHARED_PASSWORD=${SHARED_PASSWORD}
      - GOOGLE_API_KEY=${GOOGLE_API_KEY}
      - GEMINI_FILE_SEARCH_STORE=${GEMINI_FILE_SEARCH_STORE}
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "3002:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:8082

  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=loan_expert
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

## Cache Configuration

### Response Cache

```python
# In-memory cache for identical queries
CACHE_TTL = 3600  # 1 hour

# Cache key includes program to avoid cross-contamination
def _get_cache_key(message: str, program_key: str | None) -> str:
    cache_input = f"{program_key or 'general'}:{message.lower().strip()}"
    return hashlib.md5(cache_input.encode()).hexdigest()
```

### Frontend Suggestions Cache

```typescript
// SessionStorage with 5-minute TTL
const cached = sessionStorage.getItem('chat_suggestions');
const cacheAge = Date.now() - (parsed.timestamp || 0);
if (cacheAge < 300000) {  // 5 minutes
  return cached.suggestions;
}
```

---

## Security Configuration

### CORS Settings

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configure for production
    allow_credentials=True,
    allow_methods=["GET", "POST", "PATCH", "DELETE"],
    allow_headers=["*"],
)
```

### JWT Configuration

```python
JWT_ALGORITHM = "HS256"
JWT_EXPIRY_DAYS = 30
```

### Password Hashing

```python
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
```
