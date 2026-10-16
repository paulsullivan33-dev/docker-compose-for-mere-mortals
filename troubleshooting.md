# Troubleshooting

**Scope:** the failures every Compose beginner hits,
symptom → cause → fix. Unless marked otherwise, each
error below was reproduced with Docker Compose v5.6.0
via `docker compose config` (no daemon needed for file
errors). Runtime errors are marked as documented rather
than reproduced — the build machine can't run the
daemon.

## `go-yaml load error in scanner ... mapping values are not allowed in this context`

```text
go-yaml load error in scanner at L4.C11: mapping values are not allowed in this context
```

**Cause:** YAML indentation is wrong — usually a key
indented to a level where the parser expects a value, or
mixed tabs and spaces. The `L4.C11` points at the spot.

**Fix:** count spaces (two per level, spaces only).
Reproduce-and-fix loop: `docker compose config` after
every edit.

## `service "web" has neither an image nor a build context specified: invalid compose project`

**Cause:** a service with neither `image:` nor `build:`.
Every service needs one — Compose has nothing to run
otherwise.

**Fix:** add `image:` (published image) or `build:`
(Dockerfile directory). See [Services](services.md).

## `validating compose.yaml: additional properties 'servicess' not allowed`

**Cause:** a typo'd key — here `servicess` instead of
`services`. Compose validates keys strictly; near-misses
don't guess.

**Fix:** fix the spelling. When the key looks right,
check the *level* — a service-level key accidentally
indented to top level produces the same error.

## `invalid containerPort: abc`

**Cause:** a malformed `ports:` entry. Compose couldn't
parse the container port.

**Fix:** use the quoted `"HOST:CONTAINER"` form —
`"8080:80"`, quotes included. See [Ports](ports.md).

## `the attribute 'version' is obsolete, it will be ignored`

```text
level=warning msg="compose.yaml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion"
```

**Cause:** a v1-era `version: "3.8"` line at the top,
copied from an old tutorial.

**Fix:** delete the line. Modern Compose ignores it.

## `The "APP_MODE" variable is not set. Defaulting to a blank string.`

**Cause:** `${APP_MODE}` interpolation with nothing
setting it — no shell export, no `.env` entry.

**Fix:** decide if it's optional (then use
`${APP_MODE:-default}`) or required (then use
`${APP_MODE:?message}` to fail fast instead of running
with an empty value). See [Environment
Variables](environment-variables.md).

## A service never starts when using `condition: service_healthy`

No error at all — `config` accepts the file silently
(verified), and the dependent just never starts.

**Cause:** the dependency defines no `healthcheck`, so it
never reports healthy.

**Fix:** add a `healthcheck` to the dependency, with a
generous `start_period` for slow starters. See
[Depends-On and Healthchecks](depends-on-and-healthchecks.md).

## `port is already allocated` *(documented, not reproduced here)*

**Cause:** two services (or a service and a host process)
publishing the same host port.

**Fix:** only the *published* (left-hand) port must be
unique. Change one service's host port, or find the
squatter (`ss -tlnp | grep 8080` on Linux).

## Container exits immediately; `ps` shows `Exit (1)` *(documented)*

**Cause:** the app crashed on startup — bad config,
missing env var, can't reach the database.

**Fix:** `docker compose logs <service>` — the crash
message is almost always in the first lines.
`docker compose up` without `-d` shows it live.

## `permission denied` writing to a bind mount *(documented)*

**Cause:** UID mismatch — the container's user doesn't
own your host files.

**Fix:** set `user: "1000:1000"` (your UID) on the
service, use a named volume for app-owned data, or
`chown` the host directory. See [Volumes and Bind
Mounts](volumes-and-bind-mounts.md).

## `down -v` deleted my database *(documented — and preventable)*

**Cause:** `-v` removes named volumes. That's its job.

**Fix:** there is no undo — restore from backup. Going
forward: plain `down` for daily use; `-v` only for
deliberate fresh starts. See [Updating and
Recreating](updating-and-recreating.md).

**Next:** [Glossary](glossary.md)
