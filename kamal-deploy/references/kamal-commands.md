# Kamal command cheat sheet

For full docs: https://kamal-deploy.org/docs/
Source & issues: https://github.com/basecamp/kamal

## Project setup
- `kamal init` — scaffold `config/deploy.yml` and `.kamal/secrets`
- Edit `config/deploy.yml` — set `service`, `image`, `servers`, `registry`, `env`
- Edit `.kamal/secrets` — registry password, app secrets (e.g., `RAILS_MASTER_KEY`)

## First deploy
- `kamal setup` — installs Docker on servers over SSH, builds image, pushes, boots containers

## Ongoing deploys
- `kamal deploy` — build, push, restart with zero downtime
- `kamal redeploy` — same but skips lock checks
- `kamal rollback <version>` — revert to a previous image tag

## Inspecting & debugging
- `kamal app logs` — tail app container logs (add `-f` to follow)
- `kamal app exec <cmd>` — run a one-off command in the app container
- `kamal app exec -i bash` — interactive shell in a fresh container
- `kamal details` — show running containers per server

## Server & proxy
- `kamal server bootstrap` — install Docker on new servers
- `kamal proxy reboot` — restart kamal-proxy (the request router)
- `kamal accessory <name> <cmd>` — manage sidecar services (db, redis, etc.)

## Help
- `kamal help` — top-level commands
- `kamal <command> --help` — flags for a specific command
