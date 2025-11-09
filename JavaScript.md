# Modern JavaScript Cheatsheet

> Complete guide from basics to advanced patterns. Covers ES6+ features, async programming, and modern best practices.

## Overview

- Purpose: Quick reference for JavaScript fundamentals through advanced patterns
- Audience: Beginner to Advanced
- Scope: Core language features, async programming, modern ES6+ syntax, common patterns. Excludes framework-specific code (React, Vue, etc.)
- Updated: 2025-11-09

---

## JavaScript Basics

### 1. Variables & Data Types

Short explanation:

- What it is: JavaScript has dynamic typing - variables can hold any type of value
- Why it matters: Understanding types prevents bugs and helps write predictable code
- Mental model: Think of variables as labeled boxes that can hold different types of contents

```js
// Variable declarations
let name = 'Alice';        // Mutable (can reassign)
const age = 25;            // Immutable (cannot reassign)
var oldWay = 'avoid';      // Function-scoped (avoid in modern JS)

// Primitive types
const str = 'Hello';       // String
const num = 42;            // Number
const bool = true;         // Boolean
const nothing = null;      // Null (intentional absence)
const undef = undefined;   // Undefined (not assigned)
const bigInt = 123n;       // BigInt (large integers)
const sym = Symbol('id');  // Symbol (unique identifier)

// Reference types
const arr = [1, 2, 3];               // Array
const obj = { key: 'value' };        // Object
const func = () => {};               // Function
```

Common pitfalls:

- Using `var` instead of `let`/`const` (function vs block scope)
- Confusing `null` and `undefined`
- Mutating `const` object properties (const prevents reassignment, not mutation)

---

### 2. Operators

Short explanation:

- What it is: Symbols that perform operations on values
- When to use: Arithmetic, comparisons, logical operations, assignments
- Gotchas: `==` vs `===`, `&&` and `||` short-circuit behavior

```js
// Arithmetic
const sum = 5 + 3;         // 8
const diff = 10 - 4;       // 6
const product = 3 * 4;     // 12
const quotient = 15 / 3;   // 5
const remainder = 10 % 3;  // 1
const power = 2 ** 3;      // 8 (exponentiation)

// Comparison
5 === 5;                   // true (strict equality - type + value)
5 == '5';                  // true (loose equality - converts types)
5 !== '5';                 // true (strict inequality)
5 > 3;                     // true
5 <= 5;                    // true

// Logical
true && false;             // false (AND)
true || false;             // true (OR)
!true;                     // false (NOT)

// Assignment
let x = 10;
x += 5;                    // x = x + 5 → 15
x *= 2;                    // x = x * 2 → 30
x++;                       // x = x + 1 → 31
x--;                       // x = x - 1 → 30

// Nullish coalescing
const value = null ?? 'default';        // 'default'
const value2 = 0 ?? 'default';          // 0 (only null/undefined trigger ??)

// Optional chaining
const user = { profile: { name: 'Bob' } };
user?.profile?.name;                     // 'Bob'
user?.address?.street;                   // undefined (no error)
```

---

### 3. Control Flow

Short explanation:

- What it is: Statements that control the execution path
- Mental model: Like a flowchart - make decisions and repeat actions

```js
// If-else
const age = 18;
if (age >= 18) {
  console.log('Adult');
} else if (age >= 13) {
  console.log('Teen');
} else {
  console.log('Child');
}

// Ternary operator
const status = age >= 18 ? 'Adult' : 'Minor';

// Switch
const day = 'Monday';
switch (day) {
  case 'Monday':
    console.log('Start of week');
    break;
  case 'Friday':
    console.log('Almost weekend!');
    break;
  default:
    console.log('Regular day');
}
```

---

### 4. Loops

Short explanation:

- What it is: Repeat code multiple times
- When to use: Processing arrays, repeating tasks, iterating over data

```js
// For loop
for (let i = 0; i < 5; i++) {
  console.log(i);  // 0, 1, 2, 3, 4
}

// While loop
let count = 0;
while (count < 3) {
  console.log(count);
  count++;
}

// Do-while (runs at least once)
let num = 0;
do {
  console.log(num);
  num++;
} while (num < 3);

// For...of (iterate over values)
const fruits = ['apple', 'banana', 'orange'];
for (const fruit of fruits) {
  console.log(fruit);
}

// For...in (iterate over keys - avoid for arrays)
const person = { name: 'Alice', age: 25 };
for (const key in person) {
  console.log(`${key}: ${person[key]}`);
}

// Break and continue
for (let i = 0; i < 10; i++) {
  if (i === 3) continue;  // Skip 3
  if (i === 7) break;     // Stop at 7
  console.log(i);         // 0, 1, 2, 4, 5, 6
}
```

---

### 5. Functions

Short explanation:

- What it is: Reusable blocks of code
- Why it matters: DRY principle, modularity, maintainability
- Mental model: Like a recipe - define once, use many times

```js
// Function declaration
function greet(name) {
  return `Hello, ${name}!`;
}

// Function expression
const greet2 = function(name) {
  return `Hello, ${name}!`;
};

// Arrow function (ES6+)
const greet3 = (name) => `Hello, ${name}!`;
const greet4 = name => `Hello, ${name}!`;  // Single param, no parens
const add = (a, b) => a + b;               // Implicit return

// Default parameters
function multiply(a, b = 1) {
  return a * b;
}
multiply(5);     // 5 (b defaults to 1)
multiply(5, 3);  // 15

// Rest parameters
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4);  // 10

// Arrow function gotcha: no 'this' binding
const obj = {
  name: 'Object',
  regular: function() { console.log(this.name); },  // 'Object'
  arrow: () => { console.log(this.name); }          // undefined (this = window/global)
};
```

Common pitfalls:

- Forgetting `return` in arrow functions with braces `{}`
- Arrow functions don't have their own `this`
- Hoisting differences between declarations and expressions

---

## Intermediate JavaScript

### 6. Arrays

Short explanation:

- What it is: Ordered collections of values
- Mental model: Numbered list (zero-indexed)

```js
// Creating arrays
const arr = [1, 2, 3];
const arr2 = new Array(5);           // [empty × 5]
const arr3 = Array.from('hello');    // ['h', 'e', 'l', 'l', 'o']

// Accessing elements
arr[0];        // 1 (first element)
arr[arr.length - 1];  // 3 (last element)
arr.at(-1);    // 3 (negative indexing, ES2022)

// Common methods
const numbers = [1, 2, 3, 4, 5];

// Transform
numbers.map(n => n * 2);              // [2, 4, 6, 8, 10]
numbers.filter(n => n % 2 === 0);     // [2, 4]
numbers.reduce((sum, n) => sum + n, 0);  // 15

// Search
numbers.find(n => n > 3);             // 4 (first match)
numbers.findIndex(n => n > 3);        // 3 (index of first match)
numbers.includes(3);                  // true
numbers.indexOf(3);                   // 2

// Mutating methods (change original array)
numbers.push(6);                      // Add to end → [1, 2, 3, 4, 5, 6]
numbers.pop();                        // Remove from end → [1, 2, 3, 4, 5]
numbers.unshift(0);                   // Add to start → [0, 1, 2, 3, 4, 5]
numbers.shift();                      // Remove from start → [1, 2, 3, 4, 5]
numbers.splice(2, 1);                 // Remove at index 2 → [1, 2, 4, 5]
numbers.sort();                       // Sort (mutates)
numbers.reverse();                    // Reverse (mutates)

// Non-mutating alternatives
const copy = [...numbers];            // Spread operator
const sliced = numbers.slice(1, 3);   // [2, 3] (doesn't mutate)
const joined = numbers.concat([6, 7]);  // [1, 2, 3, 4, 5, 6, 7]

// Chaining
[1, 2, 3, 4]
  .filter(n => n % 2 === 0)
  .map(n => n * 2)
  .reduce((sum, n) => sum + n, 0);    // 12
```

---

### 7. Objects

Short explanation:

- What it is: Collections of key-value pairs
- Mental model: Dictionary or map - lookup by name

```js
// Creating objects
const person = {
  name: 'Alice',
  age: 25,
  greet() {                           // Method shorthand
    return `Hi, I'm ${this.name}`;
  }
};

// Accessing properties
person.name;                          // 'Alice' (dot notation)
person['age'];                        // 25 (bracket notation)
const key = 'name';
person[key];                          // 'Alice' (dynamic key)

// Adding/modifying
person.email = 'alice@example.com';   // Add property
person.age = 26;                      // Modify property
delete person.email;                  // Remove property

// Computed property names
const prop = 'firstName';
const user = {
  [prop]: 'Bob',                      // Dynamic key
  [`${prop}Length`]: 3
};

// Object methods
Object.keys(person);                  // ['name', 'age', 'greet']
Object.values(person);                // ['Alice', 26, function]
Object.entries(person);               // [['name', 'Alice'], ['age', 26], ...]

// Copying objects (shallow)
const copy1 = { ...person };          // Spread
const copy2 = Object.assign({}, person);

// Deep merge (shallow by default)
const merged = { ...person, city: 'NYC', age: 27 };  // Override age

// Check property existence
'name' in person;                     // true
person.hasOwnProperty('name');        // true

// Object destructuring
const { name, age } = person;
const { name: userName, age: userAge = 18 } = person;  // Rename + default
```

---

### 8. Destructuring & Spread

Short explanation:

- What it is: Extract values from arrays/objects into variables
- Why it matters: Cleaner code, less repetition
- Mental model: Unpacking a box into separate items

```js
// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first = 1, second = 2, rest = [3, 4, 5]

const [a, , c] = [1, 2, 3];           // Skip elements
// a = 1, c = 3

// Object destructuring
const person = { name: 'Alice', age: 25, city: 'NYC' };
const { name, age } = person;

const { name: userName, age: userAge } = person;  // Rename
const { name, country = 'USA' } = person;         // Default value

// Nested destructuring
const user = { id: 1, profile: { email: 'a@b.com' } };
const { profile: { email } } = user;

// Function parameter destructuring
function greet({ name, age }) {
  return `${name} is ${age} years old`;
}
greet(person);

// Spread operator
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];         // [1, 2, 3, 4, 5]

const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 };       // { a: 1, b: 2, c: 3 }

// Rest parameters (functions)
function sum(first, ...rest) {
  return first + rest.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4);  // 10
```

---

### 9. Classes (ES6+)

Short explanation:

- What it is: Blueprint for creating objects with shared behavior
- Mental model: Cookie cutter - same shape, different instances

```js
// Class definition
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  // Method
  greet() {
    return `Hi, I'm ${this.name}`;
  }

  // Getter
  get info() {
    return `${this.name}, ${this.age}`;
  }

  // Setter
  set age(value) {
    if (value < 0) throw new Error('Invalid age');
    this._age = value;
  }

  // Static method (called on class, not instance)
  static create(name) {
    return new Person(name, 0);
  }
}

// Usage
const alice = new Person('Alice', 25);
alice.greet();                        // "Hi, I'm Alice"
Person.create('Bob');                 // New Person with age 0

// Inheritance
class Developer extends Person {
  constructor(name, age, language) {
    super(name, age);                 // Call parent constructor
    this.language = language;
  }

  code() {
    return `${this.name} codes in ${this.language}`;
  }

  // Override parent method
  greet() {
    return `${super.greet()}. I code in ${this.language}`;
  }
}

const dev = new Developer('Charlie', 30, 'JavaScript');
dev.code();                           // "Charlie codes in JavaScript"
dev.greet();                          // "Hi, I'm Charlie. I code in JavaScript"

// Private fields (ES2022)
class BankAccount {
  #balance = 0;                       // Private field

  deposit(amount) {
    this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount();
account.deposit(100);
account.getBalance();                 // 100
account.#balance;                     // SyntaxError: Private field
```

---

## Advanced JavaScript

### 10. Async/Await & Promises

Short explanation:

- What it is: Handle asynchronous operations (network requests, timers, file I/O)
- Why it matters: JavaScript is single-threaded; async prevents blocking
- Mental model: Promises are IOUs - "I promise to give you a value later"

```js
// Creating a Promise
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('Success!');
    } else {
      reject('Failed!');
    }
  }, 1000);
});

// Using Promises (.then/.catch)
promise
  .then(result => console.log(result))
  .catch(error => console.error(error))
  .finally(() => console.log('Done'));

// Async/Await (cleaner syntax)
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
  }
}

// Parallel execution
async function fetchMultiple() {
  const [users, posts] = await Promise.all([
    fetch('/api/users').then(r => r.json()),
    fetch('/api/posts').then(r => r.json())
  ]);
  return { users, posts };
}

// Promise utilities
Promise.all([promise1, promise2]);      // Wait for all (fails if any fail)
Promise.race([promise1, promise2]);     // First to complete
Promise.allSettled([p1, p2]);           // Wait for all (never fails)
Promise.any([p1, p2]);                  // First to succeed

// Error handling
async function riskyOperation() {
  try {
    const result = await fetch('/api/data');
    if (!result.ok) throw new Error('Network error');
    return await result.json();
  } catch (error) {
    console.error('Caught:', error);
    throw error;  // Re-throw if needed
  }
}

// Common pattern: retry logic
async function fetchWithRetry(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(url).then(r => r.json());
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
  }
}

// Async Generators (streaming data)
async function* dataStream() {
  for (let i = 0; i < 10; i++) {
    await new Promise(resolve => setTimeout(resolve, 100));
    yield { id: i, timestamp: Date.now() };
  }
}

// Consuming async generators
for await (const item of dataStream()) {
  console.log(item);  // Processes one at a time
}

// Backpressure control (limit concurrent operations)
async function processWithLimit(items, fn, limit = 5) {
  const executing = [];

  for (const item of items) {
    const promise = fn(item).then(() => {
      executing.splice(executing.indexOf(promise), 1);
    });
    executing.push(promise);

    if (executing.length >= limit) {
      await Promise.race(executing);
    }
  }

  await Promise.all(executing);
}

// Usage: Process array with max 5 concurrent operations
await processWithLimit(
  [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
  async (item) => fetch(`/api/item/${item}`),
  5
);

// Pipeline pattern (compose async functions)
async function pipeline(data, ...fns) {
  let result = data;
  for (const fn of fns) {
    result = await fn(result);
  }
  return result;
}

// Usage
const processData = (data) => pipeline(
  data,
  validateData,
  transformData,
  saveData
);
```

Common pitfalls:

- Forgetting `await` (returns Promise instead of value)
- Not handling errors with `try/catch`
- Sequential awaits when parallel would work (use `Promise.all`)
- Memory issues with unbounded async generators

---

### 11. Closures

Short explanation:

- What it is: Function that remembers variables from its outer scope
- Why it matters: Data privacy, factory functions, callbacks
- Mental model: Backpack - function carries its environment

```js
// Basic closure
function outer() {
  const secret = 'hidden';

  function inner() {
    console.log(secret);  // Can access 'secret'
  }

  return inner;
}

const fn = outer();
fn();  // 'hidden' (secret is still accessible)

// Counter example (data privacy)
function createCounter() {
  let count = 0;  // Private variable

  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; }
  };
}

const counter = createCounter();
counter.increment();  // 1
counter.increment();  // 2
counter.getCount();   // 2
// counter.count is not accessible directly

// Factory pattern
function createMultiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);
double(5);  // 10
triple(5);  // 15

// Common use: Event handlers
function setupButton() {
  const buttonId = 'myButton';

  document.getElementById(buttonId).addEventListener('click', function() {
    console.log(`${buttonId} clicked`);  // Closure over buttonId
  });
}
```

---

### 12. Prototypes & Inheritance

Short explanation:

- What it is: JavaScript's inheritance model (before ES6 classes)
- Mental model: Chain of objects - if property not found, look up the chain

```js
// Every object has a prototype
const obj = {};
obj.__proto__;  // Object.prototype

// Constructor function (pre-ES6)
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function() {
  return `Hi, I'm ${this.name}`;
};

const alice = new Person('Alice', 25);
alice.greet();  // "Hi, I'm Alice"

// Prototype chain
alice.hasOwnProperty('name');        // true (own property)
alice.hasOwnProperty('greet');       // false (inherited from prototype)
'greet' in alice;                    // true (in prototype chain)

// Object.create (manual prototype)
const personProto = {
  greet() {
    return `Hi, I'm ${this.name}`;
  }
};

const bob = Object.create(personProto);
bob.name = 'Bob';
bob.greet();  // "Hi, I'm Bob"

// Checking prototypes
Object.getPrototypeOf(alice) === Person.prototype;  // true
alice instanceof Person;                            // true
```

---

### 13. this Keyword

Short explanation:

- What it is: Context object for function execution
- Pitfalls: Value depends on HOW function is called
- Mental model: "Who is calling this function?"

```js
// Regular function: this = caller
const person = {
  name: 'Alice',
  greet() {
    console.log(this.name);
  }
};
person.greet();  // 'Alice' (this = person)

const greetFn = person.greet;
greetFn();  // undefined (this = global/window)

// Arrow function: this = lexical (inherited from outer scope)
const person2 = {
  name: 'Bob',
  greet: () => {
    console.log(this.name);  // undefined (this = outer scope, not person2)
  }
};

// Binding this
const boundGreet = person.greet.bind(person);
boundGreet();  // 'Alice' (this always = person)

// call and apply
person.greet.call({ name: 'Charlie' });    // 'Charlie'
person.greet.apply({ name: 'David' });     // 'David'

// Class methods: this = instance
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(this.name);
  }

  // Arrow function preserves this
  greetArrow = () => {
    console.log(this.name);
  }
}

const alice = new Person('Alice');
const greet = alice.greet;
greet();  // undefined (lost context)

const greetArrow = alice.greetArrow;
greetArrow();  // 'Alice' (arrow function binds this)
```

---

### 14. Modules (ES6)

Short explanation:

- What it is: Split code into reusable files
- Why it matters: Organization, namespace isolation, dependency management

```js
// math.js (exporting)
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export default function multiply(a, b) {
  return a * b;
}

// app.js (importing)
import multiply from './math.js';           // Default import
import { PI, add } from './math.js';        // Named imports
import * as math from './math.js';          // Namespace import

multiply(2, 3);  // 6
add(5, 3);       // 8
math.PI;         // 3.14159

// Rename imports
import { add as sum } from './math.js';

// Re-exporting
export { PI, add } from './math.js';
export * from './math.js';

// Dynamic imports (code splitting)
async function loadModule() {
  const { add } = await import('./math.js');
  return add(2, 3);
}
```

---

### 15. Error Handling

Short explanation:

- What it is: Catch and handle runtime errors
- Mental model: Safety net for unexpected situations

```js
// Try-catch
try {
  const result = riskyOperation();
} catch (error) {
  console.error('Error:', error.message);
} finally {
  console.log('Always runs');
}

// Throwing errors
function divide(a, b) {
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
}

// Custom error types
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
  }
}

function validateAge(age) {
  if (age < 0) {
    throw new ValidationError('Age cannot be negative');
  }
}

try {
  validateAge(-5);
} catch (error) {
  if (error instanceof ValidationError) {
    console.log('Validation failed:', error.message);
  } else {
    throw error;  // Re-throw unknown errors
  }
}

// Async error handling
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Fetch failed:', error);
    return null;  // Graceful fallback
  }
}

// Global error handler (browser)
window.addEventListener('error', (event) => {
  console.error('Global error:', event.error);
});

// Unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise:', event.reason);
});
```

---

## Common Patterns

### 16. Debouncing & Throttling

Short explanation:

- What it is: Control function execution rate
- When to use: Search inputs, scroll handlers, resize events
- Mental model: Debounce = wait until user stops; Throttle = limit frequency

```js
// Debounce: Execute after delay, reset timer on each call
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}

// Usage: Search input
const searchInput = document.querySelector('#search');
const debouncedSearch = debounce((value) => {
  console.log('Searching for:', value);
  // API call here
}, 300);

searchInput.addEventListener('input', (e) => {
  debouncedSearch(e.target.value);
});

// Throttle: Execute at most once per interval
function throttle(func, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Usage: Scroll handler
const throttledScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 200);

window.addEventListener('scroll', throttledScroll);
```

---

### 17. Memoization

Short explanation:

- What it is: Cache function results to avoid recomputation
- When to use: Expensive calculations, recursive functions

```js
// Simple memoization
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Example: Fibonacci
const fibonacci = memoize(function(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

fibonacci(40);  // Fast with memoization
```

---

### 18. Currying & Partial Application

Short explanation:

- What it is: Transform function with multiple args into sequence of functions
- Mental model: Build up function arguments piece by piece

```js
// Currying
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...nextArgs) {
      return curried.apply(this, args.concat(nextArgs));
    };
  };
}

// Example
function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = curry(add);
curriedAdd(1)(2)(3);        // 6
curriedAdd(1, 2)(3);        // 6
curriedAdd(1)(2, 3);        // 6

// Partial application
const add5 = curriedAdd(5);
add5(10, 15);               // 30

// Real-world example: API calls
const fetchData = curry((baseUrl, endpoint, params) => {
  const url = `${baseUrl}${endpoint}?${new URLSearchParams(params)}`;
  return fetch(url).then(r => r.json());
});

const fetchFromAPI = fetchData('https://api.example.com');
const fetchUsers = fetchFromAPI('/users');
fetchUsers({ page: 1, limit: 10 });
```

---

### 19. Design Patterns

```js
// Singleton
const Singleton = (function() {
  let instance;

  function createInstance() {
    return { id: Math.random() };
  }

  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

const s1 = Singleton.getInstance();
const s2 = Singleton.getInstance();
s1 === s2;  // true (same instance)

// Module pattern (IIFE)
const Calculator = (function() {
  // Private
  let result = 0;

  // Public API
  return {
    add(x) { result += x; return this; },
    subtract(x) { result -= x; return this; },
    getResult() { return result; }
  };
})();

Calculator.add(5).subtract(2).getResult();  // 3

// Observer pattern (pub/sub)
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }

  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(cb => cb(data));
    }
  }

  off(event, callback) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(cb => cb !== callback);
    }
  }
}

const emitter = new EventEmitter();
emitter.on('data', (data) => console.log('Received:', data));
emitter.emit('data', { value: 42 });
```

---

## Quick Reference

Key JavaScript gotchas and conventions:

1. Always use `const` by default, `let` when reassignment needed, avoid `var`
2. Use `===` instead of `==` (strict equality)
3. Arrow functions don't bind `this`
4. Array methods like `map`, `filter`, `reduce` don't mutate original
5. `async` functions always return a Promise
6. Semicolons are optional but recommended for consistency
7. Use template literals (backticks) for string interpolation
8. Destructuring works on arrays and objects

---

## Debugging Tips

```js
// Console methods
console.log('Basic logging');
console.error('Error message');
console.warn('Warning');
console.table([{ a: 1 }, { a: 2 }]);  // Pretty table
console.time('operation');            // Start timer
console.timeEnd('operation');         // End timer

// Debugger
function buggyFunction() {
  debugger;  // Pause execution here (in DevTools)
  const value = riskyOperation();
  return value;
}

// Stack traces
console.trace('How did we get here?');

// Conditional logging
const DEBUG = true;
DEBUG && console.log('Debug info');
```

Checklist:

- Check browser console for errors and warnings
- Use `console.log` strategically (not everywhere)
- Set breakpoints in DevTools
- Use React/Vue DevTools for framework-specific debugging
- Check Network tab for API issues
- Validate data types with `typeof` and `instanceof`

---

## Performance Tips

1. **Avoid global variables** - pollutes namespace, slower lookup
2. **Use `const` and `let`** - block scope is more predictable
3. **Debounce/throttle** event handlers (scroll, resize, input)
4. **Memoize expensive calculations** - cache results
5. **Use `Promise.all`** for parallel async operations
6. **Lazy load** modules with dynamic `import()`
7. **Avoid memory leaks** - remove event listeners, clear intervals
8. **Use Web Workers** for CPU-intensive tasks (off main thread)

---

## Security & Safety

- **Validate all inputs** - never trust user data
- **Sanitize HTML** - prevent XSS attacks (use libraries like DOMPurify)
- **Use HTTPS** - encrypt data in transit
- **Don't expose secrets** - API keys, tokens (use env vars)
- **Validate JSON** - parse safely with `try/catch`
- **Use Content Security Policy (CSP)** - restrict script sources
- **Avoid `eval()`** - executes arbitrary code

---

## Modern ES6+ Features Reference

### Template Literals

```js
const name = 'Alice';
const greeting = `Hello, ${name}!`;          // String interpolation
const multiline = `Line 1
Line 2`;                                      // Multi-line strings

// Tagged templates
function highlight(strings, ...values) {
  return strings.reduce((acc, str, i) => {
    return acc + str + (values[i] ? `<mark>${values[i]}</mark>` : '');
  }, '');
}

const html = highlight`Name: ${name}, Age: ${25}`;
```

### Optional Chaining & Nullish Coalescing

```js
const user = { profile: { name: 'Alice' } };

// Optional chaining
user?.profile?.name;           // 'Alice'
user?.address?.street;         // undefined (no error)
user?.getName?.();             // undefined (safe method call)

// Nullish coalescing
const value = null ?? 'default';      // 'default'
const value2 = 0 ?? 'default';        // 0 (only null/undefined)
const value3 = '' ?? 'default';       // '' (empty string is falsy but not nullish)

// vs OR operator
const value4 = 0 || 'default';        // 'default' (0 is falsy)
```

### Numeric Separators

```js
const billion = 1_000_000_000;        // Easier to read
const hex = 0xFF_EC_DE_5E;
const binary = 0b1010_0001_1000_0101;
```

---

## Next Steps

- **Practice regularly** - build small projects (todo app, calculator, API client)
- **Learn a framework** - React, Vue, or Svelte for modern web apps
- **Explore Node.js** - JavaScript on the server
- **Study algorithms** - improve problem-solving with data structures
- **Read source code** - learn from popular libraries on GitHub
- **Join communities** - r/javascript, Discord servers, Stack Overflow

---

## Resources

- Official docs: [MDN JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- Interactive learning: [javascript.info](https://javascript.info)
- Style guide: [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- Books: *Eloquent JavaScript*, *You Don't Know JS*
- Practice: [Codewars](https://www.codewars.com), [LeetCode](https://leetcode.com)
