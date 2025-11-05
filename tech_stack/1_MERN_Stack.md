# MERN Stack Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [MongoDB](#mongodb)
3. [Express.js](#expressjs)
4. [React](#react)
5. [Node.js](#nodejs)
6. [MERN Architecture](#mern-architecture)
7. [Project Structure](#project-structure)
8. [Best Practices](#best-practices)

---

## Introduction

MERN is a full-stack JavaScript framework comprising:
- **M** - MongoDB (Database)
- **E** - Express.js (Backend Framework)
- **R** - React (Frontend Library)
- **N** - Node.js (Runtime Environment)

### Advantages
- Single language (JavaScript) across the entire stack
- Rapid development with reusable components
- Strong community support
- Excellent for building scalable web applications
- Great for real-time applications

### When to Use MERN
- Single Page Applications (SPAs)
- Real-time applications (chat, notifications)
- Content-driven applications
- Rapid prototyping and MVP development
- Applications requiring real-time data synchronization

---

## MongoDB

### What is MongoDB?
MongoDB is a NoSQL, document-oriented database that stores data in JSON-like documents (BSON format).

### Key Concepts

#### Document
- Basic unit of storage in MongoDB
- Similar to a row in relational databases
- Stored in BSON (Binary JSON) format
```javascript
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}
```

#### Collections
- Group of MongoDB documents
- Similar to tables in relational databases
- Schema is flexible and can vary between documents

#### Database
- Container for collections
- Can have multiple databases in a MongoDB instance

### Advantages of MongoDB
- **Flexible Schema** - No predefined structure required
- **Scalability** - Horizontal scaling with sharding
- **Performance** - Fast for read-heavy operations
- **JSON-like Structure** - Natural fit with JavaScript
- **Aggregation** - Powerful data processing pipeline

### Disadvantages
- **Memory Usage** - Documents stored as BSON consume more space
- **Transactions** - Limited transaction support (multi-document transactions added in v4.0)
- **Joins** - No native JOIN operations, requires aggregation
- **Data Duplication** - Denormalization can lead to data redundancy

### CRUD Operations
```javascript
// Create
db.users.insertOne({ name: "John", age: 30 })

// Read
db.users.findOne({ name: "John" })
db.users.find({ age: { $gt: 25 } })

// Update
db.users.updateOne({ name: "John" }, { $set: { age: 31 } })

// Delete
db.users.deleteOne({ name: "John" })
```

### Common Query Operators
- `$eq` - Equal
- `$ne` - Not equal
- `$gt` - Greater than
- `$lt` - Less than
- `$in` - Value in array
- `$nin` - Value not in array
- `$and` - AND logic
- `$or` - OR logic

### Indexing
Indexes improve query performance by reducing the amount of data MongoDB must examine.

```javascript
db.users.createIndex({ email: 1 })  // Ascending index
db.users.createIndex({ age: -1 })   // Descending index
db.users.createIndex({ email: 1, name: 1 })  // Compound index
```

### Aggregation Pipeline
Process documents through multiple stages to transform data:
```javascript
db.users.aggregate([
  { $match: { age: { $gt: 25 } } },
  { $group: { _id: "$country", count: { $sum: 1 } } },
  { $sort: { count: -1 } }
])
```

---

## Express.js

### What is Express.js?
Express.js is a minimal and flexible Node.js web application framework providing a robust set of features for building APIs and web applications.

### Key Features
- **Routing** - Map URLs to handler functions
- **Middleware** - Execute functions before reaching route handlers
- **Template Engines** - Support for EJS, Pug, etc.
- **Error Handling** - Built-in error handling mechanism
- **Static File Serving** - Serve CSS, images, JavaScript

### Basic Server Setup
```javascript
const express = require('express');
const app = express();

// Middleware
app.use(express.json());
app.use(express.static('public'));

// Routes
app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(5000, () => {
  console.log('Server running on port 5000');
});
```

### Middleware
Functions that have access to request and response objects and the next middleware function.

```javascript
// Custom middleware
const myMiddleware = (req, res, next) => {
  console.log('Request received');
  next(); // Pass control to next middleware/route
};

app.use(myMiddleware);

// Error handling middleware (must have 4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something went wrong!');
});
```

### Routing
```javascript
// Basic route
app.get('/users', (req, res) => {
  res.json({ users: [] });
});

// Route parameters
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.json({ id: userId });
});

// Query parameters
app.get('/search', (req, res) => {
  const query = req.query.q;
  res.json({ query });
});

// POST request
app.post('/users', (req, res) => {
  const userData = req.body;
  res.status(201).json({ created: userData });
});
```

### Response Methods
- `res.send()` - Send response
- `res.json()` - Send JSON response
- `res.sendFile()` - Send a file
- `res.status()` - Set HTTP status code
- `res.redirect()` - Redirect to another URL
- `res.render()` - Render template

### Request Properties
- `req.params` - Route parameters
- `req.query` - Query string parameters
- `req.body` - Request body (needs body parser middleware)
- `req.headers` - HTTP headers
- `req.method` - HTTP method (GET, POST, etc.)

### HTTP Status Codes
- `200` - OK
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `500` - Internal Server Error

---

## React

### What is React?
React is a JavaScript library for building user interfaces using components. It uses a virtual DOM for efficient updates.

### Core Concepts

#### Components
Reusable, independent pieces of UI logic.

```javascript
// Functional Component
const Welcome = ({ name }) => {
  return <h1>Hello, {name}</h1>;
};

// Class Component
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

#### JSX
JavaScript XML - syntax extension allowing you to write HTML in JavaScript.

```javascript
const element = <h1>Hello, World!</h1>;

const element2 = (
  <div>
    <h1>Title</h1>
    <p>Description</p>
  </div>
);
```

#### Props
Read-only data passed from parent to child components.

```javascript
const Greeting = ({ name, age }) => (
  <div>
    <p>Name: {name}</p>
    <p>Age: {age}</p>
  </div>
);

// Usage
<Greeting name="John" age={30} />
```

#### State
Mutable data managed within a component.

```javascript
const [count, setCount] = useState(0);

const increment = () => setCount(count + 1);
```

#### Hooks
Functions that let you use state and other React features in functional components.

**useState** - Manage state
```javascript
const [state, setState] = useState(initialValue);
```

**useEffect** - Side effects (API calls, subscriptions)
```javascript
useEffect(() => {
  // Code runs after render
  return () => {
    // Cleanup code
  };
}, [dependency]); // Dependency array
```

**useContext** - Access context values
```javascript
const value = useContext(MyContext);
```

**useReducer** - Complex state logic
```javascript
const [state, dispatch] = useReducer(reducer, initialState);
```

**Custom Hooks** - Reusable logic
```javascript
const useFetch = (url) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);

  return { data, loading };
};
```

#### Component Lifecycle
- **Mounting** - Component is created and inserted into DOM
- **Updating** - Component is re-rendered
- **Unmounting** - Component is removed from DOM

#### Virtual DOM
React keeps a virtual representation of the DOM in memory and updates only the changed elements for optimal performance.

#### Reconciliation
Process React uses to update the DOM efficiently by comparing old and new elements.

### Forms in React
```javascript
const [formData, setFormData] = useState({ name: '', email: '' });

const handleChange = (e) => {
  const { name, value } = e.target;
  setFormData({ ...formData, [name]: value });
};

const handleSubmit = (e) => {
  e.preventDefault();
  console.log(formData);
};

return (
  <form onSubmit={handleSubmit}>
    <input
      type="text"
      name="name"
      value={formData.name}
      onChange={handleChange}
    />
    <button type="submit">Submit</button>
  </form>
);
```

### List Rendering
```javascript
const items = [
  { id: 1, name: 'Item 1' },
  { id: 2, name: 'Item 2' }
];

return (
  <ul>
    {items.map((item) => (
      <li key={item.id}>{item.name}</li>
    ))}
  </ul>
);
```

**Important** - Always use a unique `key` prop when rendering lists for proper reconciliation.

### Conditional Rendering
```javascript
// Ternary operator
{isLoggedIn ? <Dashboard /> : <Login />}

// Logical AND
{hasError && <ErrorMessage />}

// if-else statement
let component;
if (isLoading) {
  component = <Loader />;
} else {
  component = <Data />;
}
```

### Error Boundaries
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong</h1>;
    }
    return this.props.children;
  }
}
```

---

## Node.js

### What is Node.js?
Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine, allowing you to run JavaScript outside the browser.

### Key Features
- **Event-driven** - Non-blocking I/O model
- **Single-threaded** - Handles multiple concurrent connections efficiently
- **Package Manager** - npm (Node Package Manager)
- **Built-in Modules** - fs, path, http, events, etc.

### npm (Node Package Manager)
```bash
npm init                    # Initialize project
npm install package-name    # Install package
npm install -g package      # Install globally
npm list                    # List installed packages
npm update                  # Update packages
npm uninstall package       # Remove package
npm start                   # Run start script
npm test                    # Run test script
```

### Common Node.js Modules

**fs (File System)**
```javascript
const fs = require('fs');

// Read file
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Write file
fs.writeFile('file.txt', 'content', (err) => {
  if (err) throw err;
  console.log('File written');
});
```

**path**
```javascript
const path = require('path');

path.join('/a', 'b', 'c')  // '/a/b/c'
path.basename('/a/b/file.txt')  // 'file.txt'
path.dirname('/a/b/file.txt')  // '/a/b'
path.extname('/a/b/file.txt')  // '.txt'
```

**events**
```javascript
const EventEmitter = require('events');
const emitter = new EventEmitter();

emitter.on('event', () => {
  console.log('Event fired');
});

emitter.emit('event');
```

### Asynchronous JavaScript
**Callbacks**
```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback(null, 'Data');
  }, 1000);
}

fetchData((err, data) => {
  if (err) console.error(err);
  console.log(data);
});
```

**Promises**
```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('Data');
  }, 1000);
});

promise.then(data => console.log(data))
  .catch(err => console.error(err));
```

**Async/Await**
```javascript
async function fetchData() {
  try {
    const response = await fetch(url);
    const data = await response.json();
    return data;
  } catch (error) {
    console.error(error);
  }
}
```

### Environment Variables
```javascript
// .env file
DATABASE_URL=mongodb://localhost:27017/mydb
PORT=5000

// Access in code
require('dotenv').config();
const dbUrl = process.env.DATABASE_URL;
const port = process.env.PORT;
```

---

## MERN Architecture

### Three-Tier Architecture

```
┌─────────────────────────────────────┐
│      Presentation Layer (React)     │
│  - UI Components                    │
│  - State Management                 │
│  - API Calls                        │
└──────────────┬──────────────────────┘
               │ HTTP/REST
┌──────────────▼──────────────────────┐
│  Business Logic Layer (Express.js)  │
│  - Routes                           │
│  - Controllers                      │
│  - Middleware                       │
│  - Business Logic                   │
└──────────────┬──────────────────────┘
               │ Database Queries
┌──────────────▼──────────────────────┐
│    Data Access Layer (MongoDB)      │
│  - Collections                      │
│  - Indexes                          │
│  - Data Storage                     │
└─────────────────────────────────────┘
```

### API Communication
- React sends HTTP requests to Express.js backend
- Express.js processes requests and queries MongoDB
- MongoDB returns data to Express.js
- Express.js sends JSON responses to React
- React updates UI with received data

### State Management Patterns
**Local Component State**
```javascript
const [data, setData] = useState(null);
```

**Context API** - For prop drilling avoidance
```javascript
const DataContext = createContext();

const DataProvider = ({ children }) => {
  const [data, setData] = useState(null);
  return (
    <DataContext.Provider value={{ data, setData }}>
      {children}
    </DataContext.Provider>
  );
};
```

**Redux** - For complex state management
```javascript
// Actions
const setUser = (user) => ({ type: 'SET_USER', payload: user });

// Reducer
const userReducer = (state = {}, action) => {
  switch(action.type) {
    case 'SET_USER':
      return action.payload;
    default:
      return state;
  }
};
```

---

## Project Structure

### Typical MERN Project Layout
```
mern-app/
├── client/                          # React Frontend
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Header.js
│   │   │   ├── Footer.js
│   │   │   └── Layout.js
│   │   ├── pages/
│   │   │   ├── Home.js
│   │   │   ├── About.js
│   │   │   └── Contact.js
│   │   ├── services/
│   │   │   └── api.js
│   │   ├── context/
│   │   │   └── AuthContext.js
│   │   ├── App.js
│   │   └── index.js
│   ├── package.json
│   └── .env
│
├── server/                          # Express.js Backend
│   ├── models/
│   │   ├── User.js
│   │   └── Post.js
│   ├── controllers/
│   │   ├── userController.js
│   │   └── postController.js
│   ├── routes/
│   │   ├── userRoutes.js
│   │   └── postRoutes.js
│   ├── middleware/
│   │   ├── authMiddleware.js
│   │   └── errorHandler.js
│   ├── config/
│   │   └── database.js
│   ├── server.js
│   ├── package.json
│   └── .env
│
└── .gitignore
```

### Model-View-Controller (MVC) Pattern

**Model** - Data structure and MongoDB schema
```javascript
// models/User.js
const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  password: String
});

module.exports = mongoose.model('User', userSchema);
```

**View** - React components that display data
```javascript
// React component
const UserProfile = ({ user }) => (
  <div>
    <h1>{user.name}</h1>
    <p>{user.email}</p>
  </div>
);
```

**Controller** - Business logic that handles requests
```javascript
// controllers/userController.js
exports.getUser = async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json(user);
};

exports.createUser = async (req, res) => {
  const user = new User(req.body);
  await user.save();
  res.status(201).json(user);
};
```

---

## Best Practices

### Backend Best Practices

1. **Environment Variables** - Store sensitive data in .env files
2. **Error Handling** - Use try-catch blocks and error middleware
3. **Validation** - Validate user input (use libraries like joi, validator.js)
4. **Authentication** - Use JWT tokens for secure authentication
5. **Database Indexing** - Index frequently queried fields
6. **Logging** - Log important events for debugging
7. **CORS** - Configure CORS properly for frontend requests
8. **Security Headers** - Use helmet.js for security headers
9. **Rate Limiting** - Prevent abuse with express-rate-limit
10. **Code Organization** - Follow MVC pattern for clean code

### Frontend Best Practices

1. **Component Reusability** - Create small, reusable components
2. **State Management** - Use Context API or Redux for global state
3. **Lazy Loading** - Code-split and lazy load components
4. **Error Boundaries** - Handle component errors gracefully
5. **Performance** - Use React.memo, useMemo for optimization
6. **Accessibility** - Use semantic HTML and ARIA labels
7. **Testing** - Write unit tests with Jest and React Testing Library
8. **Environmental Variables** - Use .env files for configuration
9. **Code Splitting** - Use dynamic imports for better performance
10. **Responsive Design** - Use CSS media queries or frameworks like Tailwind

### Common Libraries

**Backend**
- **mongoose** - MongoDB object modeling
- **jsonwebtoken** - JWT authentication
- **bcryptjs** - Password hashing
- **cors** - Cross-origin resource sharing
- **dotenv** - Environment variables
- **validator.js** - Input validation

**Frontend**
- **axios** - HTTP client
- **react-router-dom** - Client-side routing
- **react-query** - Data fetching and caching
- **zustand/Redux** - State management
- **tailwindcss/Material-UI** - Styling
- **jwt-decode** - Decode JWT tokens

### Security Considerations

1. **Never hardcode secrets** - Use environment variables
2. **Validate all inputs** - Prevent injection attacks
3. **Use HTTPS** - Encrypt data in transit
4. **Hash passwords** - Use bcrypt or similar
5. **Implement CORS properly** - Restrict allowed origins
6. **Use parameterized queries** - Prevent SQL injection (N/A for MongoDB but use schema validation)
7. **Implement rate limiting** - Prevent brute force attacks
8. **Sanitize output** - Prevent XSS attacks
9. **Keep dependencies updated** - Security patches
10. **Implement proper authentication** - Use JWT with refresh tokens

---

## Summary

The MERN stack combines four powerful technologies:
- **MongoDB** for flexible data storage
- **Express.js** for robust backend development
- **React** for interactive user interfaces
- **Node.js** for server-side JavaScript execution

This stack enables full-stack JavaScript development with excellent scalability, performance, and developer experience.
