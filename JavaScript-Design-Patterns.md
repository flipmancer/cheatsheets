# JavaScript Design Patterns Cheatsheet

> Essential design patterns for JavaScript/TypeScript development with practical examples and use cases.

## Table of Contents

- [JavaScript Design Patterns Cheatsheet](#javascript-design-patterns-cheatsheet)
	- [Table of Contents](#table-of-contents)
	- [Overview](#overview)
	- [Core Design Patterns](#core-design-patterns)
		- [1. Singleton Pattern](#1-singleton-pattern)
		- [2. Factory Pattern](#2-factory-pattern)
		- [3. Strategy Pattern](#3-strategy-pattern)
		- [4. Observer Pattern (Pub/Sub)](#4-observer-pattern-pubsub)
		- [5. Decorator Pattern](#5-decorator-pattern)
		- [6. Module Pattern](#6-module-pattern)
		- [7. Proxy Pattern](#7-proxy-pattern)
		- [8. Command Pattern](#8-command-pattern)
		- [9. Builder Pattern](#9-builder-pattern)
		- [10. Adapter Pattern](#10-adapter-pattern)
	- [Common Patterns in Modern JavaScript](#common-patterns-in-modern-javascript)
		- [Promise Pattern (Async Operations)](#promise-pattern-async-operations)
		- [Memoization Pattern (Caching)](#memoization-pattern-caching)
		- [Currying Pattern](#currying-pattern)
		- [Middleware Pattern (Composition)](#middleware-pattern-composition)
	- [Quick Reference](#quick-reference)
		- [When to Use Which Pattern](#when-to-use-which-pattern)
		- [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
	- [Performance Tips](#performance-tips)
	- [TypeScript-Specific Patterns](#typescript-specific-patterns)
		- [Branded Types (Compile-Time Safety)](#branded-types-compile-time-safety)
		- [Discriminated Unions (Type-Safe State)](#discriminated-unions-type-safe-state)
	- [Next Steps](#next-steps)
	- [Resources](#resources)

## Overview

- Purpose: Quick reference for common design patterns in JavaScript and TypeScript
- Audience: Intermediate to Advanced
- Scope: Creational, Structural, and Behavioral patterns commonly used in modern JavaScript development. Excludes outdated patterns and focuses on ES6+ syntax.
- Updated: 2025-11-14

---

## Core Design Patterns

### 1. Singleton Pattern

**What it is:** Ensures a class has only one instance and provides global access to it.

**Why it matters:** Useful for managing shared resources like database connections, loggers, or configuration objects.

**Mental model:** Like a government—there's only one, and everyone accesses the same instance.

```typescript
class DatabaseConnection {
    private static instance: DatabaseConnection;
    private constructor() {
        // Private constructor prevents direct instantiation
    }

    static getInstance(): DatabaseConnection {
        if (!DatabaseConnection.instance) {
            DatabaseConnection.instance = new DatabaseConnection();
        }
        return DatabaseConnection.instance;
    }

    query(sql: string) {
        console.log(`Executing: ${sql}`);
    }
}

// Usage
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2); // true - same instance
```

**Common pitfalls:**
- Can make testing difficult (hard to mock)
- Creates hidden dependencies
- Not truly "single" in module systems (each module can have its own instance)

**Modern alternative:**
```javascript
// ES6 Module singleton (simpler)
export const logger = {
    log: (msg) => console.log(msg),
    error: (err) => console.error(err)
};
```

---

### 2. Factory Pattern

**What it is:** Creates objects without specifying their exact class, delegating instantiation logic to factory functions.

**When to use:** When object creation logic is complex or when you want to decouple object creation from usage.

**Mental model:** Like a car factory—you request a car type, and the factory handles the construction details.

```typescript
// Product interface
interface Vehicle {
    drive(): void;
}

// Concrete products
class Car implements Vehicle {
    drive() { console.log("Driving a car"); }
}

class Truck implements Vehicle {
    drive() { console.log("Driving a truck"); }
}

class Motorcycle implements Vehicle {
    drive() { console.log("Riding a motorcycle"); }
}

// Factory function
function createVehicle(type: string): Vehicle {
    switch (type) {
        case 'car':
            return new Car();
        case 'truck':
            return new Truck();
        case 'motorcycle':
            return new Motorcycle();
        default:
            throw new Error(`Unknown vehicle type: ${type}`);
    }
}

// Usage
const myCar = createVehicle('car');
myCar.drive(); // "Driving a car"
```

**Gotchas:**
- Can become complex with many product types
- May violate Open/Closed Principle if not designed carefully

---

### 3. Strategy Pattern

**What it is:** Defines a family of algorithms, encapsulates each one, and makes them interchangeable.

**When to use:** When you need to select an algorithm at runtime or pass behavior as a parameter.

**Mental model:** Like payment methods—credit card, PayPal, or cash—same interface, different implementations.

```typescript
// Strategy interface
interface PaymentStrategy {
    pay(amount: number): void;
}

// Concrete strategies
class CreditCardPayment implements PaymentStrategy {
    constructor(private cardNumber: string) {}

    pay(amount: number) {
        console.log(`Paid $${amount} with credit card ${this.cardNumber}`);
    }
}

class PayPalPayment implements PaymentStrategy {
    constructor(private email: string) {}

    pay(amount: number) {
        console.log(`Paid $${amount} via PayPal (${this.email})`);
    }
}

class CashPayment implements PaymentStrategy {
    pay(amount: number) {
        console.log(`Paid $${amount} in cash`);
    }
}

// Context
class ShoppingCart {
    private paymentStrategy: PaymentStrategy;

    setPaymentStrategy(strategy: PaymentStrategy) {
        this.paymentStrategy = strategy;
    }

    checkout(amount: number) {
        this.paymentStrategy.pay(amount);
    }
}

// Usage
const cart = new ShoppingCart();
cart.setPaymentStrategy(new CreditCardPayment('1234-5678-9012-3456'));
cart.checkout(100); // Paid $100 with credit card

cart.setPaymentStrategy(new PayPalPayment('user@example.com'));
cart.checkout(50); // Paid $50 via PayPal
```

**Common use cases:**
- Sorting algorithms (quicksort, mergesort, etc.)
- Compression strategies (zip, gzip, brotli)
- Authentication methods (OAuth, JWT, session-based)

---

### 4. Observer Pattern (Pub/Sub)

**What it is:** Defines a one-to-many dependency between objects so that when one object changes state, all dependents are notified.

**Why it matters:** Foundation for event-driven architecture, reactive programming, and state management.

**Mental model:** Like a newsletter subscription—when a new issue is published, all subscribers receive it.

```typescript
class EventEmitter {
    private events: Record<string, Function[]> = {};

    on(event: string, callback: Function) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(callback);
    }

    off(event: string, callback: Function) {
        if (!this.events[event]) return;
        this.events[event] = this.events[event].filter(cb => cb !== callback);
    }

    emit(event: string, data?: any) {
        if (!this.events[event]) return;
        this.events[event].forEach(callback => callback(data));
    }
}

// Usage
const emitter = new EventEmitter();

const onUserLogin = (user: any) => console.log(`User logged in: ${user.name}`);
const onUserLogout = () => console.log('User logged out');

emitter.on('login', onUserLogin);
emitter.on('logout', onUserLogout);

emitter.emit('login', { name: 'Alice' }); // "User logged in: Alice"
emitter.emit('logout'); // "User logged out"

emitter.off('login', onUserLogin);
emitter.emit('login', { name: 'Bob' }); // Nothing happens
```

**Built-in alternatives:**
```javascript
// Browser EventTarget
const button = document.querySelector('button');
button.addEventListener('click', () => console.log('Clicked!'));

// Node.js EventEmitter
import { EventEmitter } from 'events';
const myEmitter = new EventEmitter();
myEmitter.on('event', () => console.log('Event fired!'));
myEmitter.emit('event');
```

---

### 5. Decorator Pattern

**What it is:** Adds new functionality to an object dynamically without altering its structure.

**When to use:** When you need to add responsibilities to objects without subclassing.

**Mental model:** Like adding toppings to a pizza—each topping is a decorator that enhances the base pizza.

```typescript
// Component interface
interface Coffee {
    cost(): number;
    description(): string;
}

// Concrete component
class SimpleCoffee implements Coffee {
    cost() { return 5; }
    description() { return 'Simple coffee'; }
}

// Base decorator
abstract class CoffeeDecorator implements Coffee {
    constructor(protected coffee: Coffee) {}
    abstract cost(): number;
    abstract description(): string;
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
    cost() { return this.coffee.cost() + 1; }
    description() { return this.coffee.description() + ', milk'; }
}

class SugarDecorator extends CoffeeDecorator {
    cost() { return this.coffee.cost() + 0.5; }
    description() { return this.coffee.description() + ', sugar'; }
}

class WhipDecorator extends CoffeeDecorator {
    cost() { return this.coffee.cost() + 2; }
    description() { return this.coffee.description() + ', whipped cream'; }
}

// Usage
let myCoffee: Coffee = new SimpleCoffee();
console.log(`${myCoffee.description()}: $${myCoffee.cost()}`);
// "Simple coffee: $5"

myCoffee = new MilkDecorator(myCoffee);
myCoffee = new SugarDecorator(myCoffee);
myCoffee = new WhipDecorator(myCoffee);
console.log(`${myCoffee.description()}: $${myCoffee.cost()}`);
// "Simple coffee, milk, sugar, whipped cream: $8.5"
```

**TypeScript Decorators (Experimental):**
```typescript
// Class decorator
function Logger(constructor: Function) {
    console.log(`Creating instance of ${constructor.name}`);
}

@Logger
class MyClass {
    constructor() {}
}

// Method decorator
function Measure(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    descriptor.value = function(...args: any[]) {
        const start = performance.now();
        const result = originalMethod.apply(this, args);
        const end = performance.now();
        console.log(`${propertyKey} took ${end - start}ms`);
        return result;
    };
}

class Calculator {
    @Measure
    add(a: number, b: number) {
        return a + b;
    }
}
```

---

### 6. Module Pattern

**What it is:** Encapsulates private variables and functions, exposing only a public API.

**Why it matters:** Provides data privacy and prevents namespace pollution in JavaScript.

**Mental model:** Like a capsule—internals are hidden, only the interface is exposed.

```javascript
// IIFE (Immediately Invoked Function Expression) Module
const Calculator = (function() {
    // Private variables
    let result = 0;

    // Private function
    function logOperation(operation, value) {
        console.log(`${operation}: ${value}, result: ${result}`);
    }

    // Public API
    return {
        add(value) {
            result += value;
            logOperation('add', value);
            return this;
        },
        subtract(value) {
            result -= value;
            logOperation('subtract', value);
            return this;
        },
        getResult() {
            return result;
        },
        reset() {
            result = 0;
            return this;
        }
    };
})();

// Usage
Calculator.add(5).add(3).subtract(2);
console.log(Calculator.getResult()); // 6
// Calculator.result is undefined (private)
```

**ES6+ Module (Modern):**
```javascript
// math.js
let privateCounter = 0;

function incrementCounter() {
    privateCounter++;
}

export function add(a, b) {
    incrementCounter();
    return a + b;
}

export function getCallCount() {
    return privateCounter;
}

// main.js
import { add, getCallCount } from './math.js';
console.log(add(2, 3)); // 5
console.log(getCallCount()); // 1
```

---

### 7. Proxy Pattern

**What it is:** Provides a surrogate or placeholder to control access to an object.

**When to use:** For lazy loading, access control, logging, caching, or validation.

**Mental model:** Like a security guard—controls who can access the building and logs all entries.

```javascript
// Target object
const user = {
    name: 'Alice',
    age: 30,
    email: 'alice@example.com'
};

// Proxy with validation and logging
const userProxy = new Proxy(user, {
    get(target, prop) {
        console.log(`Reading property: ${String(prop)}`);
        return target[prop];
    },
    set(target, prop, value) {
        console.log(`Setting ${String(prop)} to ${value}`);

        // Validation
        if (prop === 'age' && typeof value !== 'number') {
            throw new TypeError('Age must be a number');
        }
        if (prop === 'email' && !value.includes('@')) {
            throw new Error('Invalid email format');
        }

        target[prop] = value;
        return true;
    }
});

// Usage
console.log(userProxy.name); // Logs: "Reading property: name", outputs: "Alice"
userProxy.age = 31; // Logs: "Setting age to 31"
// userProxy.age = '31'; // Throws: TypeError: Age must be a number
```

**Practical use case - Lazy loading:**
```javascript
const lazyImage = new Proxy({}, {
    get(target, prop) {
        if (!target.loaded) {
            console.log('Loading image...');
            target.data = loadImageFromServer(); // Expensive operation
            target.loaded = true;
        }
        return target[prop];
    }
});

// Image only loads when accessed
console.log(lazyImage.data);
```

---

### 8. Command Pattern

**What it is:** Encapsulates a request as an object, allowing parameterization and queuing of requests.

**When to use:** For undo/redo functionality, task scheduling, or decoupling sender and receiver.

**Mental model:** Like a remote control—each button encapsulates a command for the TV.

```typescript
// Receiver
class TextEditor {
    private text = '';

    write(content: string) {
        this.text += content;
    }

    delete(length: number) {
        this.text = this.text.slice(0, -length);
    }

    getText() {
        return this.text;
    }
}

// Command interface
interface Command {
    execute(): void;
    undo(): void;
}

// Concrete commands
class WriteCommand implements Command {
    constructor(private editor: TextEditor, private content: string) {}

    execute() {
        this.editor.write(this.content);
    }

    undo() {
        this.editor.delete(this.content.length);
    }
}

class DeleteCommand implements Command {
    private deletedText = '';

    constructor(private editor: TextEditor, private length: number) {}

    execute() {
        const text = this.editor.getText();
        this.deletedText = text.slice(-this.length);
        this.editor.delete(this.length);
    }

    undo() {
        this.editor.write(this.deletedText);
    }
}

// Invoker
class CommandHistory {
    private history: Command[] = [];

    execute(command: Command) {
        command.execute();
        this.history.push(command);
    }

    undo() {
        const command = this.history.pop();
        if (command) {
            command.undo();
        }
    }
}

// Usage
const editor = new TextEditor();
const history = new CommandHistory();

history.execute(new WriteCommand(editor, 'Hello '));
history.execute(new WriteCommand(editor, 'World!'));
console.log(editor.getText()); // "Hello World!"

history.undo();
console.log(editor.getText()); // "Hello "

history.undo();
console.log(editor.getText()); // ""
```

---

### 9. Builder Pattern

**What it is:** Separates object construction from its representation, allowing step-by-step creation of complex objects.

**When to use:** When an object has many optional parameters or complex construction logic.

**Mental model:** Like building a custom computer—select CPU, RAM, storage, etc., then assemble.

```typescript
class HttpRequest {
    private method: string = 'GET';
    private url: string = '';
    private headers: Record<string, string> = {};
    private body?: any;
    private timeout: number = 30000;

    setMethod(method: string): this {
        this.method = method;
        return this;
    }

    setUrl(url: string): this {
        this.url = url;
        return this;
    }

    setHeaders(headers: Record<string, string>): this {
        this.headers = { ...this.headers, ...headers };
        return this;
    }

    setBody(body: any): this {
        this.body = body;
        return this;
    }

    setTimeout(timeout: number): this {
        this.timeout = timeout;
        return this;
    }

    async execute() {
        console.log(`${this.method} ${this.url}`);
        console.log('Headers:', this.headers);
        console.log('Body:', this.body);
        console.log('Timeout:', this.timeout);
        // Actual fetch logic here
    }
}

// Usage (fluent interface)
const request = new HttpRequest()
    .setMethod('POST')
    .setUrl('https://api.example.com/users')
    .setHeaders({ 'Content-Type': 'application/json' })
    .setBody({ name: 'Alice', age: 30 })
    .setTimeout(5000);

await request.execute();
```

**Alternative with separate Builder class:**
```typescript
class User {
    constructor(
        public name: string,
        public email: string,
        public age?: number,
        public address?: string,
        public phone?: string
    ) {}
}

class UserBuilder {
    private name: string = '';
    private email: string = '';
    private age?: number;
    private address?: string;
    private phone?: string;

    setName(name: string): this {
        this.name = name;
        return this;
    }

    setEmail(email: string): this {
        this.email = email;
        return this;
    }

    setAge(age: number): this {
        this.age = age;
        return this;
    }

    setAddress(address: string): this {
        this.address = address;
        return this;
    }

    setPhone(phone: string): this {
        this.phone = phone;
        return this;
    }

    build(): User {
        return new User(this.name, this.email, this.age, this.address, this.phone);
    }
}

// Usage
const user = new UserBuilder()
    .setName('Alice')
    .setEmail('alice@example.com')
    .setAge(30)
    .build();
```

---

### 10. Adapter Pattern

**What it is:** Converts the interface of a class into another interface that clients expect.

**When to use:** When you need to make incompatible interfaces work together.

**Mental model:** Like a power adapter—allows a US plug to work in a European outlet.

```typescript
// Old API (incompatible)
class OldPaymentProcessor {
    processPayment(cardNumber: string, amount: number) {
        console.log(`Processing $${amount} with card ${cardNumber}`);
    }
}

// New interface (expected by client)
interface PaymentProcessor {
    pay(amount: number, paymentDetails: { cardNumber: string }): void;
}

// Adapter
class PaymentAdapter implements PaymentProcessor {
    constructor(private oldProcessor: OldPaymentProcessor) {}

    pay(amount: number, paymentDetails: { cardNumber: string }) {
        // Adapt new interface to old implementation
        this.oldProcessor.processPayment(paymentDetails.cardNumber, amount);
    }
}

// Client code
function checkout(processor: PaymentProcessor, amount: number) {
    processor.pay(amount, { cardNumber: '1234-5678' });
}

// Usage
const oldProcessor = new OldPaymentProcessor();
const adapter = new PaymentAdapter(oldProcessor);
checkout(adapter, 100); // Works with old API via adapter
```

---

## Common Patterns in Modern JavaScript

### Promise Pattern (Async Operations)

```javascript
// Creating a promise
function fetchData(url) {
    return new Promise((resolve, reject) => {
        fetch(url)
            .then(response => response.json())
            .then(data => resolve(data))
            .catch(error => reject(error));
    });
}

// Using with async/await
async function getData() {
    try {
        const data = await fetchData('https://api.example.com/data');
        console.log(data);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

### Memoization Pattern (Caching)

```javascript
function memoize(fn) {
    const cache = new Map();

    return function(...args) {
        const key = JSON.stringify(args);

        if (cache.has(key)) {
            console.log('Cache hit!');
            return cache.get(key);
        }

        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

// Usage
const expensiveOperation = memoize((n) => {
    console.log('Computing...');
    return n * n;
});

console.log(expensiveOperation(5)); // Computing... 25
console.log(expensiveOperation(5)); // Cache hit! 25
```

### Currying Pattern

```javascript
// Basic currying
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return function(...nextArgs) {
                return curried.apply(this, args.concat(nextArgs));
            };
        }
    };
}

// Usage
function add(a, b, c) {
    return a + b + c;
}

const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1)(2, 3)); // 6
```

### Middleware Pattern (Composition)

```javascript
class Pipeline {
    constructor() {
        this.middlewares = [];
    }

    use(fn) {
        this.middlewares.push(fn);
        return this;
    }

    async execute(context) {
        let index = 0;

        const next = async () => {
            if (index < this.middlewares.length) {
                const middleware = this.middlewares[index++];
                await middleware(context, next);
            }
        };

        await next();
    }
}

// Usage (Express-style middleware)
const pipeline = new Pipeline();

pipeline
    .use(async (ctx, next) => {
        console.log('Middleware 1: Before');
        await next();
        console.log('Middleware 1: After');
    })
    .use(async (ctx, next) => {
        console.log('Middleware 2: Processing');
        ctx.result = 'Done';
        await next();
    })
    .use(async (ctx, next) => {
        console.log('Middleware 3:', ctx.result);
    });

await pipeline.execute({});
// Output:
// Middleware 1: Before
// Middleware 2: Processing
// Middleware 3: Done
// Middleware 1: After
```

---

## Quick Reference

### When to Use Which Pattern

1. **Singleton**: Shared resources (loggers, configs, database connections)
2. **Factory**: Complex object creation, decoupling instantiation
3. **Strategy**: Runtime algorithm selection, interchangeable behaviors
4. **Observer**: Event-driven systems, state change notifications
5. **Decorator**: Add functionality dynamically without subclassing
6. **Module**: Encapsulation, namespace management, data privacy
7. **Proxy**: Access control, lazy loading, validation, caching
8. **Command**: Undo/redo, task queuing, macro commands
9. **Builder**: Complex objects with many optional parameters
10. **Adapter**: Interface compatibility, legacy system integration

### Anti-Patterns to Avoid

1. **God Object**: Single class doing too much
2. **Spaghetti Code**: Tangled, unstructured control flow
3. **Callback Hell**: Deeply nested callbacks (use promises/async-await)
4. **Magic Numbers**: Unexplained constants (use named constants)
5. **Global Variables**: Polluting global namespace

---

## Performance Tips

1. **Use memoization for expensive computations** that are called repeatedly with the same arguments
2. **Prefer composition over deep inheritance** - flatter structures are easier to optimize
3. **Lazy load modules and data** - only load what's needed when it's needed
4. **Cache pattern results** - especially for factory and builder patterns
5. **Avoid premature abstraction** - apply patterns when complexity justifies them

---

## TypeScript-Specific Patterns

### Branded Types (Compile-Time Safety)

```typescript
type UserId = string & { readonly brand: unique symbol };
type ProductId = string & { readonly brand: unique symbol };

function createUserId(id: string): UserId {
    return id as UserId;
}

function createProductId(id: string): ProductId {
    return id as ProductId;
}

function getUser(id: UserId) { /* ... */ }
function getProduct(id: ProductId) { /* ... */ }

const userId = createUserId('user-123');
const productId = createProductId('prod-456');

getUser(userId); // OK
// getUser(productId); // Error: Type 'ProductId' is not assignable to type 'UserId'
```

### Discriminated Unions (Type-Safe State)

```typescript
type LoadingState = { status: 'loading' };
type SuccessState = { status: 'success'; data: any };
type ErrorState = { status: 'error'; error: string };

type State = LoadingState | SuccessState | ErrorState;

function handleState(state: State) {
    switch (state.status) {
        case 'loading':
            console.log('Loading...');
            break;
        case 'success':
            console.log('Data:', state.data); // TypeScript knows 'data' exists
            break;
        case 'error':
            console.error('Error:', state.error); // TypeScript knows 'error' exists
            break;
    }
}
```

---

## Next Steps

- Practice implementing patterns in small projects
- Study open-source libraries to see patterns in action (React, Express, RxJS)
- Explore functional programming patterns (monads, functors, lenses)
- Learn reactive programming patterns (Observables, Subjects)
- Combine patterns for more complex architectures (MVVM, MVC, Flux, Redux)

---

## Resources

- [Refactoring Guru - Design Patterns](https://refactoring.guru/design-patterns)
- [JavaScript Design Patterns (Addy Osmani)](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)
- [TypeScript Design Patterns](https://www.patterns.dev/)
- [MDN - JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
