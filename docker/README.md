# Docker Directory

This directory hosts all Docker-related configuration files.

## Directory Structure

```
docker/
├── containers/               # Service Dockerfiles
│   ├── nginx/
│   │   ├── conf.d/
│   │   │   └── default.conf  # Nginx server config
│   │   └── Dockerfile
│   ├── php/
│   │   ├── config/
│   │   │   ├── php-dev.ini   # Development PHP config
│   │   │   └── php-prod.ini  # Production PHP config
│   │   └── Dockerfile        # Multi-stage (builder → runtime → production)
│   ├── mysql/
│   │   ├── conf.d/
│   │   │   └── my.cnf        # Custom MySQL config
│   │   └── Dockerfile
│   └── composer/
│       └── Dockerfile         # Reuses PHP image via additional_contexts
├── environments/
│   ├── config.env             # Single control panel (versions, ports, paths)
│   ├── dev.env                # Dev database credentials (git-ignored)
│   ├── dev.env.example        # Template for dev.env
│   ├── staging.env.example    # Template for staging
│   └── prod.env.example       # Template for production
├── docker-compose.yml         # Base services (always loaded)
├── docker-compose.dev.yml     # Development override (volumes, ports)
├── docker-compose.prod.yml    # Production override (security, limits)
├── .dockerignore              # Build context exclusions
└── README.md                  # This file
```

## Quick Start

```bash
# Build and start (development)
make build
make up

# First-time Laravel setup
make setup

# View running containers
make ps

# Run artisan commands
make artisan cmd="migrate"

# Run composer commands
make composer cmd="install"

# Run npm commands
make npm cmd="install"
make npm cmd="run dev"
```

## Environment Control

All configuration is centralized in `environments/config.env`:
- `SYS_ENV` — switches between dev/staging/prod
- `PHP_VERSION` — PHP image version (8.1)
- `DB_VERSION` — MySQL version (8.0)
- `NODE_VERSION` — Node.js version (20)
- `APP_PORT` — Host port for web access

## Key Design Decisions

- **Non-root containers** — PHP and Nginx run as `appuser` (UID 1000)
- **Multi-stage PHP build** — Builder compiles extensions, runtime is minimal
- **Named volumes** — mysql_data, vendor_data, node_modules_data
- **Compose overrides** — Base + environment-specific configs
- **Makefile interface** — All operations via `make <command>`
