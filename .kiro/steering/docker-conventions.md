# Docker Conventions

## Makefile as Interface

- All Docker operations go through the Makefile — never run `docker compose` directly
- Use `make help` to discover available commands
- Pattern: `make <target>` or `make <target> cmd="<argument>"`

### Common Commands
```bash
make build                    # Build images
make up                       # Start services
make down                     # Stop and remove containers
make restart                  # Restart all services
make setup                    # First-time init (env, deps, key, migrate)
make composer cmd="install"   # Run composer commands
make npm cmd="run dev"        # Run npm commands
make artisan cmd="migrate"    # Run artisan commands
make shell                    # Open shell in PHP container
make logs                     # Follow container logs
make ps                       # List running containers
```

## Compose Structure

- **Base:** `docker-compose.yml` — services, networks, volumes (environment-agnostic)
- **Dev override:** `docker-compose.dev.yml` — volume mounts, exposed ports, hot-reload
- **Prod override:** `docker-compose.prod.yml` — baked code, resource limits, security hardening
- Override selection is automatic via `SYS_ENV` in `config.env`

## Environment Files

- `docker/environments/config.env` — single control panel (versions, ports, paths, SYS_ENV)
- `docker/environments/dev.env` — DB credentials for dev (gitignored)
- `docker/environments/dev.env.example` — committed template for dev.env
- Never commit real credentials — only `.example` files

## Container Principles

- **Non-root:** All containers run as `appuser` (UID 1000)
- **Named volumes:** `mysql_data`, `vendor_data`, `node_modules_data` for persistence
- **Utility containers:** `composer`, `npm`, `artisan` — run with `--rm`, not long-lived
- **Multi-stage builds:** PHP Dockerfile has `builder → runtime → production` stages
- Dev override targets `runtime` stage (no baked code, volume mounts instead)

## Adding Services

When adding a new service:
1. Add to `docker-compose.yml` (base config, no env-specific details)
2. Add dev-specific config to `docker-compose.dev.yml` (ports, volumes)
3. Add prod-specific config to `docker-compose.prod.yml` (resource limits, read-only)
4. Add a Makefile target if it needs a shorthand command
5. Document in `docker/README.md` if applicable

## File Ownership

- HOST_UID/HOST_GID passed as build args for permission alignment
- Pre-create `vendor/` and `node_modules/` in Dockerfile with `appuser` ownership
- Don't use `user: "${HOST_UID}:${HOST_GID}"` on utility containers (breaks CI)
