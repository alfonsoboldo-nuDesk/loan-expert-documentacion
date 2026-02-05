# Troubleshooting Guide

Common issues and solutions for Champions Loan Expert.

## Quick Diagnostics

### Check System Health

```bash
# Backend health
curl https://champions-backend-561975502517.us-central1.run.app/health

# Frontend (should return HTML)
curl -I https://champions-frontend-561975502517.us-central1.run.app
```

### Check Logs (Cloud Run)

```bash
# Backend logs
gcloud run logs read champions-backend --region=us-central1 --limit=50

# Frontend logs
gcloud run logs read champions-frontend --region=us-central1 --limit=50
```

---

## Common Issues

### Issue: "Unable to login"

**Symptoms:**
- Login form shows "Invalid credentials"
- Token not being set

**Possible Causes:**
1. Wrong password
2. Email domain not allowed
3. Backend not running

**Solutions:**

1. **Verify password:**
   - Contact admin for current shared password
   - Check `SHARED_PASSWORD` environment variable

2. **Verify email domain:**
   - Must end with `@championsfunding.com`
   - Check `allowed_domain` in `auth_config` table

3. **Check backend:**
   ```bash
   curl https://champions-backend-561975502517.us-central1.run.app/health
   ```

---

### Issue: "Chat not responding"

**Symptoms:**
- Message sent but no response
- Spinning indicator forever
- No SSE events received

**Possible Causes:**
1. Gemini API issues
2. File Search Store not configured
3. Network/CORS issues

**Solutions:**

1. **Check Gemini API:**
   - Verify `GOOGLE_API_KEY` is valid
   - Check Gemini API status: https://status.cloud.google.com/

2. **Check File Search Store:**
   ```bash
   # Look for this in backend logs:
   # "ChatService initialized with File Search Store: stores/..."
   ```
   If missing, set `GEMINI_FILE_SEARCH_STORE` environment variable.

3. **Check error code in logs:**
   | Error Code | Meaning | Solution |
   |------------|---------|----------|
   | `NO_FILE_SEARCH_STORE` | Store not configured | Set environment variable |
   | `RATE_LIMIT_EXCEEDED` | Too many requests | Wait and retry |
   | `QUOTA_EXCEEDED` | Billing limit | Check GCP billing |
   | `SERVICE_UNAVAILABLE` | Gemini down | Wait for recovery |

---

### Issue: "Citations not showing"

**Symptoms:**
- Response arrives but no citations panel
- "No citations" warning in logs

**Possible Causes:**
1. Query too vague/general
2. Documents not indexed
3. File Search Store misconfigured

**Solutions:**

1. **Make query more specific:**
   - Instead of "requirements" → "What are the DSCR requirements for 1-4 unit properties?"
   - Include program name in question

2. **Check indexed documents:**
   ```bash
   # Admin panel → Documents tab
   # Should show 16 documents
   ```

3. **Check grounding_metadata in logs:**
   ```
   grounding_chunks count: 0  # Bad - no documents found
   grounding_chunks count: 5  # Good - documents retrieved
   ```

---

### Issue: "Inline citations disappear on reload"

**Symptoms:**
- Citation markers [¹] visible during chat
- Markers gone after page refresh

**Status:** ✅ **FIXED in v2.5.1**

This was a bug where inline citations weren't persisted to the database. The fix adds three new columns to the `citations` table:
- `text_start_index`
- `text_end_index`
- `inline_number`

**If still experiencing:**
1. Run database migration:
   ```bash
   python scripts/run_migration.py
   ```
2. Verify migration:
   ```bash
   python scripts/run_migration.py --verify
   ```

---

### Issue: "Unknown Document" in citations

**Symptoms:**
- Citation shows "Unknown Document" instead of filename
- Raw file ID showing (e.g., "files/abc123")

**Cause:** File ID not in `file_search_name_map.json`

**Solution:**

1. Check backend logs for:
   ```
   Name map miss: 'abc123' not in map
   ```

2. Add mapping to `backend/file_search_name_map.json`:
   ```json
   {
     "abc123": "New_Document_Name.md"
   }
   ```

3. Redeploy backend

---

### Issue: "Admin panel not loading data"

**Symptoms:**
- Analytics tab shows no data
- Users tab empty
- API errors in console

**Possible Causes:**
1. Not logged in as admin
2. Session expired
3. Backend error

**Solutions:**

1. **Check user role:**
   - Admin panel requires `admin` or `manager` role
   - Check in console: user object should have `role: "admin"`

2. **Refresh token:**
   - Logout and login again
   - JWT tokens expire after 30 days

3. **Check API response:**
   ```javascript
   // In browser console
   await fetch('/api/admin/stats', {
     headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
   }).then(r => r.json())
   ```

---

### Issue: "CORS errors"

**Symptoms:**
- Console shows "Access-Control-Allow-Origin" errors
- API calls failing from frontend

**Cause:** CORS misconfiguration in backend

**Solution:**

Check `main.py` CORS settings:
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

### Issue: "Dark mode not persisting"

**Symptoms:**
- Theme resets to light on refresh
- System preference not detected

**Solution:**

1. Check localStorage:
   ```javascript
   localStorage.getItem('theme')
   // Should be 'light', 'dark', or 'system'
   ```

2. Clear and reset:
   ```javascript
   localStorage.setItem('theme', 'dark')
   window.location.reload()
   ```

---

### Issue: "Export not working"

**Symptoms:**
- Download button does nothing
- File downloads empty

**Cause:** Conversation has no messages or is archived

**Solution:**

1. Ensure conversation has messages
2. Try from an active conversation
3. Check browser download settings

---

## Database Issues

### Reset Database (Development Only)

```bash
cd backend
python scripts/setup_db.py --reset
```

### Check Database Connection

```bash
# From Cloud Run (with SQL proxy)
psql $DATABASE_URL -c "SELECT COUNT(*) FROM users;"
```

### Run Migration

```bash
cd backend
python scripts/run_migration.py
python scripts/run_migration.py --verify
```

### Rollback Migration

```bash
python scripts/run_migration.py --rollback
```

---

## Performance Issues

### Slow Response Times

**Symptoms:**
- Responses take >10 seconds
- Timeout errors

**Solutions:**

1. **Check Gemini latency:**
   - Look at `latency_ms` in run_logs table
   - Normal: 2000-5000ms
   - Slow: >10000ms

2. **Check cache hit rate:**
   - Look for "Cache hit for message" in logs
   - Repeated questions should be faster

3. **Reduce conversation history:**
   - System sends last 10 messages for context
   - Longer conversations = more tokens

### Memory Issues (Cloud Run)

**Symptoms:**
- Service restarting frequently
- "Out of memory" errors

**Solution:**
- Increase Cloud Run memory limit
- Current: 512MB recommended minimum

---

## Getting Help

1. **Check logs first:**
   - Cloud Run logs contain detailed error information
   - Look for error codes and stack traces

2. **Check this guide:**
   - Most common issues are documented here

3. **Contact support:**
   - nuDesk engineering team
   - Include: error message, timestamp, user email

---

## Error Code Reference

| Code | Meaning | Typical Solution |
|------|---------|------------------|
| `NO_FILE_SEARCH_STORE` | RAG not configured | Set GEMINI_FILE_SEARCH_STORE |
| `MODEL_NOT_FOUND` | Wrong model name | Fix GEMINI_MODEL |
| `RATE_LIMIT_EXCEEDED` | API quota hit | Wait or upgrade quota |
| `QUOTA_EXCEEDED` | Billing limit | Check GCP billing |
| `SERVICE_UNAVAILABLE` | Gemini down | Wait for recovery |
| `CONNECTION_ERROR` | Network issue | Check connectivity |
| `TIMEOUT` | Request too slow | Retry or reduce query size |
| `NO_RESPONSE` | Empty response | Retry with different query |
| `NO_CITATIONS` | No docs matched | Make query more specific |
| `INVALID_RESPONSE` | Malformed response | Contact support |
| `CONTENT_BLOCKED` | Safety filter | Rephrase query |
| `CHAT_FAILED` | Generic error | Check logs for details |
