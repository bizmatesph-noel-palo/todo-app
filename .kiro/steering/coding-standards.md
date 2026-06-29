# Coding Standards

## PSR Compliance

This project follows PSR-12 (Extended Coding Style) for all PHP code:
- 4 spaces indentation (no tabs)
- One class per file
- Opening braces on the same line for control structures, next line for classes/methods
- One blank line after namespace, after use block, before/after methods
- Visibility declared on all methods and properties
- No closing `?>` tag

Use ECS (Easy Coding Standard) for automated formatting: `make composer cmd="run ecs"`

## SOLID Principles

### Single Responsibility
- Each class has one reason to change
- Services handle one domain: `TaskService` doesn't handle auth or validation
- Controllers only orchestrate request→service→response
- Avoid "God classes" — split when a class exceeds ~200 lines or handles multiple concerns

### Open/Closed
- Extend behavior through composition, interfaces, and strategy patterns — not by modifying existing classes
- Use Laravel's event system to add side effects without modifying core logic

### Liskov Substitution
- Subtypes must be substitutable for their base types
- All implementations of `TaskRepositoryInterface` must honor the contract
- Avoid throwing unexpected exceptions in implementations

### Interface Segregation
- Prefer small, focused interfaces over large ones
- A repository interface should only declare methods that all implementations need
- If a method is only needed by one implementation, it doesn't belong in the interface

### Dependency Inversion
- High-level modules depend on abstractions, not concretions
- Inject interfaces in constructors, bind in service providers (`MyToDoAppServiceProvider`)
- Services depend on repository interfaces, not Eloquent models directly
- Simple CRUD can use models directly — don't over-abstract

## Composition Over Inheritance

- Prefer traits and composition for shared behavior across unrelated classes
- Use traits for: `InitialisableTrait`, `ArrayableEnumTrait`
- Use service injection over base controller methods
- Exception: Eloquent model inheritance is acceptable for STI patterns
- Never go deeper than one level of class inheritance in application code

## KISS (Keep It Simple)

- Choose the simplest solution that works correctly
- Don't add abstraction layers "for future use" — add them when needed
- A direct Eloquent query in a service is fine until the query gets complex enough to warrant a repository
- Avoid design patterns for their own sake — use them when they solve a real problem
- If a method needs a comment to explain what it does, rename it or simplify it

## DRY (Don't Repeat Yourself)

- Extract repeated logic into services, traits, or helper methods
- BUT don't force unrelated code into the same abstraction just because it looks similar
- "Rule of three" — tolerate duplication twice, extract on the third occurrence
- Shared constants belong in Enums or config, not magic strings scattered across files
- Database queries that appear in multiple places belong in a repository method

## Type Safety

### PHP
- `declare(strict_types=1)` in every file
- Type hints on all method parameters and return types
- Use union types (`int|null`) over mixed
- Use PHP 8.1 backed Enums for finite value sets (`TaskStatusEnum`)
- Avoid `mixed` type — be explicit about what a function accepts and returns

### JavaScript / Vue
- Use strict comparisons (`===`, `!==`)
- Define prop types and defaults in all components
- Use JSDoc annotations for composables and shared functions

## Architecture Layers

- **Controller** → thin, delegates to Service. No business logic.
- **Service** → orchestrates business logic, calls Repository. Accepts DTOs or plain arrays, NOT FormRequest objects.
- **Repository** → data access only, returns models or DTOs
- **Model** → Eloquent model, relationships, scopes, accessors/mutators, fluent setters
- **Request** → validation only (FormRequest classes)
- **Resource** → API response transformation only (BaseResource pattern)

## Error Handling

- Use `firstOrFail()` over `first()` when the record must exist
- Return proper HTTP status codes (don't wrap errors in 200 responses)
- Use custom exceptions for domain-specific errors
- Let Laravel's exception handler manage the response format
- Override `toResponse()` on error resources to set actual HTTP status

## Testing

- Feature tests for API endpoints (HTTP layer)
- Unit tests for service/repository logic (isolated)
- Use factories for test data
- One assertion per concept (not necessarily one per test)
- Run tests: `make composer cmd="run phpunit"`

## Vue.js / JavaScript

### General
- Use Composition API (`<script setup>` preferred for new components)
- Extract reusable logic into composables (`use*` functions)
- Props down, events up — no direct parent mutation
- Use `ref()` and `reactive()` appropriately (ref for primitives, reactive for objects)

### Component Structure
- Template → Script → Style (SFC order)
- Keep templates readable — extract complex logic to computed properties
- Use named exports for composables

### API Interaction
- All API calls go through composables (`useTasks`), not directly in components
- Handle loading/error states consistently
- Use axios instance with base URL configured centrally (`bootstrap.js`)
- Components receive data via props — don't fetch in child components

## Naming Conventions

### PHP
- Classes: `PascalCase` (`TaskService`, `TaskRepository`)
- Methods: `camelCase` (`findById`, `setTitle`)
- Properties/variables: `camelCase` (`$taskResource`, `$userId`)
- Constants: `SCREAMING_SNAKE_CASE`
- Enum cases: `UPPER_SNAKE` (`TaskStatusEnum::CREATED`)
- Interfaces: PascalCase with `Interface` suffix (`TaskServiceInterface`)
- Traits: PascalCase with `Trait` suffix (`InitialisableTrait`)
- Database columns: `snake_case` (`completed_at`, `created_at`)

### Vue / JavaScript
- Components: `PascalCase` files and tags (`TaskIndex.vue`, `DataTable.vue`)
- Functions/variables: `camelCase` (`listTask`, `isLoading`)
- Composables: `useCamelCase` (`useTasks`)
- Constants: `SCREAMING_SNAKE_CASE`

## Code Organization

- Group by domain/feature within the existing Laravel structure
- Related files stay close: `TaskService` near `TaskRepository`
- Tests mirror source structure: `tests/Feature/`, `tests/Unit/`
- Keep file length manageable: split at ~300 lines for classes, ~200 for components

## Static Analysis

- **PHPStan:** Level 5+ (configured in `phpstan.neon`)
- **ECS:** PSR-12 rule set (configured in `ecs.php`)
- Run before committing: `make composer cmd="run phpstan"` and `make composer cmd="run ecs"`
