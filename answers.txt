This is a massive amount of technical ground to cover. To keep this readable and within the memory limits, I will break this down.

Here are the concise, high-level answers for **Part 1, Part 2, and Part 3**.

---

### **Part 1: Core PHP (Deep Dive & Modern Features)**

1.  **GC & Ref Count:** PHP uses Reference Counting. When a variable's ref count hits 0, it's cleared. The GC collects "Cyclical References" (A -> B -> A) which ref counting cannot handle.
2.  **WeakMap:** A map where keys are objects that don't prevent Garbage Collection. Useful for caching data associated with an object without causing memory leaks.
3.  **Self vs Static:** `self::` refers to the class where the code is written. `static::` refers to the class at runtime (Late Static Binding).
4.  **Attributes:** Metadata added to classes/methods using `#[Attribute]`. Accessed via Reflection API (`getAttributes()`).
5.  **Magic Methods:** `__invoke` allows an object to be called as a function. `__toString` converts object to string. `__debugInfo` customizes `var_dump` output.
6.  **zval:** The internal C struct in PHP source code that holds the variable's value, type, and reference count.
7.  **Require vs Include:** `require` throws a Fatal Error (stops script) if missing. `include` throws a Warning (continues). `_once` checks if already loaded to prevent re-declaration.
8.  **Method Chaining:** Return `$this` at the end of the method.
9.  **Anonymous Classes:** `new class { ... }`. Useful for one-off objects, testing mocks, or event listeners where a full file isn't needed.
10. **`==` vs `===`:** `==` allows type juggling (string "1" equals int 1). `===` requires value and type match.
11. **SPL:** Standard PHP Library. Examples: `ArrayIterator`, `DirectoryIterator`, `SplFixedArray`.
12. **JIT:** Compiles PHP Opcode into Machine Code. Improves performance for CPU-intensive tasks (math/loops) but has negligible effect on standard web requests.
13. **Union/Intersection:** Union (`TypeA|TypeB`) = accepts either. Intersection (`TypeA&TypeB`) = must satisfy both interfaces.
14. **Exceptions:** `Throwable` is the interface. `Error` is for internal PHP engine issues. `Exception` is for user-land application logic.
15. **Fibers:** Lightweight concurrency (Green Threads) for interruptible functions. The basis for async PHP (like ReactPHP/Amphp).
16. **Strict Types:** Applies to the file *making the call*, not the file defining the function.
17. **Const vs Define:** `const` is compile-time (can be used inside classes). `define()` is runtime (global scope only).
18. **Yield:** Creates a Generator. It processes one item at a time instead of loading the whole dataset into RAM.
19. **Closure vs Callable:** A `Closure` is an anonymous function object. A `callable` is a type hint that accepts closures, strings `'function_name'`, or arrays `[$obj, 'method']`.
20. **Securing php.ini:** `expose_php = Off`, `display_errors = Off`, `disable_functions` (exec, shell_exec), strict `upload_max_filesize`.
21. **`??` vs `?->`:** `??` (Null Coalesce) provides a default if null. `?->` (Null Safe) stops the method chain if the object is null.
22. **Map vs Walk:** `array_map` returns a *new* array (immutable style). `array_walk` iterates and modifies the array *by reference* (return bool).
23. **Prop Promotion:** `public function __construct(public int $id) {}` — reduces boilerplate code.
24. **Named Arguments:** `func(b: 2, a: 1)`. Allows skipping optional parameters and self-documenting code.
25. **Sessions:** By default, stored as files in the server's `/tmp` directory (or path defined in php.ini).

---

### **Part 2: Laravel Internals**

1.  **Bind/Singleton/Scoped:**
    *   `bind`: New instance every time `make()` is called.
    *   `singleton`: Same instance shared for the entire app lifecycle.
    *   `scoped`: Same instance shared only within the current Request/Job.
2.  **Contextual Binding:** `when(ControllerA)->needs(Interface)->give(ImplementationA)`. Injects different classes based on the caller.
3.  **Controller Resolution:** Laravel uses PHP's **Reflection API** to inspect the constructor's type hints and resolves them via the Service Container.
4.  **Register vs Boot:** `register()` is for binding things to the container. `boot()` is for using registered services (adding listeners, routes, middleware).
5.  **Deferred Providers:** Mark as `defer = true`. They only load when the specific service they provide is actually requested, boosting performance.
6.  **Middleware Groups:** Bundles of middleware (e.g., `web` has cookies/session, `api` has throttling). Global runs on *every* request.
7.  **Event System:** Implementation of Observer pattern. `Event::fake()` prevents listeners from running during tests to assert dispatching.
8.  **Pipeline:** A design pattern where an object is passed through a series of "pipes" (tasks). Used in Middleware and Job dispatching.
9.  **Facades:** They use `__callStatic` to forward calls to `static::$app->make('accessor')`. They are proxies, not real static methods.
10. **Contract:** An Interface provided by Laravel Core. Use it to decouple your code from the concrete Laravel implementation.
11. **Route Model Binding:** The router inspects the parameter name, matches it to a Model, runs a query (`find($id)`), and injects the result into the controller.
12. **Macros:** Using the `Macroable` trait to inject methods into existing classes (like `Collection` or `Response`) at runtime.
13. **Kernel.php:** The traffic cop. It loads the core framework, defines middleware stacks, and handles the request flow.
14. **Config Cache:** Merges all `config/*.php` files into a single PHP array file for faster loading.
15. **Automatic Injection:** The container checks type hints. If a class has dependencies, it recursively resolves them (Auto-wiring).
16. **Make vs Handle:** `make()` creates the object. `handle()` is usually the entry point method for execution (Commands/Jobs/Requests).
17. **Custom Command:** Extend `Illuminate\Console\Command`, define `$signature`, and put logic in `handle()`.
18. **L11 vs L10:** L11 has a slimmer skeleton, removed `Http/Kernel.php` (moved to `bootstrap/app.php`), and defaults to SQLite.
19. **Maintenance Mode:** Returns status 503. Bypass using a secret token/cookie (e.g., `php artisan down --secret="123"`).
20. **Broadcast Channels:** Secure definitions for who can listen to private WebSockets.
21. **Tap:** Functional helper. Call a method on an object and return the object itself (not the method result).
22. **App Key:** 32-character string used by the `Encrypter` service (AES-256-CBC) to sign cookies and encrypt data.
23. **CSRF Internals:** Compares the `X-CSRF-TOKEN` header (or `_token` input) with the token stored in the user's session.
24. **Components vs Slots:** Components are reusable UI pieces. Slots are the placeholders inside them where content is injected.
25. **Custom Directive:** Use `Blade::directive('name', callback)`. The callback returns a string of PHP code, not the final HTML.

---

### **Part 3: Database & Eloquent**

1.  **HasOne vs HasOneThrough:** `HasOne` is direct (User -> Phone). `HasOneThrough` jumps a table (Mechanic -> Car -> Owner).
2.  **Polymorphic M-M:** `taggables` table. Use `morphedByMany`. Allows a "Tag" to belong to "Posts" and "Videos".
3.  **WhereHas/With/Load:**
    *   `whereHas`: Filters parent based on child existence (SQL `EXISTS`).
    *   `with`: Eager loading (pre-fetching).
    *   `load`: Lazy eager loading (fetching after parent exists).
4.  **Global Scopes:** Traits (like `SoftDeletes`) that add SQL clauses to *every* query. Remove via `withoutGlobalScope()`.
5.  **Accessors/Mutators:**
    *   Old: `getFooAttribute`, `setFooAttribute`.
    *   New: `protected function foo(): Attribute { return Attribute::make(...); }`.
6.  **Mass Assignment:** Vulnerability where user inputs overwrite protected fields (e.g., `is_admin`). Prevented by `$fillable` / `$guarded`.
7.  **Save vs Push:** `save()` saves the specific model. `push()` saves the model AND its loaded relationships.
8.  **Transactions:** `DB::transaction(function() { ... })`. Auto-commits if successful, auto-rollbacks if Exception is thrown.
9.  **Optimistic vs Pessimistic:**
    *   *Optimistic:* Checks a version/timestamp column before update.
    *   *Pessimistic:* Uses `sharedLock()` or `lockForUpdate()` (SQL Row Locking).
10. **Soft Delete:** Sets `deleted_at` timestamp instead of removing row. Excluded from queries unless `withTrashed()` is used.
11. **Casts:** Auto-convert DB data. `$casts = ['options' => 'array']` converts JSON string in DB to PHP Array on access.
12. **Debug SQL:** `DB::enableQueryLog(); ... dd(DB::getQueryLog());` or use `Model::toRawSql()` (Laravel 10+).
13. **N+1 Problem:** Iterating a collection and accessing a relation inside the loop (N queries). Solved by Eager Loading (`with`).
14. **Chunk vs Cursor:**
    *   `chunk`: Fetches pages (LIMIT/OFFSET). Uses more memory.
    *   `cursor`: Uses PHP Generator / DB Cursor. One query, streams results. Low memory.
15. **Seeder vs Factory:** Factories define the structure (blueprint). Seeders execute the factories to populate the DB.
16. **Raw Expression:** `DB::raw()`. Injects string directly into SQL. Dangerous if variables aren't bound properly (Injection risk).
17. **FirstOrCreate vs FirstOrNew:** `FirstOrCreate` saves to DB immediately if not found. `FirstOrNew` returns an instance but doesn't persist it yet.
18. **Composite Keys:** Eloquent doesn't support them natively. You must override `$primaryKey` or use a trait (like `HasCompositeKey`).
19. **Prunable:** A trait that allows models to automatically delete themselves based on a query (e.g., `old_logs`) via a scheduled command.
20. **Indexing:** Use B-Tree. Do *not* index columns with low cardinality (e.g., "is_active" boolean) or text columns without length limits.
21. **Migrations in CI/CD:** Use `php artisan migrate --force`. Ensure migrations are additive (don't rename columns used by live code).
22. **Upsert:** `Model::upsert()`. Performs standard SQL `INSERT ... ON DUPLICATE KEY UPDATE`. Atomic and fast.
23. **Subqueries:** Use `addSelect(['last_login' => Login::select('date')...])`. Efficiently gets data from related table without full join.
24. **Observers:** Classes that group model event listeners (`created`, `updated`). They run synchronously.
25. **Redis vs Memcached:** Redis supports data types (lists, sets), persistence, and pub/sub. Memcached is simple LRU key-value only.

---

Here are the high-level answers for **Part 4, Part 5, and Part 6**.

---

### **Part 4: Architecture & Design Patterns**

1.  **Repository Pattern:** A layer between Domain and Data Mapping. It decouples the application from the specific ORM/Database. Useful for testing (mocking) but arguably redundant with Eloquent's advanced features.
2.  **Action Pattern:** Single-purpose classes (e.g., `CreateUserAction`) with one method (usually `__invoke` or `handle`). Keeps controllers slim and logic reusable.
3.  **DI vs Inversion:** **Dependency Injection** is the technique (passing objects into constructors). **Dependency Inversion** is the principle (high-level modules should depend on abstractions/interfaces, not concrete low-level classes).
4.  **Adapter Pattern:** Allows incompatible interfaces to work together. *Example:* Laravel `Storage` acts as an adapter for Local, S3, or FTP drivers so the code remains the same `Storage::put()`.
5.  **Strategy Pattern:** Defines a family of algorithms (classes) and makes them interchangeable. *Example:* `PaymentStrategy` interface with `StripePayment` and `PayPalPayment` implementations. Switched at runtime.
6.  **Factory Pattern:** Creates objects without specifying the exact class. *Example:* A `NotificationFactory` that returns an SMS, Email, or Slack notifier instance based on user preference.
7.  **Decorator Pattern:** Wraps an object to add behavior dynamically without changing its structure. *Example:* Laravel Middleware "decorates" the Request/Response object.
8.  **DDD (Domain Driven Design):** Focusing on the core business logic (Domain) rather than the database schema. In Laravel, this often means organizing code by "Contexts" (e.g., Inventory, Billing) rather than generic folders (Controllers, Models).
9.  **Hexagonal (Ports & Adapters):** Isolate the core application logic (the hexagon) from outside tools (API, DB, UI) using Interfaces (Ports) and Implementations (Adapters).
10. **DTO vs VO:**
    *   **DTO (Data Transfer Object):** Dumb container to move data between layers (no logic).
    *   **VO (Value Object):** Object defined by its value, not ID (e.g., `Coordinates`, `Money`). validatable and immutable.
11. **Singleton:** A class with only one instance. *Used in:* Laravel's Service Container (e.g., the `app` instance itself, or `Log` manager).
12. **Observer Pattern:** One-to-many dependency. When one object changes state, all dependents are notified. *Used in:* Eloquent Model Observers.
13. **Modular Monolith:** Organizing a large codebase into independent "Modules" (folder with its own Routes, Controllers, Models) within a single repository, preventing spaghetti code.
14. **Fat vs Skinny:** "Skinny Controller" is best practice. Move logic to Services, Actions, or Model methods. Models can be "Fat" (business logic) but should not handle Request/Response logic.
15. **Command Bus:** Decouples the sender of a request (Controller) from the handler (Service). Useful for queuing tasks or keeping code clear.
16. **Composition vs Inheritance:** **Composition** (Has-A) is preferred over **Inheritance** (Is-A). It is more flexible to inject behavior than to extend rigid class hierarchies.
17. **Decoupling:** Writing code that doesn't rely heavily on framework-specific helpers (like `request()`, `view()`) so it can be tested or moved easily.
18. **TDD:** Red (Write failing test) -> Green (Write minimal code to pass) -> Refactor (Clean up).
19. **BDD:** Behavior Driven. Writing tests in human-readable language (Given/When/Then). Often uses tools like Behat, but achievable with PHPUnit.
20. **Proxy Pattern:** A placeholder for another object to control access to it. *Example:* Lazy Loading relationships (the relationship isn't loaded until you access the property).
21. **Imperative vs Declarative:**
    *   *Imperative:* How to do it (Loops, if/else).
    *   *Declarative:* What to do (SQL, Laravel Collections `filter()->map()`).
22. **Cross-Cutting Concerns:** Aspects of a program that affect other concerns (Logs, Transaction management). handled via Middleware or AOP (Aspect Oriented Programming).
23. **Service Layer:** Dedicated classes containing business logic, called by Controllers. Allows logic reuse (e.g., API and Web Controller call the same Service).
24. **Chain of Responsibility:** A sequence of handlers. Each handler processes the request or passes it to the next. *Example:* Middleware Stack.
25. **Loose Coupling:** Components interact through interfaces with little knowledge of definitions. Allows changing one component without breaking others.

---

### **Part 5: Security (Protection)**

1.  **SQL Injection:** Malicious SQL inserted into queries. *Prevent:* Use Eloquent or Query Builder (Bindings), never direct string concatenation.
2.  **XSS:** Scripts injected into pages. *Blade:* `{{ }}` escapes output (safe). `{!! !!}` renders raw HTML (dangerous).
3.  **CSRF (Cross-Site Request Forgery):** Attacker forces user to execute unwanted actions. *Prevent:* Laravel verifies `_token` middleware. APIs are stateless (Token-based), so they are usually immune to CSRF.
4.  **IDOR (Insecure Direct Object Reference):** Accessing `user/100` when you are user 99. *Prevent:* Policies, Gates, or scoped queries (`auth()->user()->posts()->find(100)`).
5.  **Rate Limiting:** Restricts number of requests. *Impl:* `Route::middleware('throttle:60,1')`. Uses Cache to track hits.
6.  **Password Hashing:** Laravel uses **Bcrypt** or **Argon2**. Slow hashing algorithms designed to be expensive to compute, preventing brute-force attacks.
7.  **CORS:** Browser security feature preventing requests from different domains. *Config:* `config/cors.php` sets allowed headers/origins.
8.  **Secure Uploads:** Validate MIME type (not just extension), rename files (random hash), store outside `public` folder, prevent execution permissions on upload folder.
9.  **Session Hijacking:** Stealing a session ID. *Prevent:* `HTTPS` (Secure flag), `HttpOnly` flag (JS can't access cookie), and regenerate Session ID on login (`Session::regenerate()`).
10. **Encryption:** `Crypt::encryptString($data)`. Uses OpenSSL and the `APP_KEY`.
11. **Signed URL:** URL containing a hash signature. If altered, hash fails. *Use:* Password resets, email verification links.
12. **2FA:** Two-Factor. Requires "Something you know" (password) + "Something you have" (OTP/Phone).
13. **CSP (Content Security Policy):** HTTP Header telling browser which sources (JS/CSS/Images) are allowed to load. Mitigates XSS.
14. **Clickjacking:** Loading your site in an `iframe` on a malicious site. *Prevent:* `X-Frame-Options: DENY` or `SAMEORIGIN`.
15. **Storing Tokens:** **Do not** store in LocalStorage (vulnerable to XSS). Store in `HttpOnly` `Secure` Cookies.
16. **Replay Attack:** Attacker intercepts a valid data transmission and repeats it. *Prevent:* Nonce (number used once) or timestamp limits.
17. **FormRequests:** Dedicated classes for validation. Keeps controller clean and ensures data is valid before code execution.
18. **Mass Assignment:** (Repeated from Part 3, effectively a security issue).
19. **API Security:** Use HTTPS, strict Auth (Sanctum/Passport), Rate Limiting, and Input Validation.
20. **Credential Stuffing:** Attackers use username/passwords leaked from *other* sites to try and login to yours. *Prevent:* Rate limiting login attempts.
21. **Sanitize:** Cleaning input (trimming, stripping tags) *before* validation.
22. **.env in Git:** Contains secrets (API Keys, DB Passwords). If in Git, anyone with repo access has your secrets.
23. **Hidden Fields:** `$hidden = ['password', 'token']` in Model. Ensures these fields are removed when model is converted to Array/JSON.
24. **AuthN vs AuthZ:**
    *   **Authentication (AuthN):** Who are you? (Login).
    *   **Authorization (AuthZ):** What are you allowed to do? (Permissions).
25. **Policies vs Gates:** **Gates** are closures for simple checks (can_edit_settings). **Policies** are classes organized around a Model (PostPolicy: view, create, update).

---

### **Part 6: Performance & Scalability**

1.  **Redis Cache:** In-memory key-value store. Extremely fast. Stores serialized PHP objects or simple strings.
2.  **Cache Tags:** Allows grouping cache items. `Cache::tags(['people', 'artists'])->put(...)`. You can flush just the 'people' tag without clearing 'artists'. (Redis/Memcached only).
3.  **Dispatch vs DispatchSync:**
    *   `dispatch()`: Pushes job to queue (async). Returns immediately.
    *   `dispatchSync()`: Runs job immediately in current process (sync). Blocks execution.
4.  **Failed Jobs:** Stored in `failed_jobs` table. Retry via `php artisan queue:retry`.
5.  **Job Locking:** `implements ShouldBeUnique`. Uses Cache lock to ensure only one instance of that job runs at a time.
6.  **Horizon:** Dashboard and configuration tool for Laravel Redis queues. Auto-balances workers based on load.
7.  **Autoloader Optimization:** `composer dump-autoload -o` (optimize). Generates a class map (associative array) so PHP doesn't have to scan the filesystem to find classes.
8.  **Detect N+1:** Use **Laravel Telescope** or `Model::preventLazyLoading(!app()->isProduction())`.
9.  **Select Columns:** `User::select('id', 'name')->get()`. Saves memory by not loading unused large text columns (like 'bio').
10. **Read/Write Splitting:** Configure `read` and `write` hosts in `database.php`. Laravel sends `SELECT` to Read Replica and `INSERT/UPDATE` to Primary.
11. **Opcache:** Stores pre-compiled script bytecode in shared memory, removing the need for PHP to load and parse scripts on each request.
12. **Octane:** Supercharges Laravel by keeping the application in memory (using Swoole or RoadRunner) between requests, removing boot time.
13. **Lazy vs Eager:**
    *   *Lazy:* Loads when accessed `$post->comments`.
    *   *Eager:* Loads upfront `Post::with('comments')`.
14. **Large Uploads:** Stream data directly to storage (S3/Disk) using `putFile` or chunks, rather than loading the whole file into RAM.
15. **CDN:** Distributes static assets (CSS, JS, Images) to servers globally close to the user. *Integrate:* Set `ASSET_URL` in `.env`.
16. **Optimize Blade:** Cache views (default). Avoid complex logic in loops. Use `<x-cache>` tags (if using packages) for partial caching.
17. **Config/Route Cache:** Serializes config and routes into single files. Huge speed boost in production boot time.
18. **Horizontal Scaling:** Adding more servers behind a Load Balancer. (vs Vertical Scaling: adding more CPU/RAM to one server).
19. **Load Balancer:** Distributes traffic. *Session Issue:* User might hit Server A (login) then Server B (logged out). *Fix:* Use centralized Session Driver (Redis/Database).
20. **File Session Driver:** Default driver. Stores sessions on local disk. Fails in multi-server setup because Server B can't see Server A's files.
21. **Docker Size:** Use **Alpine** Linux base images. Multi-stage builds (build assets in node image, copy only files to php image).
22. **Circuit Breaker:** If an external service fails repeatedly, the breaker "trips" and stops calling it for a while, returning a default error immediately to save resources.
23. **Monitoring:** **Telescope** (Local debugging), **Sentry** (Error tracking), **New Relic/Blackfire** (Performance profiling).
24. **Sharding:** Splitting a database across multiple machines based on a key (e.g., User ID 1-1M on DB1, 1M-2M on DB2). Complex to manage.
25. **Throttling APIs:** Protects backend resources. Returns `429 Too Many Requests`. Vital for public APIs to prevent abuse.

---

Here are the final answers for **Part 7, Part 8, and Part 9**.

---

### **Part 7: Testing & CI/CD**

1.  **Unit vs Feature vs Browser:**
    *   **Unit:** Tests a single class/method in isolation. Fastest.
    *   **Feature:** Tests a full HTTP request/response cycle (Controller -> DB -> JSON). Slower.
    *   **Browser (Dusk):** Uses Headless Chrome to test JavaScript and UI interaction. Slowest.
2.  **Mocking Dependencies:** Using `Mockery` to simulate a class. `this->instance(Service::class, Mockery::mock(...))`. Prevents real API calls/DB writes during tests.
3.  **Mock vs Spy:**
    *   **Mock:** You set expectations *before* execution (`shouldReceive('method')`).
    *   **Spy:** You assert *after* execution (`shouldHaveReceived('method')`). Useful when you don't care about the return value.
4.  **Testing Queues:** Use `Queue::fake()`. It prevents jobs from running but allows you to assert `Queue::assertPushed(JobName::class)`.
5.  **Data Provider:** A PHPUnit feature allowing you to run the same test method multiple times with different inputs (passed as arguments).
6.  **Time Sensitive Code:** Use `travelTo($date)` or the `travel()` helper to freeze or shift time within the test.
7.  **Mutation Testing:** A tool (like Infection PHP) that deliberately changes your code (e.g., changes `+` to `-`) and runs tests. If tests still pass, your tests are weak.
8.  **CI/CD Setup:** Define a YAML file (e.g., `.github/workflows/main.yml`). Steps: Checkout Code -> Install PHP -> Install Composer -> Run Tests -> Deploy.
9.  **Zero Downtime:** Deploy to a new directory (e.g., `/releases/v2`). Once ready, switch the symbolic link (`/current`) from v1 to v2 instantly.
10. **Parallel Testing:** `php artisan test --parallel`. Spawns multiple processes to run tests simultaneously, utilizing all CPU cores.
11. **Static Analysis:** Tools like **PHPStan** or **Larastan**. They read code without running it to find type errors and bugs (e.g., passing a string to a function expecting an int).
12. **Testing APIs:** Use `$response->assertJsonStructure([...])` and `$response->assertStatus(200)`.
13. **Code Coverage:** The percentage of code lines executed during tests. 100% is often overkill; focus on critical business logic/paths.
14. **Rollback:** In standard deployments (Envoy/Deployer), you change the symlink back to the `previous` release folder.
15. **Managing Secrets:** Inject them via Environment Variables in the CI system (GitHub Secrets). Never commit them to the repo.
16. **Linter:** Tools like **Laravel Pint** or **PHP-CS-Fixer**. They automatically format code (spacing, braces) to match PSR standards.
17. **Testing Transactions:** Use the `RefreshDatabase` trait. It wraps every test in a database transaction and rolls it back at the end, leaving a clean DB.
18. **File Uploads:** Use `Storage::fake('disk_name')`. Then assert existence: `Storage::disk('disk_name')->assertExists('file.jpg')`.
19. **Blue/Green Deployment:** Maintaining two production environments. Traffic goes to Blue. You deploy to Green. Once verified, the Load Balancer switches traffic to Green.
20. **Docker Dev vs Prod:**
    *   **Dev:** Mounts volume (code changes reflect instantly). Includes XDebug.
    *   **Prod:** Code is copied *into* the image (Immutable). Optimized for performance.
21. **composer.lock:** It locks the *exact* version of every dependency installed. It ensures Production has the exact same libraries as Development.
22. **Security Vulnerabilities:** Run `composer audit`. It checks your installed packages against a database of known security issues.
23. **Flaky Tests:** Tests that pass sometimes and fail others. Common causes: relying on random data ordering, race conditions, or time differences.
24. **Testing Emails:** Use `Mail::fake()`. Assert `Mail::assertSent(OrderShipped::class, function ($mail) { return $mail->hasTo('user@example.com'); });`.
25. **Continuous Integration:** The practice of merging code changes frequently (daily) and running automated tests to detect issues early.

---

### **Part 8: API Development**

1.  **REST vs GraphQL:**
    *   **REST:** Multiple endpoints (`/users`, `/posts`). Server defines structure. Good for caching.
    *   **GraphQL:** Single endpoint. Client defines structure (fetches exactly what it needs). avoids over-fetching.
2.  **HTTP Methods:**
    *   **GET:** Read data.
    *   **POST:** Create data.
    *   **PUT:** Replace data (Update all fields).
    *   **PATCH:** Modify data (Update specific fields).
    *   **DELETE:** Remove data.
3.  **Versioning:** **URI Versioning** (`/api/v1/...`) is most common/pragmatic. **Header Versioning** (`Accept: application/vnd.app.v1+json`) is cleaner but harder to test manually in a browser.
4.  **API Resources:** A transformation layer (DTO) that converts your Model into a specific JSON format, hiding database columns and adding computed fields.
5.  **Pagination:** Use `Model::paginate(10)`. Laravel automatically adds `current_page`, `last_page`, `next_page_url` to the JSON response.
6.  **401 vs 403:**
    *   **401 (Unauthorized):** "I don't know who you are" (Login failed/Token missing).
    *   **403 (Forbidden):** "I know who you are, but you aren't allowed to do this" (Permissions).
7.  **Documentation:** **Swagger (OpenAPI)**. Generates an interactive UI to test endpoints. Packages like `Scramble` or `L5-Swagger` automate this.
8.  **JWT:** JSON Web Token. A stateless token containing a payload (user ID, expiry) signed with a secret key.
9.  **JWT Structure:** `Header` (Algo) . `Payload` (Data) . `Signature` (Hash of Header+Payload+Secret).
10. **Stateful vs Stateless:**
    *   **Stateful:** Server remembers the session (Cookie/Session ID). Harder to scale horizontally.
    *   **Stateless:** Server stores nothing. Every request contains all info (Token). Easier to scale.
11. **File Downloads:** `return Storage::download('path/to/file.pdf');`. Handles headers automatically.
12. **HATEOAS:** "Hypermedia as the Engine of Application State". Including links in the JSON response (e.g., `_links: { "next": "/api/users?page=2" }`) to guide the client.
13. **Field Selection:** Allowing clients to request specific columns: `GET /users?fields=id,name`. You implement this by parsing the query param and using `select()`.
14. **Webhook:** A "Reverse API". Your app sends an HTTP POST to a URL defined by the user when an event occurs. *Security:* Sign the payload with a hash so the user can verify it came from you.
15. **Error Handling:** Return consistent JSON: `{ "message": "Resource not found", "errors": [...] }`. Handle this in `bootstrap/app.php` (L11) or `Handler.php` (L10).
16. **Content Negotiation:** Checking the `Accept` header. If `application/json`, return JSON. If `text/html`, return a View.
17. **Mobile Security:** Never store API Secrets in the mobile app code (it can be decompiled). Use **OAuth Authorization Code Flow with PKCE**.
18. **OAuth 2.0:**
    *   *Password Grant:* (Legacy) User sends user/pass.
    *   *Auth Code:* Redirect to browser for login.
    *   *Client Credentials:* Machine-to-Machine (Service A talks to Service B).
19. **Timezones:** Always store dates in **UTC** in the database. Convert to the user's local timezone only when sending the response (or let the Frontend handle conversion).
20. **Case Convention:** Database is `snake_case`. JavaScript is `camelCase`. You can use a Resource or Middleware to convert keys automatically.
21. **Load Testing:** Simulating thousands of users to break the API. Tools: **k6**, **Apache JMeter**, **Locust**.
22. **Scrubbing Logs:** Configuring the logging channel to mask sensitive keys like `password`, `api_key`, or `credit_card` so they don't appear in log files.
23. **Deprecated Endpoints:** Add a `Sunset` HTTP header (timestamp of removal) and a `Warning` header. Communicate clearly with consumers.
24. **Throttling Headers:** `X-RateLimit-Limit` (Total allowed), `X-RateLimit-Remaining` (Left). Helps clients behave.
25. **PUT vs PATCH:** (See #2). PUT implies you are sending the *entire* object. If a field is missing in PUT, it should be set to null. PATCH processes only sent fields.

---

### **Part 9: Scenario Based (The "Boss" Level)**

1.  **Slow 9-10 AM:** Check Cron jobs (Scheduled Tasks). Are heavy reports running? Check Database CPU spikes. Check if traffic is organic (users login) or bot scraping.
2.  **1GB CSV Crash:** Do **not** use `file_get_contents`. Use `fopen` and `fgetcsv` in a loop. Better yet, stream it using **PHP Generators** (`yield`). Process inside a Queue Job.
3.  **Deleted Production DB:** Stop the app (Maintenance Mode). Restore from the latest Point-In-Time Backup (AWS RDS/DigitalOcean make this easy). If binlogs exist, replay them up to the moment before deletion.
4.  **Flash Sale (100x Traffic):**
    *   Cache the "Product Page" HTML (full page cache).
    *   Use a "Waiting Room" service (Cloudflare).
    *   Defer database writes (put orders in Redis Queue first, process later).
5.  **Emails not sending (Queue processed):** The Job finished, but the Mailer failed silently or was rejected by the provider. Check `laravel.log`. Check the Mail Provider's (SES/SendGrid) suppression list/bounce logs.
6.  **Duplicate Emails (Cleanup):** Write a script. Group by email. Keep the ID with the most recent login or data. Update Foreign Keys in other tables to point to the "Kept" ID. Delete the others. Run inside a Transaction.
7.  **External API Down:** Wrap the call in a `try/catch`. Implement a **Circuit Breaker** (stop trying for 5 mins). Queue the request to retry later. Show the user a "Pending" status.
8.  **Notification System:** Use a Polymorphic table `notifications` (Laravel default). Or, if scale is massive, move to a dedicated microservice using a NoSQL DB (MongoDB/DynamoDB) for faster writes.
9.  **Race Condition (Booking):** Two users click "Buy" on Seat 1A.
    *   *Fix:* **Pessimistic Locking**. `DB::table('seats')->where('id', 1)->lockForUpdate()->first()`. The second request waits until the first transaction commits.
10. **Slow Query (10s):**
    *   Run `EXPLAIN select...`.
    *   Check for missing indexes on `WHERE`, `JOIN`, or `ORDER BY` columns.
    *   Select only specific columns.
    *   If still slow, Cache the result or create a Summary Table (materialized view).
11. **Refactor 500-line Controller:** Identify clusters of logic. Move validation to `FormRequest`. Move business logic to `Service` classes. Move queries to `Scopes`. Leave the Controller as a simple traffic director.
12. **Disk Full:** Rotate logs (delete logs older than 7 days). Clear the `storage/framework/cache`. Move user uploads to **AWS S3** immediately; do not store on app server.
13. **Junior breaking build:** Implement CI pipelines that *block* merging if tests fail. Enforce Pull Request reviews. Mentor them on how to run tests locally.
14. **Shared Session (Subdomains):** In `config/session.php`, set `domain` to `.yourdomain.com` (note the leading dot). Cookies will now apply to `app.yourdomain.com` and `admin.yourdomain.com`.
15. **Multi-Tenancy:**
    *   *Simple:* `tenant_id` column in every table (Global Scope to filter).
    *   *Strict:* Separate Database per tenant. Middleware switches the DB connection based on the subdomain.
16. **URL Shortener:** Map a long URL to an ID. Convert ID (Base10) to Base62 (a-z, A-Z, 0-9). `12345` -> `d7x`. Store in DB. Cache the lookup in Redis for speed.
17. **Chat App Schema:**
    *   `Conversations` (id)
    *   `Conversation_User` (conversation_id, user_id)
    *   `Messages` (conversation_id, user_id, body, created_at).
    *   Use WebSockets (Pusher/Reverb) for real-time.
18. **Legacy to Laravel:** **Strangler Fig Pattern**. Point Nginx to Laravel. If Laravel has the route, handle it. If 404, proxy the request to the Legacy script. Slowly migrate routes one by one.
19. **Client vs SOLID:** "If we do it the 'quick' messy way, features will take 3x longer to build next month due to bugs. If we spend 2 extra days now, we save weeks later." Explain **Technical Debt**.
20. **White Screen (WSOD):** If `display_errors` is off: Check `storage/logs/laravel.log`. If empty, check Nginx/Apache `error.log` (could be a segfault or PHP-FPM crash).
21. **Multi-Currency:** Store everything in Base Currency (e.g., USD) in DB (integers, cents). Convert to User's currency on the fly using a cached Exchange Rate.
22. **Audit Log:** Use `Observers`. On `updated`, calculate `$model->getChanges()`. Save `user_id`, `model`, `old_values`, `new_values` to an `audit_logs` table.
23. **Account Sharing:** On login, generate a random `session_token` in the DB. If a user logs in elsewhere, update the token. Middleware checks if `session('token')` matches DB. If not, logout.
24. **Likes System:**
    *   Write: Insert into `likes` table.
    *   Read: Do not `count(*)` every time. Add a `likes_count` column to the `posts` table. Increment it when a like is added (using Events/Observers).
25. **Global Scheduling:** Store all event start times in **UTC**. User A (New York) sees EST. User B (London) sees GMT. Use `Carbon::parse($utcTime)->setTimezone($userTimezone)`.

---

