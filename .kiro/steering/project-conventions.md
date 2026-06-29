# Project Conventions

## Overview

Biz Todo App — a task management web application demonstrating Kiro spec-driven development with Laravel 8, Vue.js 3, and Tailwind CSS.

## Tech Stack

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| Backend | Laravel | 8.x | Application framework |
| Language | PHP | 8.1 | Server-side logic |
| API | REST | - | JSON API endpoints |
| Frontend | Vue.js | 3.2 | SPA framework |
| Styling | Tailwind CSS | 2.x | Utility-first CSS |
| Build | Laravel Mix (Webpack) | 6.x | Frontend bundling |
| Router | Vue Router | 4.x | Client-side routing |
| Database | MySQL | 8.0 | Relational data store |
| Auth | Laravel Breeze + Sanctum | 1.10 / 2.11 | Session + API auth |
| Testing (BE) | PHPUnit | 9.5 | Backend tests |
| Linting | ECS | 12.x | PSR-12 formatting |
| Static Analysis | PHPStan | 1.10 | Type checking |
| Infrastructure | Docker Compose | - | Containerized dev/prod |

## Project Code: BTDA (Biz Todo App)

All branches, commits, and issues use the `BTDA` project code for traceability.

## Git Flow

- **Branches:** `master` (stable) ← `development` (integration) ← feature branches
- **PRs only** — no direct pushes to master or development
- **Squash merge** on feature branches into development
- **Regular merge** from development into master (preserves integration history)

### Commit Format
```
type(BTDA-XXX): short description
```

| Prefix | Use for |
|--------|---------|
| `feat(BTDA-XXX):` | New features |
| `fix(BTDA-XXX):` | Bug fixes |
| `refactor(BTDA-XXX):` | Code restructuring (no behavior change) |
| `chore(BTDA-XXX):` | Dependencies, config, tooling |
| `docs(BTDA-XXX):` | Documentation only |
| `test(BTDA-XXX):` | Adding or fixing tests |
| `style(BTDA-XXX):` | Formatting (no logic change) |
| `ci(BTDA-XXX):` | CI/CD changes |

Examples:
- `feat(BTDA-001): add Sanctum auth middleware to task routes`
- `fix(BTDA-003): handle null return from findById`
- `test(BTDA-008): add feature tests for task CRUD API`
- `chore(BTDA-000): initial project setup`

Keep the subject line under 72 characters. Use the body for context when needed.

### Branch Naming
Format: `{type}/BTDA-{number}-{short-description}`

| Prefix | Use for |
|--------|---------|
| `config/` | Project setup, tooling, CI, Docker config, steering files |
| `design/` | Specs, requirements, tech design, architecture decisions |
| `feature/` | Implementation of new functionality |
| `fix/` | Fixing broken behavior |
| `refactor/` | Code restructuring |
| `test/` | Adding test coverage |

Examples:
- `config/BTDA-000-initial-setup`
- `design/BTDA-001-secure-api-spec`
- `feature/BTDA-001-secure-api-routes`
- `fix/BTDA-005-completed-at-bug`
- `test/BTDA-008-task-crud-tests`

### Issue Tracking
- Use GitHub Issues numbered sequentially, referenced in branches and commits
- PRs reference issues: `Closes #1` in PR description

## Naming Conventions

### PHP
- Classes: `PascalCase` (`TaskService`, `TaskRepository`)
- Methods: `camelCase` (`findById`, `setTitle`)
- Properties/variables: `camelCase` (`$taskResource`, `$userId`)
- Constants: `SCREAMING_SNAKE_CASE`
- Interfaces: PascalCase with `Interface` suffix (`TaskServiceInterface`)
- Traits: PascalCase with `Trait` suffix (`InitialisableTrait`)
- Enums: PascalCase with `Enum` suffix (`TaskStatusEnum`)
- Database columns: `snake_case` (`completed_at`, `created_at`)

### Vue / JavaScript
- Components: `PascalCase` files (`TaskIndex.vue`, `DataTable.vue`)
- Composables: `useCamelCase` (`useTasks`)
- Variables/functions: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE` for true constants

### Files & Directories
- PHP: PascalCase matching class name
- Vue: PascalCase for components, camelCase for composables
- Config/env: kebab-case or snake_case as framework dictates

## File Structure

```
todo-app/
├── .kiro/
│   ├── steering/           # AI steering files
│   └── specs/              # Feature specs (requirements → design → tasks)
├── .github/
│   └── workflows/          # CI/CD
├── docker/
│   ├── containers/         # Dockerfiles per service
│   ├── environments/       # config.env, dev.env, prod.env
│   ├── docker-compose.yml
│   ├── docker-compose.dev.yml
│   └── docker-compose.prod.yml
├── src/                    # Laravel application root
│   ├── app/
│   │   ├── Enums/
│   │   ├── Http/
│   │   │   ├── Controllers/Api/
│   │   │   ├── Requests/Task/
│   │   │   └── Resources/
│   │   ├── Models/
│   │   ├── Providers/
│   │   ├── Repositories/Task/
│   │   │   ├── Interfaces/
│   │   │   └── Resources/      # DTOs
│   │   ├── Services/Task/
│   │   │   └── Interfaces/
│   │   └── Traits/
│   ├── database/
│   │   ├── migrations/
│   │   ├── seeders/
│   │   └── factories/
│   ├── resources/js/       # Vue.js SPA
│   │   ├── components/
│   │   ├── composables/
│   │   ├── router/
│   │   └── views/
│   ├── routes/
│   └── tests/
│       ├── Feature/
│       └── Unit/
├── Makefile
└── README.md
```

## Environment Strategy

- **Development:** Volume mounts, hot-reload, exposed ports, debug configs
- **Production:** Multi-stage builds, no source mounts, optimized images
- **Testing:** Runs inside Docker via `make composer cmd="run phpunit"`

## CI/CD

- GitHub Actions
- PR validation: build + smoke test on `master` and `development`
- Push on `master`: build + smoke test
- Future: lint + full test suite once test coverage is added
