# Palworld Docker Server

## What it is

A containerized Palworld dedicated server running on a Proxmox LXC container, deployed via Docker Compose with all credentials externalized to environment variables.

## Why I built it

Wanted a persistent, low-maintenance game server for friends that didn't depend on my desktop being on, and that I could rebuild from scratch in minutes if the container or host ever needed to be reprovisioned. Runs on my homelab (a Dell R630 running Proxmox) alongside other isolated services.

## Tech stack

- Docker / Docker Compose
- [`thijsvanloef/palworld-server-docker`](https://github.com/thijsvanloef/palworld-server-docker) image
- Proxmox LXC container (Ubuntu 24.04) for isolation from other homelab services
- RCON for remote admin access

## What it demonstrates

- **Secrets management**: credentials and public-facing config are injected via `.env` at runtime, never hardcoded in version control.
- **Service isolation**: the server runs in its own LXC container, separate from other homelab workloads, limiting blast radius if the container is compromised.
- **Infrastructure as code**: the entire service is reproducible from a single `docker-compose.yml` plus an `.env` file.
- **Operational hygiene**: automated save backups, `restart: unless-stopped` for resilience across host reboots.

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
