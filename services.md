# Services

A **service** is one container's recipe: which image (or
build), what name, what command, and what should happen
when it crashes. If Compose has a "main idea," services
are it.

## `image` vs `build`

Every service needs one of these — Compose errors out if
it has neither (see [Troubleshooting](troubleshooting.md)).

```yaml
services:
  web:
    image: nginx:alpine        # use a published image as-is

  worker:
    build: .                  # build from the Dockerfile in this directory
```

- **`image:`** pulls a published image. Pin a tag
  (`postgres:16`, not `postgres` floating on `latest`) so
  next month's `up` doesn't silently upgrade you
  ([Updating and Recreating](updating-and-recreating.md)).
- **`build:`** points at a directory containing a
  Dockerfile (shorthand for `build: { context: . }`). The
  long form lets you name the file and pass build args:

```yaml
  worker:
    build:
      context: .
      dockerfile: Dockerfile.worker
      args:
        APP_VERSION: "1.4"
```

You can combine them: `image:` names the built image, so
`build` + `image: myregistry/worker:1.4` builds *and*
tags it — handy before pushing to a registry.

Note: `docker compose config` validates the file even if
the Dockerfile doesn't exist yet — the build itself
happens at `up` time, not at validation time.

## Names

```yaml
services:
  web:
    image: nginx:alpine
    container_name: my-web
```

- The **service name** (`web`) is the hostname other
  services use to reach it ([Networks](networks.md)).
  Keep it short, lowercase, no spaces.
- **`container_name`** overrides the generated
  `project-web-1` name. Useful when outside tools expect a
  fixed name — but you lose the ability to scale that
  service past one container, since two containers can't
  share a name.

## `command` and `entrypoint`

Override what the container runs:

```yaml
services:
  web:
    image: nginx:alpine
    command: ["nginx", "-g", "daemon off;"]
```

`command` replaces the image's default command;
`entrypoint` replaces the executable that receives it.
Prefer the JSON list form (`["nginx", ...]`) over a bare
string — the string form gets wrapped in `sh -c`, which
changes signal handling and can make `docker compose stop`
slower to take effect.

## `restart`

What happens when the container exits — or when the
machine reboots:

| Value | Meaning |
|---|---|
| `"no"` (default) | Don't restart. Quote it — unquoted `no` is YAML for `false`. |
| `on-failure` | Restart only if it crashed (non-zero exit). |
| `unless-stopped` | Always restart, unless you stopped it by hand. The usual choice for things that should survive a reboot. |
| `always` | Always restart, even if you stopped it by hand. |

```yaml
services:
  db:
    image: postgres:16
    restart: unless-stopped
```

For one-shot tasks (migrations, seed scripts), the
default `"no"` is correct — you don't want a finished
migration restarting forever.

## A service, fully dressed

```yaml
services:
  web:
    image: nginx:alpine
    container_name: my-web
    restart: unless-stopped
    command: ["nginx", "-g", "daemon off;"]
    ports:
      - "8080:80"
    environment:
      NGINX_HOST: example.local
    volumes:
      - ./html:/usr/share/nginx/html:ro
    depends_on:
      - db
```

Each of those keys gets its own chapter from here on:
[Ports](ports.md), [Environment
Variables](environment-variables.md), [Volumes and Bind
Mounts](volumes-and-bind-mounts.md),
[Depends-On and Healthchecks](depends-on-and-healthchecks.md).

**Next:** [Networks](networks.md)
