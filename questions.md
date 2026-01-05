
### Part 1: Core PHP (Deep Dive & Modern Features)
*Focus: Memory management, new versions, and strict typing.*

1.  How does PHP’s Garbage Collection (GC) work? What is Reference Counting?
2.  What is a `WeakMap` in PHP 8 and when would you use it?
3.  Explain the difference between `self::`, `static::`, and `parent::` (Late Static Binding).
4.  What are PHP Attributes (Annotations)? How do you access them programmatically?
5.  How do `__invoke`, `__toString`, and `__debugInfo` magic methods work?
6.  Explain the concept of "zval" containers in PHP internals.
7.  What is the difference between `require`, `require_once`, `include`, and `include_once` in terms of performance and error handling?
8.  How do you implement Method Chaining in a PHP class?
9.  What are Anonymous Classes? Give a valid use case in testing.
10. Explain the difference between `==` and `===` regarding type juggling.
11. What is the Standard PHP Library (SPL)? Name three SPL iterators.
12. How does the JIT (Just-In-Time) compiler in PHP 8 change execution?
13. What are Union Types and Intersection Types?
14. How do you handle exceptions properly? What is the difference between `Throwable`, `Exception`, and `Error`?
15. What are PHP Fibers? How do they relate to asynchronous programming?
16. Explain the `strict_types=1` declaration. Does it apply to the file it is defined in or the file calling it?
17. What is the difference between `const` and `define()`?
18. How does `yield` save memory when processing large datasets?
19. What is the difference between `closure` and `callable`?
20. How would you secure a `php.ini` file for a production server?
21. What is the null coalescing operator (`??`) vs null safe operator (`?->`)?
22. How do you use `array_map` vs `array_walk`? Which one modifies the original array?
23. Explain "Constructor Property Promotion" in PHP 8.
24. What are "Named Arguments" and how do they help with legacy code?
25. How does PHP handle sessions by default? Where are they stored?

---

### Part 2: Laravel Internals (The "Under the Hood" Logic)
*Focus: The Service Container, Request Lifecycle, and Bootstrapping.*

1.  What is the difference between `bind`, `singleton`, and `scoped` in the Service Container?
2.  What is "Contextual Binding" in Laravel?
3.  How does Laravel identify which Controller method to call? (Reflection).
4.  What is a Service Provider? What is the difference between the `register` and `boot` methods?
5.  What are "Deferred Service Providers" and why use them?
6.  Explain the concept of Middleware "Groups" vs Global Middleware.
7.  How does Laravel’s Event system work? What is `Event::fake()`?
8.  What is the `Pipeline` design pattern and where does Laravel use it (besides Middleware)?
9.  How do Laravel Facades strictly work? (Explain `__callStatic` and `resolveFacadeInstance`).
10. What is a "Contract" in Laravel? Why use it over a concrete class?
11. How does Route Model Binding work internally?
12. How can you extend the Laravel framework core classes (Macros)?
13. What is the purpose of the `kernel.php` file?
14. How does Laravel handle Configuration caching (`config:cache`)?
15. Explain how Laravel’s "Automatic Injection" works.
16. What is the difference between `app()->make()` and `app()->handle()`?
17. How would you write a custom Artisan command?
18. What is the directory structure change in Laravel 11 vs Laravel 10?
19. How does Laravel handle Maintenance Mode? Can you bypass it for specific IPs?
20. What are "Broadcast Channels" and how do they relate to WebSockets?
21. How does the `Tap` helper function work?
22. What is the logic behind Laravel's encryption (App Key)?
23. How does Laravel implement CSRF protection internally?
24. What are Blade Components and Slots?
25. How do you create a custom Blade Directive?

---

### Part 3: Database & Eloquent (Optimization)
*Focus: SQL efficiency, Relationships, and Complex Queries.*

1.  What is the difference between `HasOne` and `HasOneThrough`?
2.  Explain Polymorphic Many-to-Many relationships.
3.  What is the difference between `whereHas`, `with`, and `load`?
4.  How do Global Scopes work? How do you remove one for a specific query?
5.  What are Accessors and Mutators (old syntax vs new Attribute syntax)?
6.  Explain "Mass Assignment" and the security risk involved.
7.  What is the difference between `save()` and `push()` in Eloquent?
8.  How do you handle Database Transactions in Laravel? (`DB::transaction`).
9.  What is the difference between Optimistic Locking and Pessimistic Locking?
10. How does Soft Delete work? How does it affect unique indexes?
11. What are Eloquent "Casts"? How do you cast a JSON column to an Array?
12. How do you debug SQL queries in Laravel without an external tool? (`DB::listen`, `toSql`).
13. Explain the N+1 problem in depth.
14. When should you use "Chunking" vs "Cursors" for large datasets?
15. What are Database Seeders and Factories?
16. How do you perform a "Raw Expression" query safely?
17. What is the difference between `firstOrCreate` and `firstOrNew`?
18. How do you handle composite primary keys in Eloquent?
19. What are "Prunable" models in Laravel?
20. Explain Database Indexing. When should you *not* add an index?
21. How do you manage database migrations in a CI/CD pipeline without breaking production?
22. What is `upsert` and when is it useful?
23. How do you use Subqueries in Laravel Eloquent?
24. What are Eloquent Observers? Are they synchronous or asynchronous?
25. Comparison: Redis vs Memcached for Laravel Cache.

---

### Part 4: Architecture & Design Patterns
*Focus: Code structure, maintainability, and enterprise patterns.*

1.  Explain the Repository Pattern. Is it still relevant with modern Eloquent?
2.  What is the Action Pattern (or Single Invokable Classes)?
3.  Explain Dependency Injection vs Dependency Inversion.
4.  What is the Adapter Pattern? Give a Laravel example (Filesystem/Storage).
5.  What is the Strategy Pattern? How would you use it for a Payment System?
6.  Explain the Factory Pattern (not Database Factories, but Class Factories).
7.  What is the Decorator Pattern? (e.g., Middleware).
8.  What is Domain Driven Design (DDD)? How does it fit into Laravel?
9.  What is Hexagonal Architecture (Ports and Adapters)?
10. Explain the difference between a DTO (Data Transfer Object) and a VO (Value Object).
11. What is the Singleton Pattern? Where is it used in Laravel?
12. What is the Observer Pattern?
13. How do you organize code in a large Monolithic Laravel app (Modules/Domains)?
14. Explain "Fat Model, Skinny Controller" vs "Skinny Model, Skinny Controller".
15. What is the Command Bus pattern?
16. What is the difference between Composition and Inheritance?
17. How do you decouple your application from the framework?
18. What is TDD (Test Driven Development)?
19. What is BDD (Behavior Driven Development)?
20. Explain the Proxy Pattern.
21. What is the difference between Imperative and Declarative programming?
22. How do you handle "Cross-Cutting Concerns" (Logging, Caching)?
23. What is a "Service Layer"?
24. Explain the "Chain of Responsibility" pattern (Middleware).
25. What is "Loose Coupling" and why is it important?

---

### Part 5: Security (Protection)
*Focus: OWASP Top 10, Encryption, and User Safety.*

1.  How do you prevent SQL Injection in Laravel?
2.  What is XSS (Cross Site Scripting)? How does Blade handle it?
3.  What is CSRF? Why do APIs usually not need it?
4.  How do you prevent IDOR (Insecure Direct Object References)?
5.  What is "Rate Limiting"? How do you implement it in Laravel routes?
6.  How does Laravel’s Password Hashing work (Bcrypt/Argon2)?
7.  What is CORS (Cross-Origin Resource Sharing)? How do you configure it?
8.  How do you secure uploaded files? (MIME type validation, execution prevention).
9.  What is "Session Hijacking" and how do you prevent it?
10. How do you encrypt data in the database using Laravel?
11. What is a "Signed URL" and when should you use it?
12. How do you implement Two-Factor Authentication (2FA)?
13. What is CSP (Content Security Policy)?
14. Explain "Clickjacking" and the X-Frame-Options header.
15. How do you safely store API Tokens? (Database vs Redis vs JWT).
16. What is a Replay Attack?
17. How do you validate form data securely? (`FormRequests`).
18. What is "Mass Assignment" vulnerability?
19. How do you secure a Laravel API? (Sanctum/Passport).
20. What is "Credential Stuffing"?
21. How do you sanitize user input?
22. Why should you never commit `.env` to Git?
23. How do you hide sensitive fields from JSON responses? (`$hidden`).
24. What is the difference between Authentication and Authorization?
25. How do Laravel Policies and Gates work?

---

### Part 6: Performance & Scalability
*Focus: Caching, Queues, and High-Traffic management.*

1.  How do you use Redis for Caching in Laravel?
2.  What is the difference between Cache "Tags" and standard Cache keys?
3.  Explain Laravel Queues. What is the difference between `dispatch` and `dispatchSync`?
4.  What happens if a Queue Job fails? (Failed Jobs Table, Retries).
5.  How do you prevent a Queue Job from running twice? (Job Locking).
6.  What is "Horizon"?
7.  How do you optimize Autoloader performance? (`dump-autoload -o`).
8.  What is the N+1 Query problem and how do you detect it automatically?
9.  How do you optimize database queries (Select specific columns)?
10. What is Database Replication (Read/Write Splitting) in Laravel?
11. How does PHP Opcache work?
12. What is "Octane"? How does it speed up Laravel?
13. Explain "Lazy Loading" vs "Eager Loading".
14. How do you handle large file uploads efficiently? (Streaming).
15. What is a CDN (Content Delivery Network) and how do you integrate it?
16. How do you optimize Blade rendering?
17. What is the benefit of using `route:cache` and `config:cache`?
18. How do you scale Laravel horizontally (Multiple Servers)?
19. What is a Load Balancer? How does it affect Session storage?
20. Why shouldn't you use the `file` driver for Sessions in a load-balanced app?
21. How do you minimize Docker image size for Laravel?
22. What is the "Circuit Breaker" pattern?
23. How do you monitor application performance? (Telescope, New Relic).
24. What is "Database Sharding"?
25. How do you handle "Throttling" for APIs?

---

### Part 7: Testing & CI/CD
*Focus: Quality Assurance and Deployment pipelines.*

1.  What is the difference between Unit, Feature, and Browser Tests (Dusk)?
2.  How do you mock dependencies in Laravel? (`Mockery`).
3.  What is the difference between `mock()` and `spy()`?
4.  How do you test a Queue Job without actually running it?
5.  What is a "Data Provider" in PHPUnit?
6.  How do you test time-sensitive code? (`travelTo`).
7.  What is "Mutation Testing"?
8.  How do you set up a CI/CD pipeline using GitHub Actions or GitLab CI?
9.  What is "Zero Downtime Deployment"? How do you achieve it? (`Envoy`, `Deployer`).
10. How do you run tests in parallel to speed them up?
11. What is Static Analysis? (PHPStan, Larastan).
12. How do you test APIs? (JSON structure assertions).
13. What is code coverage? Is 100% coverage necessary?
14. How do you rollback a failed deployment?
15. How do you manage secrets in CI/CD?
16. What is a "Linter" (CS Fixer/Pint)?
17. How do you test database transactions? (`RefreshDatabase` trait).
18. How do you test File Uploads? (`Storage::fake`).
19. What is "Blue/Green Deployment"?
20. How do you use Docker for local development vs production?
21. What is the `composer.lock` file and why must it be committed?
22. How do you check for security vulnerabilities in dependencies? (`composer audit`).
23. What are "Flaky Tests" and how do you fix them?
24. How do you test Emails? (`Mail::fake`).
25. Explain the concept of "Continuous Integration".

---

### Part 8: API Development (REST & GraphQL)
*Focus: Building interfaces for other systems.*

1.  What is the difference between REST and GraphQL?
2.  What are the standard HTTP methods and their idempotency?
3.  How do you handle API Versioning? (URI vs Header).
4.  What are API Resources and Collections in Laravel?
5.  How do you handle pagination in API responses?
6.  What is the difference between 401 (Unauthorized) and 403 (Forbidden)?
7.  How do you document APIs? (Swagger/OpenAPI).
8.  What is JWT (JSON Web Token)?
9.  Explain the structure of a JWT.
10. Comparison: Stateful vs Stateless authentication.
11. How do you handle file downloads via API?
12. What is HATEOAS?
13. How do you implement "Field Selection" in REST?
14. What is a "Webhook"? How do you process it securely?
15. How do you handle API errors and uniform error responses?
16. What is "Content Negotiation"?
17. How do you secure an API used by a mobile app?
18. What is OAuth 2.0? Explain the flows (Grant Types).
19. How do you handle Timezones in APIs? (UTC standard).
20. What is "Snake Case" vs "Camel Case" in JSON responses?
21. How do you test API performance (Load Testing)?
22. What is "Scrubbing" sensitive data from logs?
23. How do you handle deprecated API endpoints?
24. What is Rate Limiting "Throttling" headers (`X-RateLimit-Remaining`)?
25. Difference between PUT and PATCH?

---

### Part 9: Scenario Based (Problem Solving)
*Focus: Real-world situations.*

1.  **Scenario:** Users report that the site is slow between 9 AM and 10 AM. How do you investigate?
2.  **Scenario:** You need to import a 1GB CSV file. The server crashes. Fix it.
3.  **Scenario:** A developer pushed code that deleted the production database. What is your recovery plan?
4.  **Scenario:** You have a "Flash Sale" event. Traffic will spike 100x. Prepare the infrastructure.
5.  **Scenario:** Emails are not being sent, but the queue says "Processed". Debug this.
6.  **Scenario:** You find duplicate records in a user table that requires unique emails. How do you clean it up without downtime?
7.  **Scenario:** An external API you rely on is down. How does your app handle it?
8.  **Scenario:** Designing a "Notification System" (Email, SMS, Push). Structure the database.
9.  **Scenario:** How do you handle a "Race Condition" in a ticket booking system?
10. **Scenario:** A query takes 10 seconds. You added indexes, but it's still slow. What now?
11. **Scenario:** Refactor a Controller method that is 500 lines long.
12. **Scenario:** Your storage disk is full. How do you prevent this in the future?
13. **Scenario:** A junior dev keeps breaking the build. How do you handle it as a senior?
14. **Scenario:** You need to share a session between a main domain and a subdomain.
15. **Scenario:** How do you implement "Multi-Tenancy" (One app, multiple client databases)?
16. **Scenario:** Design a URL Shortener system.
17. **Scenario:** Design a Real-time Chat application schema.
18. **Scenario:** You need to migrate data from a Legacy PHP app to Laravel. Strategy?
19. **Scenario:** The client wants a feature that violates SOLID principles. How do you explain it?
20. **Scenario:** Debug a "White Screen of Death" in production without error display enabled.
21. **Scenario:** Handling multiple currencies in an E-commerce app.
22. **Scenario:** How do you implement an "Audit Log" for every action in the system?
23. **Scenario:** Preventing users from sharing accounts.
24. **Scenario:** Implementing a "Like" feature efficiently (Count vs Table).
25. **Scenario:** Dealing with Timezones for a Global Scheduling App.

---

### Teacher's Final Advice

This is roughly **225 high-level questions**.
*   **Don't memorize answers.** Use this list to find "Gaps" in your knowledge.
*   **Pick 3 per day.** If you don't know one, build a small demo app to understand it.
*   **The "Why" matters.** In an interview, if you don't know the exact syntax, explain the *logic*. That is what 6 years of experience buys you.