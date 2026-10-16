# Glossary

**Bind mount** — a container path mapped to a path on
your machine (`./html:/usr/share/nginx/html`). You edit
the files; the container reads them. See [Volumes and
Bind Mounts](volumes-and-bind-mounts.md).

**Build context** — the directory sent to Docker when
building an image (`build: .`). Keep it small; a
`.dockerignore` file excludes the junk.

**Compose file** — the YAML file (`compose.yaml`)
describing your stack. The thing this whole guide is
about.

**Container** — a running instance of an image. The
process, isolated.

**depends_on** — declares start order between services.
Order only — not readiness (that's what healthchecks
are for). See [Depends-On and
Healthchecks](depends-on-and-healthchecks.md).

**Environment variable** — a `KEY=value` setting passed
into a container. Configuration without rebuilding.
See [Environment Variables](environment-variables.md).

**Healthcheck** — a command Compose runs inside a
container to decide if it's actually ready
(`pg_isready`, `curl -f .../health`). Makes
`depends_on` meaningful.

**Image** — the frozen template a container starts from
(`postgres:16`). Built from a Dockerfile or pulled from
a registry.

**Interpolation** — `${VAR}` in the compose file,
filled in from the shell or `.env` at load time. With
`:-default` and `:?error` modifiers.

**Named volume** — Docker-managed persistent storage
(`pgdata:/var/lib/postgresql/data`), declared in the
top-level `volumes:` block. Survives `down`; deleted by
`down -v`.

**Network** — virtual LAN for your project's containers.
Compose makes one per project; services find each other
by service name. See [Networks](networks.md).

**Orchestration** — running containers across many
machines (Kubernetes, Swarm). Compose is not this;
Compose is one machine, well.

**Port publishing** — mapping a container port to a host
port (`"8080:80"`) so the outside world can reach in.
See [Ports](ports.md).

**Profile** — an opt-in label on a service
(`profiles: [tools]`). Profiled services only start
with `--profile` or `COMPOSE_PROFILES`. See
[Profiles](profiles.md).

**Project** — everything in one compose file, managed
as a unit. Named after the directory by default
(override with `--project-name` / `COMPOSE_PROJECT_NAME`).

**Recreate** — Compose's update mechanism: stop the old
container, start a new one from the current file/image.
Data in volumes is untouched.

**Restart policy** — what happens when a container exits
(`unless-stopped`, `on-failure`, `always`, `"no"`).
See [Services](services.md).

**Service** — one container's recipe in the compose
file: image/build, ports, volumes, environment, and
friends. The hostname other services use to reach it.

**YAML** — the indentation-based format of the compose
file. Two spaces per level, never tabs, quote your
port mappings.
