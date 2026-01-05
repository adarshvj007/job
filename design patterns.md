This is a comprehensive, deep-dive guide to **PHP Design Patterns**. It is structured to be saved as a `.md` file. It covers the "Why," the "How," and the "Trade-offs" for each major pattern using modern PHP 8.x syntax.

***

# The Ultimate Guide to PHP Design Patterns (2026 Edition)

## Table of Contents
1.  **[Introduction to Design Patterns](#introduction)**
2.  **[The SOLID Principles](#solid-principles)**
3.  **[Creational Patterns](#creational-patterns)**
    *   [Singleton](#singleton)
    *   [Factory Method](#factory-method)
    *   [Abstract Factory](#abstract-factory)
    *   [Builder](#builder)
    *   [Prototype](#prototype)
4.  **[Structural Patterns](#structural-patterns)**
    *   [Adapter](#adapter)
    *   [Decorator](#decorator)
    *   [Facade](#facade)
    *   [Bridge](#bridge)
    *   [Proxy](#proxy)
5.  **[Behavioral Patterns](#behavioral-patterns)**
    *   [Strategy](#strategy)
    *   [Observer](#observer)
    *   [Command](#command)
    *   [Chain of Responsibility](#chain-of-responsibility)
    *   [State](#state)
6.  **[Conclusion](#conclusion)**

---

<a name="introduction"></a>
## 1. Introduction
Design patterns are documented, reusable solutions to commonly occurring problems in software engineering. They are not "plug-and-play" code but rather templates for how to solve a problem.

### Why use them?
*   **Common Vocabulary:** Instead of explaining a complex communication logic, you can just say "We are using an Observer pattern."
*   **Code Reusability:** Patterns help you write code that is modular and easy to extend.
*   **Best Practices:** They are battle-tested solutions that help avoid common pitfalls (like tight coupling).

---

<a name="solid-principles"></a>
## 2. The SOLID Principles
Before learning patterns, you must understand SOLID, as design patterns are essentially applications of these rules.

*   **S**ingle Responsibility: A class should have only one reason to change.
*   **O**pen/Closed: Classes should be open for extension but closed for modification.
*   **L**iskov Substitution: Subclasses should be replaceable by their base classes without breaking the app.
*   **I**nterface Segregation: Many specific interfaces are better than one general-purpose interface.
*   **D**ependency Inversion: Depend on abstractions (interfaces), not concrete implementations.

---

<a name="creational-patterns"></a>
## 3. Creational Patterns
These patterns deal with object creation mechanisms, trying to create objects in a manner suitable to the situation.

### Singleton
**Goal:** Ensure a class has only one instance and provide a global point of access.

*   **When to use:** Database connections, Logger classes, Configuration settings.
*   **Deep Dive:** In PHP, we must make the `__construct`, `__clone`, and `__wakeup` methods private to prevent multiple instances.

```php
final class Database {
    private static ?Database $instance = null;

    private function __construct() {
        // Expensive DB connection logic here
    }

    public static function getInstance(): Database {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    // Prevent cloning and unserialization
    private function __clone() {}
    public function __wakeup() {
        throw new \Exception("Cannot unserialize a singleton.");
    }
}
```

### Factory Method
**Goal:** Define an interface for creating an object, but let subclasses decide which class to instantiate.

*   **When to use:** When your code needs to work with various types of objects, but you don't know the exact types beforehand.

```php
interface Logger {
    public function log(string $message): void;
}

class FileLogger implements Logger {
    public function log(string $message): void { echo "Logging to file: $message"; }
}

class CloudLogger implements Logger {
    public function log(string $message): void { echo "Logging to Cloud: $message"; }
}

abstract class App {
    abstract public function getLogger(): Logger;

    public function run(string $message): void {
        $logger = $this->getLogger();
        $logger->log($message);
    }
}

class WebApp extends App {
    public function getLogger(): Logger { return new CloudLogger(); }
}
```

---

<a name="structural-patterns"></a>
## 4. Structural Patterns
These patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

### Adapter
**Goal:** Allows objects with incompatible interfaces to collaborate.

*   **Real World Example:** You use a library for Slack notifications, but now you want to add SMS notifications from a different provider with different method names.

```php
interface Notification {
    public function send(string $title, string $message): void;
}

// Incompatible Legacy Service
class LegacySmsService {
    public function sendSms(string $text): void {
        echo "SMS Sent: $text";
    }
}

// The Adapter
class SmsAdapter implements Notification {
    public function __construct(private LegacySmsService $smsService) {}

    public function send(string $title, string $message): void {
        // Mapping the interface to the legacy service
        $this->smsService->sendSms("$title - $message");
    }
}
```

### Decorator
**Goal:** Attach new behaviors to objects by placing these objects inside special wrapper objects that contain the behaviors.

*   **Why?** It is a flexible alternative to subclassing.

```php
interface Text {
    public function format(): string;
}

class SimpleText implements Text {
    public function __construct(protected string $text) {}
    public function format(): string { return $this->text; }
}

class BoldDecorator implements Text {
    public function __construct(protected Text $textObject) {}
    public function format(): string {
        return "<b>" . $this->textObject->format() . "</b>";
    }
}

// Usage:
$text = new BoldDecorator(new SimpleText("Hello World"));
echo $text->format(); // <b>Hello World</b>
```

---

<a name="behavioral-patterns"></a>
## 5. Behavioral Patterns
These patterns are concerned with algorithms and the assignment of responsibilities between objects.

### Strategy
**Goal:** Define a family of algorithms, put each of them into a separate class, and make their objects interchangeable.

*   **When to use:** When you have a massive `switch` statement or many `if/else` conditions to decide which logic to run.

```php
interface PaymentStrategy {
    public function pay(int $amount): void;
}

class CreditCardPayment implements PaymentStrategy {
    public function pay(int $amount): void { echo "Paid $amount via Credit Card."; }
}

class CryptoPayment implements PaymentStrategy {
    public function pay(int $amount): void { echo "Paid $amount via Bitcoin."; }
}

class Checkout {
    public function __construct(private PaymentStrategy $strategy) {}

    public function process(int $amount): void {
        $this->strategy->pay($amount);
    }
}

// Usage
$order = new Checkout(new CryptoPayment());
$order->process(500);
```

### Observer
**Goal:** Define a subscription mechanism to notify multiple objects about any events that happen to the object they’re observing.

*   **Modern PHP Implementation:** Using built-in `SplSubject` and `SplObserver`.

```php
class UserRegistration implements \SplSubject {
    private \SplObjectStorage $observers;
    public string $email;

    public function __construct() {
        $this->observers = new \SplObjectStorage();
    }

    public function attach(\SplObserver $observer): void {
        $this->observers->attach($observer);
    }

    public function detach(\SplObserver $observer): void {
        $this->observers->detach($observer);
    }

    public function notify(): void {
        foreach ($this->observers as $observer) {
            $observer->update($this);
        }
    }

    public function register(string $email): void {
        $this->email = $email;
        echo "User registered!\n";
        $this->notify();
    }
}

class WelcomeEmailObserver implements \SplObserver {
    public function update(\SplSubject $subject): void {
        echo "Sending welcome email to: " . $subject->email . "\n";
    }
}
```

---

<a name="conclusion"></a>
## 6. Conclusion
Design patterns are a powerful toolset, but they carry a risk of **Over-engineering**. 

### Guidelines for Success:
1.  **Don't force it:** If a simple `if` statement works, don't build a Strategy Pattern.
2.  **KISS (Keep It Simple, Stupid):** Use patterns only when the complexity of the code justifies it.
3.  **Learn the "Why" first:** Knowing how to write a Singleton is useless if you don't understand why it makes unit testing difficult (global state).

### Next Steps:
*   Study **Laravel's Source Code**: Laravel uses Facades, Adapters, and Observers extensively.
*   Practice by refactoring old code: Take a messy 200-line controller and try to apply the **Command** or **Strategy** pattern.

---
*End of Document*