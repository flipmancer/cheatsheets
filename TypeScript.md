# TypeScript Cheatsheet

## Table of Contents

- [TypeScript Cheatsheet](#typescript-cheatsheet)
	- [Table of Contents](#table-of-contents)
	- [Generics Fundamentals](#generics-fundamentals)
	- [Generic Interfaces](#generic-interfaces)
	- [Advanced Generics](#advanced-generics)
	- [Abstract Classes](#abstract-classes)
	- [Type Guards](#type-guards)
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

```typescript
// User-defined type guard
function isError(obj: any): obj is Error {
    return obj instanceof Error;
}

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
```

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
