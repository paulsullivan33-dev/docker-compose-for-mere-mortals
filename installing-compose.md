# Installing Compose

Compose is a **plugin** for the Docker CLI — not a separate
program. The command is `docker compose` (a space). If you
have a current Docker installation, you may already have it.

## The check

```bash
docker compose version
```

**Check:** you want output like this (exact numbers move
with releases):

```text
Docker Compose version v5.6.0
```

If instead you see `unknown command: docker compose`,
the plugin is missing — install it per your platform below.

## Linux

On Debian/Ubuntu with Docker's official repository:

```bash
sudo apt update
sudo apt install docker-compose-plugin
docker compose version
```

On Fedora/RHEL-likes:

```bash
sudo dnf install docker-compose-plugin
```

If your distro's Docker package doesn't ship the plugin,
download the binary from the [Compose
releases](https://github.com/docker/compose/releases) page
into `~/.docker/cli-plugins/` and make it executable:

```bash
mkdir -p ~/.docker/cli-plugins
curl -SL -o ~/.docker/cli-plugins/docker-compose \
  https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64
chmod +x ~/.docker/cli-plugins/docker-compose
docker compose version
```

The CLI picks up any executable named `docker-compose` in
that directory — no root needed for this route.

## macOS and Windows

Docker Desktop includes the Compose plugin out of the box.
Install (or update) Docker Desktop and `docker compose
version` just works. No separate step.

## A note on `docker-compose` (hyphen)

Old guides invoke `docker-compose` as its own command.
That's v1, long retired. If a tutorial tells you to
`pip install docker-compose`, it's outdated — use the
plugin form. The compose *file* concepts transfer; the
command doesn't.

**Next:** [Your First Compose File](your-first-compose-file.md)
