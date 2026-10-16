# A Real Multi-Service Example

Time to put the chapters together: a small stack with a
web frontend, a database, and a cache — plus an optional
database admin tool. Every concept from the guide appears
exactly once, doing the job it was made for.

## The file

`compose.yaml`:

```yaml
services:
  db:
    image: postgres:16
    restart: unless-stopped
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: ${DB_PASSWORD:?set DB_PASSWORD before starting}
      POSTGRES_DB: appdb
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  cache:
    image: redis:7-alpine
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  web:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "${WEB_PORT:-8080}:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_healthy

  dbadmin:
    image: adminer:latest
    profiles: [tools]
    ports:
      - "127.0.0.1:8081:8080"
    depends_on:
      db:
        condition: service_healthy

volumes:
  pgdata:
```

And `html/index.html` next to it (bind-mounted
read-only into nginx):

```html
<!DOCTYPE html>
<html><head><title>Our stack is up</title></head>
<body><h1>web + db + cache, all from one file</h1></body></html>
```

## Reading it back, concept by concept

| Concept | Where it appears |
|---|---|
| Services with pinned images | `postgres:16`, `redis:7-alpine`, `nginx:alpine` |
| Named volume for data | `pgdata` keeps the database across restarts |
| Bind mount for your files | `./html` — edit the page, reload, no rebuild |
| Env interpolation + fail-fast secret | `${DB_PASSWORD:?...}` refuses to start without it |
| Healthchecks + readiness | `web` waits for *healthy* db and cache, not just started |
| Port publishing with a default | `${WEB_PORT:-8080}:80` |
| Profiles | `dbadmin` only starts with `--profile tools` |
| Localhost-only admin port | `127.0.0.1:8081:8080` — not exposed to the LAN |

## Running it

```bash
export DB_PASSWORD="pick-something-long"
docker compose config --quiet && echo "valid"   # always check first
docker compose up -d
docker compose ps
```

**Check (verified with Compose v5.6.0):**

```bash
docker compose config --services
```

```text
cache
db
web
```

The admin tool stays off until asked:

```bash
docker compose --profile tools up -d   # now dbadmin starts too
```

Then: `http://localhost:8080` serves your page, and
`http://localhost:8081` opens Adminer (log in with user
`app`, password `$DB_PASSWORD`, server `db` — the
service name, per [Networks](networks.md)).

Tear it down:

```bash
docker compose down        # containers + networks gone; pgdata kept
```

## Swapping in your real app

This example uses nginx as a stand-in frontend so every
piece runs from public images. When you have a real app,
`web` becomes either `image: your-registry/your-app:1.2`
or a `build:` section pointing at your Dockerfile — the
rest of the file (database, cache, volumes, healthchecks)
doesn't change. That's the point of the exercise: the
stack shape is stable while the pieces evolve.

**Next:** [Updating and Recreating](updating-and-recreating.md)
