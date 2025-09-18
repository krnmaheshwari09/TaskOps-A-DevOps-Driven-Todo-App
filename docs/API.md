# API Documentation

## Table of Contents
1. [API Overview](#api-overview)
2. [Authentication](#authentication)
3. [Base Configuration](#base-configuration)
4. [Task Management Endpoints](#task-management-endpoints)
5. [Health Check Endpoints](#health-check-endpoints)
6. [Error Handling](#error-handling)
7. [Rate Limiting](#rate-limiting)
8. [API Examples](#api-examples)
9. [SDK and Client Libraries](#sdk-and-client-libraries)
10. [API Testing](#api-testing)

## API Overview

The To-Do List API provides RESTful endpoints for managing tasks in a simple and intuitive way. The API follows REST conventions and returns JSON responses for all operations.

### API Version
- **Current Version**: v1
- **Base URL**: `http://localhost:3500/api`
- **Protocol**: HTTP/HTTPS
- **Data Format**: JSON

### Supported Operations
- **Create** new tasks
- **Read** all tasks or specific tasks
- **Update** existing tasks
- **Delete** tasks

## Authentication

### Current Implementation
The current API implementation **does not require authentication**. This makes it suitable for:
- Development and testing environments
- Internal applications
- Demo purposes

### Production Recommendations
For production deployments, consider implementing:
- **JWT (JSON Web Tokens)** for stateless authentication
- **API Keys** for service-to-service communication
- **OAuth 2.0** for third-party integrations

#### Example JWT Implementation
```javascript
// Middleware for future JWT implementation
const jwt = require('jsonwebtoken');

function authenticateToken(req, res, next) {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];

    if (token == null) return res.sendStatus(401);

    jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, user) => {
        if (err) return res.sendStatus(403);
        req.user = user;
        next();
    });
}
```

## Base Configuration

### Server Configuration
```javascript
const express = require("express");
const cors = require("cors");
const app = express();

// Middleware
app.use(express.json());
app.use(cors());

// Port configuration
const port = process.env.PORT || 3500;
```

### CORS Configuration
Cross-Origin Resource Sharing (CORS) is enabled for all origins:
```javascript
app.use(cors());
```

For production, configure CORS with specific origins:
```javascript
app.use(cors({
    origin: ['https://yourdomain.com', 'https://app.yourdomain.com'],
    credentials: true
}));
```

### Content Type
All requests and responses use `application/json` content type.

## Task Management Endpoints

### Base Path
All task-related endpoints are prefixed with `/api/tasks`.

---

### GET /api/tasks

Retrieve all tasks from the database.

#### Request
```http
GET /api/tasks HTTP/1.1
Host: localhost:3500
Accept: application/json
```

#### Response
```http
HTTP/1.1 200 OK
Content-Type: application/json

[
    {
        "_id": "60f7b1b3b3f3b3b3b3b3b3b3",
        "task": "Complete project documentation",
        "completed": false,
        "__v": 0
    },
    {
        "_id": "60f7b1b3b3f3b3b3b3b3b3b4",
        "task": "Review pull requests",
        "completed": true,
        "__v": 0
    }
]
```

#### Response Schema
```json
{
    "type": "array",
    "items": {
        "type": "object",
        "properties": {
            "_id": {
                "type": "string",
                "description": "MongoDB ObjectId"
            },
            "task": {
                "type": "string",
                "description": "Task description"
            },
            "completed": {
                "type": "boolean",
                "description": "Task completion status"
            },
            "__v": {
                "type": "number",
                "description": "MongoDB version key"
            }
        }
    }
}
```

#### Error Responses
- **500 Internal Server Error**: Database connection issues

---

### POST /api/tasks

Create a new task.

#### Request
```http
POST /api/tasks HTTP/1.1
Host: localhost:3500
Content-Type: application/json

{
    "task": "New task description"
}
```

#### Request Schema
```json
{
    "type": "object",
    "properties": {
        "task": {
            "type": "string",
            "required": true,
            "minLength": 1,
            "maxLength": 500,
            "description": "Task description"
        }
    }
}
```

#### Response
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
    "_id": "60f7b1b3b3f3b3b3b3b3b3b5",
    "task": "New task description",
    "completed": false,
    "__v": 0
}
```

#### Error Responses
- **400 Bad Request**: Missing or invalid task field
- **500 Internal Server Error**: Database operation failed

---

### PUT /api/tasks/:id

Update an existing task by ID.

#### Request
```http
PUT /api/tasks/60f7b1b3b3f3b3b3b3b3b3b3 HTTP/1.1
Host: localhost:3500
Content-Type: application/json

{
    "task": "Updated task description",
    "completed": true
}
```

#### URL Parameters
- **id** (required): MongoDB ObjectId of the task to update

#### Request Schema
```json
{
    "type": "object",
    "properties": {
        "task": {
            "type": "string",
            "minLength": 1,
            "maxLength": 500,
            "description": "Updated task description"
        },
        "completed": {
            "type": "boolean",
            "description": "Updated completion status"
        }
    }
}
```

#### Response
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "_id": "60f7b1b3b3f3b3b3b3b3b3b3",
    "task": "Updated task description",
    "completed": true,
    "__v": 0
}
```

#### Error Responses
- **400 Bad Request**: Invalid task ID format
- **404 Not Found**: Task with specified ID not found
- **500 Internal Server Error**: Database operation failed

---

### DELETE /api/tasks/:id

Delete a task by ID.

#### Request
```http
DELETE /api/tasks/60f7b1b3b3f3b3b3b3b3b3b3 HTTP/1.1
Host: localhost:3500
```

#### URL Parameters
- **id** (required): MongoDB ObjectId of the task to delete

#### Response
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "_id": "60f7b1b3b3f3b3b3b3b3b3b3",
    "task": "Deleted task description",
    "completed": false,
    "__v": 0
}
```

#### Error Responses
- **400 Bad Request**: Invalid task ID format
- **404 Not Found**: Task with specified ID not found
- **500 Internal Server Error**: Database operation failed

## Health Check Endpoints

The API provides several health check endpoints for monitoring and load balancer integration.

### GET /healthz

Basic health check endpoint to verify the server is running.

#### Request
```http
GET /healthz HTTP/1.1
Host: localhost:3500
```

#### Response
```http
HTTP/1.1 200 OK
Content-Type: text/plain

Healthy
```

#### Usage
- **Load Balancer**: Health check configuration
- **Monitoring**: Basic server availability
- **Kubernetes**: Liveness probe

---

### GET /ready

Readiness check endpoint that verifies database connectivity.

#### Request
```http
GET /ready HTTP/1.1
Host: localhost:3500
```

#### Response (Ready)
```http
HTTP/1.1 200 OK
Content-Type: text/plain

Ready
```

#### Response (Not Ready)
```http
HTTP/1.1 503 Service Unavailable
Content-Type: text/plain

Not Ready
```

#### Implementation Details
```javascript
app.get('/ready', (req, res) => {
    const isDbConnected = mongoose.connection.readyState === 1;
    
    if (isDbConnected) {
        res.status(200).send('Ready');
    } else {
        res.status(503).send('Not Ready');
    }
});
```

#### Usage
- **Kubernetes**: Readiness probe
- **Load Balancer**: Traffic routing decisions
- **Monitoring**: Service availability

---

### GET /started

Startup check endpoint to confirm successful server initialization.

#### Request
```http
GET /started HTTP/1.1
Host: localhost:3500
```

#### Response
```http
HTTP/1.1 200 OK
Content-Type: text/plain

Started
```

#### Usage
- **Kubernetes**: Startup probe
- **Deployment**: Rollout verification
- **Monitoring**: Initialization confirmation

## Error Handling

### Error Response Format

All error responses follow a consistent format:

```json
{
    "error": {
        "message": "Error description",
        "code": "ERROR_CODE",
        "timestamp": "2023-12-01T12:00:00.000Z",
        "path": "/api/tasks"
    }
}
```

### HTTP Status Codes

| Status Code | Description | Usage |
|-------------|-------------|-------|
| 200 | OK | Successful GET, PUT, DELETE |
| 201 | Created | Successful POST |
| 400 | Bad Request | Invalid request data |
| 404 | Not Found | Resource not found |
| 500 | Internal Server Error | Server-side errors |
| 503 | Service Unavailable | Database connectivity issues |

### Error Categories

#### Client Errors (4xx)
- **400 Bad Request**: Invalid JSON, missing required fields
- **404 Not Found**: Task ID not found in database

#### Server Errors (5xx)
- **500 Internal Server Error**: Database operation failures
- **503 Service Unavailable**: Database connection issues

### Error Handling Implementation

```javascript
// Global error handler
app.use((err, req, res, next) => {
    console.error(err.stack);
    
    res.status(500).json({
        error: {
            message: 'Internal Server Error',
            code: 'INTERNAL_ERROR',
            timestamp: new Date().toISOString(),
            path: req.path
        }
    });
});
```

## Rate Limiting

### Current Implementation
No rate limiting is currently implemented.

### Recommended Implementation
For production environments, implement rate limiting:

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // 100 requests per windowMs
    message: {
        error: {
            message: 'Too many requests',
            code: 'RATE_LIMIT_EXCEEDED'
        }
    }
});

app.use('/api/', limiter);
```

### Rate Limiting Headers
When implemented, the API will return these headers:
- `X-RateLimit-Limit`: Request limit per time window
- `X-RateLimit-Remaining`: Remaining requests in current window
- `X-RateLimit-Reset`: Time when the rate limit resets

## API Examples

### JavaScript/Node.js Client

```javascript
const axios = require('axios');

const API_BASE_URL = 'http://localhost:3500/api';

class TodoAPI {
    // Get all tasks
    async getTasks() {
        try {
            const response = await axios.get(`${API_BASE_URL}/tasks`);
            return response.data;
        } catch (error) {
            console.error('Error fetching tasks:', error);
            throw error;
        }
    }

    // Create a new task
    async createTask(taskDescription) {
        try {
            const response = await axios.post(`${API_BASE_URL}/tasks`, {
                task: taskDescription
            });
            return response.data;
        } catch (error) {
            console.error('Error creating task:', error);
            throw error;
        }
    }

    // Update a task
    async updateTask(taskId, updates) {
        try {
            const response = await axios.put(`${API_BASE_URL}/tasks/${taskId}`, updates);
            return response.data;
        } catch (error) {
            console.error('Error updating task:', error);
            throw error;
        }
    }

    // Delete a task
    async deleteTask(taskId) {
        try {
            const response = await axios.delete(`${API_BASE_URL}/tasks/${taskId}`);
            return response.data;
        } catch (error) {
            console.error('Error deleting task:', error);
            throw error;
        }
    }
}

// Usage example
const api = new TodoAPI();

async function example() {
    // Create a task
    const newTask = await api.createTask('Complete API documentation');
    console.log('Created task:', newTask);

    // Get all tasks
    const tasks = await api.getTasks();
    console.log('All tasks:', tasks);

    // Update the task
    const updatedTask = await api.updateTask(newTask._id, {
        completed: true
    });
    console.log('Updated task:', updatedTask);

    // Delete the task
    const deletedTask = await api.deleteTask(newTask._id);
    console.log('Deleted task:', deletedTask);
}
```

### cURL Examples

#### Create a Task
```bash
curl -X POST http://localhost:3500/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"task": "Learn the To-Do API"}'
```

#### Get All Tasks
```bash
curl -X GET http://localhost:3500/api/tasks \
  -H "Accept: application/json"
```

#### Update a Task
```bash
curl -X PUT http://localhost:3500/api/tasks/60f7b1b3b3f3b3b3b3b3b3b3 \
  -H "Content-Type: application/json" \
  -d '{"task": "Master the To-Do API", "completed": true}'
```

#### Delete a Task
```bash
curl -X DELETE http://localhost:3500/api/tasks/60f7b1b3b3f3b3b3b3b3b3b3
```

#### Health Check
```bash
curl -X GET http://localhost:3500/healthz
curl -X GET http://localhost:3500/ready
curl -X GET http://localhost:3500/started
```

### Python Client

```python
import requests
import json

class TodoAPI:
    def __init__(self, base_url='http://localhost:3500/api'):
        self.base_url = base_url
        self.session = requests.Session()
        self.session.headers.update({'Content-Type': 'application/json'})

    def get_tasks(self):
        """Get all tasks"""
        response = self.session.get(f'{self.base_url}/tasks')
        response.raise_for_status()
        return response.json()

    def create_task(self, task_description):
        """Create a new task"""
        data = {'task': task_description}
        response = self.session.post(f'{self.base_url}/tasks', json=data)
        response.raise_for_status()
        return response.json()

    def update_task(self, task_id, **updates):
        """Update an existing task"""
        response = self.session.put(f'{self.base_url}/tasks/{task_id}', json=updates)
        response.raise_for_status()
        return response.json()

    def delete_task(self, task_id):
        """Delete a task"""
        response = self.session.delete(f'{self.base_url}/tasks/{task_id}')
        response.raise_for_status()
        return response.json()

# Usage example
api = TodoAPI()

# Create a task
new_task = api.create_task('Complete Python integration')
print(f"Created task: {new_task}")

# Get all tasks
tasks = api.get_tasks()
print(f"All tasks: {tasks}")

# Update the task
updated_task = api.update_task(new_task['_id'], completed=True)
print(f"Updated task: {updated_task}")

# Delete the task
deleted_task = api.delete_task(new_task['_id'])
print(f"Deleted task: {deleted_task}")
```

## SDK and Client Libraries

### Frontend Service (React)

The frontend includes a service layer for API communication:

```javascript
// services/taskServices.js
import axios from 'axios';

const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:3500/api';

export const getTasks = () => {
    return axios.get(`${API_URL}/tasks`);
};

export const addTask = (task) => {
    return axios.post(`${API_URL}/tasks`, task);
};

export const updateTask = (id, task) => {
    return axios.put(`${API_URL}/tasks/${id}`, task);
};

export const deleteTask = (id) => {
    return axios.delete(`${API_URL}/tasks/${id}`);
};
```

### Recommended Client Libraries

#### JavaScript/TypeScript
- **axios**: HTTP client with request/response interceptors
- **fetch**: Native browser API
- **superagent**: Feature-rich HTTP client

#### Python
- **requests**: Simple HTTP library
- **httpx**: Async HTTP client
- **aiohttp**: Async HTTP client/server

#### Other Languages
- **Java**: OkHttp, Apache HttpClient
- **C#**: HttpClient, RestSharp
- **Go**: net/http, resty
- **PHP**: Guzzle, cURL

## API Testing

### Unit Tests

```javascript
const request = require('supertest');
const app = require('../index');

describe('Task API', () => {
    test('GET /api/tasks should return all tasks', async () => {
        const response = await request(app)
            .get('/api/tasks')
            .expect(200);
        
        expect(Array.isArray(response.body)).toBe(true);
    });

    test('POST /api/tasks should create a new task', async () => {
        const newTask = { task: 'Test task' };
        
        const response = await request(app)
            .post('/api/tasks')
            .send(newTask)
            .expect(201);
        
        expect(response.body.task).toBe('Test task');
        expect(response.body.completed).toBe(false);
    });
});
```

### Integration Tests

```javascript
const mongoose = require('mongoose');
const request = require('supertest');
const app = require('../index');

describe('Task API Integration', () => {
    beforeEach(async () => {
        await mongoose.connection.db.dropDatabase();
    });

    test('Complete task workflow', async () => {
        // Create task
        const createResponse = await request(app)
            .post('/api/tasks')
            .send({ task: 'Integration test task' })
            .expect(201);

        const taskId = createResponse.body._id;

        // Update task
        await request(app)
            .put(`/api/tasks/${taskId}`)
            .send({ completed: true })
            .expect(200);

        // Verify update
        const getResponse = await request(app)
            .get('/api/tasks')
            .expect(200);

        const updatedTask = getResponse.body.find(t => t._id === taskId);
        expect(updatedTask.completed).toBe(true);

        // Delete task
        await request(app)
            .delete(`/api/tasks/${taskId}`)
            .expect(200);
    });
});
```

### Performance Testing

```bash
# Using Apache Bench (ab)
ab -n 1000 -c 10 http://localhost:3500/api/tasks

# Using curl for load testing
for i in {1..100}; do
    curl -X GET http://localhost:3500/api/tasks &
done
wait
```

### API Documentation Testing

Use tools like:
- **Postman**: Interactive API testing
- **Insomnia**: REST client
- **Newman**: Command-line Postman runner
- **Swagger/OpenAPI**: API documentation and testing

---

This API documentation provides comprehensive information for developers integrating with the To-Do List API, including examples, error handling, and testing strategies.