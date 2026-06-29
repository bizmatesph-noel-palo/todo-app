---
inclusion: fileMatch
fileMatchPattern: "**/*migration*.php"
---

# Migration Guide

## Naming Convention

Migrations follow Laravel's timestamp format with descriptive names:
```
yyyy_mm_dd_hhmmss_{action}_{table}_table.php
```

Actions: `create`, `add`, `modify`, `drop`, `rename`

Examples:
- `2023_09_29_120331_create_tasks_table.php`
- `2026_07_01_000001_add_user_id_to_tasks_table.php`
- `2026_07_01_000002_add_priority_to_tasks_table.php`

## Schema Rules

### Required columns on all tables
```php
$table->id();
$table->timestamps();
```

### User-owned tables (after user scoping is added)
```php
$table->foreignId('user_id')->constrained()->cascadeOnDelete();
```

### String columns for enums
- Use string columns with PHP Enum casts, not MySQL ENUM type
- Easier to add new values without `ALTER TABLE`

```php
$table->string('status', 20)->default('created');
```

### Nullable columns
- Only make columns nullable if the business logic allows empty values
- Use `->nullable()` intentionally, not as a default

### Foreign keys
- Always use `foreignId()->constrained()` for referential integrity
- Ownership FKs: `cascadeOnDelete()` (user deletion removes their data)
- Reference FKs: `nullOnDelete()` (preserve record if reference is deleted)

```php
$table->foreignId('user_id')->constrained()->cascadeOnDelete();
```

## Existing Tables

### tasks
```php
$table->id();
$table->string('title');
$table->string('status', 20)->default('created');  // 'created', 'in_progress', 'completed'
$table->boolean('completed')->default(false);
$table->timestamp('completed_at')->nullable();
$table->timestamps();
```

### users (Laravel Breeze default)
```php
$table->id();
$table->string('name');
$table->string('email')->unique();
$table->timestamp('email_verified_at')->nullable();
$table->string('password');
$table->rememberToken();
$table->timestamps();
```

## Planned Schema Changes

### Add user_id to tasks (tech-debt #2)
```php
$table->foreignId('user_id')->after('id')->constrained()->cascadeOnDelete();
$table->index(['user_id', 'status'], 'idx_tasks_user_status');
```

## Migration Best Practices

1. **One concern per migration** — don't mix table creation with data seeding
2. **Always write `down()` methods** — enable clean rollbacks during development
3. **Never modify released migrations** — create new migrations to alter existing tables
4. **Test with `make fresh`** — ensure migrations run cleanly from scratch
5. **Seed after migrate** — use `make fresh` (migrate:fresh --seed) for full resets
6. **Name indexes explicitly** — `idx_{table}_{columns}` pattern for clarity
