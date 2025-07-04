# Interfaces vs. Type Aliases

## Overview
Both interfaces and type aliases allow you to define custom types in TypeScript, but they have different capabilities and use cases.

## Interface

### Definition
Interfaces define the shape of objects and can be extended or implemented.

```typescript
interface User {
  name: string;
  age: number;
  email?: string; // optional property
}

interface Admin extends User {
  permissions: string[];
}
```

### Key Features
- **Declaration merging**: Multiple interface declarations with the same name are merged
- **Extensibility**: Can extend other interfaces using `extends`
- **Implementation**: Classes can implement interfaces
- **Object-oriented**: Best for defining object shapes

### Examples

#### Basic Interface
```typescript
interface Person {
  firstName: string;
  lastName: string;
  age: number;
}

const john: Person = {
  firstName: "John",
  lastName: "Doe",
  age: 30
};
```

#### Interface with Methods
```typescript
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

class BasicCalculator implements Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
  
  subtract(a: number, b: number): number {
    return a - b;
  }
}
```

#### Declaration Merging
```typescript
interface Window {
  title: string;
}

interface Window {
  ts: TypeScriptAPI; // This gets merged with the above
}

// Now Window has both title and ts properties
```

## Type Alias

### Definition
Type aliases create a new name for any type, including primitives, unions, intersections, and complex types.

```typescript
type UserID = string | number;

type User = {
  name: string;
  age: number;
  id: UserID;
};
```

### Key Features
- **Flexibility**: Can represent any type (primitives, unions, intersections, etc.)
- **No declaration merging**: Each type alias is unique
- **Computed properties**: Can use mapped types and conditional types
- **Union and intersection types**: Perfect for complex type compositions

### Examples

#### Basic Type Alias
```typescript
type Status = "pending" | "approved" | "rejected";

type User = {
  name: string;
  status: Status;
};
```

#### Union Types
```typescript
type StringOrNumber = string | number;
type Theme = "light" | "dark" | "auto";

function setTheme(theme: Theme) {
  // Implementation
}
```

#### Intersection Types
```typescript
type Timestamped = {
  createdAt: Date;
  updatedAt: Date;
};

type User = {
  name: string;
  email: string;
};

type UserWithTimestamp = User & Timestamped;
```

#### Complex Type Manipulations
```typescript
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P];
};

type UserUpdate = Partial<User>; // All properties optional
```

## Key Differences

| Feature | Interface | Type Alias |
|---------|-----------|------------|
| **Declaration Merging** | ✅ Yes | ❌ No |
| **Extends/Implements** | ✅ Yes | ❌ No (but can use intersections) |
| **Union Types** | ❌ No | ✅ Yes |
| **Primitive Types** | ❌ No | ✅ Yes |
| **Computed Properties** | ❌ Limited | ✅ Yes |
| **Error Messages** | Better for objects | Can be less clear |

## When to Use Which?

### Use Interfaces When:
- Defining object shapes that might be extended
- Creating public APIs that others might extend
- Working with classes that need to implement contracts
- You want declaration merging capabilities

```typescript
// Good use case for interface
interface APIResponse {
  status: number;
  data: any;
}

interface UserAPIResponse extends APIResponse {
  data: User[];
}

class UserService implements UserAPIResponse {
  status = 200;
  data: User[] = [];
}
```

### Use Type Aliases When:
- Creating union or intersection types
- Aliasing primitive types
- Complex type transformations
- Working with utility types

```typescript
// Good use cases for type aliases
type EventType = "click" | "hover" | "focus";
type ID = string | number;
type Partial<T> = { [P in keyof T]?: T[P] };

type UserEvent = {
  type: EventType;
  userId: ID;
  timestamp: Date;
};
```

## Best Practices

### 1. Consistent Naming
```typescript
// Use PascalCase for both
interface User { }
type UserStatus = "active" | "inactive";
```

### 2. Prefer Interfaces for Object Shapes
```typescript
// Preferred
interface Product {
  id: string;
  name: string;
  price: number;
}

// Instead of
type Product = {
  id: string;
  name: string;
  price: number;
};
```

### 3. Use Type Aliases for Unions
```typescript
// Preferred
type Status = "loading" | "success" | "error";
type Theme = "light" | "dark";

// Not possible with interfaces
```

### 4. Composition Examples
```typescript
// Interface extension
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// Type intersection
type Animal = {
  name: string;
};

type Dog = Animal & {
  breed: string;
};
```

## Advanced Patterns

### Generic Interfaces
```typescript
interface Repository<T> {
  find(id: string): T | null;
  save(entity: T): void;
  delete(id: string): boolean;
}

class UserRepository implements Repository<User> {
  find(id: string): User | null {
    // Implementation
    return null;
  }
  
  save(user: User): void {
    // Implementation
  }
  
  delete(id: string): boolean {
    // Implementation
    return true;
  }
}
```

### Generic Type Aliases
```typescript
type ApiResponse<T> = {
  success: boolean;
  data: T;
  error?: string;
};

type UserResponse = ApiResponse<User>;
type ProductResponse = ApiResponse<Product[]>;
```

### Conditional Types
```typescript
type NonNullable<T> = T extends null | undefined ? never : T;
type UserName = NonNullable<User["name"]>; // string
```

# Generics

## Overview
Generics provide a way to create reusable components that can work with multiple types while maintaining type safety. They allow you to write flexible, type-safe code without sacrificing the benefits of static typing.

## Why Use Generics?

### Without Generics (Problems)
```typescript
// Without generics - type unsafe
function identity(arg: any): any {
  return arg;
}

let output = identity("myString"); // type is 'any'
// No type checking, could cause runtime errors

// Without generics - not reusable
function stringIdentity(arg: string): string {
  return arg;
}

function numberIdentity(arg: number): number {
  return arg;
}
// Need separate functions for each type
```

### With Generics (Solution)
```typescript
function identity<T>(arg: T): T {
  return arg;
}

let stringOutput = identity<string>("myString"); // type is 'string'
let numberOutput = identity<number>(42); // type is 'number'
let boolOutput = identity(true); // type inferred as 'boolean'
```

## Basic Generic Syntax

### Generic Functions
```typescript
// Basic generic function
function createArray<T>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

const stringArray = createArray<string>(3, "hello"); // string[]
const numberArray = createArray(3, 42); // number[] (type inferred)

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const result = pair("hello", 42); // [string, number]
```

### Generic Interfaces
```typescript
interface Container<T> {
  value: T;
  getValue(): T;
  setValue(value: T): void;
}

class StringContainer implements Container<string> {
  constructor(public value: string) {}
  
  getValue(): string {
    return this.value;
  }
  
  setValue(value: string): void {
    this.value = value;
  }
}

// Generic interface with multiple parameters
interface KeyValuePair<K, V> {
  key: K;
  value: V;
}

const userAge: KeyValuePair<string, number> = {
  key: "john",
  value: 30
};
```

### Generic Classes
```typescript
class GenericStorage<T> {
  private items: T[] = [];
  
  add(item: T): void {
    this.items.push(item);
  }
  
  remove(): T | undefined {
    return this.items.pop();
  }
  
  getItems(): T[] {
    return [...this.items];
  }
  
  get length(): number {
    return this.items.length;
  }
}

const stringStorage = new GenericStorage<string>();
stringStorage.add("hello");
stringStorage.add("world");

const numberStorage = new GenericStorage<number>();
numberStorage.add(1);
numberStorage.add(2);
```

## Generic Constraints

### Basic Constraints
```typescript
// Constraint using 'extends'
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // Now we know 'arg' has a length property
  return arg;
}

logLength("hello"); // OK - string has length
logLength([1, 2, 3]); // OK - array has length
logLength({ length: 10, value: 3 }); // OK - object has length
// logLength(3); // Error - number doesn't have length
```

### Keyof Constraints
```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "John", age: 30, email: "john@example.com" };

const name = getProperty(person, "name"); // string
const age = getProperty(person, "age"); // number
// const invalid = getProperty(person, "invalid"); // Error
```

### Using Type Parameters in Constraints
```typescript
function copyFields<T, K extends keyof T>(target: T, source: T, keys: K[]): void {
  keys.forEach(key => {
    target[key] = source[key];
  });
}

const user1 = { name: "John", age: 30, email: "john@example.com" };
const user2 = { name: "Jane", age: 25, email: "jane@example.com" };

copyFields(user1, user2, ["name", "age"]); // OK
// copyFields(user1, user2, ["invalid"]); // Error
```

## Advanced Generic Patterns

### Conditional Types
```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type Example1 = NonNullable<string | null>; // string
type Example2 = NonNullable<number | undefined>; // number

// More complex conditional type
type ApiResponse<T> = T extends string 
  ? { message: T } 
  : T extends number 
    ? { count: T } 
    : { data: T };

type StringResponse = ApiResponse<string>; // { message: string }
type NumberResponse = ApiResponse<number>; // { count: number }
type ObjectResponse = ApiResponse<object>; // { data: object }
```

### Mapped Types
```typescript
// Make all properties optional
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// Make all properties required
type Required<T> = {
  [P in keyof T]-?: T[P];
};

// Make all properties readonly
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

interface User {
  name: string;
  age: number;
  email?: string;
}

type PartialUser = Partial<User>; // All properties optional
type RequiredUser = Required<User>; // All properties required
type ReadonlyUser = Readonly<User>; // All properties readonly
```

#Understanding Partials in Detail

# What is Partial?
**Partial** is a built-in utility type that makes all properties of a type optional. It's one of the most commonly used mapped types in TypeScript for creating flexible APIs and handling partial data updates.

### How Partial Works
```typescript
// The actual implementation of Partial
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// Example transformation
interface User {
  name: string;
  age: number;
  email?: string;
}

// Partial<User> becomes:
type PartialUser = {
  name?: string;    // required becomes optional
  age?: number;     // required becomes optional
  email?: string;   // already optional stays optional
}
```

### Common Use Cases

#### 1. Object Updates
```typescript
interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
  category: string;
}

// Update function that allows partial updates
function updateProduct(id: string, updates: Partial<Product>): Product {
  const existingProduct = getProductById(id);
  return { ...existingProduct, ...updates };
}

// Usage examples
updateProduct("123", { price: 29.99 }); // Only update price
updateProduct("456", { name: "New Name", description: "New description" });
updateProduct("789", { category: "Electronics", price: 199.99 });
```

### Best Practices

#### 1. Use Partial for Optional Updates
```typescript
// Good: Clear intent for partial updates
function updateUserProfile(userId: string, updates: Partial<UserProfile>): Promise<UserProfile> {
  // Implementation
}

// Avoid: Unclear what properties are required
function updateUserProfile(userId: string, updates: any): Promise<UserProfile> {
  // Implementation
}
```

#### 2. Validate Partial Data
```typescript
function isValidPartialUser(data: Partial<User>): data is Partial<User> & { email: string } {
  return typeof data.email === 'string' && data.email.includes('@');
}

function processUserUpdate(updates: Partial<User>) {
  if (isValidPartialUser(updates)) {
    // TypeScript now knows updates.email is defined
    console.log(`Updating user with email: ${updates.email}`);
  }
}
```

#### 3. Combine with Default Values
```typescript
function createUserWithDefaults(userData: Partial<User>): User {
  const defaults: User = {
    id: generateId(),
    username: '',
    email: '',
    profile: { firstName: '', lastName: '' },
    settings: { notifications: true, privacy: 'private' }
  };
  
  return { ...defaults, ...userData };
}
```


Partial is an essential utility type that makes TypeScript code more flexible while maintaining type safety. It's particularly useful for APIs, configuration objects, and any scenario where you need to work with incomplete data structures.

