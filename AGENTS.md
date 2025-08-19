# Agents Guide

This guide enables human and AI agents to work effectively on this repository.
The project is a minimal Docker/Compose scaffold with two build targets —
`torrent` and `usenext` — intended for building downloader-style containers.
It currently ships with placeholder stages based on Ubuntu.

Use this as a base to add real logic, keep builds reproducible, and maintain a
clean developer experience.

## Repository Map

```
.
├── build/
│   └── Dockerfile        # Multi-stage: base-image, torrent, usenext
├── docker-compose.yml    # Services pointing to Dockerfile targets
├── LICENSE               # MIT License
└── README.md             # Overview, quick start, examples
```

## Primary Flows

- Build all images: `docker compose build`
- Build one target: `docker compose build torrent`
- Run a service: `docker compose up torrent` (or `usenext`)
- Interactive shell: `docker compose run --rm torrent bash`
- Rebuild without cache: `docker compose build --no-cache torrent`

## Goals and Non‑Goals

- Goals:
  - Multi-stage builds with a shared base.
  - Side-by-side service flavors (torrent, usenext).
  - Clear extension points for runtime logic and dependencies.
- Non‑Goals (for now):
  - Shipping production-ready downloader logic.
  - Complex orchestration, databases, or external infra.

## How to Extend

1) Add logic to a target stage in `build/Dockerfile`.

```dockerfile
FROM ubuntu:latest AS base-image

# Shared setup for all targets (optional)
# RUN apt-get update && apt-get install -y ca-certificates curl && rm -rf /var/lib/apt/lists/*

FROM base-image AS torrent
# Install target-specific tools and your app
# RUN apt-get update && apt-get install -y aria2 && rm -rf /var/lib/apt/lists/*
# COPY ./torrent/ /app
# WORKDIR /app
# CMD ["bash", "run.sh"]

FROM base-image AS usenext
# Similar pattern for UseNeXT
# COPY ./usenext/ /app
# WORKDIR /app
# CMD ["bash", "run.sh"]
```

2) Optionally add source directories at the repo root, e.g. `./torrent/` and
   `./usenext/`, and copy them into the respective stages.

3) If needed, extend `docker-compose.yml` with volumes, environment variables,
   ports, healthchecks, and restart policy.

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
    # healthcheck:
    #   test: ["CMD", "bash", "-lc", "curl -fsS http://localhost:8080/health || exit 1"]
    #   interval: 30s
    #   timeout: 5s
    #   retries: 3
    # restart: unless-stopped
```

## Dockerfile Best Practices

- Use multi-stage builds to minimize final image size.
- Combine `apt-get update` and `install` in one layer; clean apt lists.
- Prefer pinned package versions when stability matters.
- Avoid leaking secrets (no `ARG`/`ENV` with credentials committed to VCS).
- Run as a non-root user when possible; set `USER` and file ownership.
- Provide a long-running `CMD` or `ENTRYPOINT` so containers don’t exit.
- Order layers from least to most frequently changed to improve build caching.
- Consider `HEALTHCHECK` to detect failure modes.

## Compose Conventions

- Keep service names aligned with Dockerfile targets (e.g., `torrent`).
- Use volumes for persistent state (downloads, configs).
- Use `.env` for local overrides if needed; avoid committing secrets.
- Add `restart: unless-stopped` for long-running services.
- Expose ports only when required.

## Security & Secrets

- Do not bake credentials into images. Inject via environment, secrets, or
  mounted files at runtime.
- Prefer non-root users and minimal capabilities.
- Review third-party tools and repositories before installing.

## Definition of Done (DoD)

- Docker images build successfully for all targeted stages.
- Containers start and keep running with a clear `CMD`/`ENTRYPOINT`.
- No secrets committed; no broken or unused packages in layers.
- Reasonable image size; layers are ordered for good cache behavior.
- README is up-to-date for any new service or behavior.

## Common Tasks

- Add a new target:
  1. Create a new stage in `build/Dockerfile` (e.g., `FROM base-image AS webui`).
  2. Add a corresponding service block to `docker-compose.yml` with `target: webui`.
  3. Implement runtime logic and a long-running command.
  4. `docker compose build webui && docker compose up webui`.

- Add volumes and config:
  - Create host directories under `./data/...`.
  - Mount in compose and ensure the container user has permissions.

- Switch to a non-root user:
  - Create a user in the Dockerfile, chown files, and set `USER`.

```dockerfile
RUN useradd -m -u 10001 appuser
WORKDIR /app
COPY --chown=appuser:appuser . /app
USER appuser
```

## Troubleshooting

- Container exits immediately:
  - Define a proper `CMD` (e.g., a service or a script that tails logs).

- Build cache not updating:
  - Use `docker compose build --no-cache` and check `COPY` layer ordering.

- Permission issues on mounted volumes:
  - Align UID/GID between host and container; use `--user` or `USER` in image.

- Slow builds:
  - Reorder layers; pin fewer packages; avoid large downloads in early layers.

## Contribution Guidelines (for Agents)

- Keep changes minimal and scoped; avoid unrelated refactors.
- Update documentation when behavior or usage changes.
- Prefer small, reviewable patches over large, mixed changes.
- Follow the DoD checklist and verify builds locally.
- Avoid introducing new external dependencies without clear justification.

## Quick Checklist Before You Finish

- [ ] Builds: `docker compose build` passes for all touched targets.
- [ ] Run: `docker compose up <service>` keeps the container running.
- [ ] Image hygiene: minimal size, no secret material in history.
- [ ] Docs: README and comments reflect changes.
- [ ] Optional: Add a healthcheck if the service exposes an HTTP or TCP endpoint.

---

License: MIT (see `LICENSE`).

