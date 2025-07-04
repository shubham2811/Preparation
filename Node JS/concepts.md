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

