# Sistema de Enrutamiento de Archivos

Este documento explica cómo Champions Loan Expert resuelve los IDs de archivo opacos de Gemini a nombres de documentos legibles.

## Descripción General

Cuando Gemini retorna citas con metadatos de grounding, usa IDs de archivo internos (ej. `files/my6heaulw7lm`). El Sistema de Enrutamiento de Archivos convierte estos a nombres de documentos legibles.

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                    SISTEMA DE ENRUTAMIENTO DE ARCHIVOS                        │
├───────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  Respuesta de Gemini                                                          │
│  ───────────────────                                                          │
│  grounding_metadata.grounding_chunks[0].retrieved_context                     │
│    .title = "files/my6heaulw7lm"                                              │
│                                                                               │
│                              │                                                │
│                              ▼                                                │
│                                                                               │
│  _resolve_source_name() en chat_service.py                                    │
│  ─────────────────────────────────────────                                    │
│  1. Intentar búsqueda directa: _file_name_map["my6heaulw7lm"]                 │
│  2. Intentar extraer ID: "files/abc" → "abc"                                  │
│  3. Si tiene extensión, usar tal cual                                         │
│  4. Fallback al nombre raw                                                    │
│                                                                               │
│                              │                                                │
│                              ▼                                                │
│                                                                               │
│  file_search_name_map.json                                                    │
│  ─────────────────────────                                                    │
│  {                                                                            │
│    "my6heaulw7lm": "Accelerator-Activator_Alt_Doc_Matrix.md"                  │
│  }                                                                            │
│                                                                               │
│                              │                                                │
│                              ▼                                                │
│                                                                               │
│  Resultado: "Accelerator-Activator_Alt_Doc_Matrix.md"                         │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
```

---

## Componentes Clave

### 1. Archivo de Mapa de Nombres

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

### 2. Carga del Mapa de Nombres

**Ubicación:** `backend/app/services/chat_service.py` (líneas 63-88)

```python
NAME_MAP_PATH = Path(__file__).parent.parent.parent / "file_search_name_map.json"
_file_name_map: Dict[str, str] = {}

def _load_name_map():
    """Cargar el mapeo de ID de archivo a nombre de visualización."""
    global _file_name_map
    if NAME_MAP_PATH.exists():
        with open(NAME_MAP_PATH, "r", encoding="utf-8") as f:
            _file_name_map = json.load(f)
        print(f"[chat_service] Cargados {len(_file_name_map)} mapeos de nombres de archivo")

# Cargar al importar el módulo
_load_name_map()
```

### 3. Función de Resolución

**Ubicación:** `backend/app/services/chat_service.py` (líneas 474-497)

```python
def _resolve_source_name(self, raw_name: str) -> str:
    """Resolver un ID de archivo o URI a un nombre de visualización legible."""
    if not raw_name:
        return "Documento Desconocido"

    # Intentar búsqueda directa
    if raw_name in _file_name_map:
        return _file_name_map[raw_name]

    # Intentar extraer ID de archivo de URI (ej. "files/abc123" -> "abc123")
    if "/" in raw_name:
        file_id = raw_name.split("/")[-1]
        if file_id in _file_name_map:
            return _file_name_map[file_id]

    # Si ya parece un nombre de archivo (tiene extensión), usarlo
    if "." in raw_name and not raw_name.startswith("files/"):
        return raw_name

    # Fallback al nombre raw
    return raw_name
```

---

## Categorías de Documentos

### Documentos de Matriz (Elegibilidad Específica por Producto)

| ID de Archivo | Nombre de Documento | Programa |
|---------------|---------------------|----------|
| `my6heaulw7lm` | Accelerator-Activator_Alt_Doc_Matrix.md | Alt Doc |
| `6m7zk1jwfroi` | Accelerator-Activator_Full_Doc_Matrix.md | Full Doc |
| `y5t14gqb2gqf` | Accelerator_DSCR_1-4_Units_Matrix.md | DSCR |
| `z0p5iyqywsqc` | Accelerator_DSCR_5-8_Units_Matrix.md | DSCR |
| `m5cvwcrzgk4s` | Ally_Consumer_No_Ratio_Matrix.md | Ally |
| `b2ykwzwzlv3r` | Foreign-National-Ambassador_DSCR_Matrix.md | FN |
| `0o4wqv6dlx3f` | Foreign-National-Ambassador_Full-Alt-Doc_Matrix.md | FN |
| `0hkwdxz6a4nw` | ITIN_Accelerator_DSCR_Matrix.md | ITIN |
| `mvrqmqudz0qt` | ITIN_Accelerator_Full-Alt_Matrix.md | ITIN |
| `gbrnm6c0xv4f` | Super-Jumbo_Matrix.md | Super Jumbo |

### Documentos de Guías (Reglas de Underwriting)

| ID de Archivo | Nombre de Documento | Programa |
|---------------|---------------------|----------|
| `82njhpyj0iq6` | CF_Underwriting-Guidelines.md | General |
| `qmhjgpakpzqh` | CF_Underwriting-Guidelines_Ally.md | Ally |
| `5r6xsv94vmkp` | CF_Underwriting-Guidelines_Super_Jumbo.md | Super Jumbo |
| `s53hx36r2ey8` | super-jumbo-guidelines-extracted.md | Super Jumbo |

### Documentos de Referencia

| ID de Archivo | Nombre de Documento | Propósito |
|---------------|---------------------|-----------|
| `fh10d6v0nlhi` | Champions-Funding_Non-QM-Wholesale_Product-Catalog.md | Resumen de productos |
| `mw2rdrtfnlzl` | State-Licensing-Survey.md | Requisitos por estado |

---

## Cómo Se Usa

### Flujo de Extracción de Citas

```python
def _extract_citations(self, grounding_metadata):
    """Extraer citas de los metadatos de grounding de Gemini."""

    for chunk in grounding_metadata.grounding_chunks:
        ctx = chunk.retrieved_context

        # Obtener fuente raw (ID opaco o URI)
        raw_source = ctx.title or ctx.uri or "Unknown"

        # Resolver a nombre legible
        source_name = self._resolve_source_name(raw_source)

        # Usar en cita
        citations.append(Citation(
            source_name=source_name,  # "Accelerator_DSCR_Matrix.md"
            source_uri=ctx.uri,       # "files/y5t14gqb2gqf"
            ...
        ))
```

### Visualización en Frontend

El nombre resuelto se muestra en el Panel de Fuentes:

```
┌─────────────────────────────────────────┐
│ Fuentes                                 │
├─────────────────────────────────────────┤
│ 1. Accelerator_DSCR_1-4_Units_Matrix.md │
│    Páginas: 2, 3                        │
│                                         │
│ 2. CF_Underwriting-Guidelines.md        │
│    Páginas: 15                          │
└─────────────────────────────────────────┘
```

---

## Agregar Nuevos Documentos

### Paso 1: Subir a Gemini File Search Store

Usar la API de Gemini para subir el documento a tu File Search Store.

### Paso 2: Obtener el ID de Archivo

Después de subir, anotar el ID de archivo retornado (ej. `abc123xyz`).

### Paso 3: Actualizar Mapa de Nombres

Editar `backend/file_search_name_map.json`:

```json
{
  "id_existente": "Documento_Existente.md",
  "abc123xyz": "Nuevo_Nombre_Documento.md"
}
```

### Paso 4: Redesplegar Backend

El mapa de nombres se carga al inicio, así que redesplegar para cargar el nuevo mapeo.

---

## Solución de Problemas

### "Documento Desconocido" apareciendo en citas

**Causa:** ID de archivo no está en `file_search_name_map.json`

**Solución:**
1. Revisar logs del backend por "Name map miss: 'xxx' not in map"
2. Encontrar el ID faltante y agregarlo al archivo JSON
3. Redesplegar backend

### Citas mostrando IDs de archivo raw

**Causa:** Archivo de mapa de nombres no encontrado o falló al cargar

**Verificar:**
1. Verificar que `file_search_name_map.json` existe en `backend/`
2. Revisar logs por "[chat_service] Loaded X file name mappings"
3. Si 0 mapeos cargados, revisar ruta del archivo y sintaxis JSON

### Logs a revisar

```
[chat_service] Looking for name map at: /app/file_search_name_map.json
[chat_service] File exists: True
[chat_service] Loaded 16 file name mappings
[chat_service] Name map hit: 'my6heaulw7lm' -> 'Accelerator-Activator_Alt_Doc_Matrix.md'
```
