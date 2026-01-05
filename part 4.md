# Part 4: Advanced API Architecture, Security & Multitenancy

At a Senior level, you aren't just building "endpoints." You are building **contracts**. You need to ensure that your API is versioned, secure, and scalable, and that your multi-tenant logic is leak-proof.

---

## 1. Professional API Design & Versioning
Forget `/api/v1/users`. That’s the "easy" way, but it often leads to code duplication.

### A. Versioning Strategies
*   **Header-based Versioning:** Use the `Accept` header (e.g., `Accept: application/vnd.myapp.v2+json`). This allows you to keep the same URL for a resource while changing the internal logic.
*   **Logic Branching:** Instead of duplicating controllers, use the **Strategy Pattern** within the Service layer to handle version-specific logic.

### B. API Resources (The "Contract" Layer)
A Senior Dev uses `JsonResource` to ensure the internal DB schema never leaks to the public.
*   **Conditional Attributes:** Use `$this->when($this->isAdmin(), ...)` to hide sensitive data based on roles.
*   **Data Wrapping:** Ensure consistent top-level keys (`data`, `meta`, `links`).
*   **The "N+1" Resource Trap:** Never access a relationship in a Resource that wasn't eager-loaded. Use `$this->whenLoaded('relation')` religiously.

### C. Webhook Systems
When building for 6+ years, you know webhooks need high reliability.
*   **Signing Payloads:** Always sign your webhook payloads using an `HMAC` (SHA256) signature in the header.
*   **Idempotency:** Force the consumer to provide an `Idempotency-Key` so you don't process the same event twice.

---

## 2. Advanced Security & Hardening

### A. Sanctum vs. Passport: The Architectural Choice
*   **Sanctum:** Perfect for SPAs (Stateful/Cookie-based) and simple mobile apps. It’s lightweight and uses simple tokens.
*   **Passport:** Necessary only if you are building a **Platform**. If other companies need to integrate with you via OAuth2 (Client Credentials, Authorization Code Grant), use Passport.

### B. Defensive Coding & Sensitive Parameters
Since PHP 8.2, use the `#[SensitiveParameter]` attribute in your service methods to ensure secrets (like passwords or API keys) are redacted from stack traces and logs.

### C. Rate Limiting at Scale
Don't just use the `throttle` middleware with a static number.
*   **Dynamic Limiting:** Use `RateLimiter::for()` in `AppServiceProvider` to set limits based on the user’s subscription plan (e.g., Free: 60 rpm, Pro: 1000 rpm).
*   **Redis Backend:** Ensure your rate limiter is backed by Redis, especially in a load-balanced environment, so the "count" is shared across all web nodes.

---

## 3. Multitenancy: Architectural Patterns
Building for multiple clients (Tenants) requires a decision on data isolation.

### Strategy A: Multi-Database (Hard Isolation)
Each tenant has its own DB. 
*   **Pros:** Highest security, easy backups/restores per client.
*   **Implementation:** Use a "Landlord" database to store tenant details and use a `Middleware` to swap the `mysql` connection dynamically at runtime.
*   **The Trap:** Database migrations become complex. You must run `php artisan migrate` for every single tenant DB.

### Strategy B: Single Database (Discriminator Column)
All tenants in one table, separated by `tenant_id`.
*   **Pros:** Easy to manage, cheaper infrastructure.
*   **Implementation:** Use a **Global Scope** on all tenant-aware models.
```php
// In a Trait applied to Models
static::addGlobalScope('tenant', function (Builder $builder) {
    if (app()->bound('tenant')) {
        $builder->where('tenant_id', app('tenant')->id);
    }
});
```
*   **The Senior Warning:** Beware of the **"Leaky Tenant"**. If you forget the global scope on a custom query or a `JOIN`, Tenant A sees Tenant B’s data. Use a package like `spatie/laravel-multitenancy` or `archtechx/tenancy` for battle-tested isolation.

---

## 4. Authorization: Beyond Basic Policies
Standard Policies are fine for small apps. For complex ones, use **Attribute-Based Access Control (ABAC)**.

*   **Logic-Based Permissions:** Instead of checking `can('edit-post')`, check `can('edit', $post)`. The Policy should check if the user is the owner AND if the post is not already "Published."
*   **Gate::after():** Use this to allow "Super Admins" to bypass all checks without writing `if ($user->isAdmin())` in every single policy.

---

## 5. Data Privacy & Encryption at Rest
With GDPR/CCPA, you can't just store everything in plain text.
*   **Encrypted Casts:** Use Laravel’s `encrypted` cast for PII (Personally Identifiable Information).
```php
protected $casts = [
    'government_id' => 'encrypted',
];
```
*   **Pruning Models:** Use the `MassPrunable` trait to automatically delete old logs or soft-deleted data after 30 days to stay compliant with data retention laws.

---

### Architect's Checkpoint:
*   Can you explain why a **Stateful SPA** (Sanctum) is more secure against XSS than a **JWT stored in LocalStorage**? (Cookies with `httpOnly` and `SameSite` flags).
*   How do you handle a migration that adds a `tenant_id` to a table with 10 million rows? (Step 1: Add nullable column. Step 2: Update in chunks. Step 3: Make it non-nullable).
*   When should you use **Signed URLs**? (For temporary access like "Download Invoice" or "Unsubscribe" links without requiring a login).

**Ready for Part 5: Domain-Driven Design (DDD), Testing Patterns, and Clean Architecture?** Say **"Go Part 5"**.