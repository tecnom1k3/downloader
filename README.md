# downloader

Minimal, multi-service Docker Compose scaffold for building downloader images.

This repository provides a simple starting point with two Docker build targets
— `torrent` and `usenext` — defined in a single multi-stage Dockerfile and
wired up via `docker-compose.yml`. The images are currently placeholders based
on Ubuntu; you can extend them to install tools, add your code, and define an
entrypoint.

## Why this exists

- Keep service definitions small and reproducible with multi-stage builds.
- Work on multiple downloader flavors side-by-side (e.g., torrent vs. UseNeXT).
- Share a common base image and reduce duplication.

## Prerequisites

- Docker (Engine 24+ recommended)
- Docker Compose v2 (included in recent Docker Desktop/Engine)
- A shell environment (Linux/macOS or WSL2 on Windows)

## Quick start

Build all images:

```bash
docker compose build
```

Run a specific service (these are placeholders and may exit immediately until
you add an entrypoint):

```bash
docker compose up torrent
# or
docker compose up usenext
```

Open an interactive shell in a service image for development:

```bash
docker compose run --rm torrent bash
```

Rebuild after making Dockerfile changes (without cache):

```bash
docker compose build --no-cache torrent
```

## Project layout

```
.
├── build/
│   └── Dockerfile        # Multi-stage Dockerfile with targets: torrent, usenext
├── docker-compose.yml    # Compose services referencing build targets
├── LICENSE               # MIT License
└── README.md             # You are here
```

## How to add your logic

The `build/Dockerfile` defines a common base image and two empty targets. Extend
each target to install dependencies and add your runtime. Example skeleton:

```dockerfile
FROM ubuntu:latest AS base-image

# Shared setup (optional)
# RUN apt-get update && apt-get install -y ca-certificates curl && rm -rf /var/lib/apt/lists/*

FROM base-image AS torrent
# Install tools
# RUN apt-get update && apt-get install -y aria2 && rm -rf /var/lib/apt/lists/*
# Copy app code
# COPY ./torrent/ /app
# WORKDIR /app
# Default command
# CMD ["bash", "run.sh"]

FROM base-image AS usenext
# Similar setup tailored for UseNeXT
# COPY ./usenext/ /app
# WORKDIR /app
# CMD ["bash", "run.sh"]
```

You can organize service-specific files under `./torrent/` and `./usenext/` at
the repository root, and copy them into each respective stage. Compose already
targets these stages by name.

Update `docker-compose.yml` if you need volumes, environment, or ports. For example:

```yaml
services:
  torrent:
    build:
      context: ./build
      dockerfile: Dockerfile
      target: torrent
    volumes:
      - ./data/torrent:/data
    environment:
      - TZ=UTC
  usenext:
    build:
      context: ./build
      dockerfile: Dockerfile
      target: usenext
    volumes:
      - ./data/usenext:/data
```

## Common tasks

- Build all: `docker compose build`
- Build one: `docker compose build torrent`
- Run one: `docker compose up torrent`
- Shell in container: `docker compose run --rm torrent bash`
- Rebuild without cache: `docker compose build --no-cache torrent`
- Clean up builder cache: `docker builder prune`

## Troubleshooting

- Containers exit immediately: Define a long-running `CMD` or `ENTRYPOINT` in
  each target stage.
- Build uses stale layers: Use `--no-cache` or change `COPY` order to ensure
  invalidation when your sources change.
- File permissions on mounted volumes: Consider adding a non-root user in your
  Dockerfile and matching UID/GID with the host.
- Network-restricted environments: Pre-bake dependencies in the image or use a
  local registry/cache; ensure `apt` mirrors are reachable.

## Next steps

- Add actual downloader logic to `torrent` and `usenext` stages.
- Mount persistent volumes for downloads and configuration.
- Parameterize behavior with environment variables and secrets.
- Add CI to build and optionally push images per target.

## License

MIT — see `LICENSE` for details.
