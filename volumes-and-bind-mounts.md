# Volumes and Bind Mounts

Containers are forgetful: delete one and everything it
wrote inside itself goes with it. Volumes are how data
survives. Compose gives you two flavors, and choosing the
right one is half the chapter.

## Named volumes — for data you keep

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

- The short form is `name:/container/path`.
- The top-level `volumes:` block **declares** the volume;
  without the declaration, Compose errors out.
- `docker compose down` keeps named volumes — your
  database survives. `docker compose down -v` deletes
  them. That `-v` deserves a pause every time
  ([Updating and Recreating](updating-and-recreating.md)).

Give the volume an explicit `name:` (as above:
`myapp-pgdata`) when you want it recognizable in `docker
volume ls`. Otherwise Compose prefixes the project name.

**Use named volumes for:** databases, uploaded files, any
state the application owns and you want to survive
restarts.

## Bind mounts — for files you edit

```yaml
services:
  web:
    image: nginx:alpine
    volumes:
      - ./html:/usr/share/nginx/html:ro
```

A bind mount maps a path on *your machine* into the
container. Edit `./html/index.html` in your editor, reload
the page — no rebuild. The `:ro` suffix mounts it
read-only, so the container can't modify your files.

**Use bind mounts for:** config files, source code you're
developing, seed scripts, anything where *you* are the
author and the container is the reader.

## The long form

When the short form gets cryptic, spell it out:

```yaml
volumes:
  - type: volume
    source: pgdata
    target: /var/lib/postgresql/data
  - type: bind
    source: ./html
    target: /usr/share/nginx/html
    read_only: true
```

Same mounts, no ambiguity about which is which.

## The permissions gotcha

This is the classic volumes headache: the container runs
as some user ID, your files belong to another, and writes
fail with `permission denied`. What's happening: bind
mounts keep *your* ownership — the container sees your
UIDs, not its own usernames.

Practical fixes, in order of preference:

1. **Check if the image documents a UID.** Many do
   (e.g. `user: "1000:1000"` on the service sets the
   container's user to yours).
2. **Named volume instead of bind mount** for
   application-owned data — Docker initializes the
   volume's ownership from the image.
3. **Fix ownership on the host** (`chown -R`) when the
   files are yours to own.

## Quick reference

| | Named volume | Bind mount |
|---|---|---|
| Declared | top-level `volumes:` | not declared — path is literal |
| Survives `down` | yes | n/a (it's your disk) |
| Survives `down -v` | **no** | yes |
| Good for | databases, app state | config, code, seeds |
| Read-only | `:ro` works | `:ro` works |

**Next:** [Environment Variables](environment-variables.md)
