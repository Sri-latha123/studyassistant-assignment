# NetMentor

NetMentor is an adaptive Computer Networks tutor for VTU students, GATE aspirants, and interview candidates. It combines a model-backed tutoring policy with a small web app, persisted conversations, session exports, and a mobile-friendly interface.

## Run locally

Requirements: Python 3.11+ and an OpenAI API key.

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
# Add OPENAI_API_KEY to .env, then:
uvicorn app.main:app --reload
```

Open http://127.0.0.1:8000. The key is read only by the server. Without it, the interface remains usable for setup, but model-backed chat reports a clear configuration message.

## Docker

```sh
docker build -t netmentor .
docker run --rm -p 8000:8000 -e OPENAI_API_KEY=your-key -v netmentor-data:/app/data netmentor
```

Set `OPENAI_MODEL`, `DATABASE_PATH`, and `APP_ORIGINS` as environment variables when deploying. Keep `.env` out of version control and terminate TLS at a trusted reverse proxy. Mount `/app/data` persistently when using SQLite. For multiple application instances, replace SQLite with a managed database and move per-session locks to a shared coordination layer.

## Product behavior

- New sessions ask for exam goal, level, VTU scheme/semester, time, and topic.
- Tutoring is scoped to Computer Networks and includes a diagnostic path, hint-first practice, mistake tagging, adaptive difficulty, revision, scorecards, and Save Code exports.
- Session history persists in SQLite and is restored on the same browser. Export contains the session profile, progress state, and conversation; students can delete it by calling `DELETE /api/sessions/{id}`.
- The tutor receives a bounded recent conversation window. The system prompt tells it to treat student content as untrusted and not to provide offensive security instructions.

## Production notes

This is a deployable single-instance foundation, not a multi-tenant hosted service. Before exposing it to the public, add authentication/authorization, per-user session ownership, request rate limiting, retention/deletion policies, observability, backups, and a managed database. The current opaque session IDs are unguessable but are not an authentication system. Avoid logging prompts or API keys. Provider calls use `store=False`; local session data remains until deleted or the database volume is removed.

## Health

`GET /api/health` returns service availability and whether model credentials are configured. It does not make a model call.
