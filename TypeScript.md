# TypeScript Cheatsheet

> A comprehensive guide to TypeScript patterns, advanced types, and best practices for type-safe JavaScript development.

## Table of Contents

- [TypeScript Cheatsheet](#typescript-cheatsheet)
	- [Table of Contents](#table-of-contents)
	- [Overview](#overview)
	- [TypeScript Setup](#typescript-setup)
		- [Installation](#installation)
		- [Project Initialization](#project-initialization)
		- [Basic Configuration](#basic-configuration)
	- [Core TypeScript Concepts](#core-typescript-concepts)
		- [1. Type Annotations](#1-type-annotations)
		- [2. Interfaces vs Types](#2-interfaces-vs-types)
		- [3. Generics Fundamentals](#3-generics-fundamentals)
		- [4. Type Guards](#4-type-guards)
		- [5. Union and Intersection Types](#5-union-and-intersection-types)
	- [Advanced TypeScript Features](#advanced-typescript-features)
	- [Generics Fundamentals](#generics-fundamentals)
	- [Generic Interfaces](#generic-interfaces)
	- [Advanced Generics](#advanced-generics)
	- [Abstract Classes](#abstract-classes)
	- [Type Guards](#type-guards)
		- [1. `typeof` Type Guard (Primitive Types)](#1-typeof-type-guard-primitive-types)
		- [2. `instanceof` Type Guard (Class Instances)](#2-instanceof-type-guard-class-instances)
		- [3. Custom Type Guard Functions (User-Defined)](#3-custom-type-guard-functions-user-defined)
		- [4. `in` Operator Type Guard (Property Existence)](#4-in-operator-type-guard-property-existence)
		- [5. Discriminated Unions (Tagged Unions)](#5-discriminated-unions-tagged-unions)
		- [6. Array Type Guard (`Array.isArray`)](#6-array-type-guard-arrayisarray)
		- [7. Nullish Type Guard](#7-nullish-type-guard)
		- [8. Type Predicate with Complex Logic](#8-type-predicate-with-complex-logic)
		- [9. Assertion Functions (TypeScript 3.7+)](#9-assertion-functions-typescript-37)
		- [10. User-defined Type Guard for Error Checking](#10-user-defined-type-guard-for-error-checking)
		- [Best Practices for Type Guards](#best-practices-for-type-guards)
	- [Utility Types](#utility-types)
	- [Async Typing](#async-typing)
	- [Function Overloads](#function-overloads)
	- [Type Inference](#type-inference)
	- [Decorators (Experimental)](#decorators-experimental)
	- [Namespaces \& Modules](#namespaces--modules)
	- [Branded Types](#branded-types)
	- [Mapped Types](#mapped-types)
	- [Template Literal Types](#template-literal-types)
	- [Configuration](#configuration)
	- [Best Practices](#best-practices)

## Overview

- Purpose: Master TypeScript for building type-safe, maintainable JavaScript applications
- Audience: Intermediate to Advanced
- Scope: Core types, generics, advanced patterns, and real-world use cases. Excludes deprecated features and focuses on modern TypeScript 5.x+.
- Updated: 2025-11-14

---

## TypeScript Setup

### Installation

```shell
# Install TypeScript globally
npm install -g typescript

# Or as a dev dependency (recommended)
npm install --save-dev typescript

# Install type definitions for Node.js
npm install --save-dev @types/node

# Verify installation
tsc --version
```

### Project Initialization

```shell
# Initialize a new TypeScript project
npm init -y
npm install --save-dev typescript @types/node

# Generate tsconfig.json
npx tsc --init
```

### Basic Configuration

```json
// tsconfig.json - Essential settings
{
    "compilerOptions": {
        "target": "ES2022",
        "module": "commonjs",
        "lib": ["ES2022"],
        "outDir": "./dist",
        "rootDir": "./src",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist"]
}
```

---

## Core TypeScript Concepts

### 1. Type Annotations

**What it is:** Explicit type declarations for variables, parameters, and return values.

**Why it matters:** Catches type errors at compile time instead of runtime, improving code reliability.

**Mental model:** Like labeling boxes—tells TypeScript what should go inside.

```typescript
// Basic types
let name: string = "Alice";
let age: number = 30;
let isActive: boolean = true;
let numbers: number[] = [1, 2, 3];
let tuple: [string, number] = ["Alice", 30];

// Function annotations
function greet(name: string): string {
    return `Hello, ${name}`;
}

// Object types
const user: { name: string; age: number } = {
    name: "Alice",
    age: 30
};

// Optional and readonly
interface User {
    readonly id: number;
    name: string;
    email?: string; // Optional
}
```

**Common pitfalls:**

- Over-annotating when TypeScript can infer the type
- Using `any` instead of `unknown` (loses type safety)
- Forgetting to annotate function return types in public APIs

---

### 2. Interfaces vs Types

**What it is:** Two ways to define object shapes and contracts.

**When to use:**
- **Interfaces**: For object shapes, classes, and when you need declaration merging
- **Types**: For unions, intersections, primitives, and complex type operations

**Gotchas:** Interfaces can be extended and merged; types cannot.

```typescript
// Interface
interface User {
    id: number;
    name: string;
}

interface User {
    email: string; // Declaration merging - adds to existing User
}

// Extending interfaces
interface Admin extends User {
    role: string;
}

// Type aliases
type ID = string | number;

type Point = {
    x: number;
    y: number;
};

// Types can do unions and intersections
type Result = Success | Error;
type Combined = User & { timestamp: Date };

// When to use each
interface Animal {
    name: string;
    speak(): void;
}

type Status = "pending" | "active" | "inactive"; // Use type for unions
```

---

### 3. Generics Fundamentals

**What it is:** Write reusable code that works with any type while maintaining type safety.

**Why it matters:** Enables building flexible, type-safe data structures and functions without duplication.

**Mental model:** Like a template or placeholder—fill in the specific type when you use it.

```typescript
// Generic function
function identity<T>(value: T): T {
    return value;
}

const num = identity<number>(42); // T is number
const str = identity("hello");     // T inferred as string

// Generic class
class Queue<T> {
    private items: T[] = [];

    enqueue(item: T): void {
        this.items.push(item);
    }

    dequeue(): T | undefined {
        return this.items.shift();
    }

    peek(): T | undefined {
        return this.items[0];
    }
}

const numberQueue = new Queue<number>();
numberQueue.enqueue(1);
numberQueue.enqueue(2);

// Generic constraint
interface HasId {
    id: number;
}

function getById<T extends HasId>(items: T[], id: number): T | undefined {
    return items.find(item => item.id === id);
}

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
    return [first, second];
}

const result = pair("Alice", 30); // [string, number]
```

**Common pitfalls:**

- Not constraining generics when you need specific properties
- Over-using generics when a simple type would suffice
- Forgetting that `T` is determined at call time, not definition time

---

### 4. Type Guards

**What it is:** Runtime checks that narrow types within a conditional block.

**When to use:** Working with unions, validating unknown data, or checking object types.

**Mental model:** Like a security checkpoint—verifies the type before allowing access.

```typescript
// typeof guard (primitives)
function process(value: string | number) {
    if (typeof value === "string") {
        return value.toUpperCase(); // TypeScript knows it's string
    } else {
        return value.toFixed(2); // TypeScript knows it's number
    }
}

// instanceof guard (classes)
class Dog {
    bark() { console.log("Woof!"); }
}

class Cat {
    meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
    if (animal instanceof Dog) {
        animal.bark(); // Safe: TypeScript knows it's Dog
    } else {
        animal.meow(); // Safe: TypeScript knows it's Cat
    }
}

// User-defined type guard
function isString(value: unknown): value is string {
    return typeof value === "string";
}

function printLength(value: unknown) {
    if (isString(value)) {
        console.log(value.length); // TypeScript knows it's string
    }
}

// Discriminated unions (tagged unions)
type Success = { status: "success"; data: any };
type Failure = { status: "error"; error: string };
type Result = Success | Failure;

function handleResult(result: Result) {
    if (result.status === "success") {
        console.log(result.data); // TypeScript knows it's Success
    } else {
        console.error(result.error); // TypeScript knows it's Failure
    }
}

// in operator guard
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
    if ("swim" in animal) {
        animal.swim(); // Fish
    } else {
        animal.fly(); // Bird
    }
}
```

**Pitfalls:**

- Not using `is` in user-defined guards (loses type narrowing)
- Forgetting that guards only work within the conditional block
- Checking for properties that could be `undefined`

---

### 5. Union and Intersection Types

**What it is:**
- **Union (`|`)**: Value can be one of several types
- **Intersection (`&`)**: Value must satisfy all types

**When to use:**
- Unions: Multiple possible types (e.g., success or error)
- Intersections: Combining multiple object types

**Mental model:**
- Union: OR logic (can be A or B)
- Intersection: AND logic (must be A and B)

```typescript
// Union types
type ID = string | number;

function printId(id: ID) {
    if (typeof id === "string") {
        console.log(id.toUpperCase());
    } else {
        console.log(id.toFixed(2));
    }
}

// Literal unions (enums alternative)
type Status = "pending" | "active" | "inactive";
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

function setStatus(status: Status) {
    // Only accepts "pending", "active", or "inactive"
}

// Intersection types
type Person = {
    name: string;
    age: number;
};

type Employee = {
    employeeId: string;
    department: string;
};

type EmployeePerson = Person & Employee;

const employee: EmployeePerson = {
    name: "Alice",
    age: 30,
    employeeId: "E123",
    department: "Engineering"
};

// Combining interfaces with intersection
interface Timestamped {
    createdAt: Date;
    updatedAt: Date;
}

type TimestampedUser = User & Timestamped;
```

---

## Advanced TypeScript Features

## Generics Fundamentals
**Why:** Write reusable code that works with any type while maintaining type safety.

```typescript
// Generic function
function identity<T>(value: T): T {
    return value;
}

// Generic class
class Queue<T> {
    private items: T[] = [];

    enqueue(item: T): void {
        this.items.push(item);
    }

    dequeue(): T | undefined {
        return this.items.shift();
    }
}

// Generic constraint
function process<T extends { id: number }>(item: T): number {
    return item.id; // Safe: T must have 'id'
}
```

## Generic Interfaces
**Why:** Define contracts for producers/consumers/transformers that work with any data type.

```typescript
// Producer interface
interface Producer<T> {
    produce(): AsyncGenerator<T>;
}

// Consumer interface
interface Consumer<T> {
    consume(item: T): Promise<void>;
}

// Transformer interface
interface Transformer<TIn, TOut> {
    transform(input: TIn): Promise<TOut>;
}
```

## Advanced Generics
**Why:** Build complex type relationships for pipelines and transformations.

```typescript
// Multiple type parameters
class Pipeline<TIn, TOut> {
    constructor(
        private input: Producer<TIn>,
        private transformer: Transformer<TIn, TOut>,
        private output: Consumer<TOut>
    ) {}
}

// Generic defaults
class Cache<T = any> {
    private data: Map<string, T> = new Map();
}

// Conditional types
type Unwrap<T> = T extends Promise<infer U> ? U : T;
type Result = Unwrap<Promise<string>>; // string
```

## Abstract Classes
**Why:** Force subclasses to implement required methods while sharing common logic.

```typescript
abstract class StrategyBase<T> {
    protected abstract setup(): void;

    async execute(): Promise<void> {
        this.setup();
        await this.run();
    }

    protected abstract run(): Promise<void>;
}

class MyStrategy extends StrategyBase<number> {
    protected setup() { /* impl */ }
    protected async run() { /* impl */ }
}
```

## Type Guards
**Why:** Narrow types at runtime to safely access properties and avoid crashes.

### 1. `typeof` Type Guard (Primitive Types)
**When to use:** Checking primitive types (string, number, boolean, etc.)

```typescript
function processValue(value: string | number): string {
    if (typeof value === "string") {
        // TypeScript knows value is string here
        return value.toUpperCase();
    } else {
        // TypeScript knows value is number here
        return value.toFixed(2);
    }
}
```

### 2. `instanceof` Type Guard (Class Instances)
**When to use:** Checking if an object is an instance of a class

```typescript
class BaseProducer {
    name: string;
    constructor(name: string) {
        this.name = name;
    }
}

class CallbackProducer extends BaseProducer {
    callback: () => void;
    constructor(name: string, callback: () => void) {
        super(name);
        this.callback = callback;
    }
}

function handleNode(node: BaseProducer | CallbackProducer): void {
    if (node instanceof CallbackProducer) {
        // TypeScript knows node is CallbackProducer here
        node.callback();
    } else {
        // TypeScript knows node is BaseProducer here
        console.log(node.name);
    }
}
```

### 3. Custom Type Guard Functions (User-Defined)
**When to use:** Complex type checks that need custom logic

```typescript
interface OpenseaMessage {
    type: "opensea";
    slug: string;
    price: number;
}

interface MagicedenMessage {
    type: "magiceden";
    rune: string;
    amount: number;
}

type Message = OpenseaMessage | MagicedenMessage;

// Custom type guard function
function isOpenseaMessage(msg: Message): msg is OpenseaMessage {
    return msg.type === "opensea";
}

function isMagicedenMessage(msg: Message): msg is MagicedenMessage {
    return msg.type === "magiceden";
}

function processMessage(msg: Message): void {
    if (isOpenseaMessage(msg)) {
        // TypeScript knows msg is OpenseaMessage
        console.log(`Opensea collection: ${msg.slug}, price: ${msg.price}`);
    } else if (isMagicedenMessage(msg)) {
        // TypeScript knows msg is MagicedenMessage
        console.log(`Magiceden rune: ${msg.rune}, amount: ${msg.amount}`);
    }
}
```

### 4. `in` Operator Type Guard (Property Existence)
**When to use:** Checking if a property exists on an object

```typescript
interface Producer {
    produce: () => void;
}

interface Consumer {
    consume: () => void;
}

type Node = Producer | Consumer;

function executeNode(node: Node): void {
    if ("produce" in node) {
        // TypeScript knows node is Producer
        node.produce();
    } else {
        // TypeScript knows node is Consumer
        node.consume();
    }
}
```

### 5. Discriminated Unions (Tagged Unions)
**When to use:** Objects with a common discriminator property

```typescript
// Discriminated unions
type Result<T> =
    | { success: true; data: T }
    | { success: false; error: string };

function handle<T>(result: Result<T>) {
    if (result.success) {
        console.log(result.data); // Type: T
    } else {
        console.log(result.error); // Type: string
    }
}

// Real-world example
interface SuccessResult {
    status: "success";
    data: any;
}

interface ErrorResult {
    status: "error";
    error: string;
}

type ApiResult = SuccessResult | ErrorResult;

function handleResult(result: ApiResult): void {
    if (result.status === "success") {
        // TypeScript knows result is SuccessResult
        console.log("Data:", result.data);
    } else {
        // TypeScript knows result is ErrorResult
        console.log("Error:", result.error);
    }
}
```

### 6. Array Type Guard (`Array.isArray`)
**When to use:** Checking if a value is an array

```typescript
function processInput(input: string | string[]): string {
    if (Array.isArray(input)) {
        // TypeScript knows input is string[]
        return input.join(", ");
    } else {
        // TypeScript knows input is string
        return input.toUpperCase();
    }
}
```

### 7. Nullish Type Guard
**When to use:** Checking for null or undefined values

```typescript
function processOptional(value: string | null | undefined): string {
    if (value != null) {
        // TypeScript knows value is string (not null or undefined)
        return value.toUpperCase();
    } else {
        return "NO VALUE";
    }
}
```

### 8. Type Predicate with Complex Logic
**When to use:** Multiple conditions need to be checked

```typescript
interface RedisQueue {
    type: "redis";
    client: any;
    streamName: string;
}

interface MemoryQueue {
    type: "memory";
    items: any[];
}

type Queue = RedisQueue | MemoryQueue;

// Custom type guard with complex logic
function isRedisQueue(queue: Queue): queue is RedisQueue {
    return queue.type === "redis" && "client" in queue && "streamName" in queue;
}

function clearQueue(queue: Queue): void {
    if (isRedisQueue(queue)) {
        // TypeScript knows queue is RedisQueue
        queue.client.del(queue.streamName);
    } else {
        // TypeScript knows queue is MemoryQueue
        queue.items = [];
    }
}
```

### 9. Assertion Functions (TypeScript 3.7+)
**When to use:** Need to throw an error if type assertion fails

```typescript
function assertIsString(value: unknown): asserts value is string {
    if (typeof value !== "string") {
        throw new Error("Value is not a string");
    }
}

function processUnknown(value: unknown): string {
    assertIsString(value); // After this line, TypeScript knows value is string
    return value.toUpperCase();
}
```

### 10. User-defined Type Guard for Error Checking
**When to use:** Working with error handling patterns

```typescript
// User-defined type guard
function isError(obj: any): obj is Error {
    return obj instanceof Error;
}

function handleError(error: unknown): void {
    if (isError(error)) {
        console.error(error.message); // Safe: TypeScript knows it's Error
    } else {
        console.error("Unknown error:", error);
    }
}
```

### Best Practices for Type Guards

1. **Name type guard functions with `is` prefix** (e.g., `isOpenseaMessage`, `isRedisQueue`)
2. **Use discriminated unions** when possible (e.g., `status: "success" | "error"`)
3. **Use `instanceof` for class hierarchies** (e.g., `BaseProducer`, `CallbackProducer`)
4. **Use `in` operator for object shapes** (e.g., checking if property exists)
5. **Use assertion functions for validation** (e.g., `assertIsString`)
6. **Combine type guards with exhaustiveness checking**:
   ```typescript
   function handleNode(node: NodeType): void {
       if (isProducer(node)) {
           // handle producer
       } else if (isWorker(node)) {
           // handle worker
       } else if (isConsumer(node)) {
           // handle consumer
       } else {
           // Exhaustiveness check - will error if new node type added
           const _exhaustive: never = node;
       }
   }
   ```

**Common Pitfalls:**
- Not using `is` in user-defined guards (loses type narrowing)
- Forgetting that guards only work within the conditional block
- Checking for properties that could be `undefined`
- Using `any` instead of `unknown` in type guard parameters

## Utility Types
**Why:** Transform existing types without redefining them. Saves time and reduces duplication.

```typescript
// Partial: all properties optional
type PartialConfig = Partial<Config>;

// Required: all properties required
type RequiredConfig = Required<Partial<Config>>;

// Pick: select specific properties
type IdAndName = Pick<User, 'id' | 'name'>;

// Omit: exclude properties
type UserWithoutPassword = Omit<User, 'password'>;

// Record: key-value map
type QueueMap = Record<string, Queue<any>>;

// ReturnType: extract return type
type Result = ReturnType<typeof myFunction>;

// Awaited: unwrap Promise
type Data = Awaited<Promise<string>>; // string
```

## Async Typing
**Why:** Properly type promises and async generators to catch errors at compile time.

```typescript
// Async function
async function fetch(): Promise<Data> {
    return { id: 1 };
}

// Async generator
async function* stream(): AsyncGenerator<number> {
    yield 1;
    yield 2;
}

// Promise utilities
type ResolvedType<T> = T extends Promise<infer U> ? U : never;
```

## Function Overloads
**Why:** Provide different type signatures for the same function based on input types.

```typescript
class DataFactory {
    // Overload signatures
    process(data: string): Promise<string>;
    process(data: number): Promise<number>;
    process<T>(data: T): Promise<T>;

    // Implementation
    async process(data: any): Promise<any> {
        return data;
    }
}
```

## Type Inference
**Why:** Let TypeScript figure out types automatically. Less boilerplate, same safety.

```typescript
// Infer from usage
const queue = new Queue(); // Queue<unknown>
queue.enqueue("test");     // Now Queue<string>

// Const assertions
const config = {
    host: 'localhost',
    port: 3000
} as const;
// Type: { readonly host: "localhost"; readonly port: 3000 }

// satisfies operator (TS 4.9+)
const settings = {
    timeout: 5000,
    retries: 3
} satisfies Config; // Validates type but keeps literal types
```

## Decorators (Experimental)
**Why:** Add metadata or modify class behavior (logging, validation) without changing the class itself.

```typescript
// Enable: "experimentalDecorators": true in tsconfig.json

function log(target: any, key: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = async function(...args: any[]) {
        console.log(`Calling ${key}`);
        return await original.apply(this, args);
    };
}

class Worker {
    @log
    async process(data: any) {
        return data;
    }
}
```

## Namespaces & Modules
**Why:** Organize code into reusable, importable units.

```typescript
// Module exports
export class Producer<T> { }
export type ProducerConfig = { };

// Re-export
export { Consumer } from './consumer';
export * from './types';

// Type-only imports
import type { Config } from './config';
```

## Branded Types
**Why:** Prevent accidentally mixing similar primitive types (IDs, tokens, URLs).

```typescript
// Prevent mixing similar types
type UserId = string & { readonly brand: unique symbol };
type ProductId = string & { readonly brand: unique symbol };

function getUser(id: UserId) { }
function getProduct(id: ProductId) { }

const userId = "123" as UserId;
const productId = "456" as ProductId;

getUser(productId); // ERROR: type mismatch
```

## Mapped Types
**Why:** Transform all properties of a type systematically (make async, optional, readonly).

```typescript
// Make all properties async
type Asyncify<T> = {
    [K in keyof T]: Promise<T[K]>
};

// Make specific properties optional
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type User = { id: number; name: string; email: string };
type UserUpdate = Optional<User, 'email'>; // email is optional
```

## Template Literal Types
**Why:** Create precise string patterns at the type level (event names, routes, queue IDs).

```typescript
// String manipulation at type level
type EventName = 'click' | 'focus';
type EventHandler = `on${Capitalize<EventName>}`;
// 'onClick' | 'onFocus'

type QueueName = `queue_${string}`;
const validQueue: QueueName = 'queue_data'; // OK
const invalidQueue: QueueName = 'data'; // ERROR
```

## Configuration
**Why:** Proper tsconfig.json ensures modern features, strict checks, and optimal output.

```typescript
// tsconfig.json essentials
{
    "compilerOptions": {
        "target": "ES2022",           // Modern async/await
        "module": "commonjs",          // Node.js compatibility
        "lib": ["ES2022"],            // Latest features
        "outDir": "./dist",           // Build output
        "rootDir": "./src",           // Source root
        "strict": true,               // All strict checks
        "esModuleInterop": true,      // CommonJS interop
        "skipLibCheck": true,         // Faster builds
        "forceConsistentCasingInFileNames": true,
        "resolveJsonModule": true,    // Import JSON
        "declaration": true,          // Generate .d.ts
        "declarationMap": true,       // Sourcemaps for types
        "sourceMap": true             // Debug support
    }
}
```

## Best Practices
**Why:** Follow patterns that prevent bugs and make code easier to maintain.

```typescript
// 1. Use unknown over any
function parse(data: unknown): User {
    if (typeof data === 'object' && data !== null) {
        return data as User;
    }
    throw new Error('Invalid data');
}

// 2. Avoid type assertions, use guards
// BAD
const user = data as User;

// GOOD
function isUser(data: unknown): data is User {
    return typeof data === 'object'
        && data !== null
        && 'id' in data;
}

// 3. Use readonly for immutability
class Queue<T> {
    readonly name: string;
    private readonly items: T[] = [];
}

// 4. Explicit return types for public APIs
class Producer {
    // GOOD: explicit
    produce(): AsyncGenerator<Data> { }

    // BAD: inferred (harder to maintain)
    produce() { }
}
```
