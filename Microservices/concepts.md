# Design Patterns in Microservices

## Overview

Microservices design patterns are proven solutions to common problems in distributed systems. They help address challenges like service communication, data management, reliability, and scalability in microservices architectures.

## Categories of Microservices Patterns

| Category | Purpose | Common Patterns |
|----------|---------|----------------|
| **Decomposition** | Breaking down monoliths | Database per Service, Strangler Fig |
| **Communication** | Service-to-service interaction | API Gateway, Service Mesh |
| **Data Management** | Handling distributed data | Saga, CQRS, Event Sourcing |
| **Reliability** | Fault tolerance and resilience | Circuit Breaker, Bulkhead, Retry |
| **Observability** | Monitoring and debugging | Distributed Tracing, Health Check |
| **Deployment** | Service deployment strategies | Blue-Green, Canary, Rolling |
| **Security** | Authentication and authorization | JWT, OAuth, API Keys |

---

## 1. Decomposition Patterns

### Database per Service Pattern

Each microservice has its own database to ensure loose coupling and independent deployments.

### Strangler Fig Pattern

Gradually replace legacy systems by routing traffic to new microservices.

---

## 2. Communication Patterns

### API Gateway Pattern

Single entry point for all client requests, handling routing, authentication, and cross-cutting concerns.

### Service Mesh Pattern

Infrastructure layer that handles service-to-service communication.

---

## 3. Data Management Patterns

### Saga Pattern

Manages distributed transactions across multiple services.

### CQRS (Command Query Responsibility Segregation)

Separates read and write operations for better scalability.

```javascript
// command-service.js (Write side)
const express = require('express');
const EventEmitter = require('events');

class CommandHandler extends EventEmitter {
  constructor() {
    super();
    this.writeDatabase = new Map(); // Simulated write database
  }

  async createUser(command) {
    const { id, email, name } = command;
    
    // Validate command
    if (!email || !name) {
      throw new Error('Email and name are required');
    }

    // Store in write database
    const user = { id, email, name, version: 1, createdAt: new Date() };
    this.writeDatabase.set(id, user);

    // Emit event for read side
    this.emit('UserCreated', {
      eventId: 'evt-' + Date.now(),
      aggregateId: id,
      eventType: 'UserCreated',
      data: user,
      timestamp: new Date()
    });

    return user;
  }

  async updateUser(command) {
    const { id, email, name } = command;
    const user = this.writeDatabase.get(id);

    if (!user) {
      throw new Error('User not found');
    }

    const updatedUser = { ...user, email, name, version: user.version + 1 };
    this.writeDatabase.set(id, updatedUser);

    this.emit('UserUpdated', {
      eventId: 'evt-' + Date.now(),
      aggregateId: id,
      eventType: 'UserUpdated',
      data: updatedUser,
      timestamp: new Date()
    });

    return updatedUser;
  }
}

const commandHandler = new CommandHandler();
const app = express();
app.use(express.json());

// Command endpoints
app.post('/users', async (req, res) => {
  try {
    const user = await commandHandler.createUser(req.body);
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.put('/users/:id', async (req, res) => {
  try {
    const user = await commandHandler.updateUser({ ...req.body, id: req.params.id });
    res.json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.listen(3001, () => {
  console.log('Command service running on port 3001');
});

module.exports = commandHandler;
```

```javascript
// query-service.js (Read side)
const express = require('express');
const commandHandler = require('./command-service');

class QueryHandler {
  constructor() {
    this.readDatabase = new Map(); // Simulated read database (could be different DB)
    this.userViews = new Map(); // Materialized views
    
    // Listen to events from command side
    this.setupEventHandlers();
  }

  setupEventHandlers() {
    commandHandler.on('UserCreated', (event) => {
      this.handleUserCreated(event);
    });

    commandHandler.on('UserUpdated', (event) => {
      this.handleUserUpdated(event);
    });
  }

  handleUserCreated(event) {
    const { aggregateId, data } = event;
    
    // Store in read database
    this.readDatabase.set(aggregateId, data);
    
    // Create materialized view
    this.userViews.set(aggregateId, {
      id: aggregateId,
      email: data.email,
      name: data.name,
      displayName: `${data.name} <${data.email}>`,
      memberSince: data.createdAt,
      searchText: `${data.name} ${data.email}`.toLowerCase()
    });

    console.log(`User view created: ${aggregateId}`);
  }

  handleUserUpdated(event) {
    const { aggregateId, data } = event;
    
    // Update read database
    this.readDatabase.set(aggregateId, data);
    
    // Update materialized view
    const view = this.userViews.get(aggregateId);
    if (view) {
      this.userViews.set(aggregateId, {
        ...view,
        email: data.email,
        name: data.name,
        displayName: `${data.name} <${data.email}>`,
        searchText: `${data.name} ${data.email}`.toLowerCase()
      });
    }

    console.log(`User view updated: ${aggregateId}`);
  }

  getUser(id) {
    return this.readDatabase.get(id);
  }

  getUserView(id) {
    return this.userViews.get(id);
  }

  searchUsers(query) {
    const results = [];
    const searchTerm = query.toLowerCase();
    
    for (const [id, view] of this.userViews) {
      if (view.searchText.includes(searchTerm)) {
        results.push(view);
      }
    }
    
    return results;
  }

  getAllUsers() {
    return Array.from(this.readDatabase.values());
  }
}

const queryHandler = new QueryHandler();
const app = express();

// Query endpoints
app.get('/users', (req, res) => {
  const users = queryHandler.getAllUsers();
  res.json(users);
});

app.get('/users/:id', (req, res) => {
  const user = queryHandler.getUser(req.params.id);
  if (user) {
    res.json(user);
  } else {
    res.status(404).json({ error: 'User not found' });
  }
});

app.get('/users/:id/view', (req, res) => {
  const userView = queryHandler.getUserView(req.params.id);
  if (userView) {
    res.json(userView);
  } else {
    res.status(404).json({ error: 'User view not found' });
  }
});

app.get('/search/users', (req, res) => {
  const { q } = req.query;
  if (!q) {
    return res.status(400).json({ error: 'Query parameter "q" is required' });
  }
  
  const results = queryHandler.searchUsers(q);
  res.json(results);
});

app.listen(3002, () => {
  console.log('Query service running on port 3002');
});
```

---

## 4. Reliability Patterns

### Circuit Breaker Pattern

Prevents cascading failures by stopping calls to failing services.

### Bulkhead Pattern

Isolates resources to prevent failures from spreading.

---

## 5. Observability Patterns

### Distributed Tracing Pattern

Tracks requests across multiple services.

### Health Check Pattern

Monitors service health and availability.

---

## Summary

Microservices design patterns help address common challenges:

### **Key Pattern Categories:**

1. **Decomposition**: Break down monoliths (Database per Service, Strangler Fig)
2. **Communication**: Service interaction (API Gateway, Service Mesh)
3. **Data Management**: Distributed data (Saga, CQRS, Event Sourcing)
4. **Reliability**: Fault tolerance (Circuit Breaker, Bulkhead, Retry)
5. **Observability**: Monitoring (Distributed Tracing, Health Checks)

### **Pattern Selection Guidelines:**

- **Start Simple**: Begin with basic patterns like API Gateway and Database per Service
- **Add Complexity Gradually**: Implement advanced patterns as needs arise
- **Consider Trade-offs**: Each pattern adds complexity but solves specific problems
- **Monitor and Measure**: Use observability patterns to understand system behavior

### **Best Practices:**

- **Combine Patterns**: Use multiple patterns together for robust solutions
- **Automate Implementation**: Use tools and frameworks to implement patterns
- **Document Decisions**: Keep track of why specific patterns were chosen
- **Evolve Gradually**: Refactor and improve patterns as the system matures

These patterns provide proven solutions to common microservices challenges and help build scalable, resilient distributed systems.


# Service Discovery in Microservices

## What is Service Discovery?

Service Discovery is a mechanism that allows microservices to find and communicate with each other without hardcoding network locations. In a microservices architecture, services are distributed across multiple hosts and can dynamically scale up/down, making static configuration impractical.

## Why Service Discovery is Needed

### Problems Without Service Discovery:
1. **Dynamic IP Addresses**: Containers get new IPs when restarted
2. **Auto-scaling**: Services can spin up/down automatically
3. **Load Balancing**: Need to distribute requests across multiple instances
4. **Failure Handling**: Services may become unavailable and recover
5. **Environment Differences**: Different configurations for dev/staging/prod

## Types of Service Discovery

| Type | Description | Examples | Pros | Cons |
|------|-------------|----------|------|------|
| **Client-Side** | Client directly queries service registry | Eureka, Consul | Simple, no proxy overhead | Client complexity, language-specific |
| **Server-Side** | Load balancer queries service registry | AWS ALB, Nginx Plus | Language agnostic, centralized | Single point of failure, latency |
| **Service Mesh** | Infrastructure layer handles discovery | Istio, Linkerd | Advanced features, observability | Complex setup, overhead |

---

## 1. Client-Side Service Discovery

The client is responsible for determining network locations and load balancing.

### Using Consul for Service Discovery

```js
// consul-client.js
const consul = require('consul')({ host: 'localhost', port: 8500 });

class ServiceDiscovery {
  constructor() {
    this.consul = consul;
    this.services = new Map();
  }

  // Register a service
  async registerService(serviceName, serviceId, port, health) {
    const serviceConfig = {
      id: serviceId,
      name: serviceName,
      address: 'localhost',
      port: port,
      check: {
        http: `http://localhost:${port}${health}`,
        interval: '10s',
        timeout: '3s'
      }
    };

    try {
      await this.consul.agent.service.register(serviceConfig);
      console.log(`Service ${serviceName} registered with ID: ${serviceId}`);
    } catch (error) {
      console.error('Service registration failed:', error);
    }
  }

  // Discover services
  async discoverService(serviceName) {
    try {
      const services = await this.consul.health.service({
        service: serviceName,
        passing: true // Only healthy services
      });

      const instances = services[1].map(service => ({
        id: service.Service.ID,
        address: service.Service.Address,
        port: service.Service.Port,
        health: service.Checks.every(check => check.Status === 'passing')
      }));

      this.services.set(serviceName, instances);
      return instances;
    } catch (error) {
      console.error('Service discovery failed:', error);
      return [];
    }
  }

  // Get service instance with load balancing
  getServiceInstance(serviceName) {
    const instances = this.services.get(serviceName) || [];
    if (instances.length === 0) return null;

    // Simple round-robin load balancing
    const index = Math.floor(Math.random() * instances.length);
    return instances[index];
  }

  // Deregister service
  async deregisterService(serviceId) {
    try {
      await this.consul.agent.service.deregister(serviceId);
      console.log(`Service ${serviceId} deregistered`);
    } catch (error) {
      console.error('Service deregistration failed:', error);
    }
  }
}

module.exports = ServiceDiscovery;
```

### User Service Example

```js
// user-service.js
const express = require('express');
const axios = require('axios');
const ServiceDiscovery = require('./consul-client');

const app = express();
const PORT = process.env.PORT || 3001;
const SERVICE_ID = `user-service-${PORT}`;

const discovery = new ServiceDiscovery();

app.use(express.json());

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy', service: 'user-service' });
});

// User endpoints
app.get('/users', (req, res) => {
  res.json([
    { id: 1, name: 'John Doe', email: 'john@example.com' },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
  ]);
});

app.get('/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const user = { id: userId, name: `User ${userId}`, email: `user${userId}@example.com` };
  res.json(user);
});

// Call another service
app.get('/users/:id/orders', async (req, res) => {
  try {
    // Discover order service
    const orderInstances = await discovery.discoverService('order-service');
    const orderInstance = discovery.getServiceInstance('order-service');
    
    if (!orderInstance) {
      return res.status(503).json({ error: 'Order service unavailable' });
    }

    const response = await axios.get(
      `http://${orderInstance.address}:${orderInstance.port}/orders/user/${req.params.id}`
    );

    res.json(response.data);
  } catch (error) {
    console.error('Error calling order service:', error.message);
    res.status(500).json({ error: 'Failed to fetch user orders' });
  }
});

// Graceful shutdown
process.on('SIGINT', async () => {
  console.log('Shutting down user service...');
  await discovery.deregisterService(SERVICE_ID);
  process.exit(0);
});

app.listen(PORT, async () => {
  console.log(`User service running on port ${PORT}`);
  
  // Register service with Consul
  await discovery.registerService('user-service', SERVICE_ID, PORT, '/health');
  
  // Start service discovery for dependencies
  setInterval(async () => {
    await discovery.discoverService('order-service');
  }, 30000); // Update every 30 seconds
});
```

### Order Service Example

```js
// order-service.js
const express = require('express');
const ServiceDiscovery = require('./consul-client');

const app = express();
const PORT = process.env.PORT || 3002;
const SERVICE_ID = `order-service-${PORT}`;

const discovery = new ServiceDiscovery();

app.use(express.json());

// Mock orders data
const orders = {
  1: [
    { id: 101, userId: 1, product: 'Laptop', amount: 999.99 },
    { id: 102, userId: 1, product: 'Mouse', amount: 29.99 }
  ],
  2: [
    { id: 103, userId: 2, product: 'Keyboard', amount: 79.99 }
  ]
};

app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy', service: 'order-service' });
});

app.get('/orders/user/:userId', (req, res) => {
  const userId = parseInt(req.params.userId);
  const userOrders = orders[userId] || [];
  res.json(userOrders);
});

app.get('/orders/:id', (req, res) => {
  const orderId = parseInt(req.params.id);
  // Find order across all users
  for (const userId in orders) {
    const order = orders[userId].find(o => o.id === orderId);
    if (order) {
      return res.json(order);
    }
  }
  res.status(404).json({ error: 'Order not found' });
});

process.on('SIGINT', async () => {
  console.log('Shutting down order service...');
  await discovery.deregisterService(SERVICE_ID);
  process.exit(0);
});

app.listen(PORT, async () => {
  console.log(`Order service running on port ${PORT}`);
  await discovery.registerService('order-service', SERVICE_ID, PORT, '/health');
});
```

---

## 2. Server-Side Service Discovery

Load balancer handles service discovery and routing.

### Using NGINX with Consul Template

```nginx
# nginx.conf.template
upstream user-service {
  {{range service "user-service"}}
  server {{.Address}}:{{.Port}};
  {{end}}
}

upstream order-service {
  {{range service "order-service"}}
  server {{.Address}}:{{.Port}};
  {{end}}
}

server {
  listen 80;
  
  location /api/users {
    proxy_pass http://user-service;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  location /api/orders {
    proxy_pass http://order-service;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
  }
}
```

### API Gateway with Service Discovery

```js
// api-gateway.js
const express = require('express');
const httpProxy = require('http-proxy-middleware');
const ServiceDiscovery = require('./consul-client');

const app = express();
const PORT = 3000;
const discovery = new ServiceDiscovery();

// Service routing configuration
const serviceRoutes = {
  '/api/users': 'user-service',
  '/api/orders': 'order-service',
  '/api/products': 'product-service'
};

// Dynamic proxy middleware
const createProxyMiddleware = (serviceName) => {
  return httpProxy({
    target: 'http://localhost', // Will be overridden
    changeOrigin: true,
    router: async (req) => {
      // Discover service instance
      const instances = await discovery.discoverService(serviceName);
      const instance = discovery.getServiceInstance(serviceName);
      
      if (!instance) {
        throw new Error(`Service ${serviceName} not available`);
      }
      
      return `http://${instance.address}:${instance.port}`;
    },
    onError: (err, req, res) => {
      console.error(`Proxy error for ${serviceName}:`, err.message);
      res.status(503).json({ error: 'Service unavailable' });
    },
    pathRewrite: {
      '^/api': '' // Remove /api prefix when forwarding
    }
  });
};

// Register routes with service discovery
Object.entries(serviceRoutes).forEach(([route, serviceName]) => {
  app.use(route, createProxyMiddleware(serviceName));
});

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', service: 'api-gateway' });
});

// Service discovery monitoring
setInterval(async () => {
  for (const serviceName of Object.values(serviceRoutes)) {
    await discovery.discoverService(serviceName);
  }
}, 10000); // Update every 10 seconds

app.listen(PORT, () => {
  console.log(`API Gateway running on port ${PORT}`);
});
```

---

## 3. Service Mesh Discovery (Istio)

Infrastructure layer handles service discovery automatically.

## Summary

Service Discovery is essential for microservices architecture, enabling:

1. **Dynamic Service Location**: Services can find each other without hardcoded addresses
2. **Load Balancing**: Distribute traffic across multiple service instances
3. **Fault Tolerance**: Handle service failures gracefully
4. **Auto-scaling**: Support dynamic scaling of services
5. **Environment Flexibility**: Easy deployment across different environments

Choose the right service discovery pattern based on your architecture:
- **Client-side**: Simple setup, good for small-medium applications
- **Server-side**: Better for complex routing, language agnostic
- **Service Mesh**: Advanced features, suitable for large-scale deployments


# Anticorruption Layer Pattern in Microservices

## What is an Anticorruption Layer?

An Anticorruption Layer (ACL) is a design pattern that creates an isolating layer between different bounded contexts or external systems. It prevents the domain model of one service from being corrupted by the concepts, terminology, or data structures of another system.

## Why Use Anticorruption Layer?

### Problems It Solves:
1. **Legacy System Integration**: Protect new services from legacy system complexity
2. **Third-party APIs**: Isolate external API changes from your domain model
3. **Different Domain Models**: Prevent model pollution between bounded contexts
4. **Data Format Differences**: Handle incompatible data structures and protocols
5. **Vendor Lock-in**: Reduce dependency on specific external systems

### Benefits:
- **Domain Purity**: Keep your domain model clean and focused
- **Flexibility**: Easy to change external integrations without affecting core logic
- **Testability**: Mock external dependencies easily
- **Maintainability**: Centralized translation logic
- **Resilience**: Isolate failures from external systems

## Summary

The Anticorruption Layer pattern is essential for:

### **Key Benefits:**
1. **Domain Protection**: Keep your domain model clean and focused
2. **Integration Flexibility**: Easy to change external integrations
3. **System Evolution**: Gradually migrate from legacy systems
4. **Fault Isolation**: Prevent external failures from corrupting your system
5. **Testing**: Mock external dependencies easily

### **Implementation Strategies:**
- **Adapter Pattern**: For single external system integration
- **Facade Pattern**: For multiple external systems
- **Event-Driven**: For asynchronous integration
- **Versioned ACL**: For handling API version changes

### **Best Practices:**
- Keep ACL focused on translation only
- Use dependency injection for testability
- Handle errors gracefully with fallbacks
- Version your translations for API changes
- Comprehensive testing with mocks

The Anticorruption Layer ensures your microservices remain maintainable and independent while integrating with complex external systems.

---
