# Environment Variables

Containers get their configuration from environment
variables — database passwords, ports, feature flags.
Compose gives you three ways to set them, and the
differences matter.

## 1. `environment:` — right in the file

```yaml
services:
  web:
    image: nginx:alpine
    environment:
      APP_ENV: production
      DEBUG: "false"
```

Fine for non-secret settings. Two forms exist — the map
above, and a list (`- APP_ENV=production`). The map is
easier to read; use it.

**Do not put secrets here** if the file goes into git
(which it should). Passwords and API keys committed to a
repo are compromised the moment they're pushed — even to
a private repo you later make public, even in history
after you "remove" them.

## 2. `env_file:` — a separate file per service

```yaml
services:
  web:
    image: nginx:alpine
    env_file:
      - app.env
```

`app.env` is plain `KEY=value` lines:

```text
LOG_LEVEL=info
WORKERS=2
```

Use `env_file` for non-secret service settings you want
out of the compose file. For actual secrets, the file must
be git-ignored (add it to `.gitignore`) and created on
each machine — or better, see option 3.

## 3. `.env` + interpolation — the flexible one

A file named exactly `.env`, sitting next to
`compose.yaml`, is loaded automatically. Its values can
be *interpolated* into the compose file with `${VAR}`:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "${WEB_PORT:-8080}:80"
    environment:
      APP_ENV: ${APP_ENV}
      DB_PASSWORD: ${DB_PASSWORD:?a value is required here}
```

With `.env` containing:

```text
WEB_PORT=8080
APP_ENV=production
```

Rendered (verified with Compose v5.6.0):

```yaml
environment:
  APP_ENV: production
  DEBUG: "false"
  LOG_LEVEL: info
  WORKERS: "2"
```

Three interpolation tricks:

- **`${VAR:-default}`** — use `default` when unset. Ports
  and other "usually this" values.
- **`${VAR:?message}`** — fail fast with `message` when
  unset. Use for secrets: `error while interpolating
  services.web.environment.DB_PASSWORD: a value is
  required here` beats a container crashing at 2 AM
  because the password was empty.
- **Plain `${VAR}`** — unset means Compose warns (`The
  "APP_MODE" variable is not set. Defaulting to a blank
  string.`) and substitutes empty. Fine for optional
  things, dangerous for required ones.

## Precedence: who wins

When the same variable is set in several places:

1. `docker compose run -e KEY=...` (command line)
2. Shell environment (`export KEY=...` before `up`)
3. `environment:` in the compose file
4. `env_file:`
5. `.env` file

So the shell beats `.env` — handy for secrets: keep
`DB_PASSWORD` out of every file and `export` it (or use
your shell's env management) before `up`.

## The secrets rule

Short version: **compose files are documentation; don't
document your passwords.** Passwords and keys arrive via
the shell, a git-ignored file, or a secrets manager —
never hardcoded under `environment:` in a committed file.

**Next:** [Depends-On and
Healthchecks](depends-on-and-healthchecks.md)
