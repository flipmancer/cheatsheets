# JavaScript Core Libraries Cheatsheet

> Essential JavaScript libraries for utilities, data manipulation, and validation used in both browser and Node.js environments.

## Table of Contents

- [Overview](#overview)
- [Lodash](#lodash)
- [Day.js](#dayjs)
- [UUID](#uuid)
- [Zod](#zod)
- [Joi](#joi)
- [Axios](#axios)
- [Quick Reference](#quick-reference)
- [Resources](#resources)

## Overview

- Purpose: Quick reference for core JavaScript utility libraries used across frontend and backend
- Audience: Beginner to Intermediate
- Scope: Lodash, Day.js, UUID, Zod, Joi, Axios. Excludes framework-specific libraries (React, Vue) and backend-only libraries (Express, etc.)
- Updated: 2025-11-12

---

## Lodash

### What is Lodash?

- What it is: Modern JavaScript utility library providing 200+ functions for common programming tasks
- Why it matters: Simplifies array, object, string manipulation; handles edge cases; optimized for performance
- Mental model: Like a Swiss Army knife for JavaScript - one tool with many useful functions

### Installation

```shell
npm install lodash

# Or install individual functions (tree-shakeable)
npm install lodash.debounce lodash.clonedeep
```

### Common Operations

#### Arrays

```js
import _ from 'lodash';

// Chunk array into groups
_.chunk([1, 2, 3, 4, 5], 2);  // [[1, 2], [3, 4], [5]]

// Remove duplicates
_.uniq([1, 2, 2, 3, 3, 4]);  // [1, 2, 3, 4]

// Find difference between arrays
_.difference([1, 2, 3], [2, 3, 4]);  // [1]

// Flatten nested arrays
_.flatten([1, [2, 3], [4, [5]]]);  // [1, 2, 3, 4, [5]]
_.flattenDeep([1, [2, 3], [4, [5]]]);  // [1, 2, 3, 4, 5]

// Take first N elements
_.take([1, 2, 3, 4], 2);  // [1, 2]

// Intersection of arrays
_.intersection([1, 2, 3], [2, 3, 4]);  // [2, 3]
```

#### Objects

```js
// Deep clone (avoid reference issues)
const original = { a: 1, b: { c: 2 } };
const clone = _.cloneDeep(original);
clone.b.c = 99;
console.log(original.b.c);  // Still 2

// Get nested property safely
const user = { profile: { name: 'Alice' } };
_.get(user, 'profile.name');  // 'Alice'
_.get(user, 'profile.age', 25);  // 25 (default value)

// Set nested property
_.set(user, 'profile.age', 30);

// Omit properties
_.omit({ a: 1, b: 2, c: 3 }, ['a', 'c']);  // { b: 2 }

// Pick properties
_.pick({ a: 1, b: 2, c: 3 }, ['a', 'c']);  // { a: 1, c: 3 }

// Merge objects deeply
_.merge({ a: 1 }, { b: 2 }, { a: 3 });  // { a: 3, b: 2 }
```

#### Collections

```js
// Group by property
const users = [
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 30 },
  { name: 'Charlie', age: 25 }
];
_.groupBy(users, 'age');
// { '25': [{ name: 'Alice', age: 25 }, { name: 'Charlie', age: 25 }], '30': [{ name: 'Bob', age: 30 }] }

// Key by property (create map)
_.keyBy(users, 'name');
// { 'Alice': { name: 'Alice', age: 25 }, ... }

// Sort by multiple properties
_.orderBy(users, ['age', 'name'], ['asc', 'desc']);

// Find object
_.find(users, { age: 25 });  // { name: 'Alice', age: 25 }
```

#### Functions

```js
// Debounce (delay execution until idle)
const saveData = _.debounce(() => {
  console.log('Saving...');
}, 300);

// Call multiple times, but only executes once after 300ms idle
saveData();
saveData();
saveData();  // Only this triggers after 300ms

// Throttle (limit execution rate)
const trackScroll = _.throttle(() => {
  console.log('Scrolling...');
}, 1000);

window.addEventListener('scroll', trackScroll);  // Max once per second

// Memoize (cache results)
const expensiveCalculation = _.memoize((n) => {
  console.log('Computing...');
  return n * n;
});

expensiveCalculation(5);  // Logs "Computing...", returns 25
expensiveCalculation(5);  // Returns 25 (cached, no log)
```

#### Strings

```js
// Capitalize
_.capitalize('hello world');  // 'Hello world'

// Camel case
_.camelCase('foo bar');  // 'fooBar'

// Snake case
_.snakeCase('fooBar');  // 'foo_bar'

// Truncate
_.truncate('This is a long sentence', { length: 10 });  // 'This is...'
```

Common pitfalls:
- Using `_.clone()` instead of `_.cloneDeep()` for nested objects (shallow copy)
- Forgetting to import individual functions for tree-shaking
- Using Lodash for operations native to ES6+ (e.g., `_.map` vs `Array.map`)

---

## Day.js

### What is Day.js?

- What it is: Lightweight (2KB) alternative to Moment.js for parsing, validating, manipulating, and formatting dates
- Why it matters: Much smaller than Moment.js, immutable API, chainable
- Mental model: Like a calculator for dates - perform operations and format output

### Installation

```shell
npm install dayjs
```

### Common Operations

#### Parsing & Creating

```js
import dayjs from 'dayjs';

// Current date/time
dayjs();

// From string
dayjs('2025-11-12');
dayjs('2025-11-12 14:30:00');

// From Date object
dayjs(new Date());

// From timestamp
dayjs(1699804800000);

// From Unix timestamp (seconds)
dayjs.unix(1699804800);
```

#### Formatting

```js
const now = dayjs();

now.format();                    // '2025-11-12T14:30:00-05:00' (ISO 8601)
now.format('YYYY-MM-DD');        // '2025-11-12'
now.format('DD/MM/YYYY');        // '12/11/2025'
now.format('MMM DD, YYYY');      // 'Nov 12, 2025'
now.format('HH:mm:ss');          // '14:30:00'
now.format('h:mm A');            // '2:30 PM'
now.format('dddd, MMMM D, YYYY'); // 'Tuesday, November 12, 2025'
```

#### Manipulation

```js
const date = dayjs('2025-11-12');

// Add time
date.add(1, 'day');       // 2025-11-13
date.add(7, 'days');      // 2025-11-19
date.add(1, 'month');     // 2025-12-12
date.add(1, 'year');      // 2026-11-12

// Subtract time
date.subtract(1, 'day');  // 2025-11-11

// Start/end of period
date.startOf('month');    // 2025-11-01 00:00:00
date.endOf('month');      // 2025-11-30 23:59:59
date.startOf('day');      // 2025-11-12 00:00:00
```

#### Querying

```js
const date1 = dayjs('2025-11-12');
const date2 = dayjs('2025-11-15');

// Compare
date1.isBefore(date2);    // true
date1.isAfter(date2);     // false
date1.isSame(date2);      // false

// Difference
date2.diff(date1, 'day'); // 3

// Get components
date1.year();             // 2025
date1.month();            // 10 (0-indexed, so November)
date1.date();             // 12
date1.day();              // 2 (Tuesday, 0 = Sunday)
date1.hour();             // Current hour
```

#### Plugins

```js
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';
import utc from 'dayjs/plugin/utc';
import timezone from 'dayjs/plugin/timezone';

dayjs.extend(relativeTime);
dayjs.extend(utc);
dayjs.extend(timezone);

// Relative time
dayjs('2025-11-10').fromNow();  // '2 days ago'
dayjs('2025-11-15').fromNow();  // 'in 3 days'

// UTC
dayjs.utc();
dayjs().utc().format();

// Timezone
dayjs.tz('2025-11-12', 'America/New_York');
dayjs().tz('Asia/Tokyo').format();
```

Common pitfalls:
- Forgetting Day.js is immutable (methods return new instance)
- Using 0-indexed months (November = 10, not 11)
- Not loading required plugins for advanced features

---

## UUID

### What is UUID?

- What it is: Generate RFC4122 UUIDs (Universally Unique Identifiers)
- Why it matters: Create unique IDs without central coordination; used for request IDs, database keys, etc.
- Mental model: Like a lottery ticket number generator - astronomically unlikely to get duplicates

### Installation

```shell
npm install uuid
```

### Common Operations

```js
import { v4 as uuidv4, v5 as uuidv5, validate, version } from 'uuid';

// Generate UUID v4 (random)
const id = uuidv4();  // 'a3bb189e-8bf9-3888-9912-ace4e6543002'

// UUID v5 (namespace + name, deterministic)
const MY_NAMESPACE = '1b671a64-40d5-491e-99b0-da01ff1f3341';
const id5 = uuidv5('hello', MY_NAMESPACE);  // Always same for 'hello'

// Validate UUID
validate('a3bb189e-8bf9-3888-9912-ace4e6543002');  // true
validate('invalid-uuid');  // false

// Get UUID version
version('a3bb189e-8bf9-3888-9912-ace4e6543002');  // 4
```

### Practical Examples

```js
// Generate request ID
app.use((req, res, next) => {
  req.id = uuidv4();
  next();
});

// Generate subscription ID
const subscriptionId = uuidv4();
ws.send(JSON.stringify({
  id: subscriptionId,
  type: 'subscribe',
  payload: { channel: 'market_data' }
}));

// Generate deterministic ID
const orderId = uuidv5(`${userId}-${timestamp}`, MY_NAMESPACE);
```

Common pitfalls:
- Using UUID v1 (timestamp-based) when you need random UUIDs (use v4)
- Not validating UUIDs from external sources
- Storing UUIDs as strings when binary format is more efficient (36 bytes vs 16 bytes)

---

## Zod

### What is Zod?

- What it is: TypeScript-first schema validation library with static type inference
- Why it matters: Validate runtime data (API responses, user input) while maintaining TypeScript types
- Mental model: Like TypeScript types but for runtime - catch errors at the boundary

### Installation

```shell
npm install zod
```

### Common Schemas

#### Primitives

```ts
import { z } from 'zod';

// Primitives
const stringSchema = z.string();
const numberSchema = z.number();
const booleanSchema = z.boolean();
const dateSchema = z.date();

// Parse & validate
stringSchema.parse('hello');     // 'hello'
stringSchema.parse(123);         // Throws ZodError

// Safe parse (no throw)
const result = stringSchema.safeParse('hello');
if (result.success) {
  console.log(result.data);  // 'hello'
} else {
  console.log(result.error);
}
```

#### Objects

```ts
const userSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'user', 'guest']),
  createdAt: z.date()
});

// Infer TypeScript type from schema
type User = z.infer<typeof userSchema>;
// type User = {
//   id: string;
//   name: string;
//   email: string;
//   age?: number | undefined;
//   role: "admin" | "user" | "guest";
//   createdAt: Date;
// }

// Validate
const user = userSchema.parse({
  id: uuidv4(),
  name: 'Alice',
  email: 'alice@example.com',
  role: 'admin',
  createdAt: new Date()
});
```

#### Arrays & Records

```ts
// Arrays
const numbersSchema = z.array(z.number());
numbersSchema.parse([1, 2, 3]);  // [1, 2, 3]

// Array with min/max length
const tagsSchema = z.array(z.string()).min(1).max(10);

// Records (key-value map)
const scoresSchema = z.record(z.string(), z.number());
scoresSchema.parse({ alice: 95, bob: 87 });
```

#### Unions & Discriminated Unions

```ts
// Union
const idSchema = z.union([z.string(), z.number()]);
idSchema.parse('abc');  // 'abc'
idSchema.parse(123);    // 123

// Discriminated union
const eventSchema = z.discriminatedUnion('type', [
  z.object({ type: z.literal('click'), x: z.number(), y: z.number() }),
  z.object({ type: z.literal('keypress'), key: z.string() })
]);

type Event = z.infer<typeof eventSchema>;
// type Event = { type: 'click', x: number, y: number } | { type: 'keypress', key: string }
```

#### Transformations & Refinements

```ts
// Transform
const trimmedString = z.string().transform(s => s.trim());
trimmedString.parse('  hello  ');  // 'hello'

// Refine (custom validation)
const passwordSchema = z.string()
  .min(8)
  .refine(pw => /[A-Z]/.test(pw), 'Must contain uppercase letter')
  .refine(pw => /[0-9]/.test(pw), 'Must contain number');

// Preprocess
const dateSchema = z.preprocess(
  (arg) => typeof arg === 'string' ? new Date(arg) : arg,
  z.date()
);
dateSchema.parse('2025-11-12');  // Date object
```

### Practical Examples

```ts
// API route validation
const createOrderSchema = z.object({
  market: z.string(),
  side: z.enum(['buy', 'sell']),
  price: z.number().positive(),
  amount: z.number().positive()
});

app.post('/order', async (req, res) => {
  try {
    const order = createOrderSchema.parse(req.body);
    // order is typed and validated
    const result = await placeOrder(order);
    res.json(result);
  } catch (error) {
    if (error instanceof z.ZodError) {
      res.status(400).json({ errors: error.errors });
    }
  }
});

// Environment variable validation
const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.string().transform(Number),
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1)
});

const env = envSchema.parse(process.env);
// env.PORT is number, fully validated
```

Common pitfalls:
- Not using `.safeParse()` when you want to handle errors gracefully
- Forgetting to transform string numbers from environment variables
- Not leveraging type inference with `z.infer<>`

---

## Joi

### What is Joi?

- What it is: Powerful schema description language and data validator for JavaScript
- Why it matters: Mature, feature-rich validation with great error messages; Node.js standard
- Mental model: Define the shape of your data, Joi enforces it

### Installation

```shell
npm install joi
```

### Common Schemas

#### Basics

```js
const Joi = require('joi');

// String validation
const nameSchema = Joi.string().min(3).max(30).required();
const result = nameSchema.validate('Alice');

if (result.error) {
  console.log(result.error.details);
} else {
  console.log(result.value);  // 'Alice'
}

// Number validation
const ageSchema = Joi.number().integer().min(0).max(120);
```

#### Object Schemas

```js
const userSchema = Joi.object({
  id: Joi.string().guid({ version: 'uuidv4' }).required(),
  username: Joi.string().alphanum().min(3).max(30).required(),
  email: Joi.string().email().required(),
  password: Joi.string().pattern(/^[a-zA-Z0-9]{8,30}$/).required(),
  birthYear: Joi.number().integer().min(1900).max(2025),
  role: Joi.string().valid('admin', 'user', 'guest').default('user'),
  tags: Joi.array().items(Joi.string()).min(1).max(5),
  metadata: Joi.object().pattern(Joi.string(), Joi.any())
});

// Validate
const { error, value } = userSchema.validate({
  id: uuidv4(),
  username: 'alice123',
  email: 'alice@example.com',
  password: 'password123',
  tags: ['developer', 'javascript']
});
```

#### Alternatives & Conditionals

```js
// Alternative types
const idSchema = Joi.alternatives().try(
  Joi.string().uuid(),
  Joi.number().integer()
);

// Conditional validation
const schema = Joi.object({
  type: Joi.string().valid('card', 'paypal').required(),
  cardNumber: Joi.when('type', {
    is: 'card',
    then: Joi.string().creditCard().required(),
    otherwise: Joi.forbidden()
  }),
  paypalEmail: Joi.when('type', {
    is: 'paypal',
    then: Joi.string().email().required(),
    otherwise: Joi.forbidden()
  })
});
```

#### Custom Validation

```js
const schema = Joi.string().custom((value, helpers) => {
  if (value.startsWith('test_')) {
    return value;
  }
  return helpers.error('string.custom', { value });
}, 'custom validation');
```

### Practical Examples

```js
// Express middleware
const validateBody = (schema) => {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, { abortEarly: false });
    if (error) {
      return res.status(400).json({
        errors: error.details.map(d => ({
          field: d.path.join('.'),
          message: d.message
        }))
      });
    }
    req.body = value;  // Use validated/coerced value
    next();
  };
};

// Use in route
const createUserSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
  name: Joi.string().min(1).max(100).required()
});

app.post('/users', validateBody(createUserSchema), async (req, res) => {
  // req.body is validated
  const user = await createUser(req.body);
  res.json(user);
});
```

Common pitfalls:
- Not using `abortEarly: false` to get all validation errors at once
- Forgetting `.required()` - fields are optional by default
- Not stripping unknown keys with `.options({ stripUnknown: true })`

---

## Axios

### What is Axios?

- What it is: Promise-based HTTP client for browser and Node.js
- Why it matters: Better API than fetch, automatic JSON parsing, interceptors, timeouts, cancellation
- Mental model: Like fetch on steroids - all the features you wish fetch had

### Installation

```shell
npm install axios
```

### Basic Usage

#### GET Requests

```js
import axios from 'axios';

// Simple GET
const response = await axios.get('https://api.example.com/users');
console.log(response.data);  // Parsed JSON

// GET with query parameters
const response = await axios.get('https://api.example.com/users', {
  params: {
    page: 1,
    limit: 10,
    sort: 'name'
  }
});
// URL: https://api.example.com/users?page=1&limit=10&sort=name

// GET with headers
const response = await axios.get('https://api.example.com/profile', {
  headers: {
    'Authorization': `Bearer ${token}`,
    'X-Custom-Header': 'value'
  }
});
```

#### POST Requests

```js
// POST with JSON body
const response = await axios.post('https://api.example.com/users', {
  name: 'Alice',
  email: 'alice@example.com'
});

// POST with custom headers
const response = await axios.post(
  'https://api.example.com/orders',
  { product: 'laptop', quantity: 1 },
  {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    }
  }
);

// POST form data
const formData = new FormData();
formData.append('file', fileBlob);
formData.append('name', 'document.pdf');

const response = await axios.post('https://api.example.com/upload', formData, {
  headers: { 'Content-Type': 'multipart/form-data' }
});
```

#### Other Methods

```js
// PUT (update)
await axios.put('https://api.example.com/users/123', { name: 'Bob' });

// PATCH (partial update)
await axios.patch('https://api.example.com/users/123', { name: 'Bob' });

// DELETE
await axios.delete('https://api.example.com/users/123');

// HEAD (get headers only)
const response = await axios.head('https://api.example.com/users/123');
console.log(response.headers);
```

### Configuration

#### Instance with Defaults

```js
// Create instance with base configuration
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
    'X-API-Key': process.env.API_KEY
  }
});

// Use instance
const response = await api.get('/users');  // GET https://api.example.com/users
```

#### Interceptors

```js
// Request interceptor (add auth token)
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor (handle errors globally)
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Redirect to login
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

### Error Handling

```js
try {
  const response = await axios.get('https://api.example.com/users');
  console.log(response.data);
} catch (error) {
  if (error.response) {
    // Server responded with error status
    console.log('Status:', error.response.status);
    console.log('Data:', error.response.data);
    console.log('Headers:', error.response.headers);
  } else if (error.request) {
    // Request made but no response received
    console.log('No response:', error.request);
  } else {
    // Error setting up request
    console.log('Error:', error.message);
  }
}
```

### Advanced Features

#### Timeouts

```js
// Request-level timeout
const response = await axios.get('https://api.example.com/slow', {
  timeout: 5000  // 5 seconds
});
```

#### Cancellation

```js
const controller = new AbortController();

const request = axios.get('https://api.example.com/users', {
  signal: controller.signal
});

// Cancel request
controller.abort();

try {
  const response = await request;
} catch (error) {
  if (axios.isCancel(error)) {
    console.log('Request canceled');
  }
}
```

#### Progress Tracking

```js
// Upload progress
await axios.post('https://api.example.com/upload', formData, {
  onUploadProgress: (progressEvent) => {
    const percentCompleted = Math.round(
      (progressEvent.loaded * 100) / progressEvent.total
    );
    console.log(`Upload: ${percentCompleted}%`);
  }
});

// Download progress
await axios.get('https://api.example.com/large-file', {
  onDownloadProgress: (progressEvent) => {
    const percentCompleted = Math.round(
      (progressEvent.loaded * 100) / progressEvent.total
    );
    console.log(`Download: ${percentCompleted}%`);
  }
});
```

### Practical Examples

```js
// Retry logic
const axiosRetry = async (url, config = {}, retries = 3) => {
  for (let i = 0; i < retries; i++) {
    try {
      return await axios.get(url, config);
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
};

// Parallel requests
const [users, orders, products] = await Promise.all([
  axios.get('/users'),
  axios.get('/orders'),
  axios.get('/products')
]);

// Sequential requests with dependency
const user = await axios.get(`/users/${userId}`);
const orders = await axios.get(`/orders?userId=${user.data.id}`);
```

Common pitfalls:
- Not handling errors properly (check `error.response`, `error.request`, and `error.message`)
- Forgetting to set timeout (requests can hang indefinitely)
- Creating new axios instance for every request (use singleton with defaults)
- Not using interceptors for common logic (auth, logging)

---

## Quick Reference

### When to Use What

- **Lodash**: Array/object manipulation, debounce/throttle, deep cloning
- **Day.js**: Date parsing, formatting, manipulation (use over Moment.js)
- **UUID**: Generate unique IDs for requests, subscriptions, database keys
- **Zod**: TypeScript projects, API validation with type inference
- **Joi**: Node.js projects, mature validation with great error messages
- **Axios**: HTTP requests with interceptors, retry logic, progress tracking

### Bundle Size Comparison

- Lodash: ~72KB (full), ~4KB per function (tree-shaken)
- Day.js: ~2KB (core), +1-2KB per plugin
- UUID: ~3KB
- Zod: ~12KB
- Joi: ~146KB (Node.js only)
- Axios: ~13KB

### Performance Tips

1. **Lodash**: Import individual functions (`import debounce from 'lodash/debounce'`) for smaller bundles
2. **Day.js**: Only load plugins you need
3. **Zod**: Use `.safeParse()` in hot paths (avoids throwing)
4. **Axios**: Reuse axios instances, use interceptors instead of per-request logic

---

## Resources

### Official Documentation

- Lodash: https://lodash.com/docs
- Day.js: https://day.js.org/docs/en/installation/installation
- UUID: https://github.com/uuidjs/uuid
- Zod: https://zod.dev
- Joi: https://joi.dev/api/
- Axios: https://axios-http.com/docs/intro

### Alternatives

- **Lodash** → Ramda (functional), Underscore (classic)
- **Day.js** → date-fns (functional), Luxon (immutable)
- **Zod** → Yup, TypeBox, ArkType
- **Joi** → Yup, Ajv, Superstruct
- **Axios** → fetch (native), ky, got (Node.js)
