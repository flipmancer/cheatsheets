# Node.js Runtime Cheatsheet

> Complete guide to Node.js runtime features, I/O operations, and server-side development. Covers file system, databases, streams, and Node-specific APIs.

## Table of Contents

- [Node.js Runtime Cheatsheet](#nodejs-runtime-cheatsheet)
	- [Table of Contents](#table-of-contents)
	- [Overview](#overview)
	- [Setup \& Installation](#setup--installation)
		- [1. Installation](#1-installation)
		- [2. Package Management](#2-package-management)
	- [Core Node.js APIs](#core-nodejs-apis)
		- [3. File System (fs)](#3-file-system-fs)
		- [4. Streams](#4-streams)
		- [5. HTTP Server](#5-http-server)
		- [6. Path Module](#6-path-module)
	- [Database Integration](#database-integration)
		- [7. MongoDB](#7-mongodb)
		- [8. Redis](#8-redis)
		- [9. PostgreSQL / MySQL](#9-postgresql--mysql)
	- [Process \& Environment](#process--environment)
		- [10. Environment Variables](#10-environment-variables)
		- [11. Process Management](#11-process-management)
		- [12. Event Loop \& Timers](#12-event-loop--timers)
	- [Performance \& Optimization](#performance--optimization)
		- [13. Worker Threads](#13-worker-threads)
		- [14. Clustering](#14-clustering)
	- [Security \& Best Practices](#security--best-practices)
		- [15. Security](#15-security)
	- [Debugging \& Monitoring](#debugging--monitoring)
		- [16. Debugging](#16-debugging)
	- [Quick Reference](#quick-reference)
	- [Resources](#resources)

## Overview

- Purpose: Quick reference for Node.js runtime features and server-side development
- Audience: Intermediate to Advanced (assumes JavaScript knowledge)
- Scope: Node.js runtime APIs, I/O operations, databases, streams, worker threads, process management. Excludes general JavaScript syntax (see JAVASCRIPT-CHEATSHEET.md)
- Updated: 2025-11-09

---

## Setup & Installation

### 1. Installation

Short explanation:

- What it is: Node.js JavaScript runtime built on Chrome's V8 engine
- Why it matters: Run JavaScript outside the browser (servers, CLI tools, scripts)

```shell
# Install Node.js (Windows - using Chocolatey)
choco install nodejs

# Install Node.js (macOS - using Homebrew)
brew install node

# Install Node.js (Linux - using package manager)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version

# Run JavaScript file
node app.js

# Interactive REPL
node
```

---

### 2. Package Management

Short explanation:

- What it is: npm (Node Package Manager) manages dependencies
- Mental model: Like a library catalog - install, update, track packages

```shell
# Initialize new project
npm init -y

# Install dependencies
npm install express
npm install --save-dev typescript

# Install globally
npm install -g nodemon

# Update dependencies
npm update

# Audit for vulnerabilities
npm audit
npm audit fix

# Run scripts (defined in package.json)
npm run dev
npm start
npm test
```

**package.json basics:**

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "nodemon app.js",
    "start": "node app.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

---

## Core Node.js APIs

### 3. File System (fs)

Short explanation:

- What it is: Read/write files and directories on disk
- Why it matters: Node.js excels at I/O operations (non-blocking)
- Mental model: Async by default - don't block the event loop

```js
import { readFile, writeFile, appendFile, unlink, mkdir, readdir } from 'fs/promises';
import { existsSync } from 'fs';

// Read file (async)
const data = await readFile('data.json', 'utf-8');
const parsed = JSON.parse(data);

// Write file (overwrites)
await writeFile('output.json', JSON.stringify({ result: 'ok' }, null, 2));

// Append to file
await appendFile('log.txt', `${new Date().toISOString()} - Event logged\n`);

// Check if file exists (sync - only use for startup checks)
if (existsSync('config.json')) {
  console.log('Config found');
}

// Create directory
await mkdir('data/uploads', { recursive: true });

// List directory contents
const files = await readdir('data');
console.log(files);  // ['file1.txt', 'file2.json']

// Delete file
await unlink('temp.txt');

// Read file stats
import { stat } from 'fs/promises';
const stats = await stat('file.txt');
console.log(stats.size);         // File size in bytes
console.log(stats.isDirectory()); // false
```

Common pitfalls:

- Using sync methods in server code (blocks event loop)
- Not handling errors (file not found, permissions)
- Path issues (use `path` module for cross-platform paths)

---

### 4. Streams

Short explanation:

- What it is: Process large files/data without loading everything into memory
- Why it matters: Memory efficiency - handle GB files with MB memory
- Mental model: Water pipe - data flows in chunks

```js
import { createReadStream, createWriteStream } from 'fs';
import { pipeline } from 'stream/promises';
import { Transform } from 'stream';

// Read large file in chunks
const readStream = createReadStream('large-file.csv');

readStream.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
});

readStream.on('end', () => {
  console.log('File read complete');
});

readStream.on('error', (error) => {
  console.error('Error:', error);
});

// Write stream
const writeStream = createWriteStream('output.txt');
writeStream.write('Line 1\n');
writeStream.write('Line 2\n');
writeStream.end();

// Transform stream (process data in chunks)
const transformer = new Transform({
  transform(chunk, encoding, callback) {
    // Convert to uppercase
    const transformed = chunk.toString().toUpperCase();
    callback(null, transformed);
  }
});

// Pipeline (recommended - handles backpressure)
await pipeline(
  createReadStream('input.txt'),
  transformer,
  createWriteStream('output.txt')
);

// Practical example: Process CSV line by line
import { createInterface } from 'readline';

const fileStream = createReadStream('data.csv');
const rl = createInterface({
  input: fileStream,
  crlfDelay: Infinity
});

for await (const line of rl) {
  const [id, name, value] = line.split(',');
  console.log({ id, name, value });
}
```

When to use:

- Large files (> 100 MB)
- Real-time data processing
- HTTP uploads/downloads
- Log file processing

---

### 5. HTTP Server

Short explanation:

- What it is: Create web servers and APIs
- Mental model: Request/response cycle - client asks, server responds

```js
import { createServer } from 'http';

// Basic HTTP server
const server = createServer((req, res) => {
  // req = incoming request
  // res = outgoing response

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ message: 'Hello World' }));
});

server.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});

// Routing example
const server2 = createServer((req, res) => {
  const { method, url } = req;

  if (method === 'GET' && url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>Home Page</h1>');
  } else if (method === 'GET' && url === '/api/data') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ data: [1, 2, 3] }));
  } else if (method === 'POST' && url === '/api/data') {
    let body = '';
    req.on('data', chunk => { body += chunk; });
    req.on('end', () => {
      const data = JSON.parse(body);
      res.writeHead(201, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ success: true, data }));
    });
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});

server2.listen(3000);

// HTTPS server
import { createServer as createHttpsServer } from 'https';
import { readFileSync } from 'fs';

const options = {
  key: readFileSync('private-key.pem'),
  cert: readFileSync('certificate.pem')
};

const httpsServer = createHttpsServer(options, (req, res) => {
  res.writeHead(200);
  res.end('Secure connection');
});

httpsServer.listen(443);
```

**Using Express (recommended for production):**

```js
import express from 'express';

const app = express();

// Middleware
app.use(express.json());  // Parse JSON bodies

// Routes
app.get('/', (req, res) => {
  res.send('Home Page');
});

app.get('/api/users', async (req, res) => {
  const users = await getUsers();
  res.json(users);
});

app.post('/api/users', async (req, res) => {
  const user = await createUser(req.body);
  res.status(201).json(user);
});

// Error handling
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal Server Error' });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

---

### 6. Path Module

Short explanation:

- What it is: Cross-platform file path handling
- Why it matters: Windows uses `\`, Unix uses `/` - path module handles both

```js
import path from 'path';

// Join paths (cross-platform)
const filePath = path.join('data', 'users', 'profile.json');
// Windows: data\users\profile.json
// Unix: data/users/profile.json

// Resolve absolute path
const absolutePath = path.resolve('data', 'file.txt');
// /home/user/project/data/file.txt

// Get directory name
path.dirname('/home/user/file.txt');  // /home/user

// Get filename
path.basename('/home/user/file.txt');  // file.txt

// Get extension
path.extname('file.txt');  // .txt

// Parse path
path.parse('/home/user/file.txt');
// { root: '/', dir: '/home/user', base: 'file.txt', ext: '.txt', name: 'file' }

// Current directory and file (ES modules)
import { fileURLToPath } from 'url';
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
```

---

## Database Integration

### 7. MongoDB

Short explanation:

- What it is: NoSQL document database (JSON-like documents)
- When to use: Flexible schemas, rapid prototyping, nested data

```js
import { MongoClient } from 'mongodb';

// Connect to MongoDB
const client = new MongoClient('mongodb://localhost:27017');
await client.connect();

const db = client.db('myapp');
const users = db.collection('users');

// Insert
await users.insertOne({ name: 'Alice', age: 25 });
await users.insertMany([
  { name: 'Bob', age: 30 },
  { name: 'Charlie', age: 35 }
]);

// Find
const user = await users.findOne({ name: 'Alice' });
const allUsers = await users.find({ age: { $gte: 25 } }).toArray();

// Update
await users.updateOne(
  { name: 'Alice' },
  { $set: { age: 26 }, $inc: { loginCount: 1 } }
);

await users.updateMany(
  { age: { $lt: 30 } },
  { $set: { status: 'young' } }
);

// Delete
await users.deleteOne({ name: 'Alice' });
await users.deleteMany({ age: { $lt: 18 } });

// Aggregation pipeline
const stats = await users.aggregate([
  { $match: { status: 'active' } },
  { $group: { _id: '$country', count: { $sum: 1 }, avgAge: { $avg: '$age' } } },
  { $sort: { count: -1 } },
  { $limit: 10 }
]).toArray();

// Indexes (improve query performance)
await users.createIndex({ email: 1 }, { unique: true });
await users.createIndex({ name: 1, age: -1 });

// Bulk operations
const bulk = users.initializeUnorderedBulkOp();
bulk.insert({ name: 'David' });
bulk.find({ name: 'Eve' }).update({ $set: { updated: true } });
bulk.find({ name: 'Frank' }).remove();
await bulk.execute();

// Change streams (real-time updates)
const changeStream = users.watch();
changeStream.on('change', (change) => {
  console.log('Change:', change.operationType, change.documentKey);
});

// Cleanup
await client.close();
```

Common patterns:

- Connection pooling (reuse connections)
- Indexes on frequently queried fields
- Aggregation for complex queries
- Change streams for real-time apps

---

### 8. Redis

Short explanation:

- What it is: In-memory key-value store (cache, session storage, pub/sub)
- Why it matters: Extremely fast (microsecond latency), supports data structures
- When to use: Caching, rate limiting, sessions, queues, real-time messaging

```js
import { createClient } from 'redis';

// Connect to Redis
const redis = createClient({ url: 'redis://localhost:6379' });
await redis.connect();

// Basic key-value operations
await redis.set('user:123', JSON.stringify({ name: 'Alice' }));
const user = JSON.parse(await redis.get('user:123'));

// Expiration (TTL)
await redis.setEx('session:abc', 3600, sessionData);  // Expires in 1 hour
await redis.expire('key', 300);  // Set TTL on existing key

// Check if key exists
const exists = await redis.exists('user:123');  // 1 = exists, 0 = not found

// Delete
await redis.del('user:123');

// Atomic increment (thread-safe)
await redis.incr('page:views');
await redis.incrBy('counter', 5);

// Lists (queues, stacks)
await redis.lPush('jobs', JSON.stringify({ id: 1, task: 'process' }));  // Add to left
const job = await redis.rPop('jobs');  // Remove from right (FIFO queue)
const length = await redis.lLen('jobs');

// Sets (unique values)
await redis.sAdd('tags', 'javascript', 'nodejs', 'redis');
const tags = await redis.sMembers('tags');
const isMember = await redis.sIsMember('tags', 'javascript');

// Sorted sets (leaderboards, rankings)
await redis.zAdd('leaderboard', { score: 100, value: 'Alice' });
await redis.zAdd('leaderboard', { score: 200, value: 'Bob' });
const topPlayers = await redis.zRange('leaderboard', 0, 9, { REV: true });

// Hashes (objects)
await redis.hSet('user:123', 'name', 'Alice');
await redis.hSet('user:123', 'age', '25');
const name = await redis.hGet('user:123', 'name');
const userObj = await redis.hGetAll('user:123');

// Pub/Sub pattern (real-time messaging)
const subscriber = redis.duplicate();
await subscriber.connect();

await subscriber.subscribe('notifications', (message) => {
  console.log('Received:', message);
});

await redis.publish('notifications', JSON.stringify({ type: 'alert', text: 'New message' }));

// Pipelining (batch operations)
const pipeline = redis.multi();
pipeline.set('key1', 'value1');
pipeline.set('key2', 'value2');
pipeline.incr('counter');
await pipeline.exec();

// Cleanup
await redis.quit();
```

Common use cases:

- **Caching**: Store expensive query results
- **Session storage**: Fast user session lookup
- **Rate limiting**: Track API request counts
- **Job queues**: Background task processing
- **Pub/Sub**: Real-time notifications

---

### 9. PostgreSQL / MySQL

Short explanation:

- What it is: Relational SQL databases (structured data, ACID transactions)
- When to use: Complex relationships, data integrity, transactions

```js
// PostgreSQL example (using 'pg' library)
import pg from 'pg';
const { Pool } = pg;

const pool = new Pool({
  host: 'localhost',
  port: 5432,
  database: 'myapp',
  user: 'postgres',
  password: 'password'
});

// Query
const result = await pool.query('SELECT * FROM users WHERE age > $1', [25]);
console.log(result.rows);

// Insert
await pool.query(
  'INSERT INTO users (name, email, age) VALUES ($1, $2, $3)',
  ['Alice', 'alice@example.com', 25]
);

// Transaction
const client = await pool.connect();
try {
  await client.query('BEGIN');
  await client.query('UPDATE accounts SET balance = balance - $1 WHERE id = $2', [100, 1]);
  await client.query('UPDATE accounts SET balance = balance + $1 WHERE id = $2', [100, 2]);
  await client.query('COMMIT');
} catch (error) {
  await client.query('ROLLBACK');
  throw error;
} finally {
  client.release();
}

// Prepared statements (prevent SQL injection)
const getUser = await pool.query('SELECT * FROM users WHERE email = $1', ['alice@example.com']);

// Cleanup
await pool.end();
```

Best practices:

- Use connection pooling (don't create new connection per query)
- Parameterized queries ($1, $2) prevent SQL injection
- Use transactions for multi-step operations
- Index frequently queried columns

---

## Process & Environment

### 10. Environment Variables

Short explanation:

- What it is: Configuration values passed to the process
- Why it matters: Keep secrets out of code (API keys, passwords)

```js
// Access environment variables
const port = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL;
const nodeEnv = process.env.NODE_ENV;  // 'development', 'production'

// Using dotenv (load from .env file)
import 'dotenv/config';

// .env file:
// PORT=3000
// DATABASE_URL=mongodb://localhost:27017
// API_KEY=secret123

console.log(process.env.API_KEY);  // 'secret123'

// Environment-specific config
const config = {
  development: {
    db: 'mongodb://localhost:27017',
    logLevel: 'debug'
  },
  production: {
    db: process.env.DATABASE_URL,
    logLevel: 'error'
  }
}[process.env.NODE_ENV || 'development'];
```

**.env best practices:**

- Never commit `.env` to version control (add to `.gitignore`)
- Use `.env.example` with dummy values as template
- Different `.env` files per environment

---

### 11. Process Management

Short explanation:

- What it is: Control the Node.js process lifecycle
- When to use: Graceful shutdowns, error handling, process info

```js
// Process info
console.log(process.pid);           // Process ID
console.log(process.platform);      // 'win32', 'darwin', 'linux'
console.log(process.cwd());         // Current working directory
console.log(process.argv);          // Command-line arguments

// Exit codes
process.exit(0);   // Success
process.exit(1);   // Error

// Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, closing server...');
  await server.close();
  await db.close();
  process.exit(0);
});

process.on('SIGINT', async () => {
  console.log('SIGINT received (Ctrl+C)');
  process.exit(0);
});

// Unhandled errors (critical - prevent crashes)
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
  process.exit(1);
});

// Memory usage
console.log(process.memoryUsage());
// { rss: 4935680, heapTotal: 1826816, heapUsed: 650472, external: 0 }

// CPU usage
console.log(process.cpuUsage());
```

---

### 12. Event Loop & Timers

Short explanation:

- What it is: Node.js event loop executes async operations
- Mental model: Single-threaded, non-blocking I/O via event loop
- Why it matters: Understanding prevents blocking and ensures responsiveness

```js
// setTimeout (runs after minimum delay)
setTimeout(() => {
  console.log('Runs after 1 second');
}, 1000);

// setInterval (runs repeatedly)
const intervalId = setInterval(() => {
  console.log('Runs every 2 seconds');
}, 2000);

// Clear interval
clearInterval(intervalId);

// setImmediate (runs after I/O events)
setImmediate(() => {
  console.log('Runs after I/O phase');
});

// process.nextTick (runs before event loop continues)
process.nextTick(() => {
  console.log('Runs before next event loop iteration');
});

// Order of execution
console.log('1: Synchronous');

setTimeout(() => console.log('2: setTimeout'), 0);

setImmediate(() => console.log('3: setImmediate'));

process.nextTick(() => console.log('4: nextTick'));

console.log('5: Synchronous');

// Output order:
// 1: Synchronous
// 5: Synchronous
// 4: nextTick
// 2: setTimeout
// 3: setImmediate
```

**Event loop phases:**

1. Timers (setTimeout, setInterval)
2. Pending callbacks (I/O callbacks)
3. Idle, prepare
4. Poll (retrieve new I/O events)
5. Check (setImmediate)
6. Close callbacks

---

## Performance & Optimization

### 13. Worker Threads

Short explanation:

- What it is: Run CPU-intensive tasks in separate threads
- Why it matters: Prevent blocking the main event loop
- When to use: Image processing, cryptography, data parsing, machine learning

```js
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';

if (isMainThread) {
  // Main thread - create worker
  const worker = new Worker('./worker.js', {
    workerData: { task: 'heavy-computation', input: [1, 2, 3] }
  });

  worker.on('message', (result) => {
    console.log('Result from worker:', result);
  });

  worker.on('error', (error) => {
    console.error('Worker error:', error);
  });

  worker.on('exit', (code) => {
    console.log(`Worker exited with code ${code}`);
  });

} else {
  // Worker thread - do heavy computation
  const { task, input } = workerData;

  // CPU-intensive work here
  const result = input.reduce((sum, n) => sum + n, 0);

  parentPort.postMessage(result);
}
```

**Worker pool pattern:**

```js
import { Worker } from 'worker_threads';

class WorkerPool {
  constructor(workerScript, poolSize = 4) {
    this.workers = [];
    this.queue = [];

    for (let i = 0; i < poolSize; i++) {
      const worker = new Worker(workerScript);
      this.workers.push({ worker, busy: false });
    }
  }

  async run(data) {
    return new Promise((resolve, reject) => {
      const availableWorker = this.workers.find(w => !w.busy);

      if (availableWorker) {
        this.execute(availableWorker, data, resolve, reject);
      } else {
        this.queue.push({ data, resolve, reject });
      }
    });
  }

  execute(workerObj, data, resolve, reject) {
    workerObj.busy = true;

    workerObj.worker.once('message', (result) => {
      workerObj.busy = false;
      resolve(result);

      // Process next queued task
      if (this.queue.length > 0) {
        const next = this.queue.shift();
        this.execute(workerObj, next.data, next.resolve, next.reject);
      }
    });

    workerObj.worker.once('error', reject);
    workerObj.worker.postMessage(data);
  }
}

// Usage
const pool = new WorkerPool('./cpu-task.js', 4);
const result = await pool.run({ numbers: [1, 2, 3, 4, 5] });
```

---

### 14. Clustering

Short explanation:

- What it is: Run multiple Node.js processes to utilize all CPU cores
- When to use: Production servers (scale horizontally)

```js
import cluster from 'cluster';
import { cpus } from 'os';
import { createServer } from 'http';

const numCPUs = cpus().length;

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork();  // Restart worker
  });

} else {
  // Workers share the same TCP connection
  const server = createServer((req, res) => {
    res.writeHead(200);
    res.end(`Handled by worker ${process.pid}`);
  });

  server.listen(3000);
  console.log(`Worker ${process.pid} started`);
}
```

---

## Security & Best Practices

### 15. Security

Common security practices:

```js
// 1. Validate input
import validator from 'validator';

const email = req.body.email;
if (!validator.isEmail(email)) {
  throw new Error('Invalid email');
}

// 2. Sanitize HTML (prevent XSS)
import DOMPurify from 'isomorphic-dompurify';

const clean = DOMPurify.sanitize(userInput);

// 3. Use helmet for HTTP headers
import helmet from 'helmet';
app.use(helmet());

// 4. Rate limiting
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100  // Limit each IP to 100 requests per window
});

app.use('/api/', limiter);

// 5. Parameterized queries (prevent SQL injection)
await pool.query('SELECT * FROM users WHERE id = $1', [userId]);

// 6. Hash passwords (never store plaintext)
import bcrypt from 'bcrypt';

const hashedPassword = await bcrypt.hash(password, 10);
const match = await bcrypt.compare(password, hashedPassword);

// 7. Use environment variables for secrets
const apiKey = process.env.API_KEY;  // Never hardcode

// 8. CORS configuration
import cors from 'cors';

app.use(cors({
  origin: 'https://yourdomain.com',
  credentials: true
}));
```

---

## Debugging & Monitoring

### 16. Debugging

```js
// Built-in debugger
debugger;  // Pause execution (run with --inspect flag)

// Run with inspector
// node --inspect app.js
// Then open chrome://inspect in Chrome

// Console methods
console.log('Info');
console.error('Error');
console.warn('Warning');
console.table([{ name: 'Alice', age: 25 }]);
console.time('operation');
// ... code ...
console.timeEnd('operation');  // Logs elapsed time

// Stack traces
console.trace('How did we get here?');

// Performance measurement
import { performance } from 'perf_hooks';

const start = performance.now();
// ... operation ...
const end = performance.now();
console.log(`Operation took ${end - start}ms`);
```

**Logging best practices:**

```js
// Use a logging library (Winston, Pino)
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: {
    target: 'pino-pretty',
    options: { colorize: true }
  }
});

logger.info('Server started');
logger.error({ err: error }, 'Error occurred');
logger.debug({ data }, 'Debug info');
```

---

## Quick Reference

**Key Node.js concepts:**

1. **Event Loop**: Single-threaded, non-blocking I/O
2. **Streams**: Process large data in chunks
3. **Buffers**: Binary data (use streams when possible)
4. **Event Emitters**: Pub/sub pattern for events
5. **Worker Threads**: CPU-intensive tasks off main thread
6. **Clustering**: Multiple processes for multi-core systems

**Module systems:**

- ESM (modern): `import`/`export` (set `"type": "module"` in package.json)
- CommonJS (legacy): `require()`/`module.exports`

**Common patterns:**

- Connection pooling (databases)
- Graceful shutdown (cleanup on exit)
- Error handling (catch all errors, log, monitor)
- Environment-based config (dev, staging, prod)

---

## Resources

- Official docs: [Node.js Documentation](https://nodejs.org/docs)
- Package registry: [npm](https://www.npmjs.com)
- Best practices: [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- Security: [OWASP Node.js Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html)
- Testing: [Jest](https://jestjs.io/), [Mocha](https://mochajs.org/)
- Frameworks: [Express](https://expressjs.com/), [Fastify](https://www.fastify.io/), [NestJS](https://nestjs.com/)
