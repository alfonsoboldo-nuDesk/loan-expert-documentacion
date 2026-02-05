# Referencia de Configuración

Todas las variables de entorno y valores de configuración para Champions Loan Expert.

## Variables de Entorno

### Backend (Requeridas)

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `DATABASE_URL` | Cadena de conexión PostgreSQL | `postgresql://user:pass@host:5432/db` |
| `JWT_SECRET` | Clave de firma JWT (mín 32 chars) | `tu-clave-super-secreta-aqui-32chars` |
| `SHARED_PASSWORD` | Contraseña compartida del equipo | `contraseña_segura` |
| `GOOGLE_API_KEY` | Clave API de Gemini | `AIza...` |
| `GEMINI_FILE_SEARCH_STORE` | ID del File Search Store | `stores/abc123...` |

### Backend (Opcionales)

| Variable | Descripción | Default |
|----------|-------------|---------|
| `GEMINI_MODEL` | Nombre del modelo | `gemini-2.0-flash` |
| `LOG_LEVEL` | Nivel de logging | `INFO` |
| `CORS_ORIGINS` | Orígenes permitidos (separados por coma) | `*` |

### Frontend

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `NEXT_PUBLIC_API_URL` | URL del API Backend | `https://champions-backend-xxx.run.app` |

---

## .env.example

```bash
# Configuración Backend
DATABASE_URL=postgresql+asyncpg://postgres:password@localhost:5432/loan_expert
JWT_SECRET=tu-clave-jwt-super-secreta-al-menos-32-caracteres
SHARED_PASSWORD=champions2024

# Configuración Gemini
GOOGLE_API_KEY=AIzaSy...
GEMINI_MODEL=gemini-2.0-flash
GEMINI_FILE_SEARCH_STORE=stores/abc123def456

# Frontend (para producción)
NEXT_PUBLIC_API_URL=https://champions-backend-561975502517.us-central1.run.app
```

---

## Recursos de Google Cloud

### Servicios Cloud Run

| Servicio | URL | Región |
|----------|-----|--------|
| Frontend | `champions-frontend-561975502517.us-central1.run.app` | us-central1 |
| Backend | `champions-backend-561975502517.us-central1.run.app` | us-central1 |

### Cloud SQL

| Propiedad | Valor |
|-----------|-------|
| Instancia | PostgreSQL 14 |
| Región | us-central1 |
| Conexión | Via Cloud SQL Proxy o IP privada |

### Secret Manager

| Secreto | Propósito |
|---------|-----------|
| `jwt-secret` | Clave de firma JWT |
| `google-api-key` | Clave API de Gemini |
| `db-password` | Contraseña de base de datos |
| `shared-password` | Autenticación de usuarios |

### Gemini File Search Store

| Propiedad | Valor |
|-----------|-------|
| Store ID | `stores/...` |
| Modelo | `gemini-2.0-flash` |
| Documentos | 16 documentos de préstamos indexados |

---

## Mapa de Nombres de File Search

El `file_search_name_map.json` mapea IDs de archivos de Gemini a nombres legibles.

**Ubicación:** `backend/file_search_name_map.json`

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

## Patrones de Detección de Programas

El sistema auto-detecta programas de préstamos de los mensajes del usuario.

```python
PROGRAM_PATTERNS = {
    "full_doc": {
        "patterns": [r"\bfull\s*doc\b", r"\bfull\s+documentation\b"],
        "name": "Full Doc Accelerator",
        "matrix": "Accelerator-Activator Full Doc Matrix",
        "asset_formula": "60 meses",
    },
    "alt_doc": {
        "patterns": [r"\balt\s*doc\b", r"\balternative\s+doc\b"],
        "name": "Alt Doc Activator",
        "matrix": "Accelerator-Activator Alt Doc Matrix",
        "asset_formula": "60 meses",
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
        "asset_formula": "84 meses",
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

## Documentos Indexados

| Documento | Programa | Tipo |
|-----------|----------|------|
| `Accelerator-Activator_Alt_Doc_Matrix.md` | Alt Doc | Matriz |
| `Accelerator-Activator_Full_Doc_Matrix.md` | Full Doc | Matriz |
| `Accelerator_DSCR_1-4_Units_Matrix.md` | DSCR | Matriz |
| `Accelerator_DSCR_5-8_Units_Matrix.md` | DSCR | Matriz |
| `Ally_Consumer_No_Ratio_Matrix.md` | Ally | Matriz |
| `CF_Underwriting-Guidelines.md` | General | Guías |
| `CF_Underwriting-Guidelines_Ally.md` | Ally | Guías |
| `CF_Underwriting-Guidelines_Super_Jumbo.md` | Super Jumbo | Guías |
| `Champions-Funding_Non-QM-Wholesale_Product-Catalog.md` | Todos | Catálogo |
| `Foreign-National-Ambassador_DSCR_Matrix.md` | Foreign National | Matriz |
| `Foreign-National-Ambassador_Full-Alt-Doc_Matrix.md` | Foreign National | Matriz |
| `ITIN_Accelerator_DSCR_Matrix.md` | ITIN | Matriz |
| `ITIN_Accelerator_Full-Alt_Matrix.md` | ITIN | Matriz |
| `State-Licensing-Survey.md` | General | Referencia |
| `Super-Jumbo_Matrix.md` | Super Jumbo | Matriz |
| `super-jumbo-guidelines-extracted.md` | Super Jumbo | Guías |

---

## Configuración Docker

### docker-compose.yml

```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8080:8080"
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
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:8080

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

## Configuración de Caché

### Caché de Respuestas

```python
# Caché en memoria para consultas idénticas
CACHE_TTL = 3600  # 1 hora

# La clave de caché incluye programa para evitar contaminación cruzada
def _get_cache_key(message: str, program_key: str | None) -> str:
    cache_input = f"{program_key or 'general'}:{message.lower().strip()}"
    return hashlib.md5(cache_input.encode()).hexdigest()
```

### Caché de Sugerencias Frontend

```typescript
// SessionStorage con TTL de 5 minutos
const cached = sessionStorage.getItem('chat_suggestions');
const cacheAge = Date.now() - (parsed.timestamp || 0);
if (cacheAge < 300000) {  // 5 minutos
  return cached.suggestions;
}
```

---

## Configuración de Seguridad

### Ajustes CORS

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configurar para producción
    allow_credentials=True,
    allow_methods=["GET", "POST", "PATCH", "DELETE"],
    allow_headers=["*"],
)
```

### Configuración JWT

```python
JWT_ALGORITHM = "HS256"
JWT_EXPIRY_DAYS = 30
```

### Hash de Contraseñas

```python
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
```
