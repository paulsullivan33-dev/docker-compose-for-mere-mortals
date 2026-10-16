# Profiles

Not every service should start every time. The database
admin UI, the debug database, the load-test harness —
these are *sometimes* services. **Profiles** make a
service opt-in.

## The basics

```yaml
services:
  web:
    image: nginx:alpine

  dbadmin:
    image: adminer:latest
    profiles: [tools]
```

`dbadmin` does not start with a plain `docker compose
up`. It starts when you activate its profile:

```bash
docker compose --profile tools up -d
```

**Check (verified with Compose v5.6.0):**

```bash
docker compose config --services              # default
```

```text
web
```

```bash
docker compose --profile tools config --services
```

```text
dbadmin
web
```

## Multiple profiles

A service can belong to several profiles — it starts when
*any* of them is active:

```yaml
services:
  debugdb:
    image: postgres:16
    profiles: [tools, debug]
```

```bash
docker compose --profile tools --profile debug up -d
```

And instead of flags, the `COMPOSE_PROFILES` environment
variable (comma-separated) activates profiles for every
command in that shell:

```bash
export COMPOSE_PROFILES=tools,debug
docker compose up -d
```

## What profiles are for

| Profile idea | Services in it |
|---|---|
| `tools` | Adminer, pgAdmin, a mail-catcher |
| `debug` | A debug build of your app, verbose logging sidecar |
| `test` | A throwaway test database seeded with fixtures |
| `monitoring` | Prometheus, Grafana |

The pattern: the base file describes the app; profiles
describe the *tooling around* the app. Someone cloning
the repo gets a working stack from `up -d`, and the
extras stay out of the way until asked for.

One caution: a profiled service's dependencies still
apply. If `dbadmin` has `depends_on: db`, activating the
`tools` profile starts `db` too — which is what you
want, but don't be surprised by it.

**Next:** [Troubleshooting](troubleshooting.md)
