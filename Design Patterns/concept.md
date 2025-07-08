# SOLID Principles in JavaScript

## What are SOLID Principles?

SOLID is an acronym for five design principles that make software designs more understandable, flexible, and maintainable. These principles were introduced by Robert C. Martin (Uncle Bob) and are fundamental to object-oriented programming and software design.

**S** - Single Responsibility Principle (SRP)  
**O** - Open/Closed Principle (OCP)  
**L** - Liskov Substitution Principle (LSP)  
**I** - Interface Segregation Principle (ISP)  
**D** - Dependency Inversion Principle (DIP)

---

## 1. Single Responsibility Principle (SRP)

**Definition**: A class should have only one reason to change, meaning it should have only one job or responsibility.

### ❌ Bad Example - Violating SRP

```javascript
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  // User data management
  getName() {
    return this.name;
  }

  getEmail() {
    return this.email;
  }

  // Email functionality - WRONG! Not user's responsibility
  sendEmail(message) {
    console.log(`Sending email to ${this.email}: ${message}`);
    // Email sending logic
  }

  // Database operations - WRONG! Not user's responsibility
  save() {
    console.log(`Saving user ${this.name} to database`);
    // Database saving logic
  }

  // Validation - WRONG! Not user's responsibility
  validateEmail() {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(this.email);
  }
}
```

### ✅ Good Example - Following SRP

```javascript
// User class - Only handles user data
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  getName() {
    return this.name;
  }

  getEmail() {
    return this.email;
  }
}

// EmailService class - Only handles email operations
class EmailService {
  sendEmail(user, message) {
    console.log(`Sending email to ${user.getEmail()}: ${message}`);
    // Email sending logic
  }
}

// UserRepository class - Only handles database operations
class UserRepository {
  save(user) {
    console.log(`Saving user ${user.getName()} to database`);
    // Database saving logic
  }

  findById(id) {
    // Database retrieval logic
  }
}

// UserValidator class - Only handles validation
class UserValidator {
  validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }

  validateUser(user) {
    return this.validateEmail(user.getEmail()) && user.getName().length > 0;
  }
}

// Usage
const user = new User("John Doe", "john@example.com");
const emailService = new EmailService();
const userRepository = new UserRepository();
const userValidator = new UserValidator();

if (userValidator.validateUser(user)) {
  userRepository.save(user);
  emailService.sendEmail(user, "Welcome!");
}
```

---

## 2. Open/Closed Principle (OCP)

**Definition**: Software entities should be open for extension but closed for modification. You should be able to add new functionality without changing existing code.

### ❌ Bad Example - Violating OCP

```javascript
class DiscountCalculator {
  calculateDiscount(customer, amount) {
    if (customer.type === 'regular') {
      return amount * 0.1;
    } else if (customer.type === 'premium') {
      return amount * 0.2;
    } else if (customer.type === 'vip') {
      return amount * 0.3;
    }
    // Adding new customer type requires modifying this class
    return 0;
  }
}
```

### ✅ Good Example - Following OCP

```javascript
// Base discount strategy
class DiscountStrategy {
  calculateDiscount(amount) {
    throw new Error("calculateDiscount method must be implemented");
  }
}

// Concrete discount strategies
class RegularCustomerDiscount extends DiscountStrategy {
  calculateDiscount(amount) {
    return amount * 0.1;
  }
}

class PremiumCustomerDiscount extends DiscountStrategy {
  calculateDiscount(amount) {
    return amount * 0.2;
  }
}

class VIPCustomerDiscount extends DiscountStrategy {
  calculateDiscount(amount) {
    return amount * 0.3;
  }
}

// New discount type can be added without modifying existing code
class StudentDiscount extends DiscountStrategy {
  calculateDiscount(amount) {
    return amount * 0.15;
  }
}

// Context class
class DiscountCalculator {
  constructor(discountStrategy) {
    this.discountStrategy = discountStrategy;
  }

  calculateDiscount(amount) {
    return this.discountStrategy.calculateDiscount(amount);
  }
}

// Usage
const regularDiscount = new DiscountCalculator(new RegularCustomerDiscount());
const premiumDiscount = new DiscountCalculator(new PremiumCustomerDiscount());
const studentDiscount = new DiscountCalculator(new StudentDiscount());

console.log(regularDiscount.calculateDiscount(100)); // 10
console.log(premiumDiscount.calculateDiscount(100)); // 20
console.log(studentDiscount.calculateDiscount(100)); // 15
```

---

## 3. Liskov Substitution Principle (LSP)

**Definition**: Objects of a superclass should be replaceable with objects of its subclasses without breaking the application. Subclasses should be substitutable for their base classes.

### ❌ Bad Example - Violating LSP

```javascript
class Bird {
  fly() {
    return "Flying in the sky";
  }
}

class Eagle extends Bird {
  fly() {
    return "Eagle soaring high";
  }
}

class Penguin extends Bird {
  fly() {
    throw new Error("Penguins can't fly!"); // Violates LSP
  }
}

// This breaks when penguin is used
function makeBirdFly(bird) {
  return bird.fly(); // Will throw error for Penguin
}

const eagle = new Eagle();
const penguin = new Penguin();

console.log(makeBirdFly(eagle)); // Works
console.log(makeBirdFly(penguin)); // Throws error - LSP violated
```

### ✅ Good Example - Following LSP

```javascript
// Base class with common behavior
class Bird {
  constructor(name) {
    this.name = name;
  }

  eat() {
    return `${this.name} is eating`;
  }

  sleep() {
    return `${this.name} is sleeping`;
  }
}

// Separate interface for flying behavior
class FlyingBird extends Bird {
  fly() {
    return `${this.name} is flying`;
  }
}

// Separate interface for swimming behavior
class SwimmingBird extends Bird {
  swim() {
    return `${this.name} is swimming`;
  }
}

class Eagle extends FlyingBird {
  fly() {
    return `${this.name} is soaring high`;
  }
}

class Penguin extends SwimmingBird {
  swim() {
    return `${this.name} is swimming gracefully`;
  }
}

class Duck extends Bird {
  fly() {
    return `${this.name} is flying`;
  }

  swim() {
    return `${this.name} is swimming`;
  }
}

// Functions that work with appropriate interfaces
function makeFlyingBirdFly(bird) {
  if (bird instanceof FlyingBird || typeof bird.fly === 'function') {
    return bird.fly();
  }
  throw new Error("Bird cannot fly");
}

function makeBirdEat(bird) {
  return bird.eat(); // All birds can eat
}

// Usage
const eagle = new Eagle("Golden Eagle");
const penguin = new Penguin("Emperor Penguin");
const duck = new Duck("Mallard Duck");

console.log(makeBirdEat(eagle));   // Works
console.log(makeBirdEat(penguin)); // Works
console.log(makeBirdEat(duck));    // Works

console.log(makeFlyingBirdFly(eagle)); // Works
console.log(makeFlyingBirdFly(duck));  // Works
// console.log(makeFlyingBirdFly(penguin)); // Appropriately doesn't work
```

---

## 4. Interface Segregation Principle (ISP)

**Definition**: No client should be forced to depend on methods it does not use. It's better to have many specific interfaces than one general-purpose interface.

### ❌ Bad Example - Violating ISP

```javascript
// Large interface that forces all implementations to implement all methods
class Printer {
  print(document) {
    throw new Error("print method must be implemented");
  }

  scan(document) {
    throw new Error("scan method must be implemented");
  }

  fax(document) {
    throw new Error("fax method must be implemented");
  }

  copy(document) {
    throw new Error("copy method must be implemented");
  }
}

// Simple printer forced to implement all methods
class SimplePrinter extends Printer {
  print(document) {
    console.log(`Printing: ${document}`);
  }

  scan(document) {
    throw new Error("Simple printer cannot scan"); // Forced to implement
  }

  fax(document) {
    throw new Error("Simple printer cannot fax"); // Forced to implement
  }

  copy(document) {
    throw new Error("Simple printer cannot copy"); // Forced to implement
  }
}
```

### ✅ Good Example - Following ISP

```javascript
// Segregated interfaces
class Printable {
  print(document) {
    throw new Error("print method must be implemented");
  }
}

class Scannable {
  scan(document) {
    throw new Error("scan method must be implemented");
  }
}

class Faxable {
  fax(document) {
    throw new Error("fax method must be implemented");
  }
}

class Copyable {
  copy(document) {
    throw new Error("copy method must be implemented");
  }
}

// Simple printer only implements what it needs
class SimplePrinter extends Printable {
  print(document) {
    console.log(`Printing: ${document}`);
  }
}

// Multi-function printer implements multiple interfaces
class MultiFunctionPrinter extends Printable {
  constructor() {
    super();
    this.scanner = new Scanner();
    this.faxMachine = new FaxMachine();
  }

  print(document) {
    console.log(`Printing: ${document}`);
  }

  scan(document) {
    return this.scanner.scan(document);
  }

  fax(document) {
    return this.faxMachine.fax(document);
  }

  copy(document) {
    const scanned = this.scan(document);
    this.print(scanned);
    return `Copied: ${document}`;
  }
}

// Separate implementations for specific functionalities
class Scanner extends Scannable {
  scan(document) {
    console.log(`Scanning: ${document}`);
    return `Scanned version of ${document}`;
  }
}

class FaxMachine extends Faxable {
  fax(document) {
    console.log(`Faxing: ${document}`);
    return `Faxed: ${document}`;
  }
}

// Usage
const simplePrinter = new SimplePrinter();
const multiFunctionPrinter = new MultiFunctionPrinter();
const scanner = new Scanner();

simplePrinter.print("Document 1");
multiFunctionPrinter.print("Document 2");
multiFunctionPrinter.scan("Document 3");
multiFunctionPrinter.copy("Document 4");
scanner.scan("Document 5");
```

---

## 5. Dependency Inversion Principle (DIP)

**Definition**: High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions.

### ❌ Bad Example - Violating DIP

```javascript
// Low-level module
class MySQLDatabase {
  save(data) {
    console.log(`Saving data to MySQL: ${JSON.stringify(data)}`);
  }

  find(id) {
    console.log(`Finding data in MySQL with id: ${id}`);
    return { id, data: "sample data" };
  }
}

// High-level module directly depends on low-level module
class UserService {
  constructor() {
    this.database = new MySQLDatabase(); // Direct dependency
  }

  createUser(userData) {
    // Validation logic
    if (!userData.name || !userData.email) {
      throw new Error("Invalid user data");
    }

    // Directly using MySQL database
    this.database.save(userData);
    return userData;
  }

  getUser(id) {
    return this.database.find(id);
  }
}

// Hard to test and change database implementation
```

### ✅ Good Example - Following DIP

```javascript
// Abstraction (interface)
class DatabaseInterface {
  save(data) {
    throw new Error("save method must be implemented");
  }

  find(id) {
    throw new Error("find method must be implemented");
  }
}

// Low-level modules implementing the abstraction
class MySQLDatabase extends DatabaseInterface {
  save(data) {
    console.log(`Saving data to MySQL: ${JSON.stringify(data)}`);
  }

  find(id) {
    console.log(`Finding data in MySQL with id: ${id}`);
    return { id, data: "MySQL data" };
  }
}

class MongoDatabase extends DatabaseInterface {
  save(data) {
    console.log(`Saving data to MongoDB: ${JSON.stringify(data)}`);
  }

  find(id) {
    console.log(`Finding data in MongoDB with id: ${id}`);
    return { id, data: "MongoDB data" };
  }
}

class InMemoryDatabase extends DatabaseInterface {
  constructor() {
    super();
    this.data = new Map();
  }

  save(data) {
    this.data.set(data.id, data);
    console.log(`Saving data to memory: ${JSON.stringify(data)}`);
  }

  find(id) {
    const result = this.data.get(id);
    console.log(`Finding data in memory with id: ${id}`);
    return result || null;
  }
}

// High-level module depends on abstraction
class UserService {
  constructor(database) {
    if (!(database instanceof DatabaseInterface)) {
      throw new Error("Database must implement DatabaseInterface");
    }
    this.database = database; // Dependency injection
  }

  createUser(userData) {
    // Validation logic
    if (!userData.name || !userData.email) {
      throw new Error("Invalid user data");
    }

    // Using abstraction, not concrete implementation
    this.database.save(userData);
    return userData;
  }

  getUser(id) {
    return this.database.find(id);
  }
}

// Dependency injection container (simple example)
class DIContainer {
  constructor() {
    this.dependencies = new Map();
  }

  register(name, factory) {
    this.dependencies.set(name, factory);
  }

  resolve(name) {
    const factory = this.dependencies.get(name);
    if (!factory) {
      throw new Error(`Dependency ${name} not found`);
    }
    return factory();
  }
}

// Usage with dependency injection
const container = new DIContainer();

// Register dependencies
container.register('database', () => new MySQLDatabase());
container.register('userService', () => new UserService(container.resolve('database')));

// Easy to switch implementations
const mysqlUserService = new UserService(new MySQLDatabase());
const mongoUserService = new UserService(new MongoDatabase());
const memoryUserService = new UserService(new InMemoryDatabase());

// All services work the same way
const userData = { id: 1, name: "John Doe", email: "john@example.com" };

mysqlUserService.createUser(userData);
mongoUserService.createUser(userData);
memoryUserService.createUser(userData);

// Easy to test with mock database
class MockDatabase extends DatabaseInterface {
  constructor() {
    super();
    this.savedData = null;
  }

  save(data) {
    this.savedData = data;
    console.log("Data saved to mock database");
  }

  find(id) {
    console.log("Finding data in mock database");
    return this.savedData;
  }
}

const testUserService = new UserService(new MockDatabase());
testUserService.createUser(userData);
```

---

## SOLID Principles Summary

### Benefits of Following SOLID Principles:

1. **Maintainability**: Code is easier to understand and modify
2. **Flexibility**: Easy to extend and add new features
3. **Testability**: Components can be tested in isolation
4. **Reusability**: Code components can be reused in different contexts
5. **Reduced Coupling**: Components are loosely coupled
6. **Better Design**: Forces you to think about responsibilities and dependencies

### Quick Reference:

- **SRP**: One class, one responsibility
- **OCP**: Open for extension, closed for modification
- **LSP**: Subtypes must be substitutable for their base types
- **ISP**: Many specific interfaces are better than one general interface
- **DIP**: Depend on abstractions, not concretions

### Real-World Application:

These principles are especially important in:
- Large applications
- Team development
- Long-term maintenance
- Testing and debugging
- API design
- Framework development

Following SOLID principles leads to more robust, maintainable, and scalable JavaScript applications.


