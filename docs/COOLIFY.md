# Deploying on Coolify

Uses `docker-compose.coolify.yml`: for setups where Coolify runs the
containers but **your own Nginx Proxy Manager** (not Coolify's Traefik)
handles the domain and TLS. `web` joins the external `coolify` Docker
network (which NPM can also reach) as `spool-web`; no host ports are
published. Config comes from Coolify's environment variables, not a `.env`.

1. **New Resource -> Docker Compose**, pick this repository/branch, and set
   **Docker Compose Location** to `/docker-compose.coolify.yml`.
2. Leave the `web` service's **Domain** empty so Coolify's Traefik doesn't
   try to route it. If your Coolify network isn't named `coolify`, change
   `networks.coolify.name` in the compose file (`docker network ls`).
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
   NPM serves over HTTPS.
4. In NPM add a Proxy Host for `spool.example.com`: Scheme `http`, Forward
   Hostname `spool-web`, Forward Port `8000`, enable SSL (Let's Encrypt)
   and Force SSL. NPM must be attached to the same `coolify` network
   (`docker network connect coolify <npm-container>` if not).
5. Deploy. Postgres, Redis and uploaded media live in named volumes and
   survive redeploys. Back up `db_data`.

Notes:
- The first entry in `DJANGO_ALLOWED_HOSTS` is also used as the `Host`
  header for the web healthcheck, so keep your real domain first.
- Don't enable `SECURE_SSL_REDIRECT` unless you've confirmed it doesn't
  redirect-loop behind NPM.
- Trakt/Simkl redirect URIs should use your `https://` domain
  (see [IMPORTING.md](IMPORTING.md)).
