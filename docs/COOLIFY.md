# Deploying on Coolify

Uses `docker-compose.coolify.yml` (no host ports, no external proxy
network, config via Coolify's environment variables instead of a `.env`
file).

1. **New Resource -> Docker Compose**, pick this repository/branch, and set
   **Docker Compose Location** to `/docker-compose.coolify.yml`.
2. In the `web` service settings, set the **Domain** to
   `https://spool.example.com:8000` (the `:8000` tells Coolify which
   container port to route to; it is not part of the public URL).
3. In **Environment Variables**, set:

   | Variable | Value |
   |---|---|
   | `DJANGO_SECRET_KEY` | long random string (`openssl rand -base64 48`) |
   | `DJANGO_ALLOWED_HOSTS` | `spool.example.com` |
   | `DJANGO_CSRF_TRUSTED_ORIGINS` | `https://spool.example.com` |
   | `DB_PASSWORD` | strong password |
   | `ADMIN_USERNAME` / `ADMIN_PASSWORD` | first-run admin (only used if no profile exists) |
   | `TIME_ZONE` | e.g. `Europe/Ljubljana` |

   Optional: `TMDB_API_KEY`, `MDBLIST_API_KEY`, `TRAKT_CLIENT_ID/SECRET`,
   `SIMKL_CLIENT_ID/SECRET`, `GUNICORN_WORKERS`, `GUNICORN_THREADS`,
   `CELERY_WORKER_CONCURRENCY`. Secure cookies default to on because
   Coolify serves over HTTPS.
4. Deploy. Postgres, Redis and uploaded media live in named volumes and
   survive redeploys. Back up `db_data`.

Notes:
- The first entry in `DJANGO_ALLOWED_HOSTS` is also used as the `Host`
  header for the web healthcheck, so keep your real domain first.
- Don't enable `SECURE_SSL_REDIRECT` unless you've confirmed it doesn't
  redirect-loop behind Coolify's proxy.
- Trakt/Simkl redirect URIs should use your `https://` domain
  (see [IMPORTING.md](IMPORTING.md)).
