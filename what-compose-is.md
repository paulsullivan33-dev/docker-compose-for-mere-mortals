# What Compose Is

The Docker guide taught you to run one container:

```bash
docker run -d --name web -p 8080:80 \
  -v webdata:/usr/share/nginx/html \
  --network myapp \
  nginx:alpine
```

That works. Now imagine your app needs three containers — a
web server, a database, and a cache — each with its own
flags, volumes, and environment variables. The `docker run`
lines get long, the order matters, and six months from now
you will not remember the exact incantation. Teams solve
this with a wiki page. The wiki page goes stale.

**Compose replaces the wiki page with a file.** You describe
the whole stack — every container, its settings, how they
connect — in one YAML file, usually named `compose.yaml`.
Then:

```bash
docker compose up -d     # start everything
docker compose down      # stop everything
```

The file *is* the documentation, and unlike the wiki page,
it can't go stale: it's the thing that actually runs.

## What Compose does for you

Given a file like this (the full version lives in [A Real
Multi-Service Example](real-example.md)):

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - pgdata:/var/lib/postgresql/data

  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - db

volumes:
  pgdata:
```

Compose will:

- **Create a network** for the project, so `web` reaches the
  database at the hostname `db` — no IP addresses, no
  `--network` flags ([Networks](networks.md)).
- **Start things in order** — `db` before `web`
  ([Depends-On and Healthchecks](depends-on-and-healthchecks.md)).
- **Keep names tidy** — containers get project-prefixed
  names like `myapp-web-1`, so two projects never collide.
- **Treat the stack as one unit** — `up`, `down`, `logs`,
  `ps` all operate on the whole file
  ([The Everyday Commands](the-everyday-commands.md)).

## What Compose is not

- **Not a cluster manager.** Compose runs on *one*
  machine. When you need many machines, auto-scaling, and
  self-healing, that's Kubernetes or Swarm — separate
  subjects.
- **Not a Dockerfile replacement.** Compose doesn't build
  images from scratch; it *uses* images (and can trigger
  builds — see [Services](services.md)).
- **Not magic.** Every line in a compose file maps to
  something `docker run` could do. Compose is a better
  interface to the same engine, not a different engine.

## Compose v2 vs the old `docker-compose`

You may see old tutorials using `docker-compose` (with a
hyphen) as a separate Python program. That was Compose v1,
retired years ago. Modern Compose is a plugin: `docker
compose` (a space), written in Go, and it ships with Docker
Desktop and current Linux packages. The file format is the
same idea, but v1-era files starting with `version: "3.8"`
will print an "obsolete" warning today — the `version` key
is ignored and should be deleted ([Troubleshooting](troubleshooting.md)).

**Next:** [Installing Compose](installing-compose.md)
