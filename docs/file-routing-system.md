# File Routing System

This document explains how the citation system resolves Gemini file IDs to human-readable document names.

## Overview

When Gemini File Search returns results, it includes file references like `files/my6heaulw7lm`. These IDs need to be converted to readable names for display in the UI.

## How It Works

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         File Routing Flow                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Gemini Response                                                           │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  grounding_metadata:                                                │   │
│   │    grounding_chunks:                                                │   │
│   │      - chunk:                                                       │   │
│   │          retrieved_context:                                         │   │
│   │            uri: "files/my6heaulw7lm"  ◄── Raw file ID               │   │
│   │            title: ""                                                │   │
│   │            text: "DSCR minimum is..."                               │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  Extract file ID: "my6heaulw7lm"                                    │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  file_search_name_map.json                                          │   │
│   │  {                                                                  │   │
│   │    "my6heaulw7lm": "Accelerator-Activator_Alt_Doc_Matrix.md"        │   │
│   │  }                                                                  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  Result: "Accelerator-Activator_Alt_Doc_Matrix.md"                  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## The Name Map File

**Location:** `backend/file_search_name_map.json`

This JSON file maps Gemini file IDs to document names:

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

## Implementation

### In ChatService (`chat_service.py`)

```python
def _resolve_source_name(self, uri: str) -> str:
    """Convert Gemini file URI to readable name."""
    # Extract file ID from URI like "files/my6heaulw7lm"
    if uri.startswith("files/"):
        file_id = uri.split("/")[-1]
    else:
        file_id = uri

    # Look up in name map
    if file_id in self._name_map:
        return self._name_map[file_id]

    # Log miss for debugging
    logger.warning(f"Name map miss: '{file_id}' not in map")
    return "Unknown Document"
```

### Loading the Name Map

```python
def _load_name_map(self) -> dict:
    """Load file ID to name mapping."""
    map_path = Path(__file__).parent.parent.parent / "file_search_name_map.json"

    if map_path.exists():
        with open(map_path) as f:
            return json.load(f)

    return {}
```

## Adding New Documents

When you upload a new document to Gemini File Search:

1. **Get the file ID** from the upload response or Gemini Console

2. **Add mapping** to `file_search_name_map.json`:
   ```json
   {
     "existing_mappings": "...",
     "new_file_id": "Human_Readable_Name.md"
   }
   ```

3. **Redeploy backend** to load the new mapping

## Troubleshooting

### "Unknown Document" appears

**Cause:** File ID is not in the name map

**Solution:**
1. Check backend logs for:
   ```
   Name map miss: 'abc123xyz' not in map
   ```
2. Add the missing ID to `file_search_name_map.json`
3. Redeploy backend

### Raw file ID appears (files/abc123)

**Cause:** The `_resolve_source_name` function didn't process the URI

**Solution:**
1. Check that the file is properly indexed
2. Verify the name map file is loading correctly
3. Check for JSON parsing errors

### Name map not loading

**Cause:** File not found or invalid JSON

**Solution:**
1. Verify file exists at `backend/file_search_name_map.json`
2. Validate JSON syntax
3. Check file permissions

## Best Practices

1. **Use descriptive names:** `Accelerator_DSCR_1-4_Units_Matrix.md` not `doc1.md`

2. **Include program identifier:** Makes it clear which loan program the document covers

3. **Match original filename:** Easier to trace back to source files

4. **Update map before deployment:** Don't deploy with missing mappings

## Monitoring

Check for name map misses in logs:

```bash
gcloud run logs read champions-backend --region=us-central1 | grep "Name map miss"
```

If you see frequent misses, the name map needs updating.
