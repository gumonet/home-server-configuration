# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Docker Compose configuration for a homelab/home server. There is no application code, build step, or test suite — the "product" is a set of YAML compose files that get synced to a remote host and run with Docker Compose. Changes are verified by deploying and checking that containers come up healthy, not by running tests.

## Architecture

`docker-compose.yml` at the repo root is only an entrypoint. It has no services of its own — it uses Compose's `include:` directive to pull in per-service files organized by category under `services/`:

- `services/applications/` — user-facing apps: `git-tea.yaml` (Gitea + Gitea Actions runner), `n8n.yaml`, `proxy-manager.yaml` (Nginx Proxy Manager), `adguard-home.yaml` (DNS/ad-blocking), `samba.yaml` (file sharing)
- `services/databases/` — `postgres.yaml`, `redis.yaml`
- `services/monitoring/` — `portainer.yaml`, `cadvisor.yaml`
- `services/home-assistant/docker-compose.yaml` — Home Assistant

When adding a new service, create a new `services/<category>/<name>.yaml` file with its own `services:`/`volumes:` blocks and add it to the `include:` list in the root `docker-compose.yml` — don't add services directly to the root file.

All services join a single external Docker network named `homelab`, declared in every service file as:
```yaml
networks:
  homelab:
    external: true
```
This network is NOT created by Compose — it must exist beforehand via `docker network create homelab`. Services that need `network_mode: host` (e.g. Home Assistant, for local network discovery) or `privileged: true` bypass this.

Cross-service dependencies use the Docker DNS name of the target service (e.g. n8n connects to Postgres via host `postgres`), so container_name/service name must stay stable across files.

Most services log via:
```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
```
Apply this to new services for consistency unless there's a reason not to.

`services/monitoring/fluentbit.yaml` exists on disk but is **not** wired into the root `docker-compose.yml`'s `include:` list — it's currently inactive/orphaned. Note this if asked about logging aggregation, and confirm with the user before assuming it should be re-enabled vs. removed.

`README.md` is kept in sync with the actual included services — treat it as accurate, but if you add/remove a service from `docker-compose.yml`'s `include:` list, update the corresponding section in `README.md` too.

## Configuration and secrets

All service files parameterize secrets/paths via `${VAR}` referencing a root `.env` file (gitignored). `.env_example` documents the expected keys:

- `MEDIA`, `STORAGE` — media/download paths for Samba
- `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD` — leftover from a removed MinIO service; not currently consumed by any included compose file
- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`
- `GITEA_RUNNER_REGISTRATION_TOKEN` — obtained from the Gitea admin panel
- `HA_CONFIG_PATH`, `TZ` — Home Assistant config path and timezone

When adding a service that needs secrets, add the variable to `.env_example` (without a real value) so the pattern stays discoverable. Never read, print, or copy real values from the root `.env` file into commits, docs, or chat output.

## Common commands

Deploy/update the stack (run on the target host, from the repo root):
```sh
docker compose up -d
```

Restart a single service by name after editing its config:
```sh
./restart-container.sh <service-name>
```

Sync this repo to the remote homelab server (rsync over SSH, deletes files on the remote that were removed locally):
```sh
./sync.sh
```
This targets a hardcoded remote (`gumonet@192.16.0.20:/home/gumonet/docker-cluster/homelab`) — check `sync.sh` before assuming it applies to a different environment.

Validate a compose file's syntax without deploying:
```sh
docker compose config
```

## Known one-off host setup

AdGuard Home needs port 53; on Ubuntu hosts the built-in resolver must be disabled first (see `improvemnts.txt`):
```sh
sudo systemctl stop systemd-resolved.service
sudo systemctl disable systemd-resolved.service
```
