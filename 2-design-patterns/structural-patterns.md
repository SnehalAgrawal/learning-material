# Structural Design Patterns

### 1. Overview

Structural patterns focus on **how classes and objects are composed** to form larger structures. They ensure that if one part of a system changes, the entire structure doesn't need to be rebuilt.

For senior roles, these patterns are about:

* **Interface Compatibility** → Making two things work together that weren't designed to.
* **Composition over Inheritance** → Adding functionality by wrapping objects rather than extending classes.

👉 Think of Structural Patterns as *“The Architecture”* — they define how different blocks fit together to create a stable building.

### 2. Key Concepts

#### 2.1. **Adapter**

Allows objects with incompatible interfaces to collaborate.

💡 **Bad Example (Incompatible API)**

```javascript
// New system expects JSON
class PaymentProcessor {
  process(payment) {
    console.log(`Processing payment for user ${payment.userId}`);
  }
}

// Legacy system sends string
const legacyPayment = "userId=42";

new PaymentProcessor().process(legacyPayment); // ❌ Breaks
```

```python
class PaymentProcessor:
    def process(self, payment):
        print(f"Processing payment for user {payment['userId']}")

legacy_payment = "userId=42"
PaymentProcessor().process(legacy_payment)  # ❌ Breaks
```

❌ Problem: System expects structured data but receives raw string.

✅ **Better (Adapter)**

```javascript
class PaymentAdapter {
  constructor(legacyString) {
    const [key, value] = legacyString.split("=");
    this.payment = { [key]: value };
  }
}

const adapter = new PaymentAdapter("userId=42");
new PaymentProcessor().process(adapter.payment);
```

```python
class PaymentAdapter:
    def __init__(self, legacy_string):
        key, value = legacy_string.split("=")
        self.payment = {key: value}

adapter = PaymentAdapter("userId=42")
PaymentProcessor().process(adapter.payment)
```

#### 2.2. **Decorator**

Lets you attach new behaviors to objects by placing them inside special wrapper objects.

💡 **Bad Example (Class explosion)**

```javascript
class Notifier {}
class EmailNotifier extends Notifier {}
class SMSNotifier extends Notifier {}
class EmailSMSNotifier extends Notifier {} // ❌ Explosion
```

```python
class Notifier: pass
class EmailNotifier(Notifier): pass
class SMSNotifier(Notifier): pass
class EmailSMSNotifier(Notifier): pass  # ❌ Too many combinations
```

❌ Problem: Every combination creates a new class.

✅ **Better (Decorator)**

```javascript
class Notifier {
  send(message) {
    console.log(message);
  }
}

class EmailDecorator {
  constructor(notifier) {
    this.notifier = notifier;
  }
  send(message) {
    this.notifier.send(message);
    console.log("Sending Email:", message);
  }
}

class SMSDecorator {
  constructor(notifier) {
    this.notifier = notifier;
  }
  send(message) {
    this.notifier.send(message);
    console.log("Sending SMS:", message);
  }
}

const notifier = new SMSDecorator(new EmailDecorator(new Notifier()));
notifier.send("Hello!");
```

```python
class Notifier:
    def send(self, message):
        print(message)

class EmailDecorator:
    def __init__(self, notifier):
        self.notifier = notifier

    def send(self, message):
        self.notifier.send(message)
        print("Sending Email:", message)

class SMSDecorator:
    def __init__(self, notifier):
        self.notifier = notifier

    def send(self, message):
        self.notifier.send(message)
        print("Sending SMS:", message)

notifier = SMSDecorator(EmailDecorator(Notifier()))
notifier.send("Hello!")
```

#### 2.3. **Facade**

Provides a simplified interface to a complex set of classes (subsystem).

💡 **Example**: A `OrderFacade` that handles `Inventory`, `Payment`, and `Shipping` in one `placeOrder()` call.

```javascript
class Inventory {
  check(item) { console.log("Inventory checked"); }
}

class Payment {
  pay(amount) { console.log("Payment processed"); }
}

class Shipping {
  ship(item) { console.log("Item shipped"); }
}

// Facade
class OrderFacade {
  constructor() {
    this.inventory = new Inventory();
    this.payment = new Payment();
    this.shipping = new Shipping();
  }

  placeOrder(item, amount) {
    this.inventory.check(item);
    this.payment.pay(amount);
    this.shipping.ship(item);
    console.log("Order completed");
  }
}

new OrderFacade().placeOrder("Laptop", 1000);
```

```python
class Inventory:
    def check(self, item):
        print("Inventory checked")

class Payment:
    def pay(self, amount):
        print("Payment processed")

class Shipping:
    def ship(self, item):
        print("Item shipped")

class OrderFacade:
    def __init__(self):
        self.inventory = Inventory()
        self.payment = Payment()
        self.shipping = Shipping()

    def place_order(self, item, amount):
        self.inventory.check(item)
        self.payment.pay(amount)
        self.shipping.ship(item)
        print("Order completed")

OrderFacade().place_order("Laptop", 1000)
```

#### 2.4. **Proxy**

Provides a placeholder for another object to control access (lazy loading, security, caching).

💡 **Example**: A `VideoProxy` that only downloads a heavy video file from the server when the `play()` method is actually called.

```javascript
class RealVideo {
  constructor(filename) {
    this.filename = filename;
    this.loadFromServer();
  }

  loadFromServer() {
    console.log("Loading video from server...");
  }

  play() {
    console.log("Playing", this.filename);
  }
}

class VideoProxy {
  constructor(filename) {
    this.filename = filename;
    this.realVideo = null;
  }

  play() {
    if (!this.realVideo) {
      this.realVideo = new RealVideo(this.filename); // Lazy load
    }
    this.realVideo.play();
  }
}

const video = new VideoProxy("movie.mp4");
video.play(); // Loads + plays
video.play(); // Just plays
```

```python
class RealVideo:
    def __init__(self, filename):
        self.filename = filename
        self.load_from_server()

    def load_from_server(self):
        print("Loading video from server...")

    def play(self):
        print("Playing", self.filename)

class VideoProxy:
    def __init__(self, filename):
        self.filename = filename
        self.real_video = None

    def play(self):
        if not self.real_video:
            self.real_video = RealVideo(self.filename)  # Lazy load
        self.real_video.play()

video = VideoProxy("movie.mp4")
video.play()
video.play()
```

### 3. Real-World Usage

* **Third-Party Integration** → Using an Adapter to map a legacy SOAP API to your REST internal format.
* **Middleware** → Express.js or Python decorators (adding logging/auth to functions) are structural in nature.
* **BFF (Backend-for-Frontend)** → A Facade that aggregates 5 microservice calls into 1 response for mobile.
* **Virtual Proxies** → Heavy images in a browser that only load when scrolled into view (lazy loading).

### 4. Tradeoffs

* **Decorator Complexity** → Can result in many small, nested objects ("Matryoshka doll" effect), making debugging hard.
* **Adapter Overhead** → Adds a layer of mapping code that must be maintained.
* **Facade Limitations** → A Facade might hide *too much*, preventing developers from using advanced features of the subsystem.

### 5. When NOT to Use

* **Decorator** → If you only have one or two variations, simple inheritance or an `if` block is cleaner.
* **Adapter** → If you have control over both sides of the interface, refactor them to match instead of adding a wrapper.

### 6. Interview Focus

* **Decorator vs. Inheritance** → "Why would you use a Decorator instead of subclasses for adding features like encryption?"
* **Mapping Legacy Systems** → "How do you handle a messy third-party library without letting its bad API leak into your codebase?" (Facade/Adapter).
* **Security & Performance** → "How would you implement access control or caching for a heavy resource without changing its code?" (Proxy).

### 7. Common Mistakes

* **Over-adapting** → Creating Adapters for things that are already consistent.
* **Leaking Abstractions** → A Proxy or Facade that forces the user to handle errors from the "hidden" inner classes.
* **Confusing Proxy with Decorator** → Decorator adds **behavior**; Proxy manages **access**.
