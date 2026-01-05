# Part 5: Domain-Driven Design (DDD), Testing Patterns, and Clean Architecture

For a developer with 6+ years of experience, the standard `app/Http/Controllers` and `app/Models` directory structure eventually becomes a "Big Ball of Mud." To maintain a massive codebase, you must transition from **Framework-Driven Development** to **Domain-Driven Design (DDD)** and **Clean Architecture**.

---

## 1. Beyond MVC: Structuring for Scale
When your app hits 100+ models, the default Laravel structure increases cognitive load. 

### The "Domain" Folder Strategy
Instead of grouping by *technical* type (Controllers, Models), group by **Business Domain**.
*   `app/Domains/Billing`: (Invoices, Subscriptions, PaymentGateways)
*   `app/Domains/Orders`: (Order, OrderItem, ShippingService)
*   `app/App`: (The glue - Controllers, Middleware, Resources)

### The Application Layer vs. The Domain Layer
*   **Domain Layer:** Contains the business logic, models, and "Actions." It should ideally not know about the Web or CLI.
*   **Application Layer:** The "delivery" mechanism (HTTP Controllers, Artisan Commands). Its job is to capture input, call a Domain Action, and return a response.

---

## 2. Actions, DTOs, and Value Objects

### A. The Action Pattern
Move logic out of Controllers and Models into single-purpose classes. This makes logic reusable in both a Controller and a Scheduled Job.
```php
class CreateOrderAction 
{
    public function execute(OrderData $data): Order 
    {
        return DB::transaction(function () use ($data) {
            $order = Order::create($data->toArray());
            // complex logic...
            return $order;
        });
    }
}
```

### B. Data Transfer Objects (DTOs)
Stop passing `$request->all()` or loose arrays into your services. Arrays are "silent killers" because you don't know what keys they contain without looking at the source code.
Use **Readonly Classes** (PHP 8.2+):
```php
readonly class OrderData 
{
    public function __construct(
        public int $userId,
        public array $items,
        public ?string $couponCode,
    ) {}

    public static function fromRequest(OrderRequest $request): self 
    {
        return new self(
            userId: $request->user()->id,
            items: $request->validated('items'),
            couponCode: $request->validated('coupon'),
        );
    }
}
```

### C. Value Objects
If you have logic for an "Address" or "Money," don't leave it as a string or integer. Encapsulate the logic.
```php
readonly class Price {
    public function __construct(public int $amount, public string $currency) {}
    public function formatted(): string { ... }
}
```

---

## 3. Advanced Testing Patterns (Pest 3.x)
At 6+ years, your tests shouldn't just check if a page loads. They should enforce **Architecture**.

### A. Architecture Testing
Using Pest, you can write tests that fail if a developer breaks your architectural rules.
```php
test('models should not be called in controllers')
    ->expect('App\Http\Controllers')
    ->not->toUse('App\Models');
    
test('domain actions must be readonly')
    ->expect('App\Domains\*\Actions')
    ->toBeReadonly();
```

### B. Mocking and Fakes (The "Contract" Way)
Don't use `Mockery` for everything. Leverage Laravel’s built-in Fakes.
*   `Http::fake()`: Essential for testing 3rd party API integrations without making real requests.
*   `Storage::fake('s3')`: Test file uploads without hitting AWS.
*   `Bus::fake()`: Ensure a job was dispatched without actually running the heavy logic.

### C. Parallel Testing & CI/CD
In a large app, tests take too long.
*   Use `php artisan test --parallel` to utilize all CPU cores.
*   **Database Isolation:** Laravel handles creating separate databases (`db_test_1`, `db_test_2`) automatically.

---

## 4. Applying SOLID in Laravel

*   **S (Single Responsibility):** Every class does one thing (Actions).
*   **O (Open/Closed):** Use the **Strategy Pattern**. If you have multiple shipping providers, bind them to an interface in the Service Container.
*   **L (Liskov Substitution):** Ensure that any class implementing an interface doesn't throw unexpected exceptions that the interface doesn't define.
*   **I (Interface Segregation):** Don't create a massive `UserServiceInterface`. Split it into `UserAuthenticatable`, `UserExportable`, etc.
*   **D (Dependency Inversion):** Inject `PaymentProviderInterface` into your controller, not `StripeProvider`. Use the Service Provider to choose which one to inject.

---

## 5. Refactoring Legacy "God" Classes
Senior developers are often hired to fix "God Models" (Models with 3000 lines of code).
1.  **Extract Traits:** (Temporary measure) Group related methods.
2.  **Move to Actions:** Identify logic that doesn't belong in the model (e.g., `generatePdf()`) and move it to an Action.
3.  **Custom Query Builders:** Move complex `where` logic into a custom Builder class to keep the Model clean.

---

### Architect's Checkpoint:
*   Can you explain why **Constructor Injection** is preferred over `app()->make()`? (Testability and clarity of dependencies).
*   When is DDD "Over-engineering"? (If it's a simple CRUD internal tool, don't waste time on DTOs and Actions).
*   How do you handle "Cross-Domain" communication? (Use **Events/Listeners**. If the Billing domain needs to know an Order was created, it should listen for an `OrderCreated` event).

**Ready for Part 6: Deployment, Infrastructure (Vapor/Forge), and Monitoring (Pulse/Telescope)?** Say **"Go Part 6"**.