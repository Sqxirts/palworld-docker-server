# Palworld Docker Server

## What it is

A containerized Palworld dedicated server running on a Proxmox LXC container, deployed via Docker Compose with all credentials externalized to environment variables.

## Why I built it

Wanted a persistent, low-maintenance game server for friends that didn't depend on my desktop being on, and that I could rebuild from scratch in minutes if the container or host ever needed to be reprovisioned. Runs on my homelab (a Dell R630 running Proxmox) alongside other isolated services.

## Tech stack

- Docker / Docker Compose
- [`thijsvanloef/palworld-server-docker`](https://github.com/thijsvanloef/palworld-server-docker) image
- Proxmox LXC container (Ubuntu 24.04) for isolation from other homelab services
- REST API (`:8212`) for remote admin and monitoring; RCON (`:25575`) kept enabled as a legacy fallback

## What it demonstrates

- **Secrets management**: credentials and public-facing config are injected via `.env` at runtime, never hardcoded in version control.
- **Service isolation**: the server runs in its own LXC container, separate from other homelab workloads, limiting blast radius if the container is compromised.
- **Infrastructure as code**: the entire service is reproducible from a single `docker-compose.yml` plus an `.env` file. All game settings live here as environment variables rather than in a hand-edited config file — see the note below on why that isn't optional.
- **Least-exposure networking**: only the two UDP game ports are reachable from the internet. The admin surfaces (REST API, RCON) are bound to the LAN IP rather than `0.0.0.0`, since both authenticate over plaintext HTTP and grant full server control.
- **Operational hygiene**: automated save backups, `restart: unless-stopped` for resilience across host reboots, and a scheduled build-check that warns players in-game, saves, and backs up before applying a game update.

## Note: config lives in environment variables, not the ini

The upstream image regenerates `PalWorldSettings.ini` from environment variables on **every container start**, so editing that file directly is silently discarded on the next restart. Every game setting therefore belongs in `docker-compose.yml`. (`Engine.ini` is the opposite — it isn't regenerated, so manual edits there do persist.)

Server performance is bound by a single main thread, not by core count, so `WORKER_THREADS_SERVER` is pinned explicitly rather than left to the image's default of `nproc --all` — inside an LXC that reports the *host's* thread count and oversubscribes the container.

## Setup

1. Copy `.env.example` to `.env` and fill in real values:
   ```bash
   cp .env.example .env
   ```
2. Start the server:
   ```bash
   docker compose up -d
   ```
3. Game data persists in `./palworld-data` (gitignored — this is runtime state, not source).

## Screenshots / diagrams

None yet — could add a homelab network diagram showing how this container fits alongside other services (Proxmox host → LXC → Docker → game server).
