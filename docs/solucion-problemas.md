# Guía de Solución de Problemas

Problemas comunes y soluciones para Champions Loan Expert.

## Diagnósticos Rápidos

### Verificar Estado del Sistema

```bash
# Estado del backend
curl https://champions-backend-561975502517.us-central1.run.app/health

# Frontend (debería retornar HTML)
curl -I https://champions-frontend-561975502517.us-central1.run.app
```

### Verificar Logs (Cloud Run)

```bash
# Logs del backend
gcloud run logs read champions-backend --region=us-central1 --limit=50

# Logs del frontend
gcloud run logs read champions-frontend --region=us-central1 --limit=50
```

---

## Problemas Comunes

### Problema: "No puedo iniciar sesión"

**Síntomas:**
- Formulario de login muestra "Credenciales inválidas"
- Token no se está configurando

**Posibles Causas:**
1. Contraseña incorrecta
2. Dominio de email no permitido
3. Backend no está corriendo

**Soluciones:**

1. **Verificar contraseña:**
   - Contactar admin para la contraseña compartida actual
   - Verificar variable de entorno `SHARED_PASSWORD`

2. **Verificar dominio de email:**
   - Debe terminar con `@championsfunding.com`
   - Verificar `allowed_domain` en tabla `auth_config`

3. **Verificar backend:**
   ```bash
   curl https://champions-backend-561975502517.us-central1.run.app/health
   ```

---

### Problema: "El chat no responde"

**Síntomas:**
- Mensaje enviado pero sin respuesta
- Indicador de carga girando infinitamente
- No se reciben eventos SSE

**Posibles Causas:**
1. Problemas con API de Gemini
2. File Search Store no configurado
3. Problemas de red/CORS

**Soluciones:**

1. **Verificar API de Gemini:**
   - Verificar que `GOOGLE_API_KEY` sea válida
   - Verificar estado de API de Gemini: https://status.cloud.google.com/

2. **Verificar File Search Store:**
   ```bash
   # Buscar esto en logs del backend:
   # "ChatService initialized with File Search Store: stores/..."
   ```
   Si falta, configurar variable de entorno `GEMINI_FILE_SEARCH_STORE`.

3. **Verificar código de error en logs:**
   | Código de Error | Significado | Solución |
   |-----------------|-------------|----------|
   | `NO_FILE_SEARCH_STORE` | Store no configurado | Configurar variable de entorno |
   | `RATE_LIMIT_EXCEEDED` | Demasiadas solicitudes | Esperar y reintentar |
   | `QUOTA_EXCEEDED` | Límite de facturación | Verificar facturación GCP |
   | `SERVICE_UNAVAILABLE` | Gemini caído | Esperar recuperación |

---

### Problema: "Las citas no aparecen"

**Síntomas:**
- Respuesta llega pero sin panel de citas
- Advertencia "No citations" en logs

**Posibles Causas:**
1. Consulta muy vaga/general
2. Documentos no indexados
3. File Search Store mal configurado

**Soluciones:**

1. **Hacer consulta más específica:**
   - En lugar de "requisitos" → "¿Cuáles son los requisitos de DSCR para propiedades de 1-4 unidades?"
   - Incluir nombre del programa en la pregunta

2. **Verificar documentos indexados:**
   ```bash
   # Panel Admin → Pestaña Documentos
   # Debería mostrar 16 documentos
   ```

3. **Verificar grounding_metadata en logs:**
   ```
   grounding_chunks count: 0  # Malo - no se encontraron documentos
   grounding_chunks count: 5  # Bueno - documentos recuperados
   ```

---

### Problema: "Las citas inline desaparecen al recargar"

**Síntomas:**
- Marcadores de cita [¹] visibles durante el chat
- Marcadores desaparecen después de refrescar la página

**Estado:** ✅ **CORREGIDO en v2.5.1**

Este era un bug donde las citas inline no se persistían a la base de datos. La corrección agrega tres nuevas columnas a la tabla `citations`:
- `text_start_index`
- `text_end_index`
- `inline_number`

**Si aún experimenta el problema:**
1. Ejecutar migración de base de datos:
   ```bash
   python scripts/run_migration.py
   ```
2. Verificar migración:
   ```bash
   python scripts/run_migration.py --verify
   ```

---

### Problema: "Documento Desconocido" en citas

**Síntomas:**
- Cita muestra "Documento Desconocido" en lugar del nombre de archivo
- ID de archivo raw mostrándose (ej. "files/abc123")

**Causa:** ID de archivo no está en `file_search_name_map.json`

**Solución:**

1. Verificar logs del backend por:
   ```
   Name map miss: 'abc123' not in map
   ```

2. Agregar mapeo a `backend/file_search_name_map.json`:
   ```json
   {
     "abc123": "Nuevo_Nombre_Documento.md"
   }
   ```

3. Redesplegar backend

---

### Problema: "Panel admin no carga datos"

**Síntomas:**
- Pestaña Analytics no muestra datos
- Pestaña Users vacía
- Errores de API en consola

**Posibles Causas:**
1. No está logueado como admin
2. Sesión expirada
3. Error de backend

**Soluciones:**

1. **Verificar rol de usuario:**
   - Panel admin requiere rol `admin` o `manager`
   - Verificar en consola: objeto usuario debería tener `role: "admin"`

2. **Refrescar token:**
   - Cerrar sesión y volver a iniciar
   - Tokens JWT expiran después de 30 días

3. **Verificar respuesta de API:**
   ```javascript
   // En consola del navegador
   await fetch('/api/admin/stats', {
     headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
   }).then(r => r.json())
   ```

---

### Problema: "Errores de CORS"

**Síntomas:**
- Consola muestra errores "Access-Control-Allow-Origin"
- Llamadas API fallan desde frontend

**Causa:** Configuración CORS incorrecta en backend

**Solución:**

Verificar configuración CORS en `main.py`:
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://champions-frontend-561975502517.us-central1.run.app"],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PATCH", "DELETE"],
    allow_headers=["*"],
)
```

---

### Problema: "Modo oscuro no persiste"

**Síntomas:**
- Tema se resetea a claro al refrescar
- Preferencia del sistema no detectada

**Solución:**

1. Verificar localStorage:
   ```javascript
   localStorage.getItem('theme')
   // Debería ser 'light', 'dark', o 'system'
   ```

2. Limpiar y resetear:
   ```javascript
   localStorage.setItem('theme', 'dark')
   window.location.reload()
   ```

---

### Problema: "Exportación no funciona"

**Síntomas:**
- Botón de descarga no hace nada
- Archivo se descarga vacío

**Causa:** Conversación no tiene mensajes o está archivada

**Solución:**

1. Asegurar que la conversación tenga mensajes
2. Intentar desde una conversación activa
3. Verificar configuración de descargas del navegador

---

## Problemas de Base de Datos

### Resetear Base de Datos (Solo Desarrollo)

```bash
cd backend
python scripts/setup_db.py --reset
```

### Verificar Conexión de Base de Datos

```bash
# Desde Cloud Run (con SQL proxy)
psql $DATABASE_URL -c "SELECT COUNT(*) FROM users;"
```

### Ejecutar Migración

```bash
cd backend
python scripts/run_migration.py
python scripts/run_migration.py --verify
```

### Revertir Migración

```bash
python scripts/run_migration.py --rollback
```

---

## Problemas de Rendimiento

### Tiempos de Respuesta Lentos

**Síntomas:**
- Respuestas toman >10 segundos
- Errores de timeout

**Soluciones:**

1. **Verificar latencia de Gemini:**
   - Ver `latency_ms` en tabla run_logs
   - Normal: 2000-5000ms
   - Lento: >10000ms

2. **Verificar tasa de aciertos de caché:**
   - Buscar "Cache hit for message" en logs
   - Preguntas repetidas deberían ser más rápidas

3. **Reducir historial de conversación:**
   - Sistema envía últimos 10 mensajes para contexto
   - Conversaciones más largas = más tokens

### Problemas de Memoria (Cloud Run)

**Síntomas:**
- Servicio reiniciándose frecuentemente
- Errores "Out of memory"

**Solución:**
- Aumentar límite de memoria de Cloud Run
- Actual: 512MB mínimo recomendado

---

## Obtener Ayuda

1. **Verificar logs primero:**
   - Logs de Cloud Run contienen información detallada de errores
   - Buscar códigos de error y stack traces

2. **Verificar esta guía:**
   - La mayoría de problemas comunes están documentados aquí

3. **Contactar soporte:**
   - Equipo de ingeniería nuDesk
   - Incluir: mensaje de error, timestamp, email del usuario

---

## Referencia de Códigos de Error

| Código | Significado | Solución Típica |
|--------|-------------|-----------------|
| `NO_FILE_SEARCH_STORE` | RAG no configurado | Configurar GEMINI_FILE_SEARCH_STORE |
| `MODEL_NOT_FOUND` | Nombre de modelo incorrecto | Corregir GEMINI_MODEL |
| `RATE_LIMIT_EXCEEDED` | Cuota de API alcanzada | Esperar o aumentar cuota |
| `QUOTA_EXCEEDED` | Límite de facturación | Verificar facturación GCP |
| `SERVICE_UNAVAILABLE` | Gemini caído | Esperar recuperación |
| `CONNECTION_ERROR` | Problema de red | Verificar conectividad |
| `TIMEOUT` | Solicitud muy lenta | Reintentar o reducir tamaño de consulta |
| `NO_RESPONSE` | Respuesta vacía | Reintentar con consulta diferente |
| `NO_CITATIONS` | No hay docs coincidentes | Hacer consulta más específica |
| `INVALID_RESPONSE` | Respuesta malformada | Contactar soporte |
| `CONTENT_BLOCKED` | Filtro de seguridad | Reformular consulta |
| `CHAT_FAILED` | Error genérico | Verificar logs para detalles |
