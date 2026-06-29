# Biz Todo App

![CI Build](https://github.com/bizmatesph-noel-palo/todo-app/actions/workflows/ci-build.yml/badge.svg)

> A task management web application built with Laravel 8, Vue.js 3, and Tailwind CSS.
> Dockerized using the DLT (Docker Laravel Template) pattern.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Laravel 8, PHP 8.1 |
| Frontend | Vue.js 3, Tailwind CSS 2 |
| Build | Laravel Mix (Webpack) |
| Database | MySQL 8.0 |
| Auth | Laravel Breeze + Sanctum |
| Infrastructure | Docker Compose, Makefile |

## Prerequisites

- Docker Desktop with WSL2 backend
- WSL2 (Ubuntu 20.04)
- GNU Make

## Quick Start

```bash
# 1. Build Docker images
make build

# 2. Start services (detached)
make up

# 3. First-time setup (installs deps, generates key, runs migrations)
make setup

# 4. Compile frontend assets
make npm cmd="run dev"
```

The app is available at `http://localhost` after setup.

## Directory Structure

```
todo-app/
├── docker/
│   ├── containers/          # Service Dockerfiles (php, nginx, mysql, composer)
│   ├── environments/        # Environment configs (config.env, dev.env, prod.env)
│   ├── docker-compose.yml   # Base services
│   ├── docker-compose.dev.yml   # Dev override
│   └── docker-compose.prod.yml  # Prod override
├── src/                     # Laravel application root
│   ├── app/
│   ├── resources/js/        # Vue 3 SPA (views, components, composables)
│   ├── routes/
│   ├── database/
│   └── tests/
├── Makefile                 # Single interface for all commands
└── README.md
```

## Available Commands

Run `make` or `make help` to see all commands. Key ones:

### Container Management

```bash
make build          # Build all Docker images
make up             # Start services (detached)
make down           # Stop and remove containers
make restart        # Restart all services
make reset          # Full teardown + rebuild (removes volumes)
make ps             # List running containers
make logs           # Follow container logs
make shell          # Open shell in PHP containe
```

### Package Managers

```bash
make composer cmd="install"       # Install PHP dependencies
make composer cmd="require foo"   # Add a package
make npm cmd="install"            # Install Node dependencies
make npm cmd="run dev"            # Compile assets (dev)
make npm cmd="run prod"           # Compile assets (production)
```

### Laravel

```bash
make setup          # First-time setup (env, deps, key, migrations)
make artisan cmd="migrate"        # Run migrations
make artisan cmd="migrate:fresh --seed"  # Fresh DB + seeders
make artisan cmd="route:list"     # List routes
make cache-clear    # Clear all Laravel caches
```

## Configuration

- **Docker config:** `docker/environments/config.env` — project name, SYS_ENV
- **App secrets:** `docker/environments/dev.env` (gitignored, copy from `dev.env.example`)
- **Laravel env:** `src/.env` (gitignored, copied from `.env.example` by `make setup`)

## Features

- [x] Task CRUD (create, read, update, delete)
- [x] API endpoints (`/api/tasks`)
- [x] Repository + Service pattern
- [x] Vue 3 SPA with Vue Route
- [x] User authentication (Laravel Breeze)
- [ ] API auth middleware (Sanctum)
- [ ] Tasks scoped to use
- [ ] User Roles & Permissions
- [ ] Task filtering & search
- [ ] Dashboard with Charts and/or Tables for User Task data
