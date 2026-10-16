# The Everyday Commands

You will use about eight Compose commands. Everything
else is for special occasions. (Lifecycle commands need
the Docker daemon running — the file-validation ones
don't.)

## The core loop

```bash
docker compose up -d        # create + start everything, detached
docker compose ps           # what's running in this project
docker compose logs -f      # follow all services' logs (Ctrl+C detaches)
docker compose down         # stop + remove containers and networks
```

That's 90% of Compose. A few notes:

- **`up` without `-d`** attaches to the logs in your
  terminal — great for watching a first start,
  Ctrl+C stops everything.
- **`ps`** shows only *this project's* containers (it
  knows by the project name), unlike `docker ps` which
  shows the whole machine.
- **`logs -f`** prefixes each line with the service name.
  Add a service name (`docker compose logs -f db`) to
  follow just one. `-n 100` shows the last 100 lines;
  `--since 10m` shows the last ten minutes.
- **`down` keeps your named volumes.** Your database
  survives a `down`. Only `down -v` deletes volumes —
  and you should type that `-v` deliberately, every time
  ([Updating and Recreating](updating-and-recreating.md)).

## Reaching into containers

```bash
docker compose exec web sh       # open a shell in the running "web" container
docker compose exec db psql -U postgres   # run one command in "db"
docker compose run --rm web sh   # like exec, but starts a NEW one-off container
```

`exec` = into the running one. `run` = a fresh throwaway
(`--rm` cleans it up after). Use `run` for one-shot tasks
against the project's network and volumes.

## Images: pull, build, config

```bash
docker compose pull         # download the newest of every image: tag
docker compose build        # (re)build every service with a build: section
docker compose up -d --build  # rebuild, then start — the common combo
docker compose config       # render + validate the file, no daemon needed
```

`config` is the odd one out: it's pure file processing,
which is why every sample in this guide was validated
with it. Run it after every edit — it catches typos
before they become runtime mysteries.

Modern Compose also has `--dry-run` on `up`/`down`
(`docker compose up --dry-run -d`): it prints what *would*
happen without doing it. Needs the daemon, but it's the
safest way to preview a scary command.

## Multiple files: `-f` and the override pattern

```bash
docker compose -f compose.yaml -f compose.prod.yaml up -d
```

Later files override earlier ones — same service, same
key, last file wins. The beloved pattern:

- `compose.yaml` — the base, works everywhere.
- `compose.override.yaml` — auto-loaded when present;
  dev-only bits (bind-mounted source code, published
  debug ports).
- `compose.prod.yaml` — applied explicitly with `-f`;
  production tweaks (tighter restarts, no debug ports).

Keep secrets and environment-specific settings in the
override files, not the base.

## Targeting services

Almost every command takes service names at the end:

```bash
docker compose up -d db        # start only db (and its dependencies)
docker compose logs -f web
docker compose restart web     # restart one service
docker compose stop web        # stop one service (keeps it: start brings it back)
```

`stop` vs `down`: `stop` halts the containers but keeps
them (and the project) around; `down` tears the project
down to volumes and images.

## Command cheat sheet

| I want to... | I run... |
|---|---|
| Start everything | `docker compose up -d` |
| Watch it start | `docker compose up` (no `-d`) |
| See what's running | `docker compose ps` |
| Read the logs | `docker compose logs -f [service]` |
| Shell into a container | `docker compose exec <service> sh` |
| Stop everything, keep data | `docker compose down` |
| Nuke everything incl. data | `docker compose down -v` |
| Update images | `docker compose pull && docker compose up -d` |
| Check my YAML | `docker compose config --quiet` |

**Next:** [A Real Multi-Service
Example](real-example.md)
