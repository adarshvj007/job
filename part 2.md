# Part 2: Advanced Eloquent & Data Persistence

At the 6+ year mark, you know that Eloquent is both Laravel’s greatest strength and its biggest performance bottleneck. A Senior Architect doesn't just write queries; they manage **memory hydration**, **query execution plans**, and **data integrity at scale**.

---

## 1. The Hidden Cost of Hydration
Every time you run `User::all()`, Laravel is not just fetching rows. It is:
1. Fetching the raw array from the PDO driver.
2. Creating a new `User` object for every row.
3. Filling the `$attributes` and `$original` arrays for each instance.
4. Setting up relationships and event listeners.

**The Senior Move:**
*   **`cursor()`:** Uses PHP Generators to hydrate one model at a time. Perfect for iterating over 100k records without a `memory_limit` error.
*   **`toBase()` / `getQuery()`:** When you only need data for a table or a report, drop Eloquent and use the Query Builder. This avoids model hydration entirely.
*   **`chunkById()`:** Never use `chunk()` on columns you are updating. `chunkById()` ensures that even if the results change order due to the update, you don't skip rows.

---

## 2. Advanced Relationship Orchestration

### Subquery Selects (`addSelect`)
Instead of loading a relationship just to get one column (e.g., the last login date), use a subquery select. This keeps the result set in a single flat SQL row.
```php
return User::addSelect(['last_login_at' => Login::select('created_at')
    ->whereColumn('user_id', 'users.id')
    ->latest()
    ->limit(1)
])->get();
```

### Deferred Eager Loading
Use `loadMissing()` instead of `load()`. This prevents the N+1 problem by only fetching the relationship if it hasn't already been loaded into the collection memory.

---

## 3. Custom Collections: The Domain Logic Hub
Stop putting complex array manipulation in your Controllers or Models. If you have logic that operates on a *group* of models, override the `newCollection()` method in your Model.

```php
// In User.php
public function newCollection(array $models = []): UserCollection 
{
    return new UserCollection($models);
}

// In UserCollection.php
class UserCollection extends \Illuminate\Database\Eloquent\Collection 
{
    public function totalSubscriptionValue(): float 
    {
        return $this->sum(fn($user) => $user->subscription->price);
    }
}

// Usage:
$users = User::with('subscription')->get();
$total = $users->totalSubscriptionValue(); // Clean, encapsulated, and testable.
```

---

## 4. Scaling the Database Layer

### Read/Write Splitting & Replica Lag
In high-traffic environments, you’ll have one Primary (Write) and multiple Replicas (Read). 
*   **The Problem:** You write to Primary, then redirect to a "Success" page which reads from a Replica. But the data hasn't synced yet (Replica Lag).
*   **The Fix:** Use the `sticky` option in `config/database.php`.
```php
'mysql' => [
    'read' => ['host' => '192.168.1.1'],
    'write' => ['host' => '192.168.1.2'],
    'sticky' => true, // Very Important
],
```
Laravel will track if a write has occurred during the current request cycle and, if so, use the Primary for all subsequent reads in that request.

### Atomic Locks & Transactions
A Senior Dev knows `DB::transaction()` is not enough for high-concurrency race conditions (like a "Limited Edition" sale).
*   **Database Level:** `sharedLock()` (Lock for read) vs `lockForUpdate()` (Lock for write).
*   **Application Level:** Use the Cache Lock (Redis) to prevent two processes from entering the same block of code.
```php
Cache::lock('process-order-'.$id, 10)->block(5, function () {
    // This code is only executed by one process at a time
});
```

---

## 5. Zero-Downtime Migration Strategies
When your `users` table has 50 million rows, `Schema::table('users', ...)` will lock the table and crash your app.

**The Strategy:**
1.  **Avoid `->default()` on existing columns:** This triggers a full table rewrite.
2.  **The "Add, Sync, Swap" Pattern:** 
    *   Add a new nullable column.
    *   Sync data in background chunks.
    *   Update code to write to both columns.
    *   Drop the old column later.
3.  **Modern Tools:** Mention that for extreme scale, we use **Native JSON** columns for flexibility or **Virtual As-Generated** columns for indexing JSON data without duplicating storage.

---

## 6. Eloquent Internals: Global Scopes & Macros
At 6+ years, you should be building "Tenant-Aware" or "Status-Aware" systems.
*   **Global Scopes:** Use them for Multi-tenancy (e.g., `where('team_id', auth()->user()->team_id)`).
*   **Macros:** Extend the Query Builder globally.
```php
// In a Service Provider
Builder::macro('whereLike', function($attributes, string $searchTerm) {
    return $this->where(function (Builder $query) use ($attributes, $searchTerm) {
        foreach (array_wrap($attributes) as $attribute) {
            $query->orWhere($attribute, 'LIKE', "%{$searchTerm}%");
        }
    });
});
```

---

### Architect's Checkpoint:
*   Do you know when to use a **Polymorphic Relationship** vs. a **Table-per-type** inheritance?
*   Do you understand the performance difference between `count(*)` and `count(id)` in MySQL?
*   Are you using **Value Objects** (Casts) to handle complex data types (like Money or Address) within Eloquent?

**Ready for Part 3: The Asynchronous Ecosystem & Performance (Queues, Octane, and WebSockets)?** Say **"Go Part 3"**.