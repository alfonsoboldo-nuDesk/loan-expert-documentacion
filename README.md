<p align="center">
  <img src="https://img.shields.io/badge/FastAPI-Backend-009688?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI">
  <img src="https://img.shields.io/badge/Next.js-Frontend-000000?style=for-the-badge&logo=nextdotjs&logoColor=white" alt="Next.js">
  <img src="https://img.shields.io/badge/Gemini_3-AI-4285F4?style=for-the-badge&logo=google&logoColor=white" alt="Gemini">
  <img src="https://img.shields.io/badge/Google_Cloud-Hosted-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white" alt="GCP">
</p>

<h1 align="center">Champions Loan Expert</h1>

<p align="center">
  <strong>AI-powered assistant for Champions Funding's wholesale non-QM mortgage programs</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Backend-v2.5.1-blue?style=flat-square" alt="Backend Version">
  <img src="https://img.shields.io/badge/Frontend-v2.5.1-blue?style=flat-square" alt="Frontend Version">
  <img src="https://img.shields.io/badge/Status-Production-success?style=flat-square" alt="Status">
  <img src="https://img.shields.io/badge/License-Proprietary-red?style=flat-square" alt="License">
</p>

---

## Quick Start

<table>
<tr>
<td width="120" align="center">
<b>Step 1</b><br>
<sub>Open App</sub>
</td>
<td width="120" align="center">
<b>Step 2</b><br>
<sub>Log In</sub>
</td>
<td width="120" align="center">
<b>Step 3</b><br>
<sub>Ask Question</sub>
</td>
<td width="120" align="center">
<b>Step 4</b><br>
<sub>Get Answer</sub>
</td>
<td width="120" align="center">
<b>Step 5</b><br>
<sub>View Citations</sub>
</td>
</tr>
<tr>
<td align="center">Open the app URL</td>
<td align="center">Use @championsfunding.com email</td>
<td align="center">Type your loan question</td>
<td align="center">AI responds with sources</td>
<td align="center">Click citation markers</td>
</tr>
</table>

### Production Links
```
Application: https://champions-frontend-561975502517.us-central1.run.app
API Docs:    https://champions-backend-561975502517.us-central1.run.app/docs
```

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              Google Cloud Run                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │                         FRONTEND (Next.js 14)                           │   │
│   │                      champions-frontend-561975502517                    │   │
│   │                                                                         │   │
│   │   Chat UI ──► Conversations ──► Settings ──► Admin Panel                │   │
│   │                                                                         │   │
│   └───────────────────────────────────┬─────────────────────────────────────┘   │
│                                       │                                         │
│                            REST API + SSE Streaming                             │
│                                       │                                         │
│   ┌───────────────────────────────────▼─────────────────────────────────────┐   │
│   │                          BACKEND (FastAPI)                              │   │
│   │                      champions-backend-561975502517                     │   │
│   │                                                                         │   │
│   │   Auth ──► Chat Service ──► Conversation Service ──► Admin Service      │   │
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
            │               │   │               │   │               │
            │ • Users       │   │ • RAG Engine  │   │ • JWT Secret  │
            │ • Messages    │   │ • 16 Docs     │   │ • API Keys    │
            │ • Citations   │   │   Indexed     │   │ • DB Creds    │
            └───────────────┘   └───────────────┘   └───────────────┘
```

### How It Works

| Step | Component | Action |
|:----:|:----------|:-------|
| 1 | User | Asks question about loan programs |
| 2 | Backend | Sends query to Gemini with File Search RAG |
| 3 | Gemini | Searches 16 indexed documents for context |
| 4 | Backend | Streams response via SSE with citations |
| 5 | Frontend | Renders response with clickable citation markers |

---

## Documentation

| Document | Description |
|:---------|:------------|
| [Architecture](./docs/architecture.md) | System design and data flow |
| [Frontend](./docs/frontend.md) | React components and state management |
| [Backend](./docs/backend.md) | FastAPI services and endpoints |
| [Database](./docs/database.md) | PostgreSQL schema and relationships |
| [API Reference](./docs/api-reference.md) | REST endpoint documentation |
| [File Routing System](./docs/file-routing-system.md) | Document ID resolution |
| [Configuration](./docs/configuration.md) | Environment variables and settings |
| [Development](./docs/development.md) | Local setup instructions |
| [Deployment](./docs/deployment.md) | Cloud Run deployment guide |
| [Troubleshooting](./docs/troubleshooting.md) | Common issues and solutions |
| [Roadmap](./ROADMAP.md) | Pending features and improvements |

---

## Key Features

<table>
<tr>
<td align="center" width="33%">

### AI-Powered RAG
Gemini 3 Flash with File Search retrieves relevant context from 16 indexed loan documents

</td>
<td align="center" width="33%">

### Citation System
Every response includes clickable references to source documents with page numbers

</td>
<td align="center" width="33%">

### Role-Based Access
Three roles (Rep, Manager, Admin) with granular permissions for features

</td>
</tr>
</table>

---

## Loan Programs Covered

| Program | Matrix Documents | Guidelines |
|:--------|:-----------------|:-----------|
| **DSCR** | 1-4 Units, 5-8 Units | Accelerator DSCR |
| **Full Doc** | Full Doc Matrix | Accelerator Guidelines |
| **Alt Doc** | Alt Doc Matrix | Activator Guidelines |
| **Ally** | Consumer No Ratio | Ally Guidelines |
| **Super Jumbo** | Super Jumbo Matrix | Super Jumbo Guidelines |
| **Foreign National** | FN Ambassador Matrix | FN Guidelines |
| **ITIN** | ITIN Matrix | ITIN Guidelines |

---

## Technology Stack

<table>
<tr>
<td width="50%">

### Frontend
| Technology | Purpose |
|------------|---------|
| Next.js 14 | React framework (App Router) |
| TypeScript | Type safety |
| Tailwind CSS | Styling |
| Zustand | State management |
| Lucide Icons | UI icons |

</td>
<td width="50%">

### Backend
| Technology | Purpose |
|------------|---------|
| FastAPI | Python web framework |
| Python 3.11 | Runtime |
| SQLAlchemy | Async ORM |
| Pydantic | Data validation |
| google-genai | Gemini SDK |

</td>
</tr>
</table>

### Infrastructure
| Component | Service | ID/URL |
|-----------|---------|--------|
| Frontend | Cloud Run | `champions-frontend-561975502517` |
| Backend | Cloud Run | `champions-backend-561975502517` |
| Database | Cloud SQL | PostgreSQL 14 |
| AI/RAG | Gemini 3 Flash | File Search Store |
| Secrets | Secret Manager | JWT, API keys |

---

## Common Operations

<details>
<summary><b>Add New Loan Document</b></summary>

1. Upload document to Gemini File Search Store
2. Update `file_search_name_map.json` with ID → name mapping:
   ```json
   {
     "abc123xyz": "New_Document_Name.pdf"
   }
   ```
3. Redeploy backend to load new mapping
</details>

<details>
<summary><b>Create Admin User</b></summary>

```bash
cd backend
python scripts/create_admin.py admin@championsfunding.com
```
</details>

<details>
<summary><b>Run Database Migration</b></summary>

```bash
cd backend
python scripts/run_migration.py           # Apply migration
python scripts/run_migration.py --verify  # Verify status
python scripts/run_migration.py --rollback # Rollback if needed
```
</details>

<details>
<summary><b>Run Locally with Docker</b></summary>

```bash
docker-compose up -d
# Frontend: http://localhost:3002
# Backend:  http://localhost:8082
```
</details>

<details>
<summary><b>Deploy to Production</b></summary>

```bash
# Backend
cd backend && gcloud run deploy champions-backend --source . --region us-central1

# Frontend
cd frontend && gcloud run deploy champions-frontend --source . --region us-central1
```
</details>

---

## Repository Structure

```
champions-loan-expert-v2-claude
├── README.md
├── CHANGELOG.md
├── ROADMAP.md
├── .env.example
│
├── docs/
│   ├── architecture.md
│   ├── frontend.md
│   ├── backend.md
│   ├── database.md
│   ├── api-reference.md
│   ├── file-routing-system.md
│   ├── configuration.md
│   ├── development.md
│   ├── deployment.md
│   └── troubleshooting.md
│
├── backend/
│   ├── app/
│   │   ├── api/              # REST endpoints
│   │   ├── db/               # Database connection
│   │   ├── models/           # SQLAlchemy + Pydantic
│   │   ├── services/         # Business logic
│   │   ├── config.py
│   │   └── main.py
│   ├── scripts/              # Admin utilities
│   ├── file_search_name_map.json
│   └── Dockerfile
│
├── frontend/
│   ├── src/
│   │   ├── app/              # Pages (App Router)
│   │   ├── components/       # React components
│   │   ├── lib/              # API client, utilities
│   │   ├── store/            # Zustand stores
│   │   └── types/            # TypeScript definitions
│   └── Dockerfile
│
└── docker-compose.yml
```

---

## Quick Links

<table>
<tr>
<td align="center">
<a href="https://champions-frontend-561975502517.us-central1.run.app">
<img src="https://img.shields.io/badge/Open-Application-4CAF50?style=for-the-badge" alt="App">
</a>
</td>
<td align="center">
<a href="https://champions-backend-561975502517.us-central1.run.app/docs">
<img src="https://img.shields.io/badge/View-API_Docs-009688?style=for-the-badge" alt="API">
</a>
</td>
<td align="center">
<a href="./ROADMAP.md">
<img src="https://img.shields.io/badge/View-Roadmap-FF6D5A?style=for-the-badge" alt="Roadmap">
</a>
</td>
</tr>
</table>

---

## Roles & Permissions

| Feature | Rep | Manager | Admin |
|---------|:---:|:-------:|:-----:|
| Chat with AI | ✅ | ✅ | ✅ |
| View own conversations | ✅ | ✅ | ✅ |
| Export conversations | ✅ | ✅ | ✅ |
| View team analytics | ❌ | ✅ | ✅ |
| View all conversations | ❌ | ✅ | ✅ |
| Manage users | ❌ | ❌ | ✅ |
| Delete conversations | ❌ | ❌ | ✅ |
| Change user roles | ❌ | ❌ | ✅ |
| Invite new users | ❌ | ❌ | Coming Soon |

---

## Current Versions

| Component | Version | Last Updated |
|-----------|---------|--------------|
| Backend | v2.5.1 | 2025-02 |
| Frontend | v2.5.1 | 2025-02 |

See [CHANGELOG.md](./CHANGELOG.md) for detailed release notes.

---

<p align="center">
  <sub>Built with care by nuDesk LLC</sub><br>
  <sub>Proprietary — Champions Funding</sub>
</p>
