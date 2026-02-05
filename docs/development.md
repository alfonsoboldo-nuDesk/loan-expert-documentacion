# Local Development

How to run Champions Loan Expert locally for development.

## Prerequisites

- Docker and Docker Compose
- Node.js 20+ (for frontend development)
- Python 3.11+ (for backend development)
- Google API Key for Gemini

## Quick Start with Docker

The easiest way to run everything locally:

```bash
# From the project root
docker-compose up -d
```

This starts:
- PostgreSQL database on port 5434
- Backend API on port 8082
- Frontend on port 3002

Access the app at: http://localhost:3002

## Environment Setup

1. Copy the example environment file:
```bash
cp .env.example .env
```

2. Edit `.env` and add your Google API Key:
```
GOOGLE_API_KEY=your-key-here
```

## Running Backend Separately

If you want to run just the backend:

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set environment variables
export DATABASE_URL=postgresql+asyncpg://champions:champions_dev_password@localhost:5434/champions_loan_expert
export JWT_SECRET=dev-secret-change-in-production-minimum-32-chars
export SHARED_PASSWORD=ChampionsLoanExpert2024!
export GOOGLE_API_KEY=your-key-here
export GEMINI_MODEL=gemini-3-flash

# Initialize database (first time only)
python scripts/setup_db.py

# Run the server
uvicorn app.main:app --reload --port 8080
```

Backend will be at: http://localhost:8080
API docs at: http://localhost:8080/docs

## Running Frontend Separately

If you want to run just the frontend:

```bash
cd frontend

# Install dependencies
npm install

# Set environment variable
export NEXT_PUBLIC_API_URL=http://localhost:8080

# Run development server
npm run dev
```

Frontend will be at: http://localhost:3000

## Database

### Using Docker Database

The Docker Compose setup includes PostgreSQL. To connect:

```bash
psql -h localhost -p 5434 -U champions -d champions_loan_expert
# Password: champions_dev_password
```

### pgAdmin

For a visual database interface, start pgAdmin:

```bash
docker-compose --profile tools up pgadmin -d
```

Access at: http://localhost:5052
- Email: admin@championsfunding.com
- Password: admin

## Useful Commands

### View logs
```bash
docker-compose logs -f backend
docker-compose logs -f frontend
```

### Restart a service
```bash
docker-compose restart backend
```

### Reset database
```bash
docker-compose down -v
docker-compose up -d
```

### Run backend tests
```bash
cd backend
pytest
```

### Run frontend tests
```bash
cd frontend
npm test
```

## Ports Reference

| Service | Port | URL |
|---------|------|-----|
| Frontend | 3002 | http://localhost:3002 |
| Backend | 8082 | http://localhost:8082 |
| Database | 5434 | localhost:5434 |
| pgAdmin | 5052 | http://localhost:5052 |

## Troubleshooting

### Port already in use

Stop existing containers:
```bash
docker-compose down
```

Or change ports in `docker-compose.yml`.

### Database connection refused

Make sure the database container is running:
```bash
docker-compose ps
```

Wait for health check to pass:
```bash
docker-compose logs db
```

### Frontend can't connect to backend

Check the `NEXT_PUBLIC_API_URL` environment variable matches where the backend is running.

### Changes not reflecting

For frontend: Next.js has hot reload, but sometimes you need to restart.

For backend: If using `--reload`, changes should auto-reload. Otherwise restart the server.

---

## Development Workflow

### Making Changes

1. Create a branch for your feature
2. Make changes locally
3. Test with Docker Compose
4. Run any tests
5. Create a pull request

### Code Style

**Frontend:**
- TypeScript for type safety
- Tailwind CSS for styling
- Zustand for state management

**Backend:**
- Python 3.11+ features
- Type hints throughout
- Pydantic for validation
- Async/await patterns

### Git Workflow

```bash
# Create feature branch
git checkout -b feature/your-feature

# Make changes and commit
git add .
git commit -m "Add your feature"

# Push to remote
git push -u origin feature/your-feature
```
