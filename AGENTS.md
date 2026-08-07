# Fleet — Architecture & Conventions

Docs, scripts, and IaC for operating services across multiple hosts.

## Architecture

- `services/common/` — shared service bundles (not directly runnable)
- `hosts/<hostname>/` — host entrypoint; owns `compose.yaml`, `.env`, and the network definition
- Each host's `compose.yaml` uses `include:` to pull from `services/common/`

Pattern:
```
hosts/
  m4/
    compose.yaml                 # name: fleet + include + network
    .env                         # credentials + port mappings
  dev/
    compose.yaml
    .env

services/
  common/backbone/compose.yaml   # shared services, no network block
  common/telemetry/compose.yaml
```

## Conventions

- Always run `docker compose` from the host directory (e.g. `hosts/m4/`), never from `common/`
- Requires Docker Compose v2.20+ for `include:` support
- Project name is set via `name:` in the host compose file → resources prefixed `fleet_`
- Services communicate by service name (DNS), not container name — no `container_name` entries
- Compose-scoped network (`fleet_default`), not a globally-named network
- Host port range `8200–8299` reserved for fleet services; port vars defined in `.env`
- Relative paths in `common/` (e.g. bind mounts) resolve relative to the included file's directory

## Working with Services

### Starting a host

```bash
cd hosts/<hostname>  # e.g., hosts/m4
docker compose up -d
```

### Checking service health

```bash
cd hosts/<hostname>
docker compose ps        # View all services and health status
docker compose logs neo4j  # View neo4j logs (or any service name)
```

### Testing healthchecks

Services in `services/common/backbone/` include healthchecks:

```bash
cd hosts/m4
docker compose up neo4j
# Wait ~5-7 seconds for healthcheck to pass
docker compose ps
# neo4j should show "(healthy)" status
```

## Adding a new host

1. Create `hosts/<hostname>/compose.yaml` with:
   - `name: fleet` (project name)
   - `include:` pointing to service bundles in `services/common/`
   - `networks.default` configuration
2. Create `hosts/<hostname>/.env` with:
   - Service credentials (passwords, usernames)
   - Port assignments in the `8200–8299` range

