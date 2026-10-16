# Your First Compose File

Make a directory, put one file in it, run one command.
That's the whole ceremony.

## The file

Create `compose.yaml` (Compose also accepts
`docker-compose.yaml` / `docker-compose.yml` — `compose.yaml`
is the modern default):

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

Three things to notice:

1. **`services:`** is the top-level key. Everything you run
   is a *service* listed under it.
2. **`web:`** is the service's name — you choose it. Other
   services will reach this one at the hostname `web`.
3. **Indentation is structure.** Two spaces per level, always
   spaces, never tabs. YAML has no braces; the indent *is*
   the nesting.

## YAML survival skills

You only need five YAML habits for Compose:

| Habit | Example |
|---|---|
| Key–value pairs | `image: nginx:alpine` |
| Nested maps indent two spaces | `ports:` then indented `- "8080:80"` |
| Lists start with `- ` | `- "8080:80"` |
| Quote strings with colons or leading zeros | `"8080:80"` — unquoted, YAML may misread it |
| `#` starts a comment | `# the web frontend` |

The single most common Compose failure is indentation.
When `config` complains about a `mapping values` error,
count spaces — see [Troubleshooting](troubleshooting.md).

## Validate before you run

Compose can render and check the file without touching
Docker at all:

```bash
docker compose config --quiet && echo "valid"
```

**Check (verified with Compose v5.6.0):**

```text
valid
```

Drop `--quiet` to see the fully-resolved file — Compose
shows you the defaults it filled in (like the network it
will create):

```yaml
name: first
services:
  web:
    image: nginx:alpine
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "8080"
        protocol: tcp
networks:
  default:
    name: first_default
```

Notice `first_default`: the project name comes from the
directory, and Compose created a network for it. You didn't
ask for a network — you get one anyway
([Networks](networks.md)).

## Run it

```bash
docker compose up -d
docker compose ps
```

`-d` means detached (run in the background). `ps` shows
your project's containers. Then open
`http://localhost:8080` — that's nginx, the `web` service,
with its port 80 published as your port 8080
([Ports](ports.md)).

Bring it back down:

```bash
docker compose down
```

**Next:** [Services](services.md)
