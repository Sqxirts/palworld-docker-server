# Palworld Docker Server

A Palworld dedicated server for friends, run with Docker Compose on my home Proxmox server in its own container, so it doesn't depend on my desktop being on.

## What's in the repo

- `docker-compose.yml`: the whole server definition, built on the [`thijsvanloef/palworld-server-docker`](https://github.com/thijsvanloef/palworld-server-docker) image
- `.env.example`: the public IP, the LAN IP for admin ports, and the two passwords the server needs, kept out of the repo

## Choices worth noting

- **Settings live in environment variables only.** This image regenerates the game's settings file (`PalWorldSettings.ini`) on every start, so edits to that file get silently wiped. Every setting is in `docker-compose.yml` instead.
- **Passwords outside the code.** Admin and join passwords come from a gitignored `.env` file.
- **Admin ports stay on the home network.** Only the two game ports (UDP 15120 for players, 27015 for Steam's server query) are open to the internet. The REST API (8212) and RCON (25575) are bound to the LAN IP, not every interface, because both send passwords in plaintext and give full control of the server.
- **Worker threads set by hand.** Left unset, the image sizes its thread pool from `nproc --all`, which inside a Proxmox container reports the *host's* 32 threads instead of the container's 8 CPUs. The server oversubscribed itself until I pinned `WORKER_THREADS_SERVER: 8`.
- **Logs capped** at 3 files of 10 MB each, so a chatty server can't fill the disk.
- **Autosave every 5 minutes** (`AUTO_SAVE_SPAN: 300`; 30 seconds caused hitching), and a 30-second grace period on shutdown so the last save finishes.

## Run it

```bash
cp .env.example .env    # then fill in real values
docker compose up -d
```

Game data persists in `./palworld-data`, which is gitignored.

## Known limits

- The image is pinned to `latest`, so a rebuild can pull a newer version. Pin a tag if you need exact rebuilds.
- The generated settings file inside the container still holds the passwords in plaintext.
