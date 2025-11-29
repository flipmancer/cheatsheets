# Redis Cheatsheet

> A practical reference for using Redis with Node.js/TypeScript

## Table of Contents

- [Redis Cheatsheet](#redis-cheatsheet)
	- [Table of Contents](#table-of-contents)
	- [Overview](#overview)
	- [Redis Setup](#redis-setup)
		- [Installation](#installation)
		- [Starting Redis Server](#starting-redis-server)
		- [Connecting from Node.js/TypeScript](#connecting-from-nodejstypescript)
	- [Core Redis Concepts](#core-redis-concepts)
		- [1. Keys and Data Types](#1-keys-and-data-types)
		- [2. Persistence](#2-persistence)
		- [3. TTL (Time To Live)](#3-ttl-time-to-live)
		- [4. Pub/Sub](#4-pubsub)
		- [5. Streams](#5-streams)
		- [6. Transactions](#6-transactions)
	- [Data Types and Operations](#data-types-and-operations)
		- [Strings](#strings)
		- [Hashes](#hashes)
		- [Lists](#lists)
		- [Sets](#sets)
		- [Sorted Sets](#sorted-sets)
		- [Streams](#streams)
	- [Common Patterns](#common-patterns)
		- [Caching Pattern](#caching-pattern)
		- [Rate Limiting](#rate-limiting)
		- [Distributed Lock](#distributed-lock)
		- [Session Storage](#session-storage)
		- [Message Queue with Streams](#message-queue-with-streams)
		- [Pub/Sub Event Broadcasting](#pubsub-event-broadcasting)
	- [Quick Reference](#quick-reference)
	- [Performance Tips](#performance-tips)
	- [Security \& Safety](#security--safety)
	- [Complex Topic: Redis Streams vs Pub/Sub](#complex-topic-redis-streams-vs-pubsub)
		- [Redis Streams](#redis-streams)
		- [Redis Pub/Sub](#redis-pubsub)
	- [CLI Commands Reference](#cli-commands-reference)
		- [Server Management](#server-management)
		- [Key Operations](#key-operations)
		- [Monitoring \& Debugging](#monitoring--debugging)
		- [Stream Operations](#stream-operations)
	- [Troubleshooting](#troubleshooting)
		- [Connection Refused](#connection-refused)
		- [Authentication Failed](#authentication-failed)
		- [Memory Issues](#memory-issues)
		- [Performance Degradation](#performance-degradation)
	- [Next Steps](#next-steps)
	- [Resources](#resources)

## Overview

- **Purpose**: In-memory data store for caching, queuing, pub/sub, and real-time applications
- **Audience**: Intermediate (assumes basic JavaScript/TypeScript knowledge)
- **Scope**: Focus on Node.js/TypeScript usage with `redis` (node-redis) library
- **Updated**: November 15, 2025

---

## Redis Setup

### Installation

**Server:**

```shell
# macOS
brew install redis

# Ubuntu/Debian
sudo apt-get update
sudo apt-get install redis-server

# Windows
# Download from https://github.com/microsoftarchive/redis/releases
# Or use WSL
```

**Node.js Client:**

```shell
npm install redis
# Or
pnpm add redis
```

### Starting Redis Server

```shell
# Start Redis server (default port 6379)
redis-server

# Start with custom config
redis-server /path/to/redis.conf

# Start as background service (Linux/macOS)
sudo systemctl start redis
# Or
brew services start redis
```

**Verify it's running:**

```shell
redis-cli ping
# Should return: PONG
```

### Connecting from Node.js/TypeScript

```typescript
import { createClient } from 'redis';

// Basic connection
const client = createClient({
  url: 'redis://localhost:6379'
});

// With password
const authClient = createClient({
  url: 'redis://:yourpassword@localhost:6379'
});

// With options
const configuredClient = createClient({
  socket: {
    host: 'localhost',
    port: 6379,
    reconnectStrategy: (retries) => Math.min(retries * 50, 500)
  },
  password: 'yourpassword',
  database: 0 // 0-15
});

// Connect
await client.connect();

// Handle errors
client.on('error', (err) => console.error('Redis Client Error', err));
client.on('connect', () => console.log('Redis Client Connected'));
client.on('ready', () => console.log('Redis Client Ready'));

// Graceful shutdown
process.on('SIGINT', async () => {
  await client.quit();
  process.exit(0);
});
```

---

## Core Redis Concepts

### 1. Keys and Data Types

- **What it is**: Redis stores data as key-value pairs with different value types
- **Why it matters**: Choosing the right data type impacts performance and memory
- **Mental model**: Think of Redis as a giant HashMap with typed values

```typescript
// Keys are strings, values can be:
await client.set('string_key', 'value');           // String
await client.hSet('hash_key', 'field', 'value');   // Hash
await client.lPush('list_key', 'item');            // List
await client.sAdd('set_key', 'member');            // Set
await client.zAdd('zset_key', { score: 1, value: 'member' }); // Sorted Set
await client.xAdd('stream_key', '*', { field: 'value' }); // Stream
```

**Common pitfalls:**

- Using wrong data type for use case (e.g., list instead of set for uniqueness)
- Not setting expiration on cache keys (memory leak)
- Using overly long key names (memory waste)

---

### 2. Persistence

- **What it is**: Redis can save data to disk for durability
- **Why it matters**: Choose between speed (no persistence) and reliability (persistent)
- **Mental model**: RDB = periodic snapshots, AOF = append-only log

```typescript
// Check persistence settings
const info = await client.info('persistence');
console.log(info);

// Force snapshot (admin command)
await client.bgSave(); // Background save

// Note: Configure persistence in redis.conf, not via client
```

**Persistence modes:**

- **RDB (Snapshotting)**: Periodic disk snapshots (faster, may lose recent data)
- **AOF (Append-Only File)**: Logs every write (slower, more durable)
- **Hybrid**: Both RDB and AOF (recommended for production)

---

### 3. TTL (Time To Live)

- **What it is**: Automatic key expiration after specified time
- **When to use**: Caching, session management, temporary data
- **Gotchas**: TTL is removed if key is overwritten without KEEPTTL option

```typescript
// Set key with expiration (seconds)
await client.setEx('cache_key', 3600, 'value'); // Expires in 1 hour

// Set key then add expiration
await client.set('key', 'value');
await client.expire('key', 60); // Expire in 60 seconds

// Set expiration in milliseconds
await client.pExpire('key', 5000); // 5 seconds

// Check remaining TTL
const ttl = await client.ttl('key'); // Returns seconds (-1 = no expiry, -2 = doesn't exist)

// Remove expiration
await client.persist('key');
```

---

### 4. Pub/Sub

- **What it is**: Message broadcasting pattern (publisher → subscribers)
- **Why it matters**: Real-time event distribution across services
- **Mental model**: Like a radio station - broadcast to all listeners, no history

```typescript
// Subscriber
const subscriber = client.duplicate();
await subscriber.connect();

await subscriber.subscribe('news_channel', (message) => {
  console.log('Received:', message);
});

// Publisher
await client.publish('news_channel', 'Breaking news!');

// Pattern subscribe (wildcard)
await subscriber.pSubscribe('events:*', (message, channel) => {
  console.log(`Channel ${channel}: ${message}`);
});

await client.publish('events:user_login', 'User 123 logged in');
```

**Pitfalls:**

- Messages are lost if no subscribers are listening
- No message history or replay capability
- Cannot distribute work (all subscribers get all messages)

---

### 5. Streams

- **What it is**: Persistent, append-only log for message queues
- **Why it matters**: Reliable message delivery with consumer groups and acknowledgments
- **Mental model**: Like Kafka Lite - ordered log with multiple consumer groups

```typescript
// Add entry to stream
const id = await client.xAdd('events', '*', {
  user: 'alice',
  action: 'login',
  timestamp: Date.now().toString()
});

// Read from stream (non-blocking)
const messages = await client.xRead(
  { key: 'events', id: '0' }, // Read from beginning
  { COUNT: 10 }
);

// Create consumer group
await client.xGroupCreate('events', 'workers', '0', { MKSTREAM: true });

// Read as consumer group (blocks until message available)
const groupMessages = await client.xReadGroup(
  'workers',          // Consumer group
  'worker-1',         // Consumer name
  { key: 'events', id: '>' }, // '>' = only new messages
  { COUNT: 1, BLOCK: 5000 }   // Block for 5 seconds
);

if (groupMessages && groupMessages.length > 0) {
  const [{ messages }] = groupMessages;
  for (const { id, message } of messages) {
    console.log('Processing:', message);

    // Acknowledge message
    await client.xAck('events', 'workers', id);
  }
}

// Check pending messages
const pending = await client.xPending('events', 'workers');
console.log('Pending:', pending);
```

**Use cases:**

- Message queues with guaranteed delivery
- Event sourcing
- Activity feeds
- Distributed task queues

---

### 6. Transactions

- **What it is**: Execute multiple commands atomically
- **When to use**: When you need all-or-nothing execution
- **Gotchas**: Commands are queued, not executed until EXEC

```typescript
// Using multi/exec
const results = await client
  .multi()
  .set('key1', 'value1')
  .set('key2', 'value2')
  .incr('counter')
  .exec();

console.log(results); // Array of results

// Watch for optimistic locking
await client.watch('balance');

const balance = parseInt(await client.get('balance') || '0');

if (balance >= 100) {
  const result = await client
    .multi()
    .decrBy('balance', 100)
    .incrBy('spent', 100)
    .exec();

  if (result === null) {
    console.log('Transaction aborted - balance was modified');
  }
}

await client.unwatch();
```

---

## Data Types and Operations

### Strings

Most basic type - can store text, numbers, or binary data (up to 512MB).

```typescript
// Set and get
await client.set('name', 'Alice');
const name = await client.get('name'); // 'Alice'

// Set with options
await client.set('key', 'value', {
  EX: 60,        // Expire in 60 seconds
  NX: true       // Only set if key doesn't exist
});

// Increment/decrement (atomic)
await client.incr('counter');           // counter = 1
await client.incrBy('counter', 5);      // counter = 6
await client.decr('counter');           // counter = 5

// Multiple keys
await client.mSet({ key1: 'val1', key2: 'val2' });
const values = await client.mGet(['key1', 'key2']); // ['val1', 'val2']

// Append
await client.append('message', ' world'); // message = 'hello world'

// Get and set (atomic swap)
const oldValue = await client.getSet('key', 'new_value');

// Check exists
const exists = await client.exists('key'); // 1 if exists, 0 otherwise

// Delete
await client.del('key');
```

---

### Hashes

Key-value pairs within a key (like a nested object).

```typescript
// Set field
await client.hSet('user:1000', 'name', 'Alice');
await client.hSet('user:1000', 'email', 'alice@example.com');

// Set multiple fields
await client.hSet('user:1000', {
  name: 'Alice',
  email: 'alice@example.com',
  age: '30'
});

// Get field
const name = await client.hGet('user:1000', 'name'); // 'Alice'

// Get all fields
const user = await client.hGetAll('user:1000');
// { name: 'Alice', email: 'alice@example.com', age: '30' }

// Get multiple fields
const fields = await client.hmGet('user:1000', ['name', 'email']);
// ['Alice', 'alice@example.com']

// Increment field
await client.hIncrBy('user:1000', 'login_count', 1);

// Check if field exists
const hasEmail = await client.hExists('user:1000', 'email'); // 1 or 0

// Get all field names
const fieldNames = await client.hKeys('user:1000'); // ['name', 'email', 'age']

// Get all values
const values = await client.hVals('user:1000'); // ['Alice', 'alice@example.com', '30']

// Delete field
await client.hDel('user:1000', 'age');

// Field count
const count = await client.hLen('user:1000'); // 2
```

---

### Lists

Ordered sequences of strings (like arrays, but optimized for push/pop).

```typescript
// Push to list (left/head)
await client.lPush('queue', 'task1');
await client.lPush('queue', ['task2', 'task3']); // Multiple items

// Push to right/tail
await client.rPush('queue', 'task4');

// Pop from list
const task = await client.lPop('queue');  // Remove from head
const last = await client.rPop('queue');  // Remove from tail

// Blocking pop (waits until item available)
const item = await client.blPop('queue', 5); // Block for 5 seconds
// Returns: { key: 'queue', element: 'task1' } or null

// Get range
const tasks = await client.lRange('queue', 0, -1); // All items
const first10 = await client.lRange('queue', 0, 9);

// Get by index
const first = await client.lIndex('queue', 0);

// Set by index
await client.lSet('queue', 0, 'updated_task');

// List length
const length = await client.lLen('queue');

// Trim list (keep only range)
await client.lTrim('queue', 0, 99); // Keep first 100 items

// Remove items
await client.lRem('queue', 0, 'task1'); // Remove all occurrences
await client.lRem('queue', 2, 'task2'); // Remove first 2 occurrences
```

**Use cases:**

- Job queues (FIFO)
- Activity feeds (latest N items)
- Stacks (LIFO)

---

### Sets

Unordered collections of unique strings.

```typescript
// Add members
await client.sAdd('tags', 'javascript');
await client.sAdd('tags', ['typescript', 'nodejs', 'javascript']); // Duplicates ignored

// Check membership
const isMember = await client.sIsMember('tags', 'javascript'); // 1 or 0

// Get all members
const tags = await client.sMembers('tags'); // ['javascript', 'typescript', 'nodejs']

// Random member
const random = await client.sRandMember('tags');
const random3 = await client.sRandMemberCount('tags', 3);

// Pop random member
const popped = await client.sPop('tags');

// Remove member
await client.sRem('tags', 'nodejs');

// Set size
const count = await client.sCard('tags'); // Number of members

// Set operations
await client.sAdd('set1', ['a', 'b', 'c']);
await client.sAdd('set2', ['b', 'c', 'd']);

const union = await client.sUnion(['set1', 'set2']);       // ['a', 'b', 'c', 'd']
const inter = await client.sInter(['set1', 'set2']);       // ['b', 'c']
const diff = await client.sDiff(['set1', 'set2']);         // ['a']

// Store result in new set
await client.sUnionStore('result', ['set1', 'set2']);
```

**Use cases:**

- Tags/categories
- Unique visitor tracking
- Friendships/relationships
- Filtering duplicates

---

### Sorted Sets

Sets with scores for ordering (like priority queues).

```typescript
// Add members with scores
await client.zAdd('leaderboard', { score: 100, value: 'alice' });
await client.zAdd('leaderboard', [
  { score: 200, value: 'bob' },
  { score: 150, value: 'charlie' }
]);

// Get rank (0-indexed, low to high)
const rank = await client.zRank('leaderboard', 'alice'); // 0

// Get reverse rank (high to low)
const revRank = await client.zRevRank('leaderboard', 'bob'); // 0

// Get score
const score = await client.zScore('leaderboard', 'alice'); // 100

// Increment score
await client.zIncrBy('leaderboard', 50, 'alice'); // alice = 150

// Get range (by rank)
const top3 = await client.zRange('leaderboard', 0, 2); // ['alice', 'charlie', 'bob']

// Get range with scores
const top3WithScores = await client.zRangeWithScores('leaderboard', 0, 2);
// [{ value: 'alice', score: 150 }, ...]

// Reverse range (high to low)
const topScorers = await client.zRevRange('leaderboard', 0, 2);

// Range by score
const mediumScorers = await client.zRangeByScore('leaderboard', 100, 200);

// Count by score range
const count = await client.zCount('leaderboard', 100, 200);

// Remove member
await client.zRem('leaderboard', 'alice');

// Remove by rank
await client.zRemRangeByRank('leaderboard', 0, 0); // Remove lowest

// Remove by score
await client.zRemRangeByScore('leaderboard', 0, 100);

// Cardinality (size)
const size = await client.zCard('leaderboard');
```

**Use cases:**

- Leaderboards
- Priority queues
- Time-series data (use timestamp as score)
- Rate limiting (sliding window)

---

### Streams

Append-only logs for message queues (covered in detail above).

```typescript
// Quick reference
await client.xAdd('stream', '*', { field: 'value' });
await client.xRead({ key: 'stream', id: '0' });
await client.xGroupCreate('stream', 'group', '0');
await client.xReadGroup('group', 'consumer', { key: 'stream', id: '>' });
await client.xAck('stream', 'group', messageId);
```

---

## Common Patterns

### Caching Pattern

```typescript
async function getCachedData(key: string): Promise<any> {
  // Try cache first
  const cached = await client.get(key);
  if (cached) {
    return JSON.parse(cached);
  }

  // Cache miss - fetch from source
  const data = await fetchFromDatabase(key);

  // Store in cache with TTL
  await client.setEx(key, 3600, JSON.stringify(data)); // 1 hour

  return data;
}

// Cache-aside with hash
async function getUserById(userId: string) {
  const cacheKey = `user:${userId}`;

  const cached = await client.hGetAll(cacheKey);
  if (Object.keys(cached).length > 0) {
    return cached;
  }

  const user = await db.users.findById(userId);
  await client.hSet(cacheKey, user);
  await client.expire(cacheKey, 3600);

  return user;
}
```

---

### Rate Limiting

```typescript
// Fixed window rate limiter
async function isRateLimited(userId: string, limit: number, windowSeconds: number): Promise<boolean> {
  const key = `rate_limit:${userId}:${Math.floor(Date.now() / 1000 / windowSeconds)}`;

  const current = await client.incr(key);

  if (current === 1) {
    await client.expire(key, windowSeconds);
  }

  return current > limit;
}

// Usage
if (await isRateLimited('user:123', 100, 60)) {
  throw new Error('Rate limit exceeded');
}

// Sliding window with sorted set
async function slidingWindowRateLimit(userId: string, limit: number, windowMs: number): Promise<boolean> {
  const key = `rate:${userId}`;
  const now = Date.now();
  const windowStart = now - windowMs;

  // Remove old entries
  await client.zRemRangeByScore(key, 0, windowStart);

  // Count current requests
  const current = await client.zCard(key);

  if (current >= limit) {
    return true; // Rate limited
  }

  // Add current request
  await client.zAdd(key, { score: now, value: `${now}` });
  await client.expire(key, Math.ceil(windowMs / 1000));

  return false;
}
```

---

### Distributed Lock

```typescript
async function acquireLock(lockKey: string, ttlSeconds: number): Promise<string | null> {
  const lockValue = `${Date.now()}-${Math.random()}`;

  // SET if Not eXists with expiration
  const result = await client.set(lockKey, lockValue, {
    NX: true,
    EX: ttlSeconds
  });

  return result === 'OK' ? lockValue : null;
}

async function releaseLock(lockKey: string, lockValue: string): Promise<boolean> {
  // Use Lua script for atomic check-and-delete
  const script = `
    if redis.call("get", KEYS[1]) == ARGV[1] then
      return redis.call("del", KEYS[1])
    else
      return 0
    end
  `;

  const result = await client.eval(script, {
    keys: [lockKey],
    arguments: [lockValue]
  });

  return result === 1;
}

// Usage
const lockValue = await acquireLock('resource:123', 30);
if (lockValue) {
  try {
    // Critical section
    await doWork();
  } finally {
    await releaseLock('resource:123', lockValue);
  }
} else {
  console.log('Could not acquire lock');
}
```

---

### Session Storage

```typescript
interface Session {
  userId: string;
  username: string;
  loginTime: number;
}

async function createSession(sessionId: string, data: Session): Promise<void> {
  const key = `session:${sessionId}`;

  await client.hSet(key, {
    userId: data.userId,
    username: data.username,
    loginTime: data.loginTime.toString()
  });

  // Auto-expire after 24 hours
  await client.expire(key, 86400);
}

async function getSession(sessionId: string): Promise<Session | null> {
  const key = `session:${sessionId}`;
  const data = await client.hGetAll(key);

  if (!data || Object.keys(data).length === 0) {
    return null;
  }

  return {
    userId: data.userId,
    username: data.username,
    loginTime: parseInt(data.loginTime)
  };
}

async function destroySession(sessionId: string): Promise<void> {
  await client.del(`session:${sessionId}`);
}

// Extend session TTL on activity
async function touchSession(sessionId: string): Promise<void> {
  await client.expire(`session:${sessionId}`, 86400);
}
```

---

### Message Queue with Streams

```typescript
// Producer
async function enqueueJob(queueName: string, job: any): Promise<string> {
  const id = await client.xAdd(queueName, '*', {
    type: job.type,
    payload: JSON.stringify(job.data),
    createdAt: Date.now().toString()
  });

  return id;
}

// Consumer
async function processJobs(queueName: string, groupName: string, consumerName: string) {
  // Create consumer group if not exists
  try {
    await client.xGroupCreate(queueName, groupName, '0', { MKSTREAM: true });
  } catch (err) {
    // Group already exists
  }

  while (true) {
    const messages = await client.xReadGroup(
      groupName,
      consumerName,
      { key: queueName, id: '>' },
      { COUNT: 10, BLOCK: 5000 }
    );

    if (!messages || messages.length === 0) continue;

    for (const { messages: streamMessages } of messages) {
      for (const { id, message } of streamMessages) {
        try {
          const job = {
            type: message.type,
            data: JSON.parse(message.payload),
            createdAt: parseInt(message.createdAt)
          };

          await processJob(job);

          // Acknowledge success
          await client.xAck(queueName, groupName, id);
        } catch (err) {
          console.error(`Failed to process job ${id}:`, err);
          // Job will remain in pending list for retry
        }
      }
    }
  }
}

// Retry failed jobs
async function retryPendingJobs(queueName: string, groupName: string) {
  const pending = await client.xPending(queueName, groupName);

  if (pending.pending > 0) {
    const details = await client.xPendingRange(
      queueName,
      groupName,
      '-',
      '+',
      10
    );

    for (const entry of details) {
      const { id, millisecondsSinceLastDelivery, deliveryCount } = entry;

      // Retry if pending for > 5 minutes and < 3 retries
      if (millisecondsSinceLastDelivery > 300000 && deliveryCount < 3) {
        await client.xClaim(
          queueName,
          groupName,
          'retry-consumer',
          60000,
          [id]
        );
      }
    }
  }
}
```

---

### Pub/Sub Event Broadcasting

```typescript
// Event emitter
async function broadcastEvent(channel: string, event: any): Promise<number> {
  const subscriberCount = await client.publish(
    channel,
    JSON.stringify(event)
  );

  return subscriberCount;
}

// Event listener
async function subscribeToEvents(channels: string[], handler: (event: any, channel: string) => void) {
  const subscriber = client.duplicate();
  await subscriber.connect();

  await subscriber.subscribe(channels, (message, channel) => {
    try {
      const event = JSON.parse(message);
      handler(event, channel);
    } catch (err) {
      console.error('Failed to parse event:', err);
    }
  });

  return subscriber;
}

// Usage
const subscriber = await subscribeToEvents(['user:events', 'order:events'], (event, channel) => {
  console.log(`Event on ${channel}:`, event);
});

// Broadcast
await broadcastEvent('user:events', { type: 'login', userId: '123' });
await broadcastEvent('order:events', { type: 'created', orderId: '456' });

// Pattern subscription
await subscriber.pSubscribe('user:*', (message, channel) => {
  console.log(`User event on ${channel}:`, message);
});
```

---

## Quick Reference

**Connection:**

```typescript
const client = createClient({ url: 'redis://localhost:6379' });
await client.connect();
await client.quit();
```

**Strings:**

```typescript
await client.set('key', 'value');
const val = await client.get('key');
await client.setEx('key', 60, 'value'); // With TTL
```

**Hashes:**

```typescript
await client.hSet('hash', 'field', 'value');
const val = await client.hGet('hash', 'field');
const all = await client.hGetAll('hash');
```

**Lists:**

```typescript
await client.lPush('list', 'item');
const item = await client.lPop('list');
const all = await client.lRange('list', 0, -1);
```

**Sets:**

```typescript
await client.sAdd('set', 'member');
const members = await client.sMembers('set');
const exists = await client.sIsMember('set', 'member');
```

**Sorted Sets:**

```typescript
await client.zAdd('zset', { score: 1, value: 'member' });
const top = await client.zRange('zset', 0, 9);
const topRev = await client.zRevRange('zset', 0, 9);
```

**Streams:**

```typescript
await client.xAdd('stream', '*', { field: 'value' });
await client.xReadGroup('group', 'consumer', { key: 'stream', id: '>' });
await client.xAck('stream', 'group', id);
```

**Pub/Sub:**

```typescript
await subscriber.subscribe('channel', (msg) => console.log(msg));
await client.publish('channel', 'message');
```

**Expiration:**

```typescript
await client.expire('key', 60);
const ttl = await client.ttl('key');
await client.persist('key');
```

**Transactions:**

```typescript
const results = await client.multi().set('k1', 'v1').set('k2', 'v2').exec();
```

---

## Performance Tips

1. **Use pipelining for multiple commands** - Batch operations to reduce network round-trips:

   ```typescript
   const pipeline = client.multi();
   for (let i = 0; i < 1000; i++) {
     pipeline.set(`key:${i}`, `value${i}`);
   }
   await pipeline.exec();
   ```

2. **Use SCAN instead of KEYS** - KEYS blocks the server, SCAN iterates incrementally:

   ```typescript
   // ❌ Don't do this in production
   const keys = await client.keys('user:*');

   // ✅ Do this instead
   const keys: string[] = [];
   for await (const key of client.scanIterator({ MATCH: 'user:*', COUNT: 100 })) {
     keys.push(key);
   }
   ```

3. **Set appropriate TTLs on cached data** - Prevent memory bloat:

   ```typescript
   await client.setEx('cache:key', 3600, data); // 1 hour
   ```

4. **Use hashes for related data** - More memory efficient than separate keys:

   ```typescript
   // ❌ Less efficient
   await client.set('user:1000:name', 'Alice');
   await client.set('user:1000:email', 'alice@example.com');

   // ✅ More efficient
   await client.hSet('user:1000', { name: 'Alice', email: 'alice@example.com' });
   ```

5. **Use Lua scripts for atomic operations** - Reduce round-trips and ensure atomicity:

   ```typescript
   const script = `
     local current = redis.call('get', KEYS[1])
     if current and tonumber(current) > tonumber(ARGV[1]) then
       return redis.call('set', KEYS[1], ARGV[1])
     end
     return 0
   `;

   await client.eval(script, { keys: ['price'], arguments: ['100'] });
   ```

6. **Use connection pooling** - Reuse connections instead of creating new ones:

   ```typescript
   // Single client instance handles pooling internally
   const client = createClient({ url: 'redis://localhost:6379' });
   await client.connect();
   // Reuse this client across your application
   ```

7. **Monitor memory usage** - Use `INFO memory` to track usage:

   ```typescript
   const info = await client.info('memory');
   console.log(info);
   ```

8. **Optimize data serialization** - Use MessagePack or Protocol Buffers for large objects instead of JSON.

---

## Security & Safety

- **Use authentication** - Always set a password in production:

  ```bash
  # In redis.conf
  requirepass your_strong_password
  ```

  ```typescript
  const client = createClient({
    url: 'redis://:your_strong_password@localhost:6379'
  });
  ```

- **Bind to localhost** - Don't expose Redis to the internet:

  ```bash
  # In redis.conf
  bind 127.0.0.1 ::1
  ```

- **Use TLS/SSL** - Encrypt connections for sensitive data:

  ```typescript
  const client = createClient({
    url: 'rediss://localhost:6379', // Note: rediss:// for TLS
    socket: {
      tls: true,
      rejectUnauthorized: true
    }
  });
  ```

- **Disable dangerous commands** - Rename or disable FLUSHALL, FLUSHDB, CONFIG:

  ```bash
  # In redis.conf
  rename-command FLUSHALL ""
  rename-command FLUSHDB ""
  rename-command CONFIG ""
  ```

- **Limit max memory** - Prevent Redis from consuming all system memory:

  ```bash
  # In redis.conf
  maxmemory 2gb
  maxmemory-policy allkeys-lru
  ```

- **Validate input** - Sanitize user input before using as keys:

  ```typescript
  // ❌ Unsafe
  await client.get(userInput);

  // ✅ Safe
  const sanitized = userInput.replace(/[^a-zA-Z0-9:_-]/g, '');
  await client.get(`user:${sanitized}`);
  ```

- **Use ACLs** (Redis 6+) - Fine-grained access control:

  ```bash
  # Create user with limited permissions
  ACL SETUSER worker on >password ~jobs:* +xreadgroup +xack
  ```

---

## Complex Topic: Redis Streams vs Pub/Sub

### Redis Streams

- **What it is**: Persistent, append-only log with consumer groups and acknowledgments
- **Why it matters**: Reliable message delivery with replay capability and work distribution
- **Mental model**: Like Kafka Lite - ordered, persistent, acknowledgeable messages
- **When to use**: Message queues, task distribution, event sourcing, audit logs

```typescript
// Streams - messages persist
await client.xAdd('events', '*', { type: 'sale', amount: '100' });

// Multiple workers share the load
await client.xReadGroup('workers', 'worker-1', { key: 'events', id: '>' });
await client.xReadGroup('workers', 'worker-2', { key: 'events', id: '>' });

// Acknowledge processing
await client.xAck('events', 'workers', messageId);

// Replay from any position
await client.xRead({ key: 'events', id: '0' }); // From beginning
```

**Pitfalls:**

- Streams grow unbounded - trim periodically with `XTRIM`
- Consumer groups need explicit creation
- Pending messages need manual retry logic

---

### Redis Pub/Sub

- **What it is**: Fire-and-forget message broadcasting
- **Why it matters**: Real-time event distribution across services
- **Mental model**: Like a radio broadcast - ephemeral, all subscribers receive
- **When to use**: Notifications, live updates, cache invalidation signals

```typescript
// Pub/Sub - messages are lost if no subscribers
await client.publish('notifications', 'New message!');

// All subscribers receive the same message
await subscriber1.subscribe('notifications', (msg) => console.log(msg));
await subscriber2.subscribe('notifications', (msg) => console.log(msg));
```

**Pitfalls:**

- No persistence - messages lost if no active subscribers
- No acknowledgment or retry mechanism
- All subscribers get all messages (no load balancing)
- No message history or replay

---

**Comparison:**

| Feature | Streams | Pub/Sub |
|---------|---------|---------|
| Persistence | ✅ Yes | ❌ No (ephemeral) |
| Consumer groups | ✅ Yes | ❌ No |
| Acknowledgment | ✅ Yes | ❌ No |
| Replay | ✅ Yes | ❌ No |
| Load balancing | ✅ Yes | ❌ No (broadcast) |
| Use case | Task queues, events | Notifications, live updates |

**Recommendation:** Use Streams for reliable queues, Pub/Sub for ephemeral broadcasts.

---

## CLI Commands Reference

### Server Management

```bash
# Start/stop
redis-server                     # Start with default config
redis-server /path/to/redis.conf # Start with custom config
redis-cli shutdown               # Graceful shutdown

# Connect to server
redis-cli                        # Connect to localhost:6379
redis-cli -h host -p port        # Connect to remote
redis-cli -a password            # With auth
redis-cli --tls                  # With TLS

# Check status
redis-cli ping                   # Returns PONG if alive
redis-cli info                   # Server info
redis-cli info memory            # Memory info only
redis-cli info stats             # Stats only
```

### Key Operations

```bash
# Set and get
SET key value
GET key
SETEX key 60 value              # With 60s expiration
SETNX key value                 # Set if not exists

# Multiple keys
MSET key1 val1 key2 val2
MGET key1 key2

# Delete
DEL key
DEL key1 key2 key3

# Exists
EXISTS key

# Expiration
EXPIRE key 60                   # Expire in 60 seconds
TTL key                         # Check remaining TTL
PERSIST key                     # Remove expiration

# All keys (⚠️ blocks server)
KEYS *
KEYS user:*                     # Pattern match

# Safe iteration
SCAN 0 MATCH user:* COUNT 100
```

### Monitoring & Debugging

```bash
# Real-time monitoring
redis-cli monitor               # Show all commands in real-time

# Slow log
SLOWLOG GET 10                  # Last 10 slow queries

# Client list
CLIENT LIST                     # All connected clients

# Memory analysis
MEMORY STATS
MEMORY USAGE key                # Memory used by key

# Database size
DBSIZE                          # Number of keys

# Clear database (⚠️ destructive!)
FLUSHDB                         # Clear current database
FLUSHALL                        # Clear all databases

# Save snapshot
SAVE                            # Blocking save
BGSAVE                          # Background save
```

### Stream Operations

```bash
# Add to stream
XADD stream * field1 value1 field2 value2

# Read from stream
XREAD COUNT 10 STREAMS stream 0

# Create consumer group
XGROUP CREATE stream group 0 MKSTREAM

# Read as consumer group
XREADGROUP GROUP group consumer COUNT 1 STREAMS stream >

# Acknowledge
XACK stream group message-id

# Stream info
XINFO STREAM stream
XINFO GROUPS stream
XINFO CONSUMERS stream group

# Pending messages
XPENDING stream group
XPENDING stream group - + 10

# Stream length
XLEN stream

# Trim stream
XTRIM stream MAXLEN 1000        # Keep last 1000 entries
```

---

## Troubleshooting

### Connection Refused

**Error:** `Error: connect ECONNREFUSED 127.0.0.1:6379`

**Cause:** Redis server not running.

**Solution:**

```bash
# Check if Redis is running
ps aux | grep redis

# Start Redis
redis-server

# Or as service
sudo systemctl start redis
brew services start redis
```

### Authentication Failed

**Error:** `NOAUTH Authentication required`

**Cause:** Redis requires password but none provided.

**Solution:**

```typescript
const client = createClient({
  url: 'redis://:your_password@localhost:6379'
});
```

Or in CLI:

```bash
redis-cli -a your_password
# Or after connecting
AUTH your_password
```

### Memory Issues

**Error:** `OOM command not allowed when used memory > 'maxmemory'`

**Cause:** Redis hit configured memory limit.

**Solution:**

```bash
# Check memory usage
redis-cli INFO memory

# Increase max memory (in redis.conf)
maxmemory 4gb

# Or set eviction policy
maxmemory-policy allkeys-lru

# Manually free memory
FLUSHDB  # ⚠️ Deletes all keys in current DB
```

### Performance Degradation

**Symptoms:** Slow responses, high CPU usage

**Debug steps:**

```bash
# Check slow queries
redis-cli SLOWLOG GET 10

# Monitor commands
redis-cli monitor

# Check client connections
redis-cli CLIENT LIST

# Analyze key space
redis-cli --bigkeys

# Check latency
redis-cli --latency
redis-cli --latency-history
```

**Common causes:**

- Using `KEYS *` instead of `SCAN`
- Large values in keys
- Too many clients
- Disk I/O blocking (if persistence enabled)
- Network latency

**Solutions:**

- Use `SCAN` for iteration
- Implement pipelining
- Add more Redis instances (sharding)
- Optimize data structures
- Disable persistence if not needed

---

## Next Steps

- **Explore Redis Modules**: RedisJSON, RedisGraph, RedisTimeSeries, RediSearch
- **Learn clustering**: Redis Cluster for horizontal scaling
- **Master Lua scripting**: Write custom atomic operations
- **Implement monitoring**: Use Redis Enterprise, RedisInsight, or Prometheus
- **Study persistence modes**: RDB vs AOF vs Hybrid
- **Practice patterns**: Implement common patterns in your projects

---

## Resources

- **Official docs**: [https://redis.io/docs/](https://redis.io/docs/)
- **Node.js client**: [https://github.com/redis/node-redis](https://github.com/redis/node-redis)
- **Redis University**: [https://university.redis.com/](https://university.redis.com/)
- **Redis Commands**: [https://redis.io/commands/](https://redis.io/commands/)
- **RedisInsight**: [https://redis.io/insight/](https://redis.io/insight/) (GUI client)
- **Best practices**: [https://redis.io/docs/management/optimization/](https://redis.io/docs/management/optimization/)

---

**Built for high-performance caching, queuing, and real-time applications** 🚀
