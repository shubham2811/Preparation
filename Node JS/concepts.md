# [Difference Between Long Polling, Server-Sent Events (SSE), and WebSockets](https://medium.com/@asharsaleem4/long-polling-vs-server-sent-events-vs-websockets-a-comprehensive-guide-fb27c8e610d0)

| Feature         | Long Polling                        | Server-Sent Events (SSE)           | WebSockets                        |
|-----------------|-------------------------------------|------------------------------------|-----------------------------------|
| Protocol        | HTTP                                | HTTP (unidirectional)              | TCP (full-duplex)                 |
| Direction       | Client ↔ Server (via repeated HTTP) | Server → Client (unidirectional)   | Client ↔ Server (bidirectional)   |
| Connection      | Repeated HTTP requests              | Single persistent HTTP connection  | Single persistent TCP connection  |
| Complexity      | Simple to implement                 | Simple, built-in browser support   | More complex, requires handshake  |
| Use Case        | Notifications, chat (basic)         | Live feeds, notifications          | Real-time apps, chat, games       |
| Browser Support | Universal                           | Most modern browsers               | Most modern browsers              |

---

## Long Polling

- The client sends a request to the server and waits for a response.
- The server holds the request open until new data is available or a timeout occurs.
- After receiving a response, the client immediately sends a new request.
- Simulates real-time communication but is less efficient due to repeated HTTP requests.

**Example Use Case:** Chat applications, notifications.
```js
const express = require('express');
const app = express();
const port = 3000;

let messages = [];
app.use(express.json());

app.post('/messages', (req, res) => {
  messages.push(req.body.message);
  res.status(200).end();
});

app.get('/poll', (req, res) => {
  const interval = setInterval(() => {
    if (messages.length > 0) {
      clearInterval(interval);
      res.json({ message: messages.shift() });
    }
  }, 1000);
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
--------------------------------------------------------------------
<!DOCTYPE html>
<html>
<head>
  <title>Long Polling Example</title>
</head>
<body>
  <div id="messages"></div>
  <script>
    function poll() {
      fetch('/poll')
        .then(response => response.json())
        .then(data => {
          const messagesDiv = document.getElementById('messages');
          const newMessage = document.createElement('div');
          newMessage.textContent = data.message;
          messagesDiv.appendChild(newMessage);
          poll(); // Poll again
        });
    }

    poll(); // Start polling
  </script>
</body>
</html>
```

---

## Server-Sent Events (SSE)

- The client opens a single HTTP connection and the server pushes updates to the client as events.
- Only supports server-to-client (unidirectional) communication.
- Built-in support in most browsers via the `EventSource` API.
- Uses less overhead than long polling for server-to-client updates.

**Example Use Case:** Live news feeds, stock tickers, notifications.
```js
const express = require('express');
const app = express();
const port = 3000;

app.use(express.json());

app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  setInterval(() => {
    res.write(`data: ${JSON.stringify({ message: 'Hello from server!' })}\n\n`);
  }, 2000);
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
-----------------------------------------------------------------------
<!DOCTYPE html>
<html>
<head>
  <title>SSE Example</title>
</head>
<body>
  <div id="messages"></div>
  <script>
    const eventSource = new EventSource('/events');

    eventSource.onmessage = function(event) {
      const messagesDiv = document.getElementById('messages');
      const newMessage = document.createElement('div');
      newMessage.textContent = JSON.parse(event.data).message;
      messagesDiv.appendChild(newMessage);
    };
  </script>
</body>
</html>

```
---

## WebSockets

- Establishes a persistent, full-duplex connection between client and server over TCP.
- Allows real-time, bidirectional communication.
- More efficient for high-frequency, low-latency data exchange.
- Requires a handshake to upgrade from HTTP to WebSocket protocol.

**Example Use Case:** Online games, collaborative editing, real-time chat.
```js
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', ws => {
  ws.on('message', message => {
    console.log('Received:', message);
    ws.send(`Hello, you sent -> ${message}`);
  });

  ws.send('Welcome to the WebSocket server!');
});
--------------------------------------------------------
<!DOCTYPE html>
<html>
<head>
  <title>WebSockets Example</title>
</head>
<body>
  <div id="messages"></div>
  <script>
    const ws = new WebSocket('ws://localhost:8080');

    ws.onmessage = function(event) {
      const messagesDiv = document.getElementById('messages');
      const newMessage = document.createElement('div');
      newMessage.textContent = event.data;
      messagesDiv.appendChild(newMessage);
    };

    ws.onopen = function() {
      ws.send('Hello Server!');
    };
  </script>
</body>
</html>
```
# Multithreading and clustering in node js

## Multithreading

Node.js worker threads enable true parallelism for CPU-intensive tasks. Workers communicate through message passing and shared memory.

### Basic Worker Creation
```js
const { Worker } = require('worker_threads');

new Worker('./heavy-task.js'); // runs in parallel thread
```

## How Node.js Workers Communicate

Worker threads use **message passing** and **shared memory** to communicate. Here are the main communication mechanisms:

### 1. Message Passing Between Main Thread and Workers

#### Main Thread to Worker
```js
// main.js
const { Worker, isMainThread, parentPort } = require('worker_threads');

if (isMainThread) {
  // Main thread
  const worker = new Worker(__filename);
  
  // Send message to worker
  worker.postMessage({
    type: 'TASK',
    data: [1, 2, 3, 4, 5],
    operation: 'multiply'
  });
  
  // Listen for messages from worker
  worker.on('message', (result) => {
    console.log('Result from worker:', result);
  });
  
  worker.on('error', (error) => {
    console.error('Worker error:', error);
  });
  
  worker.on('exit', (code) => {
    console.log(`Worker stopped with exit code ${code}`);
  });
  
} else {
  // Worker thread
  parentPort.on('message', (message) => {
    console.log('Worker received:', message);
    
    if (message.type === 'TASK') {
      const result = message.data.map(n => n * 2);
      
      // Send result back to main thread
      parentPort.postMessage({
        type: 'RESULT',
        data: result,
        workerId: require('worker_threads').threadId
      });
    }
  });
}
```

### 2. Worker Pool with Communication
```js
const { Worker } = require('worker_threads');
const os = require('os');

class WorkerPool {
  constructor(workerScript, poolSize = os.cpus().length) {
    this.workers = [];
    this.taskQueue = [];
    this.workerIndex = 0;
    
    // Create worker pool
    for (let i = 0; i < poolSize; i++) {
      const worker = new Worker(workerScript);
      worker.id = i;
      worker.busy = false;
      
      worker.on('message', (result) => {
        console.log(`Worker ${worker.id} completed task:`, result);
        worker.busy = false;
        this.processQueue();
      });
      
      this.workers.push(worker);
    }
  }
  
  execute(data) {
    return new Promise((resolve, reject) => {
      const task = { data, resolve, reject };
      this.taskQueue.push(task);
      this.processQueue();
    });
  }
  
  processQueue() {
    if (this.taskQueue.length === 0) return;
    
    const availableWorker = this.workers.find(w => !w.busy);
    if (!availableWorker) return;
    
    const task = this.taskQueue.shift();
    availableWorker.busy = true;
    availableWorker.postMessage(task.data);
    
    // Store promise handlers for this worker
    availableWorker.currentTask = task;
  }
}

// Usage
const pool = new WorkerPool('./cpu-intensive-worker.js', 4);

async function processTasks() {
  const tasks = [
    { numbers: [1, 2, 3] },
    { numbers: [4, 5, 6] },
    { numbers: [7, 8, 9] }
  ];
  
  const results = await Promise.all(
    tasks.map(task => pool.execute(task))
  );
  
  console.log('All tasks completed:', results);
}
```

### 3. Shared Memory Communication (SharedArrayBuffer)
```js
// main.js
const { Worker, isMainThread } = require('worker_threads');

if (isMainThread) {
  // Create shared memory buffer
  const sharedBuffer = new SharedArrayBuffer(1024);
  const sharedArray = new Int32Array(sharedBuffer);
  
  // Initialize shared data
  sharedArray[0] = 100; // counter
  sharedArray[1] = 0;   // sum
  
  // Create multiple workers sharing the same memory
  const workers = [];
  for (let i = 0; i < 3; i++) {
    const worker = new Worker(__filename, {
      workerData: { sharedBuffer, workerId: i }
    });
    
    worker.on('message', (msg) => {
      console.log(`Worker ${i}:`, msg);
    });
    
    workers.push(worker);
  }
  
  // Monitor shared state
  setInterval(() => {
    console.log(`Shared counter: ${sharedArray[0]}, Sum: ${sharedArray[1]}`);
  }, 1000);
  
} else {
  // Worker thread
  const { workerData, parentPort } = require('worker_threads');
  const { sharedBuffer, workerId } = workerData;
  const sharedArray = new Int32Array(sharedBuffer);
  
  // Atomic operations on shared memory
  setInterval(() => {
    // Atomic decrement counter
    const oldValue = Atomics.compareExchange(sharedArray, 0, sharedArray[0], sharedArray[0] - 1);
    
    if (oldValue > 0) {
      // Atomic add to sum
      Atomics.add(sharedArray, 1, workerId + 1);
      
      parentPort.postMessage({
        action: 'decremented',
        oldValue,
        newValue: sharedArray[0],
        sum: sharedArray[1]
      });
    }
  }, 500);
}
```

### 4. Worker-to-Worker Communication via Main Thread
```js
// coordinator.js
const { Worker } = require('worker_threads');

class WorkerCoordinator {
  constructor() {
    this.workers = new Map();
    this.messageQueue = [];
  }
  
  createWorker(id, script) {
    const worker = new Worker(script, { workerData: { workerId: id } });
    
    worker.on('message', (message) => {
      this.handleWorkerMessage(id, message);
    });
    
    this.workers.set(id, worker);
    return worker;
  }
  
  handleWorkerMessage(senderId, message) {
    console.log(`Message from Worker ${senderId}:`, message);
    
    if (message.type === 'BROADCAST') {
      // Send to all other workers
      this.workers.forEach((worker, workerId) => {
        if (workerId !== senderId) {
          worker.postMessage({
            type: 'PEER_MESSAGE',
            from: senderId,
            data: message.data
          });
        }
      });
    } else if (message.type === 'DIRECT_MESSAGE') {
      // Send to specific worker
      const targetWorker = this.workers.get(message.target);
      if (targetWorker) {
        targetWorker.postMessage({
          type: 'PEER_MESSAGE',
          from: senderId,
          data: message.data
        });
      }
    }
  }
  
  broadcast(message) {
    this.workers.forEach(worker => {
      worker.postMessage({
        type: 'COORDINATOR_MESSAGE',
        data: message
      });
    });
  }
}

// Usage
const coordinator = new WorkerCoordinator();

// Create workers
coordinator.createWorker('worker1', './communicating-worker.js');
coordinator.createWorker('worker2', './communicating-worker.js');
coordinator.createWorker('worker3', './communicating-worker.js');

// Broadcast to all workers
coordinator.broadcast({ command: 'START_PROCESSING' });
```

```js
// communicating-worker.js
const { parentPort, workerData } = require('worker_threads');
const { workerId } = workerData;

// Listen for messages from coordinator or other workers
parentPort.on('message', (message) => {
  console.log(`Worker ${workerId} received:`, message);
  
  switch (message.type) {
    case 'COORDINATOR_MESSAGE':
      console.log(`Worker ${workerId} got coordinator message:`, message.data);
      break;
      
    case 'PEER_MESSAGE':
      console.log(`Worker ${workerId} got message from Worker ${message.from}:`, message.data);
      
      // Reply to sender
      parentPort.postMessage({
        type: 'DIRECT_MESSAGE',
        target: message.from,
        data: `Hello back from Worker ${workerId}!`
      });
      break;
  }
});

// Send broadcast message
setTimeout(() => {
  parentPort.postMessage({
    type: 'BROADCAST',
    data: `Broadcast from Worker ${workerId}`
  });
}, 2000);

// Send direct message to specific worker
setTimeout(() => {
  const targetWorker = workerId === 'worker1' ? 'worker2' : 'worker1';
  parentPort.postMessage({
    type: 'DIRECT_MESSAGE',
    target: targetWorker,
    data: `Direct message from Worker ${workerId}`
  });
}, 3000);
```

### 5. Advanced Communication with MessageChannel
```js
const { Worker, MessageChannel } = require('worker_threads');

// Create a message channel for direct worker-to-worker communication
const { port1, port2 } = new MessageChannel();

// Worker 1
const worker1 = new Worker(`
  const { parentPort, workerData } = require('worker_threads');
  const { port } = workerData;
  
  // Receive messages from Worker 2
  port.on('message', (data) => {
    console.log('Worker 1 received from Worker 2:', data);
  });
  
  // Send message to Worker 2
  setTimeout(() => {
    port.postMessage('Hello from Worker 1!');
  }, 1000);
  
  port.start();
`, {
  eval: true,
  workerData: { port: port1 },
  transferList: [port1]
});

// Worker 2
const worker2 = new Worker(`
  const { parentPort, workerData } = require('worker_threads');
  const { port } = workerData;
  
  // Receive messages from Worker 1
  port.on('message', (data) => {
    console.log('Worker 2 received from Worker 1:', data);
    
    // Reply back
    port.postMessage('Hello back from Worker 2!');
  });
  
  port.start();
`, {
  eval: true,
  workerData: { port: port2 },
  transferList: [port2]
});
```

### 6. Worker Communication Patterns Summary

| Pattern | Description | Use Case | Performance |
|---------|-------------|----------|-------------|
| **Message Passing** | `postMessage()` between threads | Task distribution, results | Good |
| **Shared Memory** | `SharedArrayBuffer` with atomics | High-frequency data sharing | Excellent |
| **Coordinator Pattern** | Main thread routes messages | Complex worker coordination | Good |
| **Message Channels** | Direct worker-to-worker pipes | Peer-to-peer communication | Very Good |
| **Broadcast** | One-to-many messaging | System notifications | Fair |

### 7. Best Practices for Worker Communication

```js
// Error handling and cleanup
class RobustWorkerPool {
  constructor(script, size = 4) {
    this.workers = [];
    this.createWorkers(script, size);
  }
  
  createWorkers(script, size) {
    for (let i = 0; i < size; i++) {
      this.createWorker(script, i);
    }
  }
  
  createWorker(script, id) {
    const worker = new Worker(script, { workerData: { id } });
    
    worker.on('message', (result) => {
      this.handleMessage(id, result);
    });
    
    worker.on('error', (error) => {
      console.error(`Worker ${id} error:`, error);
      this.restartWorker(script, id);
    });
    
    worker.on('exit', (code) => {
      if (code !== 0) {
        console.error(`Worker ${id} stopped with exit code ${code}`);
        this.restartWorker(script, id);
      }
    });
    
    this.workers[id] = worker;
  }
  
  restartWorker(script, id) {
    console.log(`Restarting worker ${id}`);
    this.createWorker(script, id);
  }
  
  handleMessage(workerId, message) {
    // Handle timeouts
    if (message.type === 'TIMEOUT') {
      console.log(`Worker ${workerId} task timed out`);
      this.workers[workerId].terminate();
      this.restartWorker('./worker.js', workerId);
    }
  }
  
  terminate() {
    this.workers.forEach(worker => worker.terminate());
  }
}
```

### Key Differences: Workers vs Clusters

| Feature | Worker Threads | Clusters |
|---------|----------------|----------|
| **Memory** | Can share memory | Separate memory spaces |
| **Communication** | Message passing + SharedArrayBuffer | IPC only |
| **Use Case** | CPU-intensive tasks | I/O scaling |
| **Overhead** | Lower | Higher |
| **Isolation** | Shared V8 instance | Separate processes |
## Clustering
```js
const cluster = require('cluster');
const http = require('http');
const os = require('os');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;
  for (let i = 0; i < numCPUs; i++) cluster.fork();
} else {
  http.createServer((req, res) => res.end('Handled by worker')).listen(3000);
}
```

## How Node.js Clusters Communicate

Node.js clusters use **Inter-Process Communication (IPC)** to communicate between the master process and worker processes. Here are the main communication mechanisms:

### 1. Master-Worker Communication

#### Sending Messages from Master to Workers
```js
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);
  
  // Fork workers
  for (let i = 0; i < os.cpus().length; i++) {
    const worker = cluster.fork();
    
    // Send message to specific worker
    worker.send({
      type: 'CONFIG_UPDATE',
      data: { maxConnections: 100, timeout: 5000 }
    });
  }
  
  // Broadcast message to all workers
  Object.values(cluster.workers).forEach(worker => {
    worker.send({
      type: 'BROADCAST',
      message: 'Server maintenance in 5 minutes'
    });
  });
  
} else {
  // Worker process
  console.log(`Worker ${process.pid} started`);
  
  // Listen for messages from master
  process.on('message', (msg) => {
    console.log(`Worker ${process.pid} received:`, msg);
    
    switch(msg.type) {
      case 'CONFIG_UPDATE':
        // Update worker configuration
        console.log('Updating config:', msg.data);
        break;
      case 'BROADCAST':
        console.log('Broadcast message:', msg.message);
        break;
    }
  });
}
```

#### Sending Messages from Worker to Master
```js
if (cluster.isMaster) {
  cluster.on('message', (worker, message) => {
    console.log(`Master received from worker ${worker.process.pid}:`, message);
    
    if (message.type === 'WORKER_STATUS') {
      console.log(`Worker ${worker.process.pid} status:`, message.data);
    }
  });
  
} else {
  // Worker sends message to master
  process.send({
    type: 'WORKER_STATUS',
    data: {
      pid: process.pid,
      memoryUsage: process.memoryUsage(),
      uptime: process.uptime()
    }
  });
}
```

### 2. Shared State Management

Since workers don't share memory, you need external storage for shared state:

```js
const cluster = require('cluster');
const redis = require('redis');

if (cluster.isMaster) {
  const redisClient = redis.createClient();
  
  // Master manages shared state
  cluster.on('message', async (worker, message) => {
    if (message.type === 'SET_SHARED_DATA') {
      await redisClient.set(message.key, JSON.stringify(message.value));
      
      // Notify all workers about the update
      Object.values(cluster.workers).forEach(w => {
        if (w.id !== worker.id) {
          w.postMessage({
            type: 'SHARED_DATA_UPDATED',
            key: message.key,
            value: message.value
          });
        }
      });
    }
  });
  
} else {
  // Worker process
  const setSharedData = (key, value) => {
    process.send({
      type: 'SET_SHARED_DATA',
      key,
      value
    });
  };
  
  process.on('message', (msg) => {
    if (msg.type === 'SHARED_DATA_UPDATED') {
      console.log(`Shared data updated: ${msg.key} = ${msg.value}`);
      // Update local cache or take action
    }
  });
}
```

### 3. Load Balancing and Task Distribution

```js
const cluster = require('cluster');
const http = require('http');

if (cluster.isMaster) {
  const tasks = [];
  const workerStats = new Map();
  
  // Initialize workers
  for (let i = 0; i < 4; i++) {
    const worker = cluster.fork();
    workerStats.set(worker.id, { activeTasks: 0 });
    
    worker.on('message', (message) => {
      if (message.type === 'TASK_COMPLETED') {
        const stats = workerStats.get(worker.id);
        stats.activeTasks--;
        workerStats.set(worker.id, stats);
        
        console.log(`Worker ${worker.id} completed task. Active tasks: ${stats.activeTasks}`);
      }
    });
  }
  
  // Distribute tasks to least busy worker
  const distributeTasks = () => {
    if (tasks.length > 0) {
      let leastBusyWorker = null;
      let minTasks = Infinity;
      
      for (const [workerId, stats] of workerStats) {
        if (stats.activeTasks < minTasks) {
          minTasks = stats.activeTasks;
          leastBusyWorker = cluster.workers[workerId];
        }
      }
      
      if (leastBusyWorker) {
        const task = tasks.shift();
        leastBusyWorker.send({
          type: 'NEW_TASK',
          task: task
        });
        
        const stats = workerStats.get(leastBusyWorker.id);
        stats.activeTasks++;
        workerStats.set(leastBusyWorker.id, stats);
      }
    }
  };
  
  // Add tasks periodically
  setInterval(() => {
    tasks.push({ id: Date.now(), data: 'Some task data' });
    distributeTasks();
  }, 2000);
  
} else {
  // Worker process
  process.on('message', (message) => {
    if (message.type === 'NEW_TASK') {
      console.log(`Worker ${process.pid} processing task:`, message.task.id);
      
      // Simulate task processing
      setTimeout(() => {
        process.send({
          type: 'TASK_COMPLETED',
          taskId: message.task.id
        });
      }, Math.random() * 3000);
    }
  });
}
```

### 4. Health Monitoring and Auto-Recovery

```js
const cluster = require('cluster');

if (cluster.isMaster) {
  const workerHealth = new Map();
  
  cluster.on('fork', (worker) => {
    console.log(`Worker ${worker.process.pid} forked`);
    workerHealth.set(worker.id, { lastHeartbeat: Date.now() });
    
    // Listen for heartbeat messages
    worker.on('message', (message) => {
      if (message.type === 'HEARTBEAT') {
        workerHealth.set(worker.id, { lastHeartbeat: Date.now() });
      }
    });
  });
  
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    workerHealth.delete(worker.id);
    
    // Restart dead worker
    console.log('Starting a new worker');
    cluster.fork();
  });
  
  // Health check every 10 seconds
  setInterval(() => {
    const now = Date.now();
    
    for (const [workerId, health] of workerHealth) {
      if (now - health.lastHeartbeat > 15000) { // 15 seconds timeout
        console.log(`Worker ${workerId} is unresponsive, killing it`);
        cluster.workers[workerId].kill();
      }
    }
  }, 10000);
  
  // Request heartbeat from all workers
  setInterval(() => {
    Object.values(cluster.workers).forEach(worker => {
      worker.send({ type: 'HEARTBEAT_REQUEST' });
    });
  }, 5000);
  
} else {
  // Worker heartbeat
  process.on('message', (message) => {
    if (message.type === 'HEARTBEAT_REQUEST') {
      process.send({
        type: 'HEARTBEAT',
        pid: process.pid,
        timestamp: Date.now()
      });
    }
  });
}
```

### 5. Communication Patterns Summary

| Pattern | Description | Use Case | Example |
|---------|-------------|----------|---------|
| **Master → Worker** | Master sends commands/config to workers | Configuration updates, task assignment | `worker.send(message)` |
| **Worker → Master** | Worker reports status/results to master | Status updates, task completion | `process.send(message)` |
| **Broadcast** | Master sends message to all workers | System-wide notifications | Loop through all workers |
| **Request-Response** | Two-way communication with correlation | Health checks, data queries | Message with correlation ID |
| **Pub-Sub** | Event-driven communication | Real-time updates | Using Redis/EventEmitter |

### 6. External Communication Methods

For more complex scenarios, clusters can communicate through:

```js
// Using Redis for cluster communication
const redis = require('redis');
const publisher = redis.createClient();
const subscriber = redis.createClient();

if (cluster.isMaster) {
  // Master publishes events
  publisher.publish('cluster-events', JSON.stringify({
    type: 'CONFIG_CHANGE',
    data: { newSetting: 'value' }
  }));
  
} else {
  // Workers subscribe to events
  subscriber.subscribe('cluster-events');
  subscriber.on('message', (channel, message) => {
    const event = JSON.parse(message);
    console.log(`Worker ${process.pid} received event:`, event);
  });
}

// Using shared database
const sqlite3 = require('sqlite3');
const db = new sqlite3.Database('shared.db');

// Workers can read/write shared state to database
```

# Types of Authentication in Node.js

## Overview
Authentication is the process of verifying the identity of a user or system. Node.js offers various authentication strategies depending on the application requirements.

| Authentication Type | Stateful | Security Level | Use Case | Storage |
|-------------------|----------|----------------|----------|---------|
| Session-based | Yes | Medium | Traditional web apps | Server memory/database |
| JWT | No | High | APIs, SPAs, microservices | Client-side |
| OAuth | No | Very High | Third-party integration | External provider |
| API Keys | No | Medium | API access | Client headers |
| Multi-Factor (MFA) | Varies | Very High | High-security apps | Multiple methods |

---

## 1. Session-Based Authentication

Uses server-side sessions to maintain user state. The server stores session data and sends a session ID to the client via cookies.

```js
const express = require('express');
const session = require('express-session');
const bcrypt = require('bcrypt');

const app = express();
app.use(express.json());

// Session middleware
app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: { 
    secure: false, // set to true in production with HTTPS
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));

// Mock user database
const users = [
  { id: 1, username: 'john', password: '$2b$10$...' } // hashed password
];

// Login route
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = users.find(u => u.username === username);
  
  if (user && await bcrypt.compare(password, user.password)) {
    req.session.userId = user.id;
    res.json({ message: 'Login successful' });
  } else {
    res.status(401).json({ message: 'Invalid credentials' });
  }
});

// Protected route
app.get('/profile', (req, res) => {
  if (req.session.userId) {
    res.json({ message: 'Welcome to your profile!' });
  } else {
    res.status(401).json({ message: 'Please login first' });
  }
});

// Logout
app.post('/logout', (req, res) => {
  req.session.destroy();
  res.json({ message: 'Logged out successfully' });
});
```

---

## 2. JSON Web Token (JWT) Authentication

Stateless authentication using signed tokens that contain user information. Tokens are stored on the client side.

```js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

const JWT_SECRET = 'your-jwt-secret';

// Login and generate JWT
app.post('/auth/login', async (req, res) => {
  const { username, password } = req.body;
  const user = users.find(u => u.username === username);
  
  if (user && await bcrypt.compare(password, user.password)) {
    const token = jwt.sign(
      { userId: user.id, username: user.username },
      JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    res.json({ 
      message: 'Login successful',
      token,
      user: { id: user.id, username: user.username }
    });
  } else {
    res.status(401).json({ message: 'Invalid credentials' });
  }
});

// JWT Middleware
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
  
  if (!token) {
    return res.status(401).json({ message: 'Access token required' });
  }
  
  jwt.verify(token, JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ message: 'Invalid or expired token' });
    }
    req.user = user;
    next();
  });
};

// Protected route with JWT
app.get('/auth/profile', authenticateToken, (req, res) => {
  res.json({ 
    message: 'Access granted!',
    user: req.user 
  });
});

// Refresh token example
app.post('/auth/refresh', authenticateToken, (req, res) => {
  const newToken = jwt.sign(
    { userId: req.user.userId, username: req.user.username },
    JWT_SECRET,
    { expiresIn: '24h' }
  );
  
  res.json({ token: newToken });
});
```

---

## 3. OAuth Authentication

Allows users to authenticate using third-party providers (Google, Facebook, GitHub, etc.) without sharing passwords.

```js
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

// Configure Google OAuth strategy
passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: "/auth/google/callback"
}, (accessToken, refreshToken, profile, done) => {
  // Save user to database or find existing user
  const user = {
    id: profile.id,
    name: profile.displayName,
    email: profile.emails[0].value,
    provider: 'google'
  };
  return done(null, user);
}));

passport.serializeUser((user, done) => done(null, user));
passport.deserializeUser((user, done) => done(null, user));

app.use(passport.initialize());
app.use(passport.session());

// OAuth routes
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] })
);

app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    // Successful authentication
    res.redirect('/dashboard');
  }
);

// GitHub OAuth example
const GitHubStrategy = require('passport-github2').Strategy;

passport.use(new GitHubStrategy({
  clientID: process.env.GITHUB_CLIENT_ID,
  clientSecret: process.env.GITHUB_CLIENT_SECRET,
  callbackURL: "/auth/github/callback"
}, (accessToken, refreshToken, profile, done) => {
  return done(null, profile);
}));
```

---

## 4. API Key Authentication

Simple authentication method using unique keys for API access. Commonly used for service-to-service communication.

```js
// API Key middleware
const authenticateApiKey = (req, res, next) => {
  const apiKey = req.headers['x-api-key'] || req.query.apiKey;
  
  if (!apiKey) {
    return res.status(401).json({ message: 'API key required' });
  }
  
  // Validate API key (check against database)
  const validApiKeys = ['key123', 'key456']; // In real app, store in database
  
  if (!validApiKeys.includes(apiKey)) {
    return res.status(403).json({ message: 'Invalid API key' });
  }
  
  req.apiKey = apiKey;
  next();
};

// Protected API endpoint
app.get('/api/data', authenticateApiKey, (req, res) => {
  res.json({ 
    message: 'API access granted',
    data: ['item1', 'item2', 'item3'],
    apiKey: req.apiKey
  });
});

// Rate limiting with API keys
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each API key to 100 requests per windowMs
  keyGenerator: (req) => req.apiKey,
  message: 'Too many requests from this API key'
});

app.use('/api/', apiLimiter);
```

---

## 5. Multi-Factor Authentication (MFA)

Adds an extra layer of security by requiring multiple forms of verification.

```js
const speakeasy = require('speakeasy');
const QRCode = require('qrcode');

// Generate TOTP secret for user
app.post('/auth/setup-mfa', authenticateToken, async (req, res) => {
  const secret = speakeasy.generateSecret({
    name: `MyApp (${req.user.username})`,
    issuer: 'MyApp'
  });
  
  // Save secret to user profile in database
  // users[req.user.userId].mfaSecret = secret.base32;
  
  // Generate QR code for user to scan
  const qrCodeUrl = await QRCode.toDataURL(secret.otpauth_url);
  
  res.json({
    secret: secret.base32,
    qrCode: qrCodeUrl,
    manualEntryKey: secret.base32
  });
});

// Verify MFA token
app.post('/auth/verify-mfa', authenticateToken, (req, res) => {
  const { token } = req.body;
  const userSecret = 'user-stored-secret'; // Get from database
  
  const verified = speakeasy.totp.verify({
    secret: userSecret,
    encoding: 'base32',
    token: token,
    window: 2 // Allow some time drift
  });
  
  if (verified) {
    // Generate enhanced JWT with MFA flag
    const enhancedToken = jwt.sign(
      { 
        userId: req.user.userId,
        username: req.user.username,
        mfaVerified: true
      },
      JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    res.json({ 
      message: 'MFA verification successful',
      token: enhancedToken
    });
  } else {
    res.status(401).json({ message: 'Invalid MFA token' });
  }
});

// MFA-protected route
const requireMFA = (req, res, next) => {
  if (!req.user.mfaVerified) {
    return res.status(403).json({ message: 'MFA verification required' });
  }
  next();
};

app.get('/admin/sensitive-data', authenticateToken, requireMFA, (req, res) => {
  res.json({ message: 'Access to sensitive data granted' });
});
```

---

## 6. Biometric Authentication (Advanced)

Modern authentication using fingerprint, face recognition, or other biometric data.

```js
// WebAuthn API for biometric authentication
const { generateRegistrationOptions, verifyRegistrationResponse,
        generateAuthenticationOptions, verifyAuthenticationResponse } = require('@simplewebauthn/server');

// Registration
app.post('/auth/webauthn/register-begin', authenticateToken, async (req, res) => {
  const options = generateRegistrationOptions({
    rpName: 'MyApp',
    rpID: 'localhost',
    userID: req.user.userId,
    userName: req.user.username,
    userDisplayName: req.user.username,
    attestationType: 'none',
  });
  
  // Store challenge in session or cache
  req.session.challenge = options.challenge;
  
  res.json(options);
});

app.post('/auth/webauthn/register-finish', authenticateToken, async (req, res) => {
  const { body } = req;
  
  const verification = await verifyRegistrationResponse({
    response: body,
    expectedChallenge: req.session.challenge,
    expectedOrigin: 'http://localhost:3000',
    expectedRPID: 'localhost',
  });
  
  if (verification.verified) {
    // Save authenticator info to user profile
    res.json({ verified: true });
  } else {
    res.status(400).json({ verified: false });
  }
});
```

---

## Authentication Best Practices

### Security Considerations
1. **Password Hashing**: Always hash passwords using bcrypt or similar
2. **HTTPS**: Use SSL/TLS for all authentication endpoints
3. **Token Expiration**: Set appropriate expiration times for tokens
4. **Rate Limiting**: Implement rate limiting to prevent brute force attacks
5. **Input Validation**: Validate and sanitize all user inputs
6. **Secure Headers**: Use security headers (helmet.js)

### Example Security Setup
```js
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

// Security headers
app.use(helmet());

// Rate limiting for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // limit each IP to 5 requests per windowMs
  message: 'Too many authentication attempts, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/auth', authLimiter);

// Password hashing utility
const hashPassword = async (password) => {
  const saltRounds = 12;
  return await bcrypt.hash(password, saltRounds);
};

// Secure cookie configuration
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production', // HTTPS only in production
    httpOnly: true, // Prevent XSS attacks
    maxAge: 24 * 60 * 60 * 1000, // 24 hours
    sameSite: 'strict' // CSRF protection
  }
}));
```

### Choosing the Right Authentication Method

- **Session-based**: Traditional web applications with server-side rendering
- **JWT**: RESTful APIs, SPAs, microservices, mobile applications
- **OAuth**: When you need third-party authentication or don't want to handle passwords
- **API Keys**: Service-to-service communication, simple API access
- **MFA**: High-security applications, financial services, admin panels
- **Biometric**: Modern web/mobile apps requiring maximum security and user convenience

# Difference Between `import` and `require` Statements

Node.js supports two module systems: **CommonJS** (`require`) and **ES Modules** (`import`). Here are the key differences:

## Overview Comparison

| Feature | `require` (CommonJS) | `import` (ES Modules) |
|---------|---------------------|----------------------|
| **Loading** | Synchronous | Asynchronous |
| **When executed** | Runtime | Compile time (static) |
| **Hoisting** | No | Yes |
| **Tree shaking** | No | Yes |
| **Dynamic imports** | Yes (always) | Yes (with `import()`) |
| **File extension** | `.js` (CommonJS) | `.mjs` or `.js` with `"type": "module"` |
| **Default in Node.js** | Yes (legacy) | Opt-in |
| **Browser support** | No (needs bundler) | Native support |

---

## 1. When to Use Which?

### Use CommonJS (`require`) when:
- Working with legacy Node.js projects
- Need synchronous loading
- Using older packages that don't support ES modules
- Simple scripts and tools

### Use ES Modules (`import`) when:
- Building modern applications
- Need tree shaking and better bundling
- Working with frontend frameworks
- Want static analysis and better tooling
- Need top-level await
- Building libraries for both Node.js and browsers

## Summary

**ES Modules** are the future of JavaScript modules with better static analysis, tree shaking, and browser compatibility. **CommonJS** remains important for legacy support and simpler use cases. Modern Node.js applications should prefer ES modules while maintaining CommonJS compatibility when needed.

---

# Node.js Event Loop Phases

## Overview

The Node.js event loop is the heart of Node.js's asynchronous, non-blocking I/O model. It processes callbacks and executes JavaScript code in a single thread through different phases.

## Event Loop Architecture

The event loop has **6 main phases** that execute in a specific order:

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

---

## 1. Timers Phase

**Purpose**: Executes callbacks scheduled by `setTimeout()` and `setInterval()`.

**What happens**:
- Checks if any timer callbacks are ready to execute
- Executes expired timer callbacks
- Controlled by a min-heap of timers

```js
console.log('Start');

setTimeout(() => {
  console.log('Timer 1 - 0ms');
}, 0);

setTimeout(() => {
  console.log('Timer 2 - 10ms');
}, 10);

console.log('End');

// Output:
// Start
// End
// Timer 1 - 0ms
// Timer 2 - 10ms
```

### Timer Precision Example
```js
const start = Date.now();

setTimeout(() => {
  console.log(`Timer executed after: ${Date.now() - start}ms`);
}, 1);

// Output might be: Timer executed after: 2ms (not exactly 1ms)
// This is because the timer phase has to wait for the event loop cycle
```

---

## 2. Pending Callbacks Phase

**Purpose**: Executes I/O callbacks deferred to the next loop iteration.

**What happens**:
- Executes callbacks for some system operations (like TCP errors)
- Handles callbacks that were deferred from the previous cycle
- Most I/O callbacks are handled in the poll phase, not here

```js
const fs = require('fs');

// This callback will be executed in the poll phase, not pending callbacks
fs.readFile('file.txt', (err, data) => {
  console.log('File read complete');
});

// TCP error callbacks might be executed in pending callbacks phase
const net = require('net');
const client = net.createConnection({ port: 8080 }, () => {
  console.log('Connected to server');
});

client.on('error', (err) => {
  console.log('Connection error:', err.message);
  // This error callback might be executed in pending callbacks phase
});
```

---

## 3. Idle, Prepare Phase

**Purpose**: Internal use only - used by Node.js internally.

**What happens**:
- `idle`: Runs some internal Node.js housekeeping
- `prepare`: Prepares for the poll phase
- Not directly accessible to user code

```js
// No user code examples - this is internal to Node.js
// The libuv library uses this phase for internal operations
```

---

## 4. Poll Phase

**Purpose**: Fetches new I/O events and executes I/O related callbacks.

**What happens**:
- Executes callbacks for most I/O operations (file operations, network requests)
- Polls for new I/O events
- Blocks here if no other phases have callbacks waiting

```js
const fs = require('fs');
const https = require('https');

console.log('Start');

// File I/O - callback executed in poll phase
fs.readFile('package.json', (err, data) => {
  console.log('File read callback');
});

// Network I/O - callback executed in poll phase
https.get('https://api.github.com/users/octocat', (res) => {
  console.log('HTTP request callback');
});

console.log('End');

// Output:
// Start
// End
// File read callback
// HTTP request callback
```

### Poll Phase Behavior
```js
const fs = require('fs');

console.log('Start');

// Poll phase will execute this callback
fs.readFile('file.txt', (err, data) => {
  console.log('File read - poll phase');
  
  // This setTimeout will be executed in the next timers phase
  setTimeout(() => {
    console.log('Timer inside file callback');
  }, 0);
  
  // This will be executed in the next check phase
  setImmediate(() => {
    console.log('Immediate inside file callback');
  });
});

console.log('End');
```

---

## 5. Check Phase

**Purpose**: Executes `setImmediate()` callbacks.

**What happens**:
- Executes callbacks scheduled by `setImmediate()`
- Always executes after the poll phase
- Allows you to execute callbacks immediately after the poll phase

```js
console.log('Start');

setImmediate(() => {
  console.log('Immediate 1');
});

setImmediate(() => {
  console.log('Immediate 2');
});

console.log('End');

// Output:
// Start
// End
// Immediate 1
// Immediate 2
```

### setImmediate vs setTimeout
```js
// Outside of I/O callbacks - order is not guaranteed
setTimeout(() => {
  console.log('Timer');
}, 0);

setImmediate(() => {
  console.log('Immediate');
});

// Output could be either:
// Timer, Immediate OR Immediate, Timer

// Inside I/O callbacks - setImmediate always executes first
const fs = require('fs');

fs.readFile('file.txt', () => {
  setTimeout(() => {
    console.log('Timer inside I/O');
  }, 0);
  
  setImmediate(() => {
    console.log('Immediate inside I/O');
  });
});

// Output (guaranteed):
// Immediate inside I/O
// Timer inside I/O
```

---

## 6. Close Callbacks Phase

**Purpose**: Executes close event callbacks.

**What happens**:
- Executes callbacks for `close` events
- Handles cleanup when sockets or handles are closed abruptly

```js
const net = require('net');

const server = net.createServer((socket) => {
  socket.on('close', () => {
    console.log('Socket closed - close callbacks phase');
  });
});

server.listen(8080, () => {
  console.log('Server started');
  
  // Create a client connection
  const client = net.createConnection({ port: 8080 }, () => {
    console.log('Client connected');
    client.end(); // This will trigger close event
  });
});
```

---

## Process.nextTick() and Microtasks

**Important**: These are not part of the event loop phases but have higher priority.

### Process.nextTick()
```js
console.log('Start');

process.nextTick(() => {
  console.log('NextTick 1');
});

process.nextTick(() => {
  console.log('NextTick 2');
});

setImmediate(() => {
  console.log('Immediate');
});

console.log('End');

// Output:
// Start
// End
// NextTick 1
// NextTick 2
// Immediate
```

### Microtasks (Promises)
```js
console.log('Start');

Promise.resolve().then(() => {
  console.log('Promise 1');
});

process.nextTick(() => {
  console.log('NextTick');
});

Promise.resolve().then(() => {
  console.log('Promise 2');
});

console.log('End');

// Output:
// Start
// End
// NextTick
// Promise 1
// Promise 2
```

### Priority Order
1. **process.nextTick()** callbacks (highest priority)
2. **Microtasks** (Promise callbacks)
3. **Event loop phases** (timers, poll, check, etc.)

---

## Complete Event Loop Example

```js
const fs = require('fs');

console.log('1: Start');

// Microtask
Promise.resolve().then(() => {
  console.log('2: Promise');
});

// NextTick (highest priority)
process.nextTick(() => {
  console.log('3: NextTick');
});

// Timer phase
setTimeout(() => {
  console.log('4: Timer');
}, 0);

// Check phase
setImmediate(() => {
  console.log('5: Immediate');
});

// Poll phase
fs.readFile('package.json', () => {
  console.log('6: File read');
  
  // These will execute in the next event loop iteration
  setTimeout(() => {
    console.log('7: Timer inside I/O');
  }, 0);
  
  setImmediate(() => {
    console.log('8: Immediate inside I/O');
  });
  
  process.nextTick(() => {
    console.log('9: NextTick inside I/O');
  });
});

console.log('10: End');

// Output:
// 1: Start
// 10: End
// 3: NextTick
// 2: Promise
// 4: Timer (or 5: Immediate - order not guaranteed outside I/O)
// 5: Immediate (or 4: Timer)
// 6: File read
// 9: NextTick inside I/O
// 8: Immediate inside I/O
// 7: Timer inside I/O
```

---

## Event Loop Phases Summary

| Phase | Purpose | Examples |
|-------|---------|----------|
| **Timers** | Execute `setTimeout()` and `setInterval()` callbacks | Timer callbacks |
| **Pending Callbacks** | Execute I/O callbacks deferred to next iteration | Some TCP error callbacks |
| **Idle, Prepare** | Internal housekeeping | Internal Node.js operations |
| **Poll** | Fetch new I/O events and execute I/O callbacks | File operations, network requests |
| **Check** | Execute `setImmediate()` callbacks | Immediate callbacks |
| **Close Callbacks** | Execute close event callbacks | Socket close events |

## Key Takeaways

1. **Single-threaded**: JavaScript execution is single-threaded
2. **Non-blocking**: I/O operations don't block the event loop
3. **Phase-based**: Each phase handles specific types of callbacks
4. **Priority**: `process.nextTick()` > Microtasks > Event loop phases
5. **Deterministic**: Inside I/O callbacks, `setImmediate()` always executes before `setTimeout()`

Understanding these phases helps you write more efficient Node.js applications and debug timing-related issues.

---


