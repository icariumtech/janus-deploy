# janus-deploy

Homelab deployment for [JANUS — Sci-Fi Campaign Console](https://github.com/icariumtech/janus-console).

This repo is everything a server needs to run JANUS. No source code, no build tools — just Docker Compose pulling a pre-built image from GitHub Container Registry.

## What's in here

```
janus-deploy/
├── compose.yml           # Service definitions (app + mcp)
├── .env.example          # Environment variable template
├── .gitignore            # Keeps .env and db.sqlite3 out of git
└── data/
    └── campaign/
        └── standby.yaml  # Standby screen title/subtitle
```

Your campaign data lives in `data/` and is bind-mounted into the container. Add locations, crew, sessions, and other YAML files here — they persist across image updates.

## First-time setup

**Prerequisites:** Docker with the Compose plugin (`docker compose version`).

```bash
# 1. Clone this repo on your server
git clone https://github.com/icariumtech/janus-deploy.git
cd janus-deploy

# 2. Create your .env from the example
cp .env.example .env
```

Edit `.env` — at minimum set:
- `SECRET_KEY` — run `openssl rand -hex 32` to generate one
- `DJANGO_SUPERUSER_PASSWORD` — your GM admin password
- `ALLOWED_HOSTS` — add your server's LAN IP so players can connect

```bash
# 3. Create the database file (Docker makes it a directory if missing)
touch db.sqlite3

# 4. Pull the image and start
docker compose pull
docker compose up -d
```

The GM console is at `http://your-server-ip:8000/gmconsole/` — log in with the admin credentials you set. Players connect to `http://your-server-ip:8000/terminal/`.

## Updating

When a new version of JANUS is released:

```bash
docker compose pull
docker compose up -d
```

Your `data/` directory and `db.sqlite3` are on the host — they survive image updates.

## Customising the standby screen

Edit `data/campaign/standby.yaml`:

```yaml
title: "MOTHERSHIP"
subtitle: "The Outer Veil"
```

Changes take effect immediately — no restart needed.

## Services

| Service | Port | Purpose |
|---------|------|---------|
| `app` | 8000 | Main Django app — GM console + player terminal |
| `mcp` | 8001 | MCP server — campaign AI tools for external agents |

The `mcp` service waits for `app` to be healthy before starting.

## Environment variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SECRET_KEY` | **Yes** | Django secret key |
| `DJANGO_SUPERUSER_PASSWORD` | **Yes** | GM admin password |
| `DJANGO_SUPERUSER_USERNAME` | No | GM admin username (default: `admin`) |
| `DJANGO_SUPERUSER_EMAIL` | No | GM admin email (default: `admin@localhost`) |
| `ALLOWED_HOSTS` | No | Comma-separated hosts/IPs (default: `localhost,127.0.0.1`) |
| `ANTHROPIC_API_KEY` | No | Required for JANUS AI terminal |
| `OBSIDIAN_VAULT_PATH` | No | Path to Obsidian vault for campaign lore |
| `DEBUG` | No | Set `True` only for troubleshooting (default: `False`) |
