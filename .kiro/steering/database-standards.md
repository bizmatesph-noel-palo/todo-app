---
inclusion: fileMatch
fileMatchPattern: "**/*.php"
---

# Database Standards

## Query Patterns

### Eager Load Relationships
- Use `with()` to prevent N+1 queries
- Define default eager loads in model's `$with` property for always-needed relationships

```php
// Bad: N+1
$tasks = Task::all();
foreach ($tasks as $task) {
    echo $task->user->name; // Query per task
}

// Good: Eager loaded
$tasks = Task::with('user')->get();
```

### Scope Queries to User (when user_id is added)
- All task queries MUST be scoped to the authenticated user
- Use a global scope or relationship chaining:

```php
// Via relationship
$tasks = auth()->user()->tasks()->get();

// Via global scope
trait BelongsToAuthenticatedUser
{
    protected static function booted(): void
    {
        static::addGlobalScope('user', function (Builder $builder) {
            if (auth()->check()) {
                $builder->where('user_id', auth()->id());
            }
        });

        static::creating(function (Model $model) {
            if (auth()->check() && !$model->user_id) {
                $model->user_id = auth()->id();
            }
        });
    }
}
```

### Use Database Transactions for Multi-Step Operations
- Wrap related writes in `DB::transaction()`

```php
DB::transaction(function () use ($data) {
    $task = Task::create($data);
    // any related operations
});
```

## Indexing Strategy

### When to Add an Index
- Any column used in `WHERE` clauses regularly
- Any column used in `ORDER BY` on large tables
- Foreign keys (Laravel adds these automatically with `foreignId`)
- Composite indexes: most-selective column first

### Index Naming
- Format: `idx_{table}_{columns}` (e.g., `idx_tasks_user_status`)
- Unique constraints: `uq_{table}_{columns}`

## Data Integrity

### Foreign Key Behavior
- `user_id` → `cascadeOnDelete()` (delete user = delete their tasks)
- Future FKs should follow: cascade for ownership, nullOnDelete for references

### Validation at Multiple Levels
1. **Form Request** — input format validation (required, string, max length)
2. **Service** — business rule validation (status transitions, ownership)
3. **Database** — constraints as final safety net (foreign keys, NOT NULL)

### Enum Storage
- Store as strings in the database (`VARCHAR`), not MySQL ENUM type
- MySQL ENUM requires `ALTER TABLE` to add values — string columns don't
- Validate against PHP Enum cases in form requests
- Cast to PHP Enum in model `$casts`

```php
// Model
protected $casts = [
    'status' => TaskStatusEnum::class,
];

// Migration
$table->string('status', 20)->default('created');
```

## Query Performance

### Avoid
- `SELECT *` in complex queries — specify columns
- Subqueries in loops — use joins or eager loading
- Unbounded `LIKE '%search%'` — use indexed prefix matches
- `count()` separately when you already have the collection

### Prefer
- Aggregate queries at the database level (`SUM`, `COUNT`) over PHP-side calculation
- Pagination for all list endpoints — never return unbounded result sets
- Query builder for complex reports, Eloquent for CRUD operations

## Seeding

- Use factories for test data, seeders for reference data
- Development seeders can generate sample tasks for UI testing
- Run with: `make artisan cmd="db:seed"`
