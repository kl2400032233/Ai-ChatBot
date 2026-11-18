# Lumora — updated project

This folder is a clean copy of the Lumora RAG chatbot with the sign-in error
handling fixes applied. Build artifacts, virtual environments, installed packages,
and local secrets are intentionally excluded.

## What was fixed

The deployed site showed `Failed to fetch` when signing in because the configured
Supabase project hostname briefly stopped resolving in DNS. The project is
reachable again, and the client now handles this situation better:

- `frontend/src/AuthView.jsx` retries sign-in once on a transient network failure
  and shows an actionable message instead of `Failed to fetch`. It also explains
  invalid credentials, unconfirmed email, rate limits, and weak passwords.
- `frontend/src/api.js` reports which API URL could not be reached.
- `frontend/src/App.jsx` shows a clear "Service unreachable" message when the API
  or auth service is offline.
- `README.md` includes a "Troubleshooting Failed to fetch on sign-in" section with
  DNS and health-check commands.

## Run locally

Backend (from this folder):

```bash
python -m venv .venv
.venv\Scripts\activate          # macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
copy .env.example .env          # macOS/Linux: cp .env.example .env
ollama pull llama3.2:3b
uvicorn api:app --reload --host 127.0.0.1 --port 8000
```

Frontend (second terminal):

```bash
cd frontend
npm install
npm run dev
```

Open http://127.0.0.1:5173.

This build verifies every API request against Supabase, so a Supabase project is
required even for local use. Add the same four values to the backend `.env` and
to `frontend/.env`:

```text
# .env (backend)
SUPABASE_URL=https://<project-ref>.supabase.co
SUPABASE_ANON_KEY=<publishable-or-anon-key>

# frontend/.env
VITE_API_BASE_URL=http://127.0.0.1:8000
VITE_SUPABASE_URL=https://<project-ref>.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=<publishable-or-anon-key>
```

Then restart both processes. Without these values the UI stops at the sign-in
setup screen and the API returns `503 Supabase Auth is not configured`.

## Hosted deployment

The production build reads these values at build time:

```text
frontend (GitHub Actions secrets)
  VITE_API_BASE_URL=https://lumen-rag-api-rsmi.onrender.com
  VITE_SUPABASE_URL=https://<project-ref>.supabase.co
  VITE_SUPABASE_PUBLISHABLE_KEY=<publishable-or-anon-key>

backend (Render environment)
  SUPABASE_URL=https://<project-ref>.supabase.co
  SUPABASE_ANON_KEY=<publishable-or-anon-key>
```

Never commit a Supabase service-role key or a `.env` file.

## If "Failed to fetch" comes back

```bash
nslookup <project-ref>.supabase.co 8.8.8.8
curl https://<project-ref>.supabase.co/auth/v1/health
```

`NXDOMAIN` means the Supabase project is paused, restoring, or missing its DNS
record. Restore it in the Supabase dashboard, wait for a healthy status, then
reload the site (and flush DNS locally if needed).
