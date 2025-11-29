# Node.js Backend Libraries Cheatsheet

> Essential Node.js libraries for building servers, APIs, and backend applications. Covers Express, Fastify, WebSockets, databases, and authentication.

## Table of Contents

- [Overview](#overview)
- [Web Frameworks](#web-frameworks)
  - [Express.js](#expressjs)
  - [Fastify](#fastify)
- [WebSockets & Real-Time](#websockets--real-time)
  - [ws](#ws)
  - [Socket.io](#socketio)
- [Authentication & Security](#authentication--security)
  - [jsonwebtoken](#jsonwebtoken)
  - [bcrypt](#bcrypt)
  - [helmet](#helmet)
- [Logging](#logging)
  - [Winston](#winston)
  - [Pino](#pino)
- [Task Scheduling](#task-scheduling)
  - [node-cron](#node-cron)
- [Quick Reference](#quick-reference)
- [Resources](#resources)

## Overview

- Purpose: Quick reference for essential Node.js backend libraries and frameworks
- Audience: Intermediate to Advanced
- Scope: Express, Fastify, ws, Socket.io, JWT, bcrypt, helmet, Winston, Pino, node-cron. Excludes ORMs (covered separately), frontend libraries, and framework-specific tools.
- Updated: 2025-11-12

---

## Web Frameworks

### Express.js

#### What is Express?

- What it is: Minimal, unopinionated web framework for Node.js
- Why it matters: Industry standard, massive ecosystem, simple API
- Mental model: Like a restaurant manager - handles requests, routes them to the right handler, sends responses

#### Installation

```shell
npm install express
```

#### Basic Server

```js
import express from 'express';

const app = express();

// Middleware to parse JSON bodies
app.use(express.json());

// Basic route
app.get('/', (req, res) => {
  res.send('Hello World');
});

// JSON response
app.get('/api/users', (req, res) => {
  res.json({ users: ['Alice', 'Bob'] });
});

// Start server
app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

#### Routing

```js
// Route parameters
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.json({ id: userId, name: 'Alice' });
});

// Query parameters
app.get('/search', (req, res) => {
  const { q, limit = 10 } = req.query;
  // URL: /search?q=javascript&limit=5
  res.json({ query: q, limit });
});

// POST with body
app.post('/users', (req, res) => {
  const { name, email } = req.body;
  res.status(201).json({ id: 123, name, email });
});

// Multiple HTTP methods
app.route('/article/:id')
  .get((req, res) => { /* get article */ })
  .put((req, res) => { /* update article */ })
  .delete((req, res) => { /* delete article */ });
```

#### Middleware

```js
// Application-level middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();  // Pass control to next middleware
});

// Router-level middleware
const router = express.Router();
router.use((req, res, next) => {
  req.startTime = Date.now();
  next();
});

// Error-handling middleware (4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal server error' });
});

// Built-in middleware
app.use(express.json());                    // Parse JSON bodies
app.use(express.urlencoded({ extended: true }));  // Parse URL-encoded bodies
app.use(express.static('public'));          // Serve static files
```

#### Practical Examples

```js
// Authentication middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  try {
    req.user = verifyToken(token);
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};

// Protected route
app.get('/api/profile', authenticate, (req, res) => {
  res.json({ user: req.user });
});

// Request validation middleware
const validateBody = (schema) => (req, res, next) => {
  const { error } = schema.validate(req.body);
  if (error) {
    return res.status(400).json({ error: error.message });
  }
  next();
};

// CORS middleware
import cors from 'cors';
app.use(cors({
  origin: 'https://example.com',
  credentials: true
}));

// Rate limiting
import rateLimit from 'express-rate-limit';
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100  // Limit each IP to 100 requests per windowMs
});
app.use('/api/', limiter);
```

Common pitfalls:

- Forgetting to call `next()` in middleware (request hangs)
- Not handling async errors (use `express-async-errors` or try-catch)
- Placing error-handling middleware before routes (it won't catch errors)

---

### Fastify

#### What is Fastify?

- What it is: Fast, low-overhead web framework focused on performance
- Why it matters: 2-3x faster than Express, built-in schema validation, TypeScript-friendly
- Mental model: Like Express but optimized for speed - every millisecond counts

#### Installation

```shell
npm install fastify
```

#### Basic Server

```js
import Fastify from 'fastify';

const fastify = Fastify({ logger: true });

// Route with schema validation
fastify.get('/users/:id', {
  schema: {
    params: {
      type: 'object',
      properties: {
        id: { type: 'string' }
      }
    },
    response: {
      200: {
        type: 'object',
        properties: {
          id: { type: 'string' },
          name: { type: 'string' }
        }
      }
    }
  }
}, async (request, reply) => {
  return { id: request.params.id, name: 'Alice' };
});

// Start server
await fastify.listen({ port: 3000 });
```

#### Schema Validation

```js
// POST with body validation
fastify.post('/users', {
  schema: {
    body: {
      type: 'object',
      required: ['name', 'email'],
      properties: {
        name: { type: 'string', minLength: 1 },
        email: { type: 'string', format: 'email' }
      }
    },
    response: {
      201: {
        type: 'object',
        properties: {
          id: { type: 'number' },
          name: { type: 'string' },
          email: { type: 'string' }
        }
      }
    }
  }
}, async (request, reply) => {
  const { name, email } = request.body;
  reply.code(201);
  return { id: 123, name, email };
});
```

#### Hooks & Plugins

```js
// Hooks (like middleware)
fastify.addHook('onRequest', async (request, reply) => {
  request.startTime = Date.now();
});

fastify.addHook('onResponse', async (request, reply) => {
  const duration = Date.now() - request.startTime;
  console.log(`${request.method} ${request.url} - ${duration}ms`);
});

// Plugin
async function authPlugin(fastify, options) {
  fastify.decorate('authenticate', async (request, reply) => {
    const token = request.headers.authorization?.split(' ')[1];
    if (!token) {
      throw new Error('No token provided');
    }
    request.user = verifyToken(token);
  });
}

fastify.register(authPlugin);

// Protected route
fastify.get('/profile', {
  onRequest: [fastify.authenticate]
}, async (request, reply) => {
  return { user: request.user };
});
```

Common pitfalls:

- Not returning values from route handlers (Fastify requires explicit return or `reply.send()`)
- Using Express middleware directly (use `@fastify/express` plugin)
- Forgetting to `await fastify.ready()` before making requests in tests

---

## WebSockets & Real-Time

### ws

#### What is ws?

- What it is: Lightweight, fast WebSocket client/server library for Node.js
- Why it matters: Low-level control, minimal overhead, works in browser and Node.js
- Mental model: Like a telephone line - persistent two-way communication channel

#### Installation

```shell
npm install ws
```

#### Server

```js
import { WebSocketServer } from 'ws';

const wss = new WebSocketServer({ port: 8080 });

wss.on('connection', (ws) => {
  console.log('Client connected');

  // Receive message
  ws.on('message', (data) => {
    console.log('Received:', data.toString());

    // Echo back
    ws.send(`Echo: ${data}`);
  });

  // Handle errors
  ws.on('error', (error) => {
    console.error('WebSocket error:', error);
  });

  // Handle close
  ws.on('close', () => {
    console.log('Client disconnected');
  });

  // Send welcome message
  ws.send('Welcome to the server!');
});
```

#### Client

```js
import WebSocket from 'ws';

const ws = new WebSocket('ws://localhost:8080');

ws.on('open', () => {
  console.log('Connected to server');
  ws.send('Hello server!');
});

ws.on('message', (data) => {
  console.log('Received:', data.toString());
});

ws.on('close', () => {
  console.log('Disconnected from server');
});

ws.on('error', (error) => {
  console.error('Error:', error);
});
```

#### Broadcast to All Clients

```js
wss.on('connection', (ws) => {
  ws.on('message', (data) => {
    // Broadcast to all connected clients
    wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(data);
      }
    });
  });
});
```

#### Ping/Pong (Keep-Alive)

```js
const wss = new WebSocketServer({ port: 8080 });

wss.on('connection', (ws) => {
  ws.isAlive = true;

  ws.on('pong', () => {
    ws.isAlive = true;
  });
});

// Ping all clients every 30 seconds
const interval = setInterval(() => {
  wss.clients.forEach((ws) => {
    if (!ws.isAlive) {
      return ws.terminate();
    }
    ws.isAlive = false;
    ws.ping();
  });
}, 30000);

wss.on('close', () => {
  clearInterval(interval);
});
```

#### Practical Example (Real-Time Price Updates)

```js
import { WebSocketServer } from 'ws';

const wss = new WebSocketServer({ port: 8080 });
const subscriptions = new Map();  // clientId -> Set<symbols>

wss.on('connection', (ws) => {
  const clientId = generateId();
  subscriptions.set(clientId, new Set());

  ws.on('message', (data) => {
    const message = JSON.parse(data.toString());

    if (message.type === 'subscribe') {
      subscriptions.get(clientId).add(message.symbol);
      ws.send(JSON.stringify({ type: 'subscribed', symbol: message.symbol }));
    } else if (message.type === 'unsubscribe') {
      subscriptions.get(clientId).delete(message.symbol);
    }
  });

  ws.on('close', () => {
    subscriptions.delete(clientId);
  });
});

// Simulate price updates
setInterval(() => {
  const priceUpdate = {
    type: 'price',
    symbol: 'BTC',
    price: Math.random() * 50000 + 20000
  };

  wss.clients.forEach((ws) => {
    if (ws.readyState === WebSocket.OPEN) {
      ws.send(JSON.stringify(priceUpdate));
    }
  });
}, 1000);
```

Common pitfalls:

- Not checking `readyState` before sending (causes errors if client disconnected)
- Forgetting to implement ping/pong for connection health checks
- Not handling `Buffer` vs string messages (use `.toString()`)

---

### Socket.io

#### What is Socket.io?

- What it is: Real-time bidirectional event-based communication library
- Why it matters: Auto-reconnection, rooms/namespaces, fallback to long-polling, easier API than raw WebSockets
- Mental model: Like ws but with superpowers - handles all the hard parts for you

#### Installation

```shell
npm install socket.io socket.io-client
```

#### Server

```js
import { Server } from 'socket.io';
import { createServer } from 'http';

const httpServer = createServer();
const io = new Server(httpServer, {
  cors: {
    origin: 'http://localhost:3000',
    credentials: true
  }
});

io.on('connection', (socket) => {
  console.log('Client connected:', socket.id);

  // Listen for events
  socket.on('message', (data) => {
    console.log('Received:', data);

    // Emit back to sender
    socket.emit('message', `Echo: ${data}`);
  });

  // Broadcast to all clients
  socket.on('broadcast', (data) => {
    io.emit('message', data);  // To all clients
    // socket.broadcast.emit('message', data);  // To all except sender
  });

  socket.on('disconnect', () => {
    console.log('Client disconnected:', socket.id);
  });
});

httpServer.listen(3000);
```

#### Client

```js
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000');

socket.on('connect', () => {
  console.log('Connected:', socket.id);
  socket.emit('message', 'Hello server!');
});

socket.on('message', (data) => {
  console.log('Received:', data);
});

socket.on('disconnect', () => {
  console.log('Disconnected');
});
```

#### Rooms & Namespaces

```js
// Join/leave rooms
io.on('connection', (socket) => {
  socket.on('join-room', (room) => {
    socket.join(room);
    socket.emit('joined', room);
    socket.to(room).emit('user-joined', socket.id);
  });

  socket.on('leave-room', (room) => {
    socket.leave(room);
    socket.to(room).emit('user-left', socket.id);
  });

  // Send to specific room
  socket.on('message-to-room', (room, message) => {
    io.to(room).emit('message', message);
  });
});

// Namespaces (separate channels)
const adminNamespace = io.of('/admin');
adminNamespace.on('connection', (socket) => {
  console.log('Admin connected');
});
```

#### Acknowledgments (Request-Response)

```js
// Server
socket.on('create-order', (data, callback) => {
  try {
    const order = createOrder(data);
    callback({ success: true, order });
  } catch (error) {
    callback({ success: false, error: error.message });
  }
});

// Client
socket.emit('create-order', { symbol: 'BTC', amount: 1 }, (response) => {
  if (response.success) {
    console.log('Order created:', response.order);
  } else {
    console.error('Error:', response.error);
  }
});
```

Common pitfalls:

- Not handling reconnections properly (events may be lost during disconnect)
- Confusing `socket.emit()` (to sender) vs `io.emit()` (to all) vs `socket.broadcast.emit()` (to all except sender)
- Not cleaning up room subscriptions on disconnect

---

## Authentication & Security

### jsonwebtoken

#### What is JWT?

- What it is: Library to create and verify JSON Web Tokens for authentication
- Why it matters: Stateless authentication, no server-side sessions needed
- Mental model: Like a passport stamp - proves identity without checking database

#### Installation

```shell
npm install jsonwebtoken
```

#### Basic Usage

```js
import jwt from 'jsonwebtoken';

const SECRET = process.env.JWT_SECRET || 'your-secret-key';

// Create token
const token = jwt.sign(
  { userId: 123, email: 'alice@example.com' },
  SECRET,
  { expiresIn: '1h' }
);

// Verify token
try {
  const decoded = jwt.verify(token, SECRET);
  console.log(decoded);  // { userId: 123, email: 'alice@example.com', iat: ..., exp: ... }
} catch (error) {
  console.error('Invalid token:', error.message);
}
```

#### Practical Example (Express Middleware)

```js
// Generate token on login
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await authenticateUser(email, password);

  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  const token = jwt.sign(
    { userId: user.id, email: user.email, role: user.role },
    SECRET,
    { expiresIn: '7d' }
  );

  res.json({ token, user: { id: user.id, email: user.email } });
});

// Verify token middleware
const authenticate = (req, res, next) => {
  const authHeader = req.headers.authorization;
  const token = authHeader?.split(' ')[1];  // "Bearer <token>"

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const decoded = jwt.verify(token, SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
};

// Protected route
app.get('/profile', authenticate, (req, res) => {
  res.json({ user: req.user });
});
```

#### Refresh Tokens

```js
const ACCESS_TOKEN_SECRET = process.env.ACCESS_TOKEN_SECRET;
const REFRESH_TOKEN_SECRET = process.env.REFRESH_TOKEN_SECRET;

// Generate both tokens
app.post('/login', async (req, res) => {
  const user = await authenticateUser(req.body.email, req.body.password);

  const accessToken = jwt.sign(
    { userId: user.id },
    ACCESS_TOKEN_SECRET,
    { expiresIn: '15m' }
  );

  const refreshToken = jwt.sign(
    { userId: user.id },
    REFRESH_TOKEN_SECRET,
    { expiresIn: '7d' }
  );

  // Store refresh token in database
  await saveRefreshToken(user.id, refreshToken);

  res.json({ accessToken, refreshToken });
});

// Refresh access token
app.post('/refresh', async (req, res) => {
  const { refreshToken } = req.body;

  try {
    const decoded = jwt.verify(refreshToken, REFRESH_TOKEN_SECRET);
    const isValid = await checkRefreshToken(decoded.userId, refreshToken);

    if (!isValid) {
      return res.status(401).json({ error: 'Invalid refresh token' });
    }

    const newAccessToken = jwt.sign(
      { userId: decoded.userId },
      ACCESS_TOKEN_SECRET,
      { expiresIn: '15m' }
    );

    res.json({ accessToken: newAccessToken });
  } catch (error) {
    res.status(401).json({ error: 'Invalid refresh token' });
  }
});
```

Common pitfalls:

- Storing secret key in code (use environment variables)
- Not setting expiration (`expiresIn` option)
- Not validating token before trusting payload
- Using weak secrets (use long, random strings)

---

### bcrypt

#### What is bcrypt?

- What it is: Library to hash passwords securely with salt
- Why it matters: Never store plain-text passwords; bcrypt is slow by design (prevents brute-force)
- Mental model: Like a one-way shredder - you can't un-shred the paper

#### Installation

```shell
npm install bcrypt
```

#### Basic Usage

```js
import bcrypt from 'bcrypt';

const SALT_ROUNDS = 10;  // Higher = more secure but slower

// Hash password
const password = 'userPassword123';
const hashedPassword = await bcrypt.hash(password, SALT_ROUNDS);
console.log(hashedPassword);  // $2b$10$...

// Verify password
const isValid = await bcrypt.compare(password, hashedPassword);
console.log(isValid);  // true
```

#### Practical Example (User Registration)

```js
// Register user
app.post('/register', async (req, res) => {
  const { email, password } = req.body;

  // Validate password strength
  if (password.length < 8) {
    return res.status(400).json({ error: 'Password must be at least 8 characters' });
  }

  // Hash password
  const hashedPassword = await bcrypt.hash(password, 10);

  // Save user
  const user = await createUser({ email, password: hashedPassword });

  res.status(201).json({ id: user.id, email: user.email });
});

// Login user
app.post('/login', async (req, res) => {
  const { email, password } = req.body;

  // Find user
  const user = await findUserByEmail(email);
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Compare password
  const isValid = await bcrypt.compare(password, user.password);
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Generate JWT token
  const token = jwt.sign({ userId: user.id }, SECRET, { expiresIn: '7d' });
  res.json({ token });
});
```

Common pitfalls:

- Using too few salt rounds (< 10 is not secure enough)
- Using synchronous methods in production (`hashSync`, `compareSync` block event loop)
- Not handling bcrypt errors (e.g., invalid hash format)

---

### helmet

#### What is helmet?

- What it is: Express middleware to set secure HTTP headers
- Why it matters: Protects against common web vulnerabilities (XSS, clickjacking, etc.)
- Mental model: Like a security guard - adds protective layers to your responses

#### Installation

```shell
npm install helmet
```

#### Basic Usage

```js
import express from 'express';
import helmet from 'helmet';

const app = express();

// Use helmet with default settings
app.use(helmet());

// Or customize
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'", "https://cdn.example.com"]
    }
  },
  hsts: {
    maxAge: 31536000,  // 1 year
    includeSubDomains: true,
    preload: true
  }
}));
```

#### Headers Set by Helmet

```js
// Content Security Policy (prevents XSS)
helmet.contentSecurityPolicy()

// X-DNS-Prefetch-Control (controls DNS prefetching)
helmet.dnsPrefetchControl()

// X-Frame-Options (prevents clickjacking)
helmet.frameguard()

// X-Powered-By removal (hides Express)
helmet.hidePoweredBy()

// Strict-Transport-Security (forces HTTPS)
helmet.hsts()

// X-Download-Options (prevents IE from executing downloads)
helmet.ieNoOpen()

// X-Content-Type-Options (prevents MIME sniffing)
helmet.noSniff()

// X-Permitted-Cross-Domain-Policies (controls Adobe Flash/PDF)
helmet.permittedCrossDomainPolicies()

// Referrer-Policy
helmet.referrerPolicy()

// X-XSS-Protection (XSS filter)
helmet.xssFilter()
```

Common pitfalls:

- Breaking functionality with overly strict CSP (test thoroughly)
- Not configuring CORS separately (helmet doesn't handle CORS)
- Using helmet after routes (it won't apply to those routes)

---

## Logging

### Winston

#### What is Winston?

- What it is: Versatile logging library with multiple transports (console, file, HTTP, etc.)
- Why it matters: Production-grade logging with log levels, formats, and outputs
- Mental model: Like a recording studio - capture logs in different formats and send to different destinations

#### Installation

```shell
npm install winston
```

#### Basic Usage

```js
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Add console transport for development
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// Log messages
logger.error('Error message');
logger.warn('Warning message');
logger.info('Info message');
logger.debug('Debug message');
```

#### Custom Format

```js
const customFormat = winston.format.printf(({ level, message, timestamp, ...meta }) => {
  return `${timestamp} [${level}]: ${message} ${Object.keys(meta).length ? JSON.stringify(meta) : ''}`;
});

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.errors({ stack: true }),
    customFormat
  ),
  transports: [new winston.transports.Console()]
});
```

#### Practical Example (Express Integration)

```js
import winston from 'winston';
import expressWinston from 'express-winston';

// Request logging
app.use(expressWinston.logger({
  transports: [
    new winston.transports.File({ filename: 'requests.log' })
  ],
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  meta: true,
  msg: "HTTP {{req.method}} {{req.url}}",
  expressFormat: true
}));

// Error logging
app.use(expressWinston.errorLogger({
  transports: [
    new winston.transports.File({ filename: 'errors.log' })
  ],
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  )
}));
```

Common pitfalls:

- Not rotating log files (use `winston-daily-rotate-file`)
- Logging sensitive data (passwords, tokens, API keys)
- Using synchronous transports in production (blocks event loop)

---

### Pino

#### What is Pino?

- What it is: Extremely fast JSON logger for Node.js
- Why it matters: 5-10x faster than Winston, minimal overhead, structured logging
- Mental model: Like Winston but optimized for speed - logs are fire-and-forget

#### Installation

```shell
npm install pino pino-pretty
```

#### Basic Usage

```js
import pino from 'pino';

// Production logger (JSON output)
const logger = pino();

// Development logger (pretty output)
const logger = pino({
  transport: {
    target: 'pino-pretty',
    options: {
      colorize: true,
      translateTime: 'SYS:standard',
      ignore: 'pid,hostname'
    }
  }
});

// Log messages
logger.info('Server started');
logger.error({ err: new Error('Failed') }, 'Error occurred');
logger.debug({ userId: 123 }, 'User logged in');
```

#### Child Loggers

```js
const logger = pino();

// Create child logger with context
const requestLogger = logger.child({ requestId: 'abc-123' });
requestLogger.info('Processing request');
// Output: {"level":30,"time":...,"requestId":"abc-123","msg":"Processing request"}
```

#### Express Integration

```js
import express from 'express';
import pino from 'pino';
import pinoHttp from 'pino-http';

const app = express();
const logger = pino();

// HTTP request logging
app.use(pinoHttp({ logger }));

app.get('/', (req, res) => {
  req.log.info('Handling request');
  res.send('Hello');
});
```

Common pitfalls:

- Logging large objects (increases JSON serialization time)
- Not using child loggers for request-specific context
- Expecting pretty logs in production (use JSON + log aggregation)

---

## Task Scheduling

### node-cron

#### What is node-cron?

- What it is: Lightweight task scheduler using cron syntax
- Why it matters: Run tasks at specific intervals (backups, cleanup, data sync)
- Mental model: Like a digital alarm clock - triggers code at scheduled times

#### Installation

```shell
npm install node-cron
```

#### Basic Usage

```js
import cron from 'node-cron';

// Run every minute
cron.schedule('* * * * *', () => {
  console.log('Running every minute');
});

// Run every hour at minute 30
cron.schedule('30 * * * *', () => {
  console.log('Running at :30 of every hour');
});

// Run at 2:30 AM every day
cron.schedule('30 2 * * *', () => {
  console.log('Running daily at 2:30 AM');
});

// Run every Monday at 9 AM
cron.schedule('0 9 * * 1', () => {
  console.log('Running every Monday at 9 AM');
});
```

#### Cron Syntax

```txt
 ┌────────────── second (optional, 0-59)
 │ ┌──────────── minute (0-59)
 │ │ ┌────────── hour (0-23)
 │ │ │ ┌──────── day of month (1-31)
 │ │ │ │ ┌────── month (1-12)
 │ │ │ │ │ ┌──── day of week (0-7, 0 and 7 are Sunday)
 │ │ │ │ │ │
 * * * * * *
```

#### Practical Examples

```js
// Cleanup old logs every day at midnight
cron.schedule('0 0 * * *', async () => {
  console.log('Cleaning up old logs...');
  await cleanupOldLogs();
});

// Sync data every 5 minutes
cron.schedule('*/5 * * * *', async () => {
  console.log('Syncing data...');
  await syncDataFromAPI();
});

// Send weekly report every Sunday at 8 PM
cron.schedule('0 20 * * 0', async () => {
  console.log('Generating weekly report...');
  await sendWeeklyReport();
});

// Controllable task
const task = cron.schedule('* * * * *', () => {
  console.log('Running task');
}, {
  scheduled: false  // Don't start immediately
});

// Start task
task.start();

// Stop task
task.stop();

// Destroy task
task.destroy();
```

Common pitfalls:

- Not handling async errors in scheduled tasks (use try-catch)
- Running CPU-intensive tasks in the same process (use worker threads or separate processes)
- Forgetting timezone differences (use `timezone` option)

---

## Quick Reference

### When to Use What

- **Express**: General-purpose web apps, REST APIs, large ecosystem
- **Fastify**: Performance-critical APIs, microservices, schema validation
- **ws**: Low-level WebSocket control, minimal overhead
- **Socket.io**: Real-time apps with rooms, auto-reconnection, fallbacks
- **JWT**: Stateless authentication, API tokens
- **bcrypt**: Password hashing (never store plain passwords)
- **helmet**: Security headers for all Express apps
- **Winston**: Feature-rich logging, multiple outputs
- **Pino**: High-performance logging, structured logs
- **node-cron**: Simple scheduled tasks

### Performance Comparison

- **Fastify** vs **Express**: 2-3x faster for JSON APIs
- **Pino** vs **Winston**: 5-10x faster logging
- **ws** vs **Socket.io**: Lower overhead but less features

### Security Checklist

- [ ] Use helmet middleware
- [ ] Hash passwords with bcrypt
- [ ] Use JWT for authentication
- [ ] Validate all inputs (Joi, Zod)
- [ ] Set rate limits
- [ ] Use HTTPS in production
- [ ] Keep dependencies updated
- [ ] Never log sensitive data

---

## Resources

### Official Documentation

- Express: <https://expressjs.com/>
- Fastify: <https://www.fastify.io/>
- ws: <https://github.com/websockets/ws>
- Socket.io: <https://socket.io/docs/>
- JWT: <https://github.com/auth0/node-jsonwebtoken>
- bcrypt: <https://github.com/kelektiv/node.bcrypt.js>
- helmet: <https://helmetjs.github.io/>
- Winston: <https://github.com/winstonjs/winston>
- Pino: <https://getpino.io/>
- node-cron: <https://github.com/node-cron/node-cron>

### Best Practices

- Use environment variables for secrets (dotenv)
- Implement proper error handling middleware
- Log all errors but not sensitive data
- Use TypeScript for type safety
- Write tests for critical paths
- Monitor performance and errors in production
