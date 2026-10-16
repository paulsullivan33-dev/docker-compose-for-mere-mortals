# Networks

Compose creates a network for your project automatically,
puts every service on it, and gives each service a DNS
name. Most of the time, that's the whole story — this
chapter is about the minority of times it isn't.

## The default network

With no `networks:` keys anywhere, `docker compose up`
creates one bridge network named `<project>_default` (you
saw `first_default` in [Your First Compose
File](your-first-compose-file.md)). Every service joins
it.

On that network, **services reach each other by service
name**. If your file has a service called `db`, then from
the `web` container, `db` resolves to the database
container's IP:

```yaml
services:
  web:
    image: myapp:latest
    environment:
      DATABASE_URL: postgres://app:secret@db:5432/appdb
```

`db:5432` — that's the service name as a hostname, plus
the port *inside* the container. No published port needed
for service-to-service traffic; `ports:` is only for
reaching containers from *outside* ([Ports](ports.md)).

This replaces the old `--link` flag entirely. If a
tutorial tells you to add `links:`, it's outdated — delete
it.

## Custom networks

Declare networks at the top level, attach services to
them. A service on two networks can talk on both; two
services with no shared network can't talk at all — which
is the point:

```yaml
services:
  web:
    image: nginx:alpine
    networks:
      - frontend
      - backend

  db:
    image: postgres:16
    networks:
      - backend

networks:
  frontend:
  backend:
    name: myapp-backend
```

Here `web` talks to the world (`frontend`) and to the
database (`backend`), but nothing on `frontend` can reach
`db` directly. That's a real, useful security boundary for
a few lines of YAML.

Rendered (verified with Compose v5.6.0), note the naming:
undeclared `name:` gets the project prefix, an explicit
`name:` is used as-is:

```yaml
networks:
  backend:
    name: myapp-backend
    driver: bridge
  frontend:
    name: networks_frontend
```

## When you need the options

| Situation | What to do |
|---|---|
| Talk to a container *outside* Compose | Put both on an external network: `networks: {proxy: {external: true}}`, then attach the service to `proxy`. Compose won't create or delete it. |
| Fixed IPs | `ipv4_address` under the service's network entry — rarely needed; service-name DNS covers nearly everything. |
| No network at all | `network_mode: none` on the service, or `network_mode: "host"` to share the host's network (Linux only; skips port publishing entirely). |

**Rule of thumb:** if you're reaching for network
options beyond a second named network, pause and check
whether service-name DNS already solves it. It usually
does.

**Next:** [Volumes and Bind Mounts](volumes-and-bind-mounts.md)
