---
inclusion: fileMatch
fileMatchPattern: "**/*.php"
---

# Backend Standards

## Architecture Layers

```
Request → Controller → Service → Repository/Model → Database
                          ↕
                     DTO / Enum
```

Each layer has a single responsibility. Data flows down through the layers, and responses flow back up.

## Controllers (Thin Orchestrators)

- Accept the request, delegate to a service, return the response
- No business logic, no direct model queries, no data transformation
- RESTful resource methods (index, store, show, update, destroy)
- Use Form Requests for validation (injected automatically)
- Return API Resources for response transformation

```php
final class ApiTaskController extends ApiBaseController
{
    public function __construct(
        private readonly TaskServiceInterface $taskService,
    ) {}

    public function store(TaskRequest $request): TaskResource
    {
        $task = $this->taskService->store($request);

        return new TaskResource($task);
    }
}
```

## Services (Business Logic)

- Contain all business rules and orchestration
- One service per domain: `TaskService`
- Accept DTOs or validated arrays — NOT FormRequest objects (decouple from HTTP layer)
- Wrap multi-step operations in database transactions
- Throw domain-specific exceptions for business rule violations

```php
final class TaskService implements TaskServiceInterface
{
    public function __construct(
        private readonly TaskRepositoryInterface $taskRepository,
    ) {}

    public function store(TaskData $data): Task
    {
        return $this->taskRepository->create($data);
    }
}
```

## Repositories (Data Access)

- Encapsulate queries and data manipulation
- Return Eloquent models, collections, or DTOs
- Use `AbstractRepository` as base class for common operations
- Use `firstOrFail()` when a record must exist (never return null silently)

```php
final class TaskRepository extends AbstractRepository implements TaskRepositoryInterface
{
    public function __construct(Task $task)
    {
        parent::__construct($task);
    }

    public function findById(int $id): Task
    {
        return $this->getQuery()->findOrFail($id);
    }

    public function create(TaskData $data): Task
    {
        return $this->model
            ->setTitle($data->title)
            ->setStatus($data->status);
        // save and return
    }
}
```

## DTOs (Data Transfer Objects)

- Use for passing structured data between layers
- Immutable — all properties should be `readonly`
- Created from request data via a static factory method
- Replaces passing FormRequest into the service layer

```php
final readonly class TaskData
{
    public function __construct(
        public string $title,
        public TaskStatusEnum $status = TaskStatusEnum::CREATED,
        public ?bool $completed = false,
    ) {}

    public static function fromRequest(TaskRequest $request): self
    {
        return new self(
            title: $request->validated('title'),
            status: TaskStatusEnum::from($request->validated('status', 'created')),
            completed: $request->boolean('completed'),
        );
    }
}
```

## Enums

- Use PHP 8.1 backed string enums for all finite value sets
- Store as strings in the database (not MySQL ENUM type for easier migrations)
- Cast in model `$casts` for automatic hydration
- Use `ArrayableEnumTrait` for helper methods

```php
enum TaskStatusEnum: string
{
    use ArrayableEnumTrait;

    case CREATED = 'created';
    case IN_PROGRESS = 'in_progress';
    case COMPLETED = 'completed';
}
```

## API Resources (Response Transformation)

- Extend `BaseResource` for consistent response envelope
- Handle data formatting and conditional includes
- Never expose raw database columns without intention

```php
final class TaskResource extends BaseResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'status' => $this->status->value,
            'completed' => $this->completed,
            'completed_at' => $this->completed_at?->toIso8601String(),
            'created_at' => $this->created_at->toIso8601String(),
            'updated_at' => $this->updated_at->toIso8601String(),
        ];
    }
}
```

## Form Requests (Validation)

- One request class per action (or shared if rules are identical for store/update)
- Validation rules only — no business logic
- Use enum validation: `Rule::in(TaskStatusEnum::values())`

```php
final class TaskRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'status' => ['sometimes', 'string', Rule::in(TaskStatusEnum::values())],
            'completed' => ['sometimes', 'boolean'],
        ];
    }
}
```

## Service Provider Bindings

- Bind interfaces to implementations in `MyToDoAppServiceProvider`
- One binding per interface
- Allows swapping implementations for testing

```php
public function register(): void
{
    $this->app->bind(TaskRepositoryInterface::class, TaskRepository::class);
    $this->app->bind(TaskServiceInterface::class, TaskService::class);
}
```

## Error Handling

- Use `firstOrFail()` — Laravel automatically returns 404 for `ModelNotFoundException`
- Override `toResponse()` on error resources to set actual HTTP status codes
- Don't wrap errors in HTTP 200 responses
- Let Laravel's exception handler manage the response format
- Log unexpected errors with context

## Factories (Test Data)

- Every model should have a factory
- Factories produce realistic data using Faker
- Define states for common variations

```php
final class TaskFactory extends Factory
{
    protected $model = Task::class;

    public function definition(): array
    {
        return [
            'title' => fake()->sentence(3),
            'status' => fake()->randomElement(TaskStatusEnum::cases()),
            'completed' => false,
            'completed_at' => null,
        ];
    }

    public function completed(): static
    {
        return $this->state([
            'status' => TaskStatusEnum::COMPLETED,
            'completed' => true,
            'completed_at' => now(),
        ]);
    }
}
```
