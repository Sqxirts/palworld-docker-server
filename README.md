# Palworld Docker Server

A Palworld dedicated server for friends, run with Docker Compose on my home Proxmox server in its own container, so it doesn't depend on my desktop being on.

## What's in the repo

- `docker-compose.yml`: the whole server definition, built on the [`thijsvanloef/palworld-server-docker`](https://github.com/thijsvanloef/palworld-server-docker) image
- `.env.example`: the public IP and the two passwords the server needs, kept out of the repo

## Choices worth noting

- **Settings live in environment variables only.** This image regenerates the game's settings file on every start, so edits to that file get wiped. Every setting is in `docker-compose.yml` instead.
- **Passwords outside the code.** Admin and join passwords come from a gitignored `.env` file.
- **Only the game ports are published.** UDP 15120 for players and 27015 for Steam's server query. RCON stays inside the container.
- **Autosave every 5 minutes** (`AUTO_SAVE_SPAN: 300`), and a 30-second grace period on shutdown so the last save finishes.

## Run it

```bash
cp .env.example .env    # then fill in real values
docker compose up -d
```

Game data persists in `./palworld-data`, which is gitignored.

## Known limits

- The image is pinned to `latest`, so a rebuild can pull a newer version. Pin a tag if you need exact rebuilds.
