# Updating and Recreating

Stacks change: new image versions, edited compose files,
rotated passwords. Compose's model is **recreate, don't
mutate** — it replaces containers from the current file.
Your data lives in volumes, which recreation doesn't
touch. Understanding that split is the whole chapter.

## Updating images

```bash
docker compose pull          # fetch the newest of each pinned tag
docker compose up -d         # recreate anything whose image changed
```

`up` compares what's running to what the file (and fresh
images) describe, and recreates only what's stale. This
is why pinned tags matter ([Services](services.md)): with
`postgres:16`, `pull` gets the latest 16.x patch — a
safe update. With `postgres` (floating `latest`), `pull`
can jump major versions overnight.

## Changing the compose file

Edit the file, then `up -d` again. Compose diffs the
running containers against the new file and recreates
what changed. Added an environment variable? The
container gets recreated with it. Changed a port? Same.
This is the normal development loop:

```bash
# edit compose.yaml
docker compose config --quiet && docker compose up -d
```

Validate first, recreate second. (`config` needs no
daemon — there's no reason to skip it.)

## Forcing a fresh container

```bash
docker compose up -d --force-recreate   # recreate even if nothing changed
docker compose up -d --build           # rebuild images, then recreate
```

`--force-recreate` is for "I suspect the container is in
a weird state." `--build` is for after you edited a
Dockerfile.

## The data-safety split

| Command | Containers | Networks | Named volumes | Images |
|---|---|---|---|---|
| `down` | removed | removed | **kept** | kept |
| `down -v` | removed | removed | **removed** | kept |
| `down --rmi local` | removed | removed | kept | removed (locally built) |

Plain `down` is safe: `up` later finds your database
intact. `down -v` is the "fresh start" button — it
deletes named volumes, i.e. your data. Type it only when
you mean it, and never from muscle memory.

**Backups are still your job.** Volumes surviving `down`
is convenience, not a backup strategy. If the data
matters, back up the volume (`docker run --volumes-from`
or a bind-mounted backup sidecar) on a schedule.

## One-shot updates without downtime (roughly)

For a single service: `docker compose pull web &&
docker compose up -d web`. Only `web` recreates; the
database never blinks. True zero-downtime deploys
(health-gated rolling restarts) are orchestrator
territory — Compose restarts the container, and there's
a gap. For personal stacks, the gap is seconds.

**Next:** [Profiles](profiles.md)
