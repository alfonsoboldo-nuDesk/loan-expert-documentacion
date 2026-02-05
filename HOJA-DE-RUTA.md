# Hoja de Ruta - Champions Loan Expert

Este documento rastrea funcionalidades planificadas, mejoras y deuda técnica.

---

## 🚀 Próximas Funcionalidades

### Prioridad 1: Crítico

#### ✅ Citas Inline Persistentes (COMPLETADO)
**Estado:** Implementado en v2.5.1

Los marcadores de cita [¹] ahora persisten después de recargar la página.

**Cambios realizados:**
- Agregadas columnas `text_start_index`, `text_end_index`, `inline_number` a la tabla citations
- Modificado `conversation_service.py` para guardar datos de citas inline
- Modificado `chat_service.py` para incluir inline_citations en la respuesta
- Modificado `conversations.py` para retornar inline_citations por mensaje
- Actualizado frontend para usar `message.inline_citations`

---

### Prioridad 2: Alta

#### 🔜 Registro Automático de Usuarios por Email
**Estado:** Planificado

Permitir a los admins invitar nuevos usuarios por email directamente desde el panel de admin.

**Implementación Propuesta:**

```
┌──────────────────────────────────────────┐
│  Panel Admin - Pestaña Usuarios           │
│  ┌────────────────────────────────────┐  │
│  │ [+ Invitar Usuario]     [Actualizar]│  │
│  └────────────────────────────────────┘  │
│                                          │
│  Clic en [+ Invitar Usuario]:            │
│  ┌────────────────────────────────────┐  │
│  │  Email: [________________________] │  │
│  │  Rol:   [Rep ▼]                    │  │
│  │                                    │  │
│  │     [Cancelar]    [Enviar Invitación]│ │
│  └────────────────────────────────────┘  │
│                                          │
│  Resultado:                              │
│  ┌────────────────────────────────────┐  │
│  │  ✓ Usuario Creado                  │  │
│  │  Email: nuevo@championsfunding.com │  │
│  │  Contraseña: Abc123XyZ!@#          │  │
│  │                                    │  │
│  │  [Copiar Credenciales]    [Cerrar] │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

**Archivos a modificar:**

| Archivo | Cambio |
|------|--------|
| `backend/app/api/admin.py` | Agregar endpoint `POST /admin/users/invite` |
| `backend/app/models/schemas.py` | Agregar `UserInvite`, `UserInviteResponse` |
| `frontend/src/lib/api.ts` | Agregar `adminApi.inviteUser()` |
| `frontend/src/app/admin/page.tsx` | Agregar modal de invitación UI |

**Diseño de API:**
```python
@router.post("/users/invite")
async def invite_user(
    invite_data: UserInvite,  # email, role
    current_user: AdminUser,
    db: DbSession
) -> UserInviteResponse:
    # 1. Verificar que el email no exista
    # 2. Generar contraseña aleatoria de 12 caracteres
    # 3. Crear usuario con contraseña hasheada
    # 4. Retornar email + contraseña temporal
```

---

### Prioridad 3: Media

#### 🔜 Flujo de Restablecimiento de Contraseña
**Estado:** Planificado

Permitir a los usuarios restablecer su contraseña.

**Opciones:**
1. **Iniciado por admin:** Admin genera nueva contraseña para usuario
2. **Autoservicio (futuro):** Restablecimiento de contraseña por email

---

#### 🔜 Dashboard de Actividad de Usuarios
**Estado:** Planificado

Analytics mejorados mostrando:
- Conteo de mensajes por usuario
- Usuarios más activos
- Calidad de respuestas (ratio de feedback)
- Uso de tokens por usuario

---

#### 🔜 Plantillas de Conversación
**Estado:** Idea

Iniciadores de conversación predefinidos para escenarios comunes:
- "Verificación de calificación de nuevo prestatario"
- "Solicitud de comparación de programas"
- "Requisitos de documentación"

---

### Prioridad 4: Baja / Deseable

#### 🔜 Mejoras de Responsividad Móvil
**Estado:** Planificado

La UI actual funciona en móvil pero podría mejorar:
- Sidebar colapsable
- Mejores objetivos táctiles
- Gestos de deslizamiento

---

#### 🔜 Atajos de Teclado
**Estado:** Idea

- `Ctrl+N` - Nueva conversación
- `Ctrl+K` - Buscar conversaciones
- `Escape` - Cerrar modales

---

#### 🔜 Exportar a PDF
**Estado:** Idea

Exportar conversaciones como PDF además de Markdown.

---

## 🔧 Deuda Técnica

### Base de Datos

#### Considerar Alembic para Migraciones
**Prioridad:** Media

Actualmente usando scripts SQL manuales. Considerar agregar Alembic para:
- Seguimiento de versiones
- Rollback automático
- Múltiples desarrolladores

**Enfoque actual:**
```bash
python scripts/run_migration.py
```

**Enfoque con Alembic:**
```bash
alembic revision --autogenerate -m "descripcion"
alembic upgrade head
```

---

### Backend

#### Agregar Tests Unitarios
**Prioridad:** Alta

Cobertura de tests actual: Baja

**Áreas que necesitan tests:**
- `chat_service.py` - Extracción de citas
- `conversation_service.py` - Operaciones CRUD
- `auth_service.py` - Validación JWT

---

#### Agregar Validación de Requests
**Prioridad:** Media

Algunos endpoints carecen de validación de entrada apropiada:
- Límites de longitud de strings
- Validación de formato de email
- Validación de formato UUID

---

### Frontend

#### Agregar Tests E2E
**Prioridad:** Media

Considerar Playwright o Cypress para:
- Flujo de login
- Flujo de chat
- Panel de admin

---

#### Biblioteca de Componentes
**Prioridad:** Baja

Extraer componentes comunes a una biblioteca compartida:
- Variantes de botón
- Componente modal
- Notificaciones toast

---

## 📊 Métricas a Rastrear

### Rendimiento
- [ ] Latencia promedio de respuesta
- [ ] Tasa de acierto de caché
- [ ] Tendencias de uso de tokens

### Uso
- [ ] Usuarios activos diarios
- [ ] Mensajes por día
- [ ] Preguntas más comunes

### Calidad
- [ ] Ratio de feedback (pulgar arriba vs abajo)
- [ ] Precisión de citas
- [ ] Tasa de errores

---

## 🗓️ Historial de Versiones

| Versión | Fecha | Destacados |
|---------|------|------------|
| 2.5.1 | 2025-02 | Citas inline persistentes |
| 2.5.0 | 2025-01 | Manejo de errores, códigos de error |
| 2.4.0 | 2025-01 | Refuerzo de seguridad |
| 2.3.0 | 2025-01 | Sugerencias dinámicas |
| 2.2.0 | 2025-01 | Mejoras de admin |
| 2.1.0 | 2025-01 | Modo oscuro, i18n |
| 2.0.0 | 2025-01 | Lanzamiento inicial |

---

## 💡 Solicitudes de Funcionalidades

### De Usuarios
1. "¿Podemos exportar todas las conversaciones a la vez?"
2. "¿Podemos tener prompts/plantillas guardados?"
3. "¿Puede el sistema recordar mis programas preferidos?"

### Internas
1. Registro de auditoría para cumplimiento
2. Límite de tasa por usuario
3. Seguimiento de versiones de documentos

---

## 🎯 Criterios de Éxito

### Corto plazo (Q1 2025)
- [ ] Citas inline persisten correctamente
- [ ] Flujo de invitación de usuarios funcionando
- [ ] 95% de uptime

### Mediano plazo (Q2 2025)
- [ ] Cobertura de tests > 60%
- [ ] UI amigable para móvil
- [ ] Dashboard de analytics de uso

### Largo plazo (2025)
- [ ] Soporte multi-tenant
- [ ] Carga de documentos personalizados
- [ ] API para integraciones externas
