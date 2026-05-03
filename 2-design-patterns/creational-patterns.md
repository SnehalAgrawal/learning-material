# Creational Design Patterns

### 1. Overview

Creational patterns are all about **object creation mechanisms**. Instead of instantiating objects directly (which can couple your code to specific classes), these patterns provide ways to create objects while hiding the creation logic. 

For senior developers, these patterns are about:
*   **Decoupling** → Your code shouldn't care *how* an object is made, just that it works.
*   **Flexibility** → Changing the type of object created without rewriting half the system.

👉 Think of Creational Patterns as *“The Factory Floor”* — they handle the messy details of construction so the rest of the app can focus on business.

### 2. Key Concepts

#### 2.1. **Singleton**

Ensures a class has only one instance and provides a global point of access.

💡 **Bad Example (Multiple instances causing state conflict)**

```javascript
class Database {
  constructor() {
    this.connection = Math.random();
  }
}

const db1 = new Database();
const db2 = new Database(); // Different connection!
```

❌ Problem: Every `new Database()` creates a new connection, wasting resources and losing shared state.

✅ **Better (Singleton)**

```javascript
class Database {
  constructor() {
    if (Database.instance) return Database.instance;
    this.connection = "connected";
    Database.instance = this;
  }
}
```

```python
class Database:
    _instance = None
    def __new__(cls):
        if not cls._instance:
            cls._instance = super().__new__(cls)
            cls._instance.connection = "connected"
        return cls._instance
```

#### 2.2. **Factory Method**

Defines an interface for creating an object but lets subclasses decide which class to instantiate.

💡 **Bad Example (Hardcoded dependencies)**

```javascript
class EmailNotification {
  send(msg) {
    console.log("Sending Email:", msg);
  }
}

class SMSNotification {
  send(msg) {
    console.log("Sending SMS:", msg);
  }
}

// Client code
function notify(type, msg) {
  let notifier;

  if (type === "email") {
    notifier = new EmailNotification();
  } else if (type === "sms") {
    notifier = new SMSNotification();
  }

  notifier.send(msg);
}
```

❌ Problem: Adding a "Plane" requires modifying the core creation function (OCP violation).

✅ **Better (Factory Method)**

```javascript
class NotificationFactory {
  constructor() {
    this.creators = {};
  }

  register(type, creator) {
    this.creators[type] = creator;
  }

  createNotification(type) {
    const Creator = this.creators[type];
    if (!Creator) throw new Error("Unknown type");
    return new Creator();
  }
}

// Usage
const factory = new NotificationFactory();
factory.register("email", EmailNotification);
factory.register("sms", SMSNotification);

factory.createNotification("sms").send("Hi!");
```

```python
class NotificationFactory:
    _creators = {}

    @classmethod
    def register(cls, type, creator):
        cls._creators[type] = creator

    @classmethod
    def create_notification(cls, type):
        if type not in cls._creators:
            raise ValueError("Unknown type")
        return cls._creators[type]()

# Register
NotificationFactory.register("email", EmailNotification)
NotificationFactory.register("sms", SMSNotification)

# Use
notifier = NotificationFactory.create_notification("sms")
notifier.send("Factory pattern in action!")
```

#### 2.3. **Builder**

Separates the construction of a complex object from its representation.

💡 **Bad Example (The "Telescoping Constructor")**

```javascript
class Pizza {
  constructor(size, cheese, pepperoni, bacon, onions, mushrooms) {
    // Hard to read: new Pizza(12, true, false, true, true, false)
  }
}
```

❌ Problem: Massive constructors with many optional parameters are error-prone and unreadable.

✅ **Better (Builder)**

```javascript
class PizzaBuilder {
  addCheese() { this.cheese = true; return this; }
  addPepperoni() { this.pepperoni = true; return this; }
  build() { return new Pizza(this); }
}

const pizza = new PizzaBuilder().addCheese().addPepperoni().build();
```

```python
class PizzaBuilder:
    def __init__(self):
        self.pizza = Pizza()
    def add_cheese(self):
        self.pizza.cheese = True
        return self
    def build(self):
        return self.pizza
```

#### 2.4. **Abstract Factory**

Provides an interface for creating families of related objects without specifying concrete classes.

💡 **Example**: Creating a `UIFactory` that can produce both a `Button` and a `Checkbox` in either `MacOS` or `Windows` style.
```javascript
// Abstract (interfaces via base classes)
class Button {
  render() {}
}

class Checkbox {
  render() {}
}

// 🟦 MacOS implementations
class MacButton extends Button {
  render() {
    console.log("🍎 MacOS Button");
  }
}

class MacCheckbox extends Checkbox {
  render() {
    console.log("🍎 MacOS Checkbox");
  }
}

// 🟩 Windows implementations
class WinButton extends Button {
  render() {
    console.log("🪟 Windows Button");
  }
}

class WinCheckbox extends Checkbox {
  render() {
    console.log("🪟 Windows Checkbox");
  }
}

// 🏭 Abstract Factory
class UIFactory {
  createButton() {}
  createCheckbox() {}
}

// Concrete Factories
class MacFactory extends UIFactory {
  createButton() {
    return new MacButton();
  }

  createCheckbox() {
    return new MacCheckbox();
  }
}

class WindowsFactory extends UIFactory {
  createButton() {
    return new WinButton();
  }

  createCheckbox() {
    return new WinCheckbox();
  }
}

// 🧑‍💻 Client code
function renderUI(factory) {
  const button = factory.createButton();
  const checkbox = factory.createCheckbox();

  button.render();
  checkbox.render();
}

// Usage
const factory = new MacFactory(); // switch to WindowsFactory easily
renderUI(factory);
```

```python
from abc import ABC, abstractmethod

# Abstract Products
class Button(ABC):
    @abstractmethod
    def render(self):
        pass

class Checkbox(ABC):
    @abstractmethod
    def render(self):
        pass


# 🟦 MacOS
class MacButton(Button):
    def render(self):
        print("🍎 MacOS Button")

class MacCheckbox(Checkbox):
    def render(self):
        print("🍎 MacOS Checkbox")


# 🟩 Windows
class WinButton(Button):
    def render(self):
        print("🪟 Windows Button")

class WinCheckbox(Checkbox):
    def render(self):
        print("🪟 Windows Checkbox")


# Abstract Factory
class UIFactory(ABC):
    @abstractmethod
    def create_button(self): pass

    @abstractmethod
    def create_checkbox(self): pass


# Concrete Factories
class MacFactory(UIFactory):
    def create_button(self):
        return MacButton()

    def create_checkbox(self):
        return MacCheckbox()


class WindowsFactory(UIFactory):
    def create_button(self):
        return WinButton()

    def create_checkbox(self):
        return WinCheckbox()


# Client
def render_ui(factory: UIFactory):
    button = factory.create_button()
    checkbox = factory.create_checkbox()

    button.render()
    checkbox.render()


# Usage
factory = WindowsFactory()
render_ui(factory)
```

#### 2.5. **Prototype**

Creates new objects by copying an existing object (cloning).

💡 **Example**: When creating an object is expensive (e.g., requires a heavy DB query), you clone an existing instance and modify only what's needed.
```javascript
class Prototype {
  clone() {
    throw new Error("Clone method not implemented");
  }
}

class User extends Prototype {
  constructor(name, preferences) {
    super();
    this.name = name;
    this.preferences = preferences;
  }

  clone() {
    // Deep copy important!
    return new User(
      this.name,
      JSON.parse(JSON.stringify(this.preferences))
    );
  }
}

// Usage
const original = new User("Snehal", { theme: "dark" });

const copy = original.clone();
copy.name = "Rahul";
copy.preferences.theme = "light";

console.log(original);
console.log(copy);
```

```python
import copy

class User:
    def __init__(self, name, preferences):
        self.name = name
        self.preferences = preferences

    def clone(self):
        return copy.deepcopy(self)


# Usage
original = User("Snehal", {"theme": "dark"})

copy_user = original.clone()
copy_user.name = "Rahul"
copy_user.preferences["theme"] = "light"

print(original.__dict__)
print(copy_user.__dict__)
```

### 3. Real-World Usage

*   **Database Connections** → Singleton for connection pools.
*   **UI Frameworks** → Abstract Factory for themed components (Material vs iOS).
*   **Query Builders** → Builder pattern for complex SQL queries (e.g., Knex.js, SQLAlchemy).
*   **Logging** → Factory pattern for different sinks (Console, File, CloudWatch).

### 4. Tradeoffs

*   **Singleton Complexity** → Global state is hard to test; often considered an anti-pattern in favor of DI.
*   **Factory Boilerplate** → Requires many small classes, which can feel like overkill for simple apps.
*   **Builder Verbosity** → Requires writing an extra builder class for every complex entity.

### 5. When NOT to Use

*   **Singleton** → When state needs to be isolated for testing or multiple concurrent user sessions.
*   **Factory** → For simple `new User()` calls where no abstraction or logic is required (Avoid YAGNI).

### 6. Interview Focus

*   **Singleton vs. DI** → "If Singleton is a pattern, why do we use Dependency Injection instead today?"
*   **Builder Choice** → "When would you use a Builder over a regular constructor with an Options object?"
    ```javascript
    // Options object
    new Car({
      engine: "V8",
      color: "red",
      sunroof: true,
      gps: true,
      heatedSeats: true,
      sportPackage: true,
      suspension: "sport",
      ...
    });

    // Builder
    class Car {
      constructor(builder) {
        this.engine = builder.engine;
        this.color = builder.color;
        this.sunroof = builder.sunroof;
      }
    }

    class CarBuilder {
      setEngine(engine) {
        this.engine = engine;
        return this;
      }

      setColor(color) {
        this.color = color;
        return this;
      }

      enableSunroof() {
        this.sunroof = true;
        return this;
      }

      build() {
        if (!this.engine) {
          throw new Error("Engine is required");
        }
        return new Car(this);
      }
    }

    // Usage
    const car = new CarBuilder()
      .setEngine("V8")
      .setColor("black")
      .enableSunroof()
      .build();
    ```
*   **Abstract Factory** → "How do you ensure a family of objects (e.g., Dark Mode components) stay consistent?"

### 7. Common Mistakes

*   **Non-Thread-Safe Singletons** → Implementing Singletons without handling race conditions (double-checked locking).
*   **Shallow Copying in Prototype** → Cloning an object but accidentally sharing nested references.
*   **The "One Giant Factory"** → Creating a factory that knows about everything, violating SRP.
