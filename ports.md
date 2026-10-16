# Ports

Ports are how the outside world reaches your containers —
and the number-one source of "why is this port weird"
confusion. The rule is simple once you see it.

## The order: `"HOST:CONTAINER"`

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

Read it left to right: **your machine's port 8080 →
the container's port 80**. The container thinks it's
serving on 80 (its normal port); you visit
`http://localhost:8080`.

The quotes matter: unquoted `8080:80` can be misread by
YAML (it's not a valid number, not quite a string).
Always quote port mappings.

Rendered (verified with Compose v5.6.0) — Compose is
explicit about which end is which:

```yaml
ports:
  - mode: ingress
    target: 80
    published: "8080"
    protocol: tcp
```

`target` = inside the container. `published` = on your
machine. When in doubt, read the rendered output.

## Binding to localhost only

```yaml
ports:
  - "127.0.0.1:8443:443"
```

Three parts: `HOST_IP:HOST_PORT:CONTAINER_PORT`. This
publishes the port on localhost only — reachable from
your machine, invisible to the LAN. Use it for admin
interfaces and databases you don't want the whole
network poking at.

## `expose` vs `ports`

```yaml
expose:
  - "8080"
```

`expose` declares a port for *other services* (documentation
plus inter-container firewalling in some setups) — it
does **not** publish anything to your machine. If you can
reach it from your browser, that's `ports`. If only other
containers need it, you usually need neither: services on
the same network reach each other's container ports
directly ([Networks](networks.md)).

## The long form

For everything the short form can't say:

```yaml
ports:
  - target: 80
    published: 9090
    protocol: tcp
    mode: ingress
```

## The classic failure: port already allocated

Two services (or a service and something on your host)
can't publish the same host port. Compose fails at `up`
with `port is already allocated`. Fix: find who's holding
it (`ss -tlnp | grep 8080` on Linux, or just pick another
host port — only the *published* side needs to be unique;
ten containers can all use container-port 80).

Note: this one needs a running daemon to reproduce, so
the message above is quoted from the Compose reference
rather than a live run here.

**Next:** [The Everyday Commands](the-everyday-commands.md)
