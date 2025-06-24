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
```js
const { Worker } = require('worker_threads');

new Worker('./heavy-task.js'); // runs in parallel thread

```
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

### Summary
| Feature         | Multithreading (`worker_threads`) | Clustering (`cluster`)           |
| --------------- | --------------------------------- | -------------------------------- |
| Execution model | Threads in same process           | Separate processes               |
| Memory sharing  | Possible                          | Not shared                       |
| Use case        | CPU-bound tasks                   | Scaling web servers              |
| Communication   | Message passing (fast)            | IPC (slower)                     |
| Overhead        | Lower                             | Higher                           |
| Port sharing    | No                                | Yes (can share HTTP server port) |

