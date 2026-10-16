# Depends-On and Healthchecks

Your web app needs the database *ready*, not just
*started*. Compose's `depends_on` handles the easy half
of that. This chapter is about the other half.

## `depends_on`: start order, nothing more

```yaml
services:
  web:
    image: myapp:latest
    depends_on:
      - db
```

This starts `db` before `web`. That's *all* it promises.
The database container existing is not the database
accepting connections — postgres takes seconds to
initialize, and your app will crash into "connection
refused" if it doesn't retry. (Well-written apps retry
their DB connection on startup; that's the real fix, and
it protects you everywhere, not just in Compose.)

## `condition: service_healthy`: order that means something

Pair `depends_on` with a **healthcheck** — a command
Compose runs inside the container to ask "are you
actually ready?":

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  web:
    image: myapp:latest
    depends_on:
      db:
        condition: service_healthy
```

Now `web` waits until postgres passes `pg_isready`.
The healthcheck fields:

| Field | Meaning |
|---|---|
| `test` | The probe. `CMD-SHELL` runs it in a shell; `CMD` runs it directly (no shell). |
| `interval` | How often to probe (default 30s). |
| `timeout` | Give up on one probe after this (default 30s). |
| `retries` | Consecutive failures before "unhealthy" (default 3). |
| `start_period` | Grace period while the container boots — failures here don't count. Give databases a generous one. |

Common probes: `pg_isready` for postgres, `redis-cli
ping` for redis (expect `PONG`), `curl -f
http://localhost:8080/health` for web apps (needs curl in
the image).

## The silent-hang trap

`condition: service_healthy` against a service with **no**
healthcheck never becomes true — Compose prints no
warning (verified with v5.6.0: `config` accepts it
silently) and the dependent simply never starts. If a
service hangs at startup, check that every dependency it
waits on actually defines a healthcheck.

## `depends_on` conditions, complete

| Condition | Waits for... |
|---|---|
| `service_started` (default) | The container exists and started. |
| `service_healthy` | Healthcheck passing. |
| `service_completed_successfully` | A one-shot container (migration, seed) that exited 0. |

The last one is the pattern for startup tasks:

```yaml
services:
  migrate:
    image: myapp:latest
    command: ["./migrate.sh"]
  web:
    image: myapp:latest
    depends_on:
      migrate:
        condition: service_completed_successfully
```

Migrations run, then the app starts. If the migration
fails, the app never starts — which is what you want.

**Next:** [Ports](ports.md)
