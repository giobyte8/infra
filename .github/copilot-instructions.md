# Fleet — Copilot Instructions

Docs, scripts, and IaC for operating services across multiple hosts.

## Architecture

- `services/common/` — shared service bundles (not directly runnable)
- `services/<host>/` — host entrypoint; owns `compose.yaml`, `.env`, and the network definition
- Each host's `compose.yaml` uses `include:` to pull from `services/common/`

Pattern:
```
services/
  common/backbone/compose.yaml   # shared services, no network block
  m4/
    compose.yaml                 # name: fleet + include + network
    .env                         # credentials + port mappings
```

## Conventions

- Always run `docker compose` from the host directory (e.g. `services/m4/`), never from `common/`
- Requires Docker Compose v2.20+ for `include:` support
- Project name is set via `name:` in the host compose file → resources prefixed `fleet_`
- Services communicate by service name (DNS), not container name — no `container_name` entries
- Compose-scoped network (`fleet_default`), not a globally-named network
- Host port range `8200–8299` reserved for fleet services; port vars defined in `.env`
- Relative paths in `common/` (e.g. bind mounts) resolve relative to the included file's directory

## Adding a new host

1. Create `services/<hostname>/compose.yaml` with `name:`, `include:`, and `networks.default`
2. Create `services/<hostname>/.env` with credentials and port assignments in the 8200–8299 range
