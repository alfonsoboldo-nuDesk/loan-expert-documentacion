<p align="center">
  <img src="https://img.shields.io/badge/FastAPI-Backend-009688?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI">
  <img src="https://img.shields.io/badge/Next.js-Frontend-000000?style=for-the-badge&logo=nextdotjs&logoColor=white" alt="Next.js">
  <img src="https://img.shields.io/badge/Gemini_2.0-IA-4285F4?style=for-the-badge&logo=google&logoColor=white" alt="Gemini">
  <img src="https://img.shields.io/badge/Google_Cloud-Hospedado-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white" alt="GCP">
</p>

<h1 align="center">Champions Loan Expert</h1>

<p align="center">
  <strong>Asistente impulsado por IA para los programas hipotecarios non-QM mayoristas de Champions Funding</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Backend-v2.5.0-blue?style=flat-square" alt="Versión Backend">
  <img src="https://img.shields.io/badge/Frontend-v2.5.0-blue?style=flat-square" alt="Versión Frontend">
  <img src="https://img.shields.io/badge/Estado-Producción-success?style=flat-square" alt="Estado">
  <img src="https://img.shields.io/badge/Licencia-Propietaria-red?style=flat-square" alt="Licencia">
</p>

---

## 🚀 Inicio Rápido

<table>
<tr>
<td width="120" align="center">
<b>Paso 1</b><br>
<sub>Abrir App</sub>
</td>
<td width="120" align="center">
<b>Paso 2</b><br>
<sub>Iniciar Sesión</sub>
</td>
<td width="120" align="center">
<b>Paso 3</b><br>
<sub>Hacer Pregunta</sub>
</td>
<td width="120" align="center">
<b>Paso 4</b><br>
<sub>Obtener Respuesta</sub>
</td>
<td width="120" align="center">
<b>Paso 5</b><br>
<sub>Ver Citas</sub>
</td>
</tr>
<tr>
<td align="center">🌐</td>
<td align="center">🔐</td>
<td align="center">💬</td>
<td align="center">🤖</td>
<td align="center">📄</td>
</tr>
</table>

### 🔗 Enlaces de Producción
```
Aplicación: https://champions-frontend-561975502517.us-central1.run.app
Docs API:   https://champions-backend-561975502517.us-central1.run.app/docs
```

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              Google Cloud Run                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │                         FRONTEND (Next.js 14)                           │   │
│   │                      champions-frontend-561975502517                    │   │
│   │                                                                         │   │
│   │   Chat UI ──► Conversaciones ──► Ajustes ──► Panel Admin                │   │
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
            │  Cloud SQL    │   │ Gemini 2.0    │   │    Secret     │
            │  PostgreSQL   │   │ File Search   │   │    Manager    │
            │               │   │               │   │               │
            │ • Usuarios    │   │ • Motor RAG   │   │ • JWT Secret  │
            │ • Mensajes    │   │ • 16 Docs     │   │ • API Keys    │
            │ • Citas       │   │   Indexados   │   │ • DB Creds    │
            └───────────────┘   └───────────────┘   └───────────────┘
```

### Cómo Funciona

| Paso | Componente | Acción |
|:----:|:-----------|:-------|
| 1 | Usuario | Hace pregunta sobre programas de préstamo |
| 2 | Backend | Envía consulta a Gemini con File Search RAG |
| 3 | Gemini | Busca en 16 documentos indexados por contexto |
| 4 | Backend | Transmite respuesta vía SSE con citas |
| 5 | Frontend | Renderiza respuesta con marcadores de cita clicables |

---

## 📚 Documentación

| Documento | Descripción |
|:---------|:------------|
| 📖 [Arquitectura](./docs/arquitectura.md) | Diseño del sistema y flujo de datos |
| 🔑 [Referencia API](./docs/referencia-api.md) | Documentación de endpoints REST |
| 📁 [Sistema de Enrutamiento de Archivos](./docs/sistema-enrutamiento-archivos.md) | Resolución de IDs de documentos |
| ⚙️ [Configuración](./docs/configuracion.md) | Variables de entorno y ajustes |
| 🔧 [Solución de Problemas](./docs/solucion-problemas.md) | Problemas comunes y soluciones |
| 🗺️ [Hoja de Ruta](./HOJA-DE-RUTA.md) | Funcionalidades pendientes y mejoras |

---

## 🔑 Características Principales

<table>
<tr>
<td align="center" width="33%">

### 🤖 RAG con IA
Gemini 2.0 Flash con File Search recupera contexto relevante de 16 documentos de préstamos indexados

</td>
<td align="center" width="33%">

### 📄 Sistema de Citas
Cada respuesta incluye referencias clicables a documentos fuente con números de página

</td>
<td align="center" width="33%">

### 🔒 Acceso por Roles
Tres roles (Rep, Manager, Admin) con permisos granulares para funcionalidades

</td>
</tr>
</table>

---

## 🎯 Programas de Préstamo Cubiertos

| Programa | Documentos Matrix | Guías |
|:--------|:-----------------|:-----------|
| **DSCR** | 1-4 Unidades, 5-8 Unidades | Accelerator DSCR |
| **Full Doc** | Full Doc Matrix | Guías Accelerator |
| **Alt Doc** | Alt Doc Matrix | Guías Activator |
| **Ally** | Consumer No Ratio | Guías Ally |
| **Super Jumbo** | Super Jumbo Matrix | Guías Super Jumbo |
| **Foreign National** | FN Ambassador Matrix | Guías FN |
| **ITIN** | ITIN Matrix | Guías ITIN |

---

## 🛠 Stack Tecnológico

<table>
<tr>
<td width="50%">

### Frontend
| Tecnología | Propósito |
|------------|---------|
| Next.js 14 | Framework React (App Router) |
| TypeScript | Seguridad de tipos |
| Tailwind CSS | Estilos |
| Zustand | Gestión de estado |
| Lucide Icons | Íconos UI |

</td>
<td width="50%">

### Backend
| Tecnología | Propósito |
|------------|---------|
| FastAPI | Framework web Python |
| Python 3.11 | Runtime |
| SQLAlchemy | ORM Async |
| Pydantic | Validación de datos |
| google-genai | SDK de Gemini |

</td>
</tr>
</table>

### Infraestructura
| Componente | Servicio | ID/URL |
|-----------|---------|--------|
| Frontend | Cloud Run | `champions-frontend-561975502517` |
| Backend | Cloud Run | `champions-backend-561975502517` |
| Base de Datos | Cloud SQL | PostgreSQL 14 |
| IA/RAG | Gemini 2.0 Flash | File Search Store |
| Secretos | Secret Manager | JWT, API keys |

---

## ⚙️ Operaciones Comunes

<details>
<summary><b>➕ Agregar Nuevo Documento de Préstamo</b></summary>

1. Subir documento a Gemini File Search Store
2. Actualizar `file_search_name_map.json` con mapeo ID → nombre:
   ```json
   {
     "abc123xyz": "Nuevo_Nombre_Documento.pdf"
   }
   ```
3. Redesplegar backend para cargar nuevo mapeo
</details>

<details>
<summary><b>🔐 Crear Usuario Admin</b></summary>

```bash
cd backend
python scripts/create_admin.py admin@championsfunding.com
```
</details>

<details>
<summary><b>🗄️ Ejecutar Migración de Base de Datos</b></summary>

```bash
cd backend
python scripts/run_migration.py           # Aplicar migración
python scripts/run_migration.py --verify  # Verificar estado
python scripts/run_migration.py --rollback # Revertir si es necesario
```
</details>

---

## 📁 Estructura del Repositorio

```
📦 champions-loan-expert-v2-claude
├── 📄 README.md
├── 📄 CHANGELOG.md
├── 📄 HOJA-DE-RUTA.md
├── 📄 .env.example
│
├── 📂 docs/
│   ├── 📖 arquitectura.md
│   ├── 🔑 referencia-api.md
│   ├── 📁 sistema-enrutamiento-archivos.md
│   ├── ⚙️ configuracion.md
│   └── 🔧 solucion-problemas.md
│
├── 📂 backend/
│   ├── 📂 app/
│   │   ├── 📂 api/              # Endpoints REST
│   │   ├── 📂 db/               # Conexión a base de datos
│   │   ├── 📂 models/           # SQLAlchemy + Pydantic
│   │   ├── 📂 services/         # Lógica de negocio
│   │   ├── config.py
│   │   └── main.py
│   ├── 📂 scripts/              # Utilidades admin
│   ├── 📄 file_search_name_map.json
│   └── 📄 Dockerfile
│
├── 📂 frontend/
│   ├── 📂 src/
│   │   ├── 📂 app/              # Páginas (App Router)
│   │   ├── 📂 components/       # Componentes React
│   │   ├── 📂 lib/              # Cliente API, utilidades
│   │   ├── 📂 store/            # Stores Zustand
│   │   └── 📂 types/            # Definiciones TypeScript
│   └── 📄 Dockerfile
│
└── 📂 infrastructure/           # Terraform (opcional)
```

---

## 🔗 Enlaces Rápidos

<table>
<tr>
<td align="center">
<a href="https://champions-frontend-561975502517.us-central1.run.app">
<img src="https://img.shields.io/badge/Abrir-Aplicación-4CAF50?style=for-the-badge" alt="App">
</a>
</td>
<td align="center">
<a href="https://champions-backend-561975502517.us-central1.run.app/docs">
<img src="https://img.shields.io/badge/Ver-Docs_API-009688?style=for-the-badge" alt="API">
</a>
</td>
<td align="center">
<a href="./HOJA-DE-RUTA.md">
<img src="https://img.shields.io/badge/Ver-Hoja_de_Ruta-FF6D5A?style=for-the-badge" alt="Roadmap">
</a>
</td>
</tr>
</table>

---

## 👥 Roles y Permisos

| Funcionalidad | Rep | Manager | Admin |
|---------|:---:|:-------:|:-----:|
| Chat con IA | ✅ | ✅ | ✅ |
| Ver conversaciones propias | ✅ | ✅ | ✅ |
| Exportar conversaciones | ✅ | ✅ | ✅ |
| Ver analytics del equipo | ❌ | ✅ | ✅ |
| Ver todas las conversaciones | ❌ | ✅ | ✅ |
| Gestionar usuarios | ❌ | ❌ | ✅ |
| Eliminar conversaciones | ❌ | ❌ | ✅ |
| Cambiar roles de usuario | ❌ | ❌ | ✅ |
| Invitar nuevos usuarios | ❌ | ❌ | 🔜 |

---

## 📊 Versiones Actuales

| Componente | Versión | Última Actualización |
|-----------|---------|--------------|
| Backend | v2.5.0 | 2025-01-27 |
| Frontend | v2.5.0 | 2025-01-27 |

Ver [CHANGELOG.md](./CHANGELOG.md) para notas de versión detalladas.

---

<p align="center">
  <sub>Construido con ❤️ por nuDesk LLC</sub><br>
  <sub>Propietario — Champions Funding</sub>
</p>
