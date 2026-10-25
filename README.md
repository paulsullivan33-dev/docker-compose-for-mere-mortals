# Docker Compose for Mere Mortals

One container is a toy; real applications are several — a web
server, a database, maybe a cache. **Docker Compose** describes
the whole stack in a single YAML file and runs it as one unit:
`docker compose up` and everything starts, in order, on its own
network, with its data volumes attached.

This guide picks up where [Docker for Mere
Mortals](https://paulsullivan33-dev.github.io/docker-for-mere-mortals/)
leaves off. It assumes you know what an image and a container
are, and that Docker itself is installed. Everything Compose adds
on top — the file format, services, networks, volumes,
environment, healthchecks, and the daily command set — is
explained from zero.

## What this guide covers

- [**What Compose is**](what-compose-is.md) — the one-file
  pitch, and what Compose is *not*
- [**Installing Compose**](installing-compose.md) — the plugin
  on Linux, macOS, and Windows, and the version check
- [**Your first Compose file**](your-first-compose-file.md) —
  YAML survival skills and a working `compose.yaml`
- [**Services**](services.md) — `image` vs `build`, names,
  restart policies, overriding commands
- [**Networks**](networks.md) — the automatic network,
  service-name DNS, custom networks
- [**Volumes and bind mounts**](volumes-and-bind-mounts.md) —
  named volumes vs bind mounts, read-only mounts, the
  permissions gotcha
- [**Environment variables**](environment-variables.md) —
  `environment`, `env_file`, `.env` interpolation, and
  keeping secrets out of git
- [**Depends-on and healthchecks**](depends-on-and-healthchecks.md) —
  start order vs readiness, and the healthcheck that makes
  `depends_on` actually work
- [**Ports**](ports.md) — publishing ports, the
  `"8080:80"` order, `expose` vs `ports`
- [**The everyday commands**](the-everyday-commands.md) —
  `up`, `down`, `ps`, `logs`, `exec`, `pull`, `config`
- [**A real multi-service example**](real-example.md) — web +
  database + cache, all wired together
- [**Updating and recreating**](updating-and-recreating.md) —
  new images, config changes, and keeping your data safe
- [**Profiles**](profiles.md) — optional services that only
  start when you ask
- [**Troubleshooting**](troubleshooting.md) — symptom → fix,
  with the real error messages
- [**Glossary**](glossary.md) — the terms, collected in one
  place

## What it doesn't cover

Kubernetes and Swarm (multi-machine orchestration),
writing Dockerfiles (see the Docker guide), and
production hardening (TLS, secrets managers, registries
with authentication). This guide is about running a stack
on one machine, well.

## How to use this guide

Every `compose.yaml` in these chapters was validated with
`docker compose config` against the real thing — **Check:**
blocks show what success looks like. Type the files
yourself; YAML punishes copy-paste indentation errors the
same either way, and typing teaches the shape.

## Validation notes

- All compose files were parsed and rendered with Docker
  Compose v5.6.0 (`docker compose config`); outputs shown
  are the actual outputs.
- Quoted error messages in [Troubleshooting](troubleshooting.md)
  were reproduced with Compose v5.6.0 unless marked
  otherwise.
- `up`/`down` runs were **not** performed here — the build
  machine's kernel cannot run the Docker daemon (no bridge
  networking support), so container lifecycle commands are
  documented from the official Compose reference, not from
  live runs. `config`, which is the validation that matters
  for file correctness, needs no daemon and was run on
  every sample.
- Version-sensitive facts were verified in October 2026;
  treat them as "check again in a year."

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Docker for Mere Mortals](https://paulsullivan33-dev.github.io/docker-for-mere-mortals/)
- [Podman for Mere Mortals](https://paulsullivan33-dev.github.io/podman-for-mere-mortals/)
- [Caddy for Mere Mortals](https://paulsullivan33-dev.github.io/caddy-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/docker-compose-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/docker-compose-for-mere-mortals/blob/main/LICENSE-CODE).
