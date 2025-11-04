# E-Commerce System Design: In-Depth Guide

## Table of Contents
1. [Problem Statement & Requirements](#problem-statement--requirements)
2. [Capacity Estimation](#capacity-estimation)
3. [High-Level Architecture](#high-level-architecture)
4. [Database Design](#database-design)
5. [API Design](#api-design)
6. [Microservices Deep Dive](#microservices-deep-dive)
7. [Caching Strategy](#caching-strategy)
8. [Search & Indexing](#search--indexing)
9. [Payment Processing](#payment-processing)
10. [Real-Time Inventory](#real-time-inventory)
11. [Notifications & Messaging](#notifications--messaging)
12. [Scaling & Optimization](#scaling--optimization)
13. [Reliability & Fault Tolerance](#reliability--fault-tolerance)
14. [Monitoring & Logging](#monitoring--logging)
15. [Security](#security)
16. [Complete System Diagram](#complete-system-diagram)

---

## Problem Statement & Requirements

### Functional Requirements

What must the system do?

**For Customers:**
- Browse products with filters/search
- Add items to cart
- Checkout and place orders
- View order history
- Track shipments
- Review and rate products
- Wishlist functionality
- User authentication

**For Sellers:**
- Upload and manage products
- View inventory
- Manage prices
- View sales analytics
- Process refunds
- Respond to reviews

**For Admin:**
- Manage users
- Monitor fraud
- Analytics dashboard
- Manage categories

---

### Non-Functional Requirements

**Performance:**
- Page load: < 200ms
- Product search: < 500ms
- Checkout: < 2s

**Scale:**
- 10 million daily active users (DAU)
- 100+ million products
- 1 billion requests per day
- Peak traffic: 3x average (during sales)

**Availability:**
- 99.99% uptime (4 minutes downtime/year)
- System stays operational even if some services fail

**Consistency:**
- Order data: Strong consistency (critical)
- Product catalog: Eventually consistent (non-critical)
- User profiles: Eventually consistent

**Regions:**
- Global deployment (US, EU, Asia)
- Local inventory for fast delivery

---

## Capacity Estimation

### Calculate QPS (Queries Per Second)

```
Daily Active Users (DAU) = 10,000,000

Requests per user per day:
- Browse: 20 requests
- Search: 2 requests
- Cart: 2 requests
- Checkout: 1 request
Total: ~25 requests per user per day

Total requests per day = 10M × 25 = 250M requests

Requests per second (average) = 250M / 86,400 seconds = ~2,900 RPS

Peak traffic (3x average) = 2,900 × 3 = ~8,700 RPS

Remember: Not all are equal
- 80% read (browse, search)
- 20% write (cart, order)

Read QPS = 8,700 × 0.8 = ~7,000 RPS
Write QPS = 8,700 × 0.2 = ~1,700 RPS
```

### Storage Estimation

```
Products = 100M
Per product: ~2KB (name, description, price, images URL)
Total: 100M × 2KB = 200 GB

Product Images (external storage/CDN)
Average 5 images per product: 100M × 5 × 500KB = 250 TB

Orders (5 years retention)
10M DAU × 50 orders/user/year = 500M orders
Per order: 1KB
Total: 500GB

User Data = 10M × 1KB = 10 GB

Total Database Size: ~1-2 TB
Total Storage (with replicas): ~5-10 TB
```

---

## High-Level Architecture

### Complete Flow

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT TIER                           │
│  Web Browser  │  Mobile App  │  Admin Dashboard         │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────────┐
│              API GATEWAY / Load Balancer                │
│  (Rate limiting, Authentication, Routing)              │
└──────────────────────────┬───────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ↓                  ↓                  ↓
┌───────────────────────────────────────────────────────────┐
│              MICROSERVICES LAYER                         │
├───────────────────────────────────────────────────────────┤
│                                                           │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │
│ │   Product    │ │    Cart      │ │   Order      │      │
│ │   Service    │ │   Service    │ │   Service    │      │
│ └──────────────┘ └──────────────┘ └──────────────┘      │
│                                                           │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │
│ │   Payment    │ │ Inventory    │ │   User       │      │
│ │   Service    │ │   Service    │ │   Service    │      │
│ └──────────────┘ └──────────────┘ └──────────────┘      │
│                                                           │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │
│ │ Notification │ │   Shipping   │ │  Review &    │      │
│ │   Service    │ │   Service    │ │   Rating     │      │
│ └──────────────┘ └──────────────┘ └──────────────┘      │
│                                                           │
└───────────────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ↓                  ↓                  ↓
┌───────────────────────────────────────────────────────────┐
│              CACHING & MESSAGE LAYER                     │
│  Cache (Redis)  │  Message Queue (Kafka)                │
└───────────────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ↓                  ↓                  ↓
┌───────────────────────────────────────────────────────────┐
│              DATA PERSISTENCE LAYER                      │
├───────────────────────────────────────────────────────────┤
│  SQL Database  │  NoSQL Database  │  Search Engine      │
│ (PostgreSQL)   │  (MongoDB)       │ (Elasticsearch)     │
│ - Users        │  - Product       │ - Full-text search  │
│ - Orders       │    catalog       │ - Analytics         │
│ - Transactions │  - Reviews       │                     │
│                │  - User activity │                     │
└───────────────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ↓                  ↓                  ↓
┌───────────────────────────────────────────────────────────┐
│              EXTERNAL SERVICES & STORAGE                 │
│  Payment Gateway  │  Image Storage  │  Analytics       │
│  (Stripe/PayPal)  │  (S3/CDN)       │  (BigQuery)      │
└───────────────────────────────────────────────────────────┘
```

---

## Database Design

### 1. Product Service Database (SQL: PostgreSQL)

```sql
-- Users Table
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    INDEX (email)
);

-- User Addresses
CREATE TABLE user_addresses (
    address_id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    address_line_1 VARCHAR(255),
    address_line_2 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(50),
    postal_code VARCHAR(20),
    country VARCHAR(50),
    is_default BOOLEAN,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    INDEX (user_id)
);

-- Categories
CREATE TABLE categories (
    category_id BIGINT PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    parent_category_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (parent_category_id) REFERENCES categories(category_id),
    INDEX (parent_category_id)
);

-- Products
CREATE TABLE products (
    product_id BIGINT PRIMARY KEY,
    seller_id BIGINT NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category_id BIGINT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    rating DECIMAL(3, 2),
    review_count INT DEFAULT 0,
    FOREIGN KEY (category_id) REFERENCES categories(category_id),
    INDEX (seller_id),
    INDEX (category_id),
    INDEX (created_at),
    FULLTEXT INDEX (name, description)
);

-- Product Images
CREATE TABLE product_images (
    image_id BIGINT PRIMARY KEY,
    product_id BIGINT NOT NULL,
    image_url VARCHAR(500),
    display_order INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    INDEX (product_id)
);

-- Product Attributes (size, color, etc.)
CREATE TABLE product_attributes (
    attribute_id BIGINT PRIMARY KEY,
    product_id BIGINT NOT NULL,
    attribute_name VARCHAR(100),
    attribute_value VARCHAR(255),
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    INDEX (product_id)
);
```

### 2. Order & Payment Database (SQL: PostgreSQL)

```sql
-- Cart Items
CREATE TABLE cart_items (
    cart_item_id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    UNIQUE KEY unique_user_product (user_id, product_id),
    INDEX (user_id)
);

-- Orders
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    total_amount DECIMAL(12, 2) NOT NULL,
    currency VARCHAR(3),
    status ENUM('PENDING', 'CONFIRMED', 'SHIPPED', 'DELIVERED', 'CANCELLED') DEFAULT 'PENDING',
    payment_status ENUM('PENDING', 'COMPLETED', 'FAILED', 'REFUNDED'),
    shipping_address_id BIGINT NOT NULL,
    billing_address_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (shipping_address_id) REFERENCES user_addresses(address_id),
    FOREIGN KEY (billing_address_id) REFERENCES user_addresses(address_id),
    INDEX (user_id),
    INDEX (order_number),
    INDEX (status),
    INDEX (created_at)
);

-- Order Items (line items in an order)
CREATE TABLE order_items (
    order_item_id BIGINT PRIMARY KEY,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    seller_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    total_price DECIMAL(12, 2) NOT NULL,
    status ENUM('PENDING', 'CONFIRMED', 'SHIPPED', 'DELIVERED', 'CANCELLED') DEFAULT 'PENDING',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    INDEX (order_id),
    INDEX (seller_id)
);

-- Payments
CREATE TABLE payments (
    payment_id BIGINT PRIMARY KEY,
    order_id BIGINT NOT NULL,
    amount DECIMAL(12, 2) NOT NULL,
    currency VARCHAR(3),
    payment_method VARCHAR(50),
    status ENUM('PENDING', 'PROCESSING', 'COMPLETED', 'FAILED', 'REFUNDED') DEFAULT 'PENDING',
    transaction_id VARCHAR(255) UNIQUE,
    payment_gateway_response JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    INDEX (order_id),
    INDEX (transaction_id)
);

-- Refunds
CREATE TABLE refunds (
    refund_id BIGINT PRIMARY KEY,
    order_id BIGINT NOT NULL,
    payment_id BIGINT NOT NULL,
    amount DECIMAL(12, 2) NOT NULL,
    reason VARCHAR(255),
    status ENUM('PENDING', 'APPROVED', 'REJECTED', 'COMPLETED') DEFAULT 'PENDING',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (payment_id) REFERENCES payments(payment_id),
    INDEX (order_id)
);
```

### 3. Inventory Database (SQL: PostgreSQL)

```sql
-- Inventory / Stock
CREATE TABLE inventory (
    inventory_id BIGINT PRIMARY KEY,
    product_id BIGINT NOT NULL UNIQUE,
    seller_id BIGINT NOT NULL,
    quantity_available INT NOT NULL,
    quantity_reserved INT DEFAULT 0,
    quantity_sold INT DEFAULT 0,
    reorder_level INT,
    last_restocked TIMESTAMP,
    warehouse_location VARCHAR(255),
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    INDEX (seller_id),
    INDEX (quantity_available)
);

-- Stock Movements (audit trail)
CREATE TABLE stock_movements (
    movement_id BIGINT PRIMARY KEY,
    product_id BIGINT NOT NULL,
    order_id BIGINT,
    movement_type ENUM('IN', 'OUT', 'ADJUSTMENT', 'RETURN'),
    quantity INT NOT NULL,
    reason VARCHAR(255),
    created_by BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    INDEX (product_id),
    INDEX (created_at)
);
```

### 4. NoSQL Database for Product Catalog (MongoDB)

```javascript
// Products Collection (denormalized for fast reads)
db.products.insertOne({
    _id: ObjectId(),
    product_id: 123,
    name: "iPhone 14 Pro",
    description: "Latest Apple iPhone",
    category_id: 456,
    price: 999.99,
    currency: "USD",
    seller_id: 789,
    images: [
        "https://cdn.example.com/image1.jpg",
        "https://cdn.example.com/image2.jpg"
    ],
    attributes: {
        color: "Space Black",
        storage: "128GB",
        condition: "New"
    },
    rating: 4.8,
    review_count: 1250,
    in_stock: true,
    availability_count: 50,
    tags: ["smartphone", "apple", "latest"],
    created_at: ISODate("2024-01-15"),
    updated_at: ISODate("2024-01-16")
});

// Reviews Collection
db.reviews.insertOne({
    _id: ObjectId(),
    review_id: 1001,
    product_id: 123,
    user_id: 456,
    rating: 5,
    title: "Excellent phone!",
    comment: "Amazing camera and performance",
    helpful_count: 250,
    unhelpful_count: 5,
    images: ["url1", "url2"],
    created_at: ISODate("2024-01-10"),
    updated_at: ISODate("2024-01-10")
});

// User Activity/Browsing History
db.user_activity.insertOne({
    _id: ObjectId(),
    user_id: 456,
    viewed_products: [123, 124, 125],
    last_viewed: ISODate("2024-01-16T10:30:00Z"),
    search_queries: ["iPhone", "laptop", "headphones"],
    wishlist: [123, 456, 789]
});
```

### 5. Search Engine (Elasticsearch)

```json
// Product Index Mapping
PUT /products
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 2,
    "analysis": {
      "analyzer": {
        "autocomplete": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "stop"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "product_id": {"type": "keyword"},
      "name": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "autocomplete": {
            "type": "text",
            "analyzer": "autocomplete"
          }
        }
      },
      "description": {"type": "text"},
      "category_id": {"type": "keyword"},
      "price": {"type": "float"},
      "rating": {"type": "float"},
      "seller_id": {"type": "keyword"},
      "tags": {"type": "keyword"},
      "in_stock": {"type": "boolean"},
      "created_at": {"type": "date"}
    }
  }
}

// Sample Product Document
POST /products/_doc
{
  "product_id": 123,
  "name": "iPhone 14 Pro",
  "description": "Latest Apple iPhone with advanced camera",
  "category_id": 456,
  "price": 999.99,
  "rating": 4.8,
  "seller_id": 789,
  "tags": ["smartphone", "apple", "latest"],
  "in_stock": true,
  "created_at": "2024-01-15"
}
```

### Database Selection Rationale

| Data Type | Database | Reason |
|-----------|----------|--------|
| Users, Orders, Payments | PostgreSQL | ACID compliance, complex transactions, consistency |
| Product Catalog, Reviews | MongoDB | Flexible schema, denormalization, high reads |
| Search Queries | Elasticsearch | Full-text search, real-time indexing, analytics |
| Cache Layer | Redis | Ultra-fast reads, TTL support, session storage |
| Analytics | Data Warehouse | Historical analysis, OLAP queries |

---

## API Design

### 1. Product APIs

```
GET /api/v1/products
Query Parameters:
  - category_id: Filter by category
  - search: Full-text search
  - min_price, max_price: Price range
  - rating_min: Minimum rating
  - page: Pagination
  - limit: Items per page
  - sort: price_asc, price_desc, rating, newest

Response:
{
  "status": "success",
  "data": {
    "products": [
      {
        "product_id": 123,
        "name": "iPhone 14 Pro",
        "price": 999.99,
        "rating": 4.8,
        "image_url": "...",
        "seller": {
          "seller_id": 789,
          "name": "Apple Store"
        },
        "in_stock": true
      }
    ],
    "total_count": 1500,
    "page": 1,
    "limit": 20
  }
}
```

```
GET /api/v1/products/{product_id}
Response:
{
  "status": "success",
  "data": {
    "product_id": 123,
    "name": "iPhone 14 Pro",
    "description": "...",
    "price": 999.99,
    "currency": "USD",
    "images": ["url1", "url2"],
    "attributes": {
      "color": ["Space Black", "Silver", "Gold"],
      "storage": ["128GB", "256GB", "512GB"]
    },
    "rating": 4.8,
    "reviews_count": 1250,
    "seller": {
      "seller_id": 789,
      "name": "Apple Store",
      "rating": 4.9
    },
    "availability": {
      "in_stock": true,
      "quantity": 50
    }
  }
}
```

---

### 2. Cart APIs

```
POST /api/v1/cart/items
Body:
{
  "product_id": 123,
  "quantity": 2,
  "selected_attributes": {
    "color": "Space Black",
    "storage": "256GB"
  }
}

Response:
{
  "status": "success",
  "data": {
    "cart_id": "cart_456",
    "items": [
      {
        "cart_item_id": 1,
        "product_id": 123,
        "product_name": "iPhone 14 Pro",
        "quantity": 2,
        "unit_price": 999.99,
        "total_price": 1999.98
      }
    ],
    "subtotal": 1999.98,
    "tax": 160.00,
    "shipping": 10.00,
    "total": 2169.98
  }
}
```

```
GET /api/v1/cart
Response:
{
  "status": "success",
  "data": {
    "items": [...],
    "subtotal": 1999.98,
    "tax": 160.00,
    "shipping": 10.00,
    "discount": 50.00,
    "total": 2119.98,
    "last_updated": "2024-01-16T10:30:00Z"
  }
}
```

```
DELETE /api/v1/cart/items/{cart_item_id}
Response: 204 No Content
```

---

### 3. Order APIs

```
POST /api/v1/orders
Body:
{
  "cart_id": "cart_456",
  "shipping_address_id": 123,
  "billing_address_id": 123,
  "payment_method": "credit_card",
  "coupon_code": "SAVE10"
}

Response:
{
  "status": "success",
  "data": {
    "order_id": 5001,
    "order_number": "ORD-2024-001234",
    "status": "PENDING",
    "items": [...],
    "total_amount": 2119.98,
    "payment_status": "PENDING",
    "created_at": "2024-01-16T10:35:00Z"
  }
}
```

```
GET /api/v1/orders/{order_id}
Response:
{
  "status": "success",
  "data": {
    "order_id": 5001,
    "order_number": "ORD-2024-001234",
    "status": "CONFIRMED",
    "items": [
      {
        "order_item_id": 1,
        "product_id": 123,
        "quantity": 2,
        "unit_price": 999.99,
        "status": "CONFIRMED"
      }
    ],
    "shipping": {
      "address": {...},
      "status": "NOT_SHIPPED",
      "tracking_number": null
    },
    "payment": {
      "amount": 2119.98,
      "status": "COMPLETED",
      "method": "credit_card",
      "transaction_id": "txn_123456"
    },
    "timeline": [
      {
        "event": "ORDER_PLACED",
        "timestamp": "2024-01-16T10:35:00Z"
      },
      {
        "event": "PAYMENT_CONFIRMED",
        "timestamp": "2024-01-16T10:36:00Z"
      }
    ]
  }
}
```

```
GET /api/v1/orders
Query Parameters:
  - status: PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
  - from_date, to_date: Date range
  - page, limit: Pagination

Response:
{
  "status": "success",
  "data": {
    "orders": [...],
    "total_count": 150,
    "page": 1,
    "limit": 10
  }
}
```

---

### 4. Payment APIs

```
POST /api/v1/payments/{order_id}/process
Body:
{
  "payment_method": "credit_card",
  "card_token": "tok_123456",
  "amount": 2119.98
}

Response:
{
  "status": "success",
  "data": {
    "payment_id": 9001,
    "order_id": 5001,
    "status": "PROCESSING",
    "amount": 2119.98,
    "transaction_id": "txn_123456"
  }
}
```

```
GET /api/v1/payments/{payment_id}
Response:
{
  "status": "success",
  "data": {
    "payment_id": 9001,
    "order_id": 5001,
    "status": "COMPLETED",
    "amount": 2119.98,
    "currency": "USD",
    "method": "credit_card",
    "transaction_id": "txn_123456",
    "created_at": "2024-01-16T10:36:00Z",
    "updated_at": "2024-01-16T10:37:00Z"
  }
}
```

---

### 5. Review APIs

```
POST /api/v1/products/{product_id}/reviews
Body:
{
  "rating": 5,
  "title": "Amazing product!",
  "comment": "Exceeded my expectations",
  "images": ["url1", "url2"]
}

Response: 201 Created
{
  "status": "success",
  "data": {
    "review_id": 1001,
    "product_id": 123,
    "rating": 5,
    "title": "Amazing product!",
    "comment": "Exceeded my expectations",
    "created_at": "2024-01-16T10:40:00Z"
  }
}
```

```
GET /api/v1/products/{product_id}/reviews
Query Parameters:
  - rating: Filter by rating
  - sort: helpful, recent
  - page, limit: Pagination

Response:
{
  "status": "success",
  "data": {
    "reviews": [
      {
        "review_id": 1001,
        "user": {
          "user_id": 456,
          "name": "John Doe"
        },
        "rating": 5,
        "title": "Amazing product!",
        "comment": "...",
        "helpful_count": 250,
        "created_at": "2024-01-16T10:40:00Z"
      }
    ],
    "average_rating": 4.8,
    "total_count": 1250,
    "distribution": {
      "5": 800,
      "4": 300,
      "3": 100,
      "2": 30,
      "1": 20
    }
  }
}
```

---

## Microservices Deep Dive

### 1. Product Service

**Responsibilities:**
- Product catalog management
- Product search
- Category management
- Product details

**Database:** PostgreSQL + MongoDB + Elasticsearch

**Key Endpoints:**
- GET /products - List products
- GET /products/{id} - Get product details
- GET /search - Search products
- POST /products (seller) - Create product
- PUT /products/{id} (seller) - Update product

**Internal APIs (Service-to-Service):**
- GET /internal/products/{id} - Get product by ID
- POST /internal/products/{id}/reserve - Reserve inventory
- POST /internal/products/{id}/release - Release reservation

**Caching:**
```
// Cache product details (expires in 24 hours)
CACHE_KEY = "product:{product_id}"
CACHE_TTL = 86400

// Cache search results (expires in 1 hour)
CACHE_KEY = "search:{query}:{page}"
CACHE_TTL = 3600
```

---

### 2. Cart Service

**Responsibilities:**
- Add/remove items from cart
- Update quantities
- Calculate totals
- Apply coupons

**Database:** Redis (primary), PostgreSQL (backup)

**Why Redis?**
- Cart data is temporary
- Requires fast reads/writes
- High throughput
- Session-like behavior

**Key Endpoints:**
- POST /cart/items - Add item
- GET /cart - Get cart
- PUT /cart/items/{id} - Update item
- DELETE /cart/items/{id} - Remove item
- POST /cart/apply-coupon - Apply discount

**Data Structure:**
```javascript
// Cart in Redis
CART_KEY = "cart:{user_id}"

{
  "items": [
    {
      "product_id": 123,
      "quantity": 2,
      "unit_price": 999.99,
      "selected_attributes": {
        "color": "Space Black",
        "storage": "256GB"
      }
    }
  ],
  "subtotal": 1999.98,
  "tax": 160.00,
  "shipping": 10.00,
  "discounts": [
    {
      "coupon_code": "SAVE10",
      "discount_amount": 50.00
    }
  ],
  "total": 2119.98,
  "updated_at": "2024-01-16T10:30:00Z"
}
```

**Cart Expiration:**
- Abandon cart after 30 days
- TTL: 2,592,000 seconds

---

### 3. Order Service

**Responsibilities:**
- Create orders from cart
- Manage order lifecycle
- Order status updates
- Order history

**Database:** PostgreSQL (ACID required)

**Key Endpoints:**
- POST /orders - Create order
- GET /orders/{id} - Get order details
- GET /orders - List user orders
- PUT /orders/{id}/cancel - Cancel order
- GET /orders/{id}/timeline - Order timeline

**Order Creation Workflow:**
```
1. Validate cart
   ├─ Check items exist
   ├─ Verify prices haven't changed
   └─ Apply discounts

2. Reserve inventory
   ├─ Call Inventory Service
   └─ Lock stock

3. Create order record
   ├─ Save to PostgreSQL
   └─ Set status: PENDING

4. Process payment
   ├─ Call Payment Service
   └─ Update payment status

5. Confirm order
   ├─ Update order status: CONFIRMED
   ├─ Release inventory (or update to SOLD)
   └─ Clear cart

6. Send notifications
   ├─ Order confirmation email
   ├─ SMS notification
   └─ Push notification

7. Publish event
   └─ "OrderCreated" to Kafka
```

**Error Handling (Saga Pattern):**
```
If payment fails:
├─ Release inventory reservation
├─ Cancel order
├─ Send cancellation notification
└─ Return error to user

If inventory unavailable:
├─ Return out-of-stock error
├─ Offer alternatives
└─ Suggest pre-order option
```

---

### 4. Payment Service

**Responsibilities:**
- Payment processing
- Payment gateway integration
- Refund handling
- Payment security

**Database:** PostgreSQL (immutable audit trail)

**Integration:**
- Stripe
- PayPal
- Square

**Key Endpoints:**
- POST /payments/{order_id}/process - Process payment
- GET /payments/{id} - Get payment status
- POST /payments/{id}/refund - Refund payment

**Payment Processing Flow:**
```
1. Receive payment request
   ├─ Validate amount
   └─ Validate currency

2. Tokenize card (PCI compliance)
   ├─ Don't store raw card data
   └─ Use payment gateway tokens

3. Process with gateway
   ├─ Call Stripe/PayPal API
   └─ Get transaction ID

4. Handle response
   ├─ SUCCESS: Update payment status
   ├─ FAILED: Retry logic (exponential backoff)
   └─ PENDING: Webhook polling

5. Update order status
   └─ Publish "PaymentConfirmed" event

6. Log transaction (immutable)
   ├─ Store full response
   ├─ Store error details
   └─ Audit trail
```

**Idempotency:**
```
Ensure payment isn't processed twice:

// Use idempotency key
REQUEST HEADER: Idempotency-Key: uuid
├─ Check if key exists
├─ If yes: Return previous response
└─ If no: Process and cache response

// Database constraint
UNIQUE KEY (order_id, transaction_id)
```

---

### 5. Inventory Service

**Responsibilities:**
- Stock management
- Reservation logic
- Real-time availability
- Stock movements tracking

**Database:** PostgreSQL (consistency critical)

**Key Endpoints:**
- GET /inventory/{product_id} - Check stock
- POST /inventory/{product_id}/reserve - Reserve stock
- POST /inventory/{product_id}/release - Release reservation
- PUT /inventory/{product_id} - Update stock

**Inventory States:**
```
total_quantity = quantity_available + quantity_reserved + quantity_sold

Example:
total = 100
available = 50
reserved = 30
sold = 20
```

**Reservation Process:**
```
1. Check availability
   └─ quantity_available >= requested_quantity?

2. Create reservation
   ├─ Decrement quantity_available
   ├─ Increment quantity_reserved
   └─ Return reservation_id

3. Timeout handling (30 minutes)
   ├─ If order not confirmed within 30 min
   ├─ Release reservation
   └─ Add back to quantity_available

4. On order confirmation
   ├─ Decrement quantity_available
   ├─ Decrement quantity_reserved
   ├─ Increment quantity_sold
   └─ Delete reservation record
```

**Stock Movement Audit Trail:**
```
Every stock change logged:
├─ movement_type: IN, OUT, ADJUSTMENT, RETURN
├─ quantity_changed
├─ reason
├─ created_by
└─ timestamp
```

---

### 6. Notification Service

**Responsibilities:**
- Email notifications
- SMS notifications
- Push notifications
- Notification history

**Integrations:**
- SendGrid / AWS SES (Email)
- Twilio (SMS)
- Firebase Cloud Messaging (Push)

**Message Queue:** Kafka

**Notification Types:**
```
Order Placed
├─ Email: Order confirmation
├─ SMS: Order number
└─ Push: Order created

Payment Confirmed
├─ Email: Payment receipt
└─ Push: Payment successful

Order Shipped
├─ Email: Shipping details + tracking
├─ SMS: Tracking link
└─ Push: Order shipped

Order Delivered
├─ Email: Delivery confirmation
├─ Push: Package delivered
└─ Request review

Payment Failed
├─ Email: Payment failed
├─ SMS: Action needed
└─ Push: Retry payment

Refund Issued
├─ Email: Refund confirmation
├─ Push: Money refunded
└─ SMS: Refund date
```

**Implementation:**
```javascript
// Subscribe to Kafka topics
Kafka Topics:
├─ order-events
├─ payment-events
├─ shipment-events
└─ inventory-events

// Process and send notifications
Process Event:
├─ 1. Enrich with user data
├─ 2. Select notification channels
├─ 3. Generate message content
├─ 4. Send (with retry logic)
└─ 5. Log to database
```

---

### 7. Shipping Service

**Responsibilities:**
- Shipping label generation
- Carrier integration
- Real-time tracking
- Shipping cost calculation

**Integrations:**
- FedEx API
- UPS API
- DHL API
- USPS API

**Key Endpoints:**
- POST /shipments - Create shipment
- GET /shipments/{id} - Get shipment details
- GET /shipments/{id}/track - Track shipment
- POST /shipments/{id}/cancel - Cancel shipment

**Shipping Process:**
```
1. Receive order_items
2. Validate address
3. Calculate shipping cost
4. Select carrier (cheapest/fastest)
5. Generate shipping label
6. Integrate with warehouse system
7. Return tracking number
8. Subscribe to tracking updates
9. Update order status
```

---

## Caching Strategy

### Cache Layer Architecture

```
┌─────────────────────────────────────────┐
│           Client Request                │
└────────────────┬────────────────────────┘
                 │
                 ↓
        ┌────────────────┐
        │  Redis Cache   │
        │                │
        │ L1 Cache:      │
        │ • Hot data     │
        │ • User sessions│
        │ • Cart         │
        └────────┬───────┘
                 │
          Cache Hit? ✓
                 │
                 └────→ Return cached data
                 │
            Cache Miss ✗
                 │
                 ↓
        ┌────────────────┐
        │   Database     │
        │ (PostgreSQL)   │
        └────────┬───────┘
                 │
        Update Cache & Return
```

### 1. Cache-Aside Pattern (Most Common)

```javascript
// Get product data
GET product(productId):
    1. Check Redis cache
    2. If found: return data (cache hit)
    3. If not found: (cache miss)
        a. Query database
        b. Update Redis cache (TTL: 24 hours)
        c. Return data

// Pseudo code
function getProduct(productId) {
    // Try cache first
    let product = redis.get(`product:${productId}`);

    if (product) {
        return product;  // Cache hit
    }

    // Cache miss - get from DB
    product = db.query(`SELECT * FROM products WHERE id = ${productId}`);

    // Update cache
    redis.setex(`product:${productId}`, 86400, JSON.stringify(product));

    return product;
}
```

### 2. Write-Through Pattern (Order Data)

```javascript
// Write order
POST createOrder(orderData):
    1. Write to cache
    2. Write to database
    3. Return success

// Pseudo code
function createOrder(orderData) {
    // Create object with ID
    const orderWithId = {
        order_id: generateId(),
        ...orderData
    };

    // Write to cache first
    redis.setex(`order:${orderWithId.order_id}`, 3600, JSON.stringify(orderWithId));

    // Write to database
    db.insert('orders', orderWithId);

    return orderWithId;
}
```

### 3. Write-Behind Pattern (High Write Volume)

```javascript
// Write cart items
POST addToCart(userId, productId):
    1. Write to cache immediately (fast)
    2. Queue async write to database
    3. Return success to client

// Pseudo code
function addToCart(userId, productId) {
    const cartKey = `cart:${userId}`;
    const cartData = redis.get(cartKey) || { items: [] };

    // Add to cache immediately (fast response)
    cartData.items.push({
        product_id: productId,
        added_at: Date.now()
    });

    redis.setex(cartKey, 2592000, JSON.stringify(cartData));

    // Queue database write (async)
    messageQueue.publish('cart-updates', {
        user_id: userId,
        action: 'add_item',
        product_id: productId
    });

    return { success: true };
}
```

### Cache Keys Strategy

```javascript
// Consistent naming convention
Products:
  product:{product_id}              // Single product
  products:page:{page_num}          // Product list pagination
  products:category:{cat_id}        // Products by category
  search:{query}:{page}             // Search results

User Data:
  user:{user_id}                    // User profile
  user:addresses:{user_id}          // User addresses

Cart & Orders:
  cart:{user_id}                    // Shopping cart
  order:{order_id}                  // Order details

Sessions:
  session:{session_id}              // User session

Rankings:
  trending:products                 // Trending products
  top:sellers                       // Top sellers
```

### Cache Expiration Strategy

| Data Type | TTL | Reason |
|-----------|-----|--------|
| Product details | 24 hours | Rarely changes |
| Product availability | 5 minutes | Changes frequently |
| Cart | 30 days | User session lifetime |
| Order | 1 hour | Quick reference |
| User session | 7 days | User login persistence |
| Search results | 1 hour | Query results |
| Trending data | 15 minutes | Real-time updates |

### Cache Invalidation

```javascript
// On product update
function updateProduct(productId, updates) {
    // Update database
    db.update('products', { id: productId }, updates);

    // Invalidate cache
    redis.delete(`product:${productId}`);
    redis.delete(`products:category:*`);  // All category queries
    redis.delete(`search:*`);              // All searches

    // Publish cache invalidation event
    kafka.publish('cache-invalidation', {
        type: 'product_updated',
        product_id: productId
    });
}

// On inventory change
function updateInventory(productId, newQuantity) {
    db.update('inventory', { product_id: productId }, { quantity_available: newQuantity });

    // Short-lived cache invalidation
    redis.delete(`product:${productId}`);
}

// On order creation
function createOrder(orderData) {
    db.insert('orders', orderData);

    // Invalidate user's order list cache
    redis.delete(`user:orders:${orderData.user_id}`);

    // Invalidate inventory cache
    for (let item of orderData.items) {
        redis.delete(`product:${item.product_id}`);
    }
}
```

---

## Search & Indexing

### Elasticsearch Integration

```javascript
// Index product into Elasticsearch
function indexProduct(product) {
    const doc = {
        product_id: product.id,
        name: product.name,
        description: product.description,
        category_id: product.category_id,
        price: product.price,
        rating: product.rating,
        seller_id: product.seller_id,
        tags: product.tags,
        in_stock: product.inventory.available > 0,
        created_at: product.created_at
    };

    // Index with product_id as document ID
    elasticsearch.index({
        index: 'products',
        id: product.id,
        body: doc
    });
}
```

### Search Queries

```javascript
// Full-text search
function searchProducts(query, filters, page = 1, limit = 20) {
    const from = (page - 1) * limit;

    const esQuery = {
        index: 'products',
        body: {
            query: {
                bool: {
                    must: [
                        {
                            multi_match: {
                                query: query,
                                fields: ['name^2', 'description', 'tags'],
                                fuzziness: 'AUTO'
                            }
                        }
                    ],
                    filter: [
                        { range: { price: { gte: filters.min_price, lte: filters.max_price } } },
                        { term: { in_stock: true } },
                        { range: { rating: { gte: filters.min_rating } } }
                    ]
                }
            },
            sort: [
                { _score: { order: 'desc' } },
                { rating: { order: 'desc' } }
            ],
            from: from,
            size: limit,
            // Aggregations for facets
            aggs: {
                categories: {
                    terms: { field: 'category_id', size: 10 }
                },
                price_range: {
                    range: {
                        field: 'price',
                        ranges: [
                            { to: 50 },
                            { from: 50, to: 100 },
                            { from: 100, to: 500 },
                            { from: 500, to: 1000 },
                            { from: 1000 }
                        ]
                    }
                }
            }
        }
    };

    return elasticsearch.search(esQuery);
}

// Autocomplete (prefix search)
function getAutocomplete(query) {
    return elasticsearch.search({
        index: 'products',
        body: {
            query: {
                match_phrase_prefix: {
                    name: {
                        query: query,
                        boost: 2
                    }
                }
            },
            size: 10,
            _source: ['product_id', 'name']
        }
    });
}

// Faceted search (filters)
function getFacets(query) {
    return elasticsearch.search({
        index: 'products',
        body: {
            query: {
                match: { name: query }
            },
            aggs: {
                categories: {
                    terms: { field: 'category_id', size: 20 }
                },
                sellers: {
                    terms: { field: 'seller_id', size: 20 }
                },
                rating: {
                    terms: { field: 'rating', size: 5 }
                },
                price_buckets: {
                    range: {
                        field: 'price',
                        ranges: [
                            { to: 50 },
                            { from: 50, to: 500 },
                            { from: 500, to: 1000 },
                            { from: 1000 }
                        ]
                    }
                }
            },
            size: 0  // Don't need actual documents, just facets
        }
    });
}
```

### Search Performance

**Indexing Performance:**
```
Real-time indexing:
├─ Bulk API for batch indexing
├─ Refresh interval: 1 second (for real-time)
└─ ~10K docs/second per shard

Query Performance:
├─ Average response: < 500ms
├─ Use shards for parallelization
└─ Index caching for common queries
```

---

## Payment Processing

### Payment Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│                  PAYMENT FLOW                           │
└─────────────────────────────────────────────────────────┘

1. CLIENT SIDE (PCI Compliance)
   ├─ Never send raw card data to your server
   ├─ Use Stripe.js / PayPal SDK
   └─ Get token from payment gateway

2. SERVER SIDE
   ├─ Receive token from client
   ├─ Create payment record (status: PENDING)
   ├─ Call payment gateway (Stripe/PayPal)
   └─ Handle response

3. ASYNC PROCESSING
   ├─ Payment may take time
   ├─ Set status to PROCESSING
   ├─ Use webhooks for updates
   └─ Eventually get COMPLETED/FAILED

4. WEBHOOK HANDLER
   ├─ Verify webhook signature
   ├─ Update payment status
   ├─ Update order status
   └─ Send notifications
```

### Payment Implementation

```javascript
// Client-side (simplified)
async function processPayment(orderData) {
    // 1. Create payment method token (Stripe.js)
    const { paymentMethod, error } = await stripe.createPaymentMethod({
        type: 'card',
        card: cardElement  // DOM element
    });

    if (error) {
        showError(error.message);
        return;
    }

    // 2. Send token to server
    const response = await fetch('/api/v1/payments/process', {
        method: 'POST',
        body: JSON.stringify({
            order_id: orderData.order_id,
            payment_method_id: paymentMethod.id,
            amount: orderData.total
        })
    });

    const result = await response.json();

    // 3. Handle response
    if (result.status === 'succeeded') {
        // Payment successful
        redirectToOrderConfirmation(result.order_id);
    } else if (result.status === 'requires_action') {
        // 3D Secure or other verification needed
        handleAdditionalAuthentication(result);
    } else {
        showError(result.error);
    }
}

// Server-side payment processing
async function processPayment(req, res) {
    const { order_id, payment_method_id, amount } = req.body;

    try {
        // 1. Validate order exists and amount matches
        const order = await db.query('SELECT * FROM orders WHERE id = ?', [order_id]);
        if (!order || order.total_amount !== amount) {
            return res.status(400).json({ error: 'Invalid amount' });
        }

        // 2. Create idempotency key (prevent duplicate payments)
        const idempotencyKey = `${order_id}_${Date.now()}`;

        // 3. Call Stripe API
        const paymentIntent = await stripe.paymentIntents.create({
            amount: Math.round(amount * 100),  // Convert to cents
            currency: 'usd',
            payment_method: payment_method_id,
            confirm: true,  // Automatically confirm
            metadata: {
                order_id: order_id
            }
        }, {
            idempotencyKey: idempotencyKey
        });

        // 4. Update payment status
        if (paymentIntent.status === 'succeeded') {
            await db.query(
                'UPDATE payments SET status = ?, transaction_id = ? WHERE order_id = ?',
                ['COMPLETED', paymentIntent.id, order_id]
            );

            // 5. Update order status
            await db.query(
                'UPDATE orders SET status = ?, payment_status = ? WHERE id = ?',
                ['CONFIRMED', 'COMPLETED', order_id]
            );

            // 6. Publish event for other services
            await kafka.publish('payment-events', {
                event: 'payment_confirmed',
                order_id: order_id,
                payment_id: paymentIntent.id,
                amount: amount
            });

            return res.json({
                status: 'succeeded',
                payment_id: paymentIntent.id,
                order_id: order_id
            });
        } else if (paymentIntent.status === 'requires_action') {
            return res.json({
                status: 'requires_action',
                client_secret: paymentIntent.client_secret
            });
        } else {
            return res.status(400).json({
                status: 'failed',
                error: 'Payment failed'
            });
        }
    } catch (error) {
        console.error('Payment error:', error);

        // Log for audit
        await db.query(
            'INSERT INTO payment_errors (order_id, error_message) VALUES (?, ?)',
            [order_id, error.message]
        );

        return res.status(500).json({ error: 'Payment processing failed' });
    }
}

// Webhook handler (Stripe → Your Server)
app.post('/webhooks/stripe', express.raw({type: 'application/json'}), async (req, res) => {
    const sig = req.headers['stripe-signature'];
    let event;

    try {
        // Verify webhook signature
        event = stripe.webhooks.constructEvent(
            req.body,
            sig,
            process.env.STRIPE_WEBHOOK_SECRET
        );
    } catch (error) {
        return res.status(400).send(`Webhook Error: ${error.message}`);
    }

    // Handle different event types
    switch (event.type) {
        case 'payment_intent.succeeded':
            const paymentIntent = event.data.object;
            await handlePaymentSucceeded(paymentIntent);
            break;

        case 'payment_intent.payment_failed':
            const failedPayment = event.data.object;
            await handlePaymentFailed(failedPayment);
            break;

        case 'charge.refunded':
            const refund = event.data.object;
            await handleRefund(refund);
            break;
    }

    res.json({received: true});
});
```

### Refund Process

```javascript
async function refundPayment(paymentId, reason) {
    try {
        // 1. Get original payment
        const payment = await db.query(
            'SELECT * FROM payments WHERE id = ?',
            [paymentId]
        );

        // 2. Create refund via Stripe
        const refund = await stripe.refunds.create({
            payment_intent: payment.transaction_id,
            reason: 'requested_by_customer',
            metadata: {
                reason: reason
            }
        });

        // 3. Record refund in database
        await db.query(
            'INSERT INTO refunds (payment_id, amount, status, transaction_id) VALUES (?, ?, ?, ?)',
            [paymentId, payment.amount, 'COMPLETED', refund.id]
        );

        // 4. Update payment status
        await db.query(
            'UPDATE payments SET status = ? WHERE id = ?',
            ['REFUNDED', paymentId]
        );

        // 5. Publish event
        await kafka.publish('payment-events', {
            event: 'payment_refunded',
            payment_id: paymentId,
            amount: payment.amount
        });

        return { success: true, refund_id: refund.id };
    } catch (error) {
        console.error('Refund error:', error);
        throw error;
    }
}
```

---

## Real-Time Inventory

### Problem: Double Booking

```
Scenario: Last iPhone in stock, 2 customers trying to buy

WRONG APPROACH (Race Condition):
Customer A                  Customer B
├─ Check stock: 1           ├─ Check stock: 1
├─ Looks good!              ├─ Looks good!
└─ Checkout...              └─ Checkout...
   ├─ Both place order
   ├─ Both reserved 1
   └─ 2 orders, but only 1 item!

RIGHT APPROACH (Transactional):
Customer A
├─ BEGIN TRANSACTION
├─ Lock row (FOR UPDATE)
├─ Check stock: 1
├─ Reserve: stock = 0
├─ COMMIT
└─ Success!

Customer B
├─ BEGIN TRANSACTION
├─ Lock row (FOR UPDATE)
├─ Check stock: 0 (updated)
├─ Insufficient stock!
├─ ROLLBACK
└─ Error: Out of stock
```

### Implementation with Database Locks

```sql
-- Atomic inventory check and reservation
BEGIN TRANSACTION;

-- Lock the row (no other process can modify it)
SELECT * FROM inventory
WHERE product_id = 123
FOR UPDATE;

-- Check availability
-- Assuming: quantity_available >= 1

-- Reserve inventory
UPDATE inventory
SET quantity_available = quantity_available - 1,
    quantity_reserved = quantity_reserved + 1
WHERE product_id = 123;

-- Create reservation record
INSERT INTO reservations (product_id, order_id, user_id, expires_at)
VALUES (123, 456, 789, DATE_ADD(NOW(), INTERVAL 30 MINUTE));

COMMIT TRANSACTION;
```

### Implementation in Code

```javascript
async function reserveInventory(productId, quantity, orderId) {
    const connection = await pool.getConnection();

    try {
        await connection.beginTransaction();

        // 1. Lock and check availability
        const [inventory] = await connection.query(
            'SELECT quantity_available FROM inventory WHERE product_id = ? FOR UPDATE',
            [productId]
        );

        if (!inventory || inventory[0].quantity_available < quantity) {
            await connection.rollback();
            throw new Error('Insufficient inventory');
        }

        // 2. Update inventory
        await connection.query(
            'UPDATE inventory SET quantity_available = quantity_available - ?, quantity_reserved = quantity_reserved + ? WHERE product_id = ?',
            [quantity, quantity, productId]
        );

        // 3. Create reservation record
        const [result] = await connection.query(
            'INSERT INTO reservations (product_id, order_id, quantity, expires_at) VALUES (?, ?, ?, DATE_ADD(NOW(), INTERVAL 30 MINUTE))',
            [productId, orderId, quantity]
        );

        await connection.commit();

        return {
            success: true,
            reservation_id: result.insertId,
            expires_at: new Date(Date.now() + 30 * 60 * 1000)
        };
    } catch (error) {
        await connection.rollback();
        throw error;
    } finally {
        connection.release();
    }
}
```

### Handling Reservation Timeouts

```javascript
// Background job to release expired reservations
async function releaseExpiredReservations() {
    const connection = await pool.getConnection();

    try {
        await connection.beginTransaction();

        // Find expired reservations
        const [expiredReservations] = await connection.query(
            'SELECT * FROM reservations WHERE expires_at < NOW() AND status = "ACTIVE"'
        );

        for (let reservation of expiredReservations) {
            // Release back to inventory
            await connection.query(
                'UPDATE inventory SET quantity_available = quantity_available + ?, quantity_reserved = quantity_reserved - ? WHERE product_id = ?',
                [reservation.quantity, reservation.quantity, reservation.product_id]
            );

            // Mark reservation as expired
            await connection.query(
                'UPDATE reservations SET status = "EXPIRED" WHERE id = ?',
                [reservation.id]
            );
        }

        await connection.commit();
        console.log(`Released ${expiredReservations.length} expired reservations`);
    } catch (error) {
        await connection.rollback();
        console.error('Error releasing reservations:', error);
    } finally {
        connection.release();
    }
}

// Schedule to run every 5 minutes
setInterval(releaseExpiredReservations, 5 * 60 * 1000);
```

---

## Notifications & Messaging

### Kafka Architecture

```
┌──────────────────────────────────────────────────────────┐
│                    MESSAGE BROKER (Kafka)                │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Topics:                                                 │
│  ├─ order-events (Partitions: 10)                        │
│  ├─ payment-events (Partitions: 5)                       │
│  ├─ inventory-events (Partitions: 10)                    │
│  ├─ user-events (Partitions: 5)                          │
│  └─ notification-events (Partitions: 3)                  │
│                                                           │
└──────────────────────────────────────────────────────────┘

Producers:                Consumers:
├─ Order Service    →     ├─ Notification Service
├─ Payment Service  →     ├─ Analytics Service
├─ Inventory Service →    ├─ Search Service
└─ User Service     →     └─ Reporting Service
```

### Event Publishing

```javascript
// Order Service publishes events
async function createOrder(orderData) {
    // ... create order in database ...

    // Publish event to Kafka
    await kafka.produce({
        topic: 'order-events',
        messages: [
            {
                key: `order:${order.id}`,
                value: JSON.stringify({
                    event_type: 'ORDER_CREATED',
                    order_id: order.id,
                    order_number: order.order_number,
                    user_id: order.user_id,
                    total_amount: order.total_amount,
                    items: order.items,
                    timestamp: new Date().toISOString()
                })
            }
        ]
    });
}

// Inventory Service publishes events
async function updateInventory(productId, changeQuantity) {
    // ... update inventory in database ...

    await kafka.produce({
        topic: 'inventory-events',
        messages: [
            {
                key: `product:${productId}`,
                value: JSON.stringify({
                    event_type: 'INVENTORY_UPDATED',
                    product_id: productId,
                    quantity_changed: changeQuantity,
                    new_quantity: newQuantity,
                    timestamp: new Date().toISOString()
                })
            }
        ]
    });
}

// Payment Service publishes events
async function confirmPayment(paymentId) {
    // ... update payment status ...

    await kafka.produce({
        topic: 'payment-events',
        messages: [
            {
                key: `payment:${paymentId}`,
                value: JSON.stringify({
                    event_type: 'PAYMENT_CONFIRMED',
                    payment_id: paymentId,
                    order_id: payment.order_id,
                    amount: payment.amount,
                    timestamp: new Date().toISOString()
                })
            }
        ]
    });
}
```

### Event Consuming (Notification Service)

```javascript
const consumer = kafka.consumer({ groupId: 'notification-service-group' });

await consumer.subscribe({ topics: ['order-events', 'payment-events', 'inventory-events'] });

await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
        const event = JSON.parse(message.value.toString());

        try {
            switch (topic) {
                case 'order-events':
                    await handleOrderEvent(event);
                    break;
                case 'payment-events':
                    await handlePaymentEvent(event);
                    break;
                case 'inventory-events':
                    await handleInventoryEvent(event);
                    break;
            }
        } catch (error) {
            console.error('Error processing event:', error);
            // Send to dead-letter queue for retry
            await sendToDeadLetterQueue(topic, message);
        }
    }
});

async function handleOrderEvent(event) {
    switch (event.event_type) {
        case 'ORDER_CREATED':
            // Send order confirmation email
            await sendOrderConfirmationEmail({
                order_id: event.order_id,
                order_number: event.order_number,
                user_id: event.user_id
            });

            // Send SMS
            await sendSMS(user.phone_number, `Your order ${event.order_number} has been placed!`);

            // Send push notification
            await sendPushNotification(event.user_id, {
                title: 'Order Confirmed',
                message: `Order ${event.order_number} placed successfully`
            });
            break;
    }
}

async function sendOrderConfirmationEmail(data) {
    const user = await db.query('SELECT * FROM users WHERE id = ?', [data.user_id]);
    const order = await db.query('SELECT * FROM orders WHERE id = ?', [data.order_id]);

    const email = {
        to: user.email,
        subject: `Order Confirmation - ${data.order_number}`,
        template: 'order-confirmation',
        data: {
            order_number: data.order_number,
            total_amount: order.total_amount,
            items: order.items,
            tracking_link: `https://example.com/orders/${data.order_id}`
        }
    };

    await sendGrid.send(email);
}
```

---

## Scaling & Optimization

### Horizontal Scaling Strategy

```
BEFORE (Single Server):
┌─────────────────────────────┐
│    Monolithic Server        │
│  • User Service             │
│  • Product Service          │
│  • Order Service            │
│  • Payment Service          │
│  ┌─────────────────────┐    │
│  │   Single Database   │    │
│  └─────────────────────┘    │
└─────────────────────────────┘

BOTTLENECK:
- CPU: 95% (too many requests)
- Database: Slow queries
- Single point of failure

AFTER (Microservices + Scale Out):
┌────────────────────────────────────────────┐
│         API Gateway / Load Balancer        │
├────────────────────────────────────────────┤
│                                            │
│  User Service        Product Service      │
│  ├─ Instance 1       ├─ Instance 1        │
│  ├─ Instance 2       ├─ Instance 2        │
│  └─ Instance 3       └─ Instance 3        │
│                                            │
│  Order Service       Payment Service      │
│  ├─ Instance 1       ├─ Instance 1        │
│  └─ Instance 2       └─ Instance 2        │
│                                            │
├────────────────────────────────────────────┤
│  Cache Layer (Redis Cluster)              │
│  Database Layer                           │
│  ├─ Primary (Write)                       │
│  ├─ Read Replica 1                        │
│  ├─ Read Replica 2                        │
│  └─ Read Replica 3                        │
└────────────────────────────────────────────┘
```

### Database Sharding Strategy

```
PROBLEM: Single database reaches capacity
├─ Write throughput: Limited
├─ Storage: Limited
└─ Read scaling: Limited by replication

SOLUTION: Horizontal Partitioning (Sharding)

Shard by User ID:
┌──────────────────────────────────────────┐
│  API Gateway                             │
│  (Shard Router: user_id % shard_count)   │
├──────────────────────────────────────────┤
│         │              │              │   │
│         ↓              ↓              ↓   │
│    ┌────────────┐ ┌────────────┐ ┌────────────┐
│    │  Shard 0   │ │  Shard 1   │ │  Shard 2   │
│    │ Users:     │ │ Users:     │ │ Users:     │
│    │ 0-333k     │ │ 333k-666k  │ │ 666k-1M    │
│    │ Orders:    │ │ Orders:    │ │ Orders:    │
│    │ User 0-333k│ │ User 333k- │ │ User 666k- │
│    └────────────┘ └────────────┘ └────────────┘
│
└──────────────────────────────────────────────┘

Benefits:
✓ Unlimited write throughput
✓ Distributed storage
✓ Parallel processing
✓ Fault isolation

Challenges:
✗ Complex queries across shards
✗ Uneven distribution (hot shards)
✗ Rebalancing complexity
```

### Database Replication (Read Scaling)

```
Master-Slave Replication:

┌────────────────┐
│  Write Master  │
│ (Primary DB)   │
└────────┬───────┘
         │
    Replication Log
    (Binlog)
         │
    ┌────┴──────────┬──────────┐
    ↓               ↓          ↓
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Slave 1  │  │ Slave 2  │  │ Slave 3  │
│ (Read)   │  │ (Read)   │  │ (Read)   │
└──────────┘  └──────────┘  └──────────┘

Routes:
- Write queries → Master
- Read queries → Slaves (load-balanced)

Ratio: 1 write, 3+ reads typical
```

---

## Reliability & Fault Tolerance

### Circuit Breaker Pattern

```javascript
class CircuitBreaker {
    constructor(action, threshold = 5, timeout = 60000) {
        this.action = action;
        this.failureCount = 0;
        this.failureThreshold = threshold;
        this.timeout = timeout;
        this.state = 'CLOSED';  // CLOSED, OPEN, HALF_OPEN
        this.nextAttemptTime = null;
    }

    async execute() {
        if (this.state === 'OPEN') {
            // Circuit is open, fail fast
            if (Date.now() < this.nextAttemptTime) {
                throw new Error('Circuit breaker is OPEN');
            }
            this.state = 'HALF_OPEN';
        }

        try {
            const result = await this.action();
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }

    onSuccess() {
        this.failureCount = 0;
        this.state = 'CLOSED';
    }

    onFailure() {
        this.failureCount++;

        if (this.failureCount >= this.failureThreshold) {
            this.state = 'OPEN';
            this.nextAttemptTime = Date.now() + this.timeout;
            console.log(`Circuit breaker OPEN. Will retry at ${new Date(this.nextAttemptTime)}`);
        }
    }
}

// Usage
const paymentServiceBreaker = new CircuitBreaker(
    async () => {
        return await fetch('https://payment-service/process', {
            method: 'POST'
        });
    },
    5,      // Open after 5 failures
    60000   // Retry after 60 seconds
);

async function processOrder(orderData) {
    try {
        const result = await paymentServiceBreaker.execute();
        // Process payment
    } catch (error) {
        if (error.message.includes('Circuit breaker is OPEN')) {
            // Fall back to queue for later processing
            await queueForRetry(orderData);
            return { status: 'queued_for_later' };
        }
        throw error;
    }
}
```

### Retry with Exponential Backoff

```javascript
async function retryWithBackoff(fn, maxRetries = 3, baseDelay = 1000) {
    for (let attempt = 0; attempt < maxRetries; attempt++) {
        try {
            return await fn();
        } catch (error) {
            if (attempt === maxRetries - 1) {
                throw error;  // Final attempt failed
            }

            // Calculate exponential backoff with jitter
            const delay = baseDelay * Math.pow(2, attempt) + Math.random() * 1000;
            console.log(`Attempt ${attempt + 1} failed. Retrying in ${delay}ms...`);

            await new Promise(resolve => setTimeout(resolve, delay));
        }
    }
}

// Usage
await retryWithBackoff(async () => {
    return await paymentGateway.processPayment(paymentData);
}, 3, 1000);
```

### Graceful Degradation

```javascript
async function getProductDetails(productId) {
    try {
        // Try to get full details with images and reviews
        return await getFullProductDetails(productId);
    } catch (error) {
        console.warn('Failed to get full product details, degrading...');

        try {
            // Fall back to basic details
            return await getBasicProductDetails(productId);
        } catch (error) {
            console.warn('Failed to get basic details, returning cached version...');

            // Use cache as last resort
            return await getCachedProductDetails(productId);
        }
    }
}

async function getOrderStatus(orderId) {
    try {
        return await fetchFromPrimaryDatabase(orderId);
    } catch (error) {
        // Try read replica
        try {
            return await fetchFromReadReplica(orderId);
        } catch (error) {
            // Return cached value if available
            return await getCachedOrderStatus(orderId) || { status: 'UNKNOWN' };
        }
    }
}
```

### Health Checks & Monitoring

```javascript
// Health check endpoint
app.get('/health', async (req, res) => {
    const health = {
        status: 'UP',
        timestamp: new Date(),
        checks: {}
    };

    try {
        // Check database
        await database.query('SELECT 1');
        health.checks.database = { status: 'UP' };
    } catch (error) {
        health.status = 'DOWN';
        health.checks.database = { status: 'DOWN', error: error.message };
    }

    try {
        // Check cache (Redis)
        await redis.ping();
        health.checks.cache = { status: 'UP' };
    } catch (error) {
        health.checks.cache = { status: 'DOWN', error: error.message };
    }

    try {
        // Check Elasticsearch
        await elasticsearch.ping();
        health.checks.search = { status: 'UP' };
    } catch (error) {
        health.checks.search = { status: 'DOWN', error: error.message };
    }

    try {
        // Check Kafka
        await kafka.admin().listTopics();
        health.checks.messaging = { status: 'UP' };
    } catch (error) {
        health.checks.messaging = { status: 'DOWN', error: error.message };
    }

    const statusCode = health.status === 'UP' ? 200 : 503;
    res.status(statusCode).json(health);
});

// Load balancer periodically calls /health
// If 503 is returned, service is removed from rotation
```

---

## Monitoring & Logging

### Metrics to Track

```javascript
// Application Metrics
├─ Request Latency
│  ├─ p50, p90, p95, p99 (percentiles)
│  ├─ max_latency
│  └─ avg_latency
│
├─ Error Rate
│  ├─ Errors per second
│  ├─ Error types (4xx, 5xx)
│  └─ Error rate by endpoint
│
├─ Throughput
│  ├─ Requests per second
│  ├─ Bytes in/out
│  └─ Transactions per second
│
├─ Cache Metrics
│  ├─ Hit ratio
│  ├─ Miss rate
│  ├─ Eviction rate
│  └─ Memory usage
│
├─ Database Metrics
│  ├─ Query latency
│  ├─ Slow queries (> 1s)
│  ├─ Connection pool usage
│  └─ Replication lag
│
└─ System Metrics
   ├─ CPU usage
   ├─ Memory usage
   ├─ Disk I/O
   └─ Network I/O
```

### Implementation with Prometheus

```javascript
const prometheus = require('prom-client');

// Create metrics
const httpRequestDuration = new prometheus.Histogram({
    name: 'http_request_duration_seconds',
    help: 'Duration of HTTP requests in seconds',
    labelNames: ['method', 'route', 'status_code'],
    buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new prometheus.Counter({
    name: 'http_requests_total',
    help: 'Total HTTP requests',
    labelNames: ['method', 'route', 'status_code']
});

const cacheHits = new prometheus.Counter({
    name: 'cache_hits_total',
    help: 'Total cache hits',
    labelNames: ['cache_name']
});

const cacheMisses = new prometheus.Counter({
    name: 'cache_misses_total',
    help: 'Total cache misses',
    labelNames: ['cache_name']
});

// Middleware to track requests
app.use((req, res, next) => {
    const start = Date.now();

    res.on('finish', () => {
        const duration = (Date.now() - start) / 1000;

        httpRequestDuration
            .labels(req.method, req.route.path, res.statusCode)
            .observe(duration);

        httpRequestTotal
            .labels(req.method, req.route.path, res.statusCode)
            .inc();
    });

    next();
});

// Track cache metrics
async function getFromCache(key) {
    const value = await redis.get(key);

    if (value) {
        cacheHits.labels('redis').inc();
        return value;
    } else {
        cacheMisses.labels('redis').inc();
        return null;
    }
}

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
    res.set('Content-Type', prometheus.register.contentType);
    res.end(await prometheus.register.metrics());
});
```

### Logging Strategy

```javascript
// Structured logging with Winston
const winston = require('winston');

const logger = winston.createLogger({
    level: process.env.LOG_LEVEL || 'info',
    format: winston.format.json(),
    transports: [
        // File logs
        new winston.transports.File({
            filename: 'logs/error.log',
            level: 'error'
        }),
        new winston.transports.File({
            filename: 'logs/combined.log'
        }),
        // Console (development)
        new winston.transports.Console({
            format: winston.format.simple()
        })
    ]
});

// Log different events
logger.info('Service started', {
    service: 'order-service',
    version: '1.0.0',
    port: 3000
});

logger.info('Order created', {
    order_id: 5001,
    user_id: 789,
    amount: 2119.98,
    timestamp: new Date(),
    correlation_id: 'abc123'  // For request tracing
});

logger.warn('Slow database query', {
    query: 'SELECT * FROM orders WHERE user_id = ?',
    duration_ms: 1500,
    threshold_ms: 1000
});

logger.error('Payment processing failed', {
    payment_id: 9001,
    error: 'Stripe API timeout',
    retry_attempt: 1,
    correlation_id: 'abc123'
});
```

---

## Security

### Authentication & Authorization

```javascript
// JWT Token Generation
const jwt = require('jsonwebtoken');

function generateToken(userId, email) {
    const token = jwt.sign(
        {
            user_id: userId,
            email: email,
            role: 'customer',
            iat: Math.floor(Date.now() / 1000)
        },
        process.env.JWT_SECRET,
        {
            expiresIn: '7d',
            algorithm: 'HS256'
        }
    );

    return token;
}

// Token Verification Middleware
function verifyToken(req, res, next) {
    const token = req.headers.authorization?.split(' ')[1];

    if (!token) {
        return res.status(401).json({ error: 'No token provided' });
    }

    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = decoded;
        next();
    } catch (error) {
        return res.status(403).json({ error: 'Invalid token' });
    }
}

// Authorization Middleware
function authorize(requiredRole) {
    return (req, res, next) => {
        if (req.user.role !== requiredRole) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        next();
    };
}

// Usage
app.post('/api/v1/orders', verifyToken, async (req, res) => {
    // User is authenticated
    const userId = req.user.user_id;
    // ...
});

app.put('/api/v1/products/:id', verifyToken, authorize('admin'), async (req, res) => {
    // Only admin can update products
    // ...
});
```

### API Security Best Practices

```javascript
// 1. Rate Limiting
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,  // 15 minutes
    max: 100,                   // 100 requests per window
    message: 'Too many requests'
});

app.use('/api/', limiter);

// 2. HTTPS Only
app.use((req, res, next) => {
    if (process.env.NODE_ENV === 'production' && req.header('x-forwarded-proto') !== 'https') {
        return res.redirect(`https://${req.header('host')}${req.url}`);
    }
    next();
});

// 3. CORS Configuration
const cors = require('cors');

app.use(cors({
    origin: process.env.ALLOWED_ORIGINS.split(','),
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));

// 4. Input Validation
const { body, validationResult } = require('express-validator');

app.post('/api/v1/cart/items',
    body('product_id').isInt({ min: 1 }),
    body('quantity').isInt({ min: 1, max: 100 }),
    (req, res) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({ errors: errors.array() });
        }
        // Process request
    }
);

// 5. SQL Injection Prevention (Parameterized Queries)
// WRONG:
db.query(`SELECT * FROM users WHERE id = ${userId}`);

// RIGHT:
db.query('SELECT * FROM users WHERE id = ?', [userId]);

// 6. XSS Prevention (Output Encoding)
// Express automatically escapes JSON responses
// For HTML: use a templating engine that escapes by default

// 7. CSRF Protection
const csrf = require('csurf');
const cookieParser = require('cookie-parser');

app.use(cookieParser());
app.use(csrf({ cookie: true }));

// 8. Sensitive Data Masking
function maskCardNumber(cardNumber) {
    return cardNumber.replace(/\d(?=\d{4})/g, '*');
}

function maskEmail(email) {
    const [name, domain] = email.split('@');
    return `${name[0]}***@${domain}`;
}
```

### Data Protection

```javascript
// Encryption at Rest
const crypto = require('crypto');

function encryptSSN(ssn) {
    const cipher = crypto.createCipher('aes-256-cbc', process.env.ENCRYPTION_KEY);
    let encrypted = cipher.update(ssn, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    return encrypted;
}

function decryptSSN(encrypted) {
    const decipher = crypto.createDecipher('aes-256-cbc', process.env.ENCRYPTION_KEY);
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    return decrypted;
}

// Hashing Passwords
const bcrypt = require('bcryptjs');

async function hashPassword(password) {
    const salt = await bcrypt.genSalt(10);
    return await bcrypt.hash(password, salt);
}

async function verifyPassword(password, hash) {
    return await bcrypt.compare(password, hash);
}

// User Registration
app.post('/api/v1/users/register', async (req, res) => {
    const { email, password } = req.body;

    const hashedPassword = await hashPassword(password);

    await db.query(
        'INSERT INTO users (email, password_hash) VALUES (?, ?)',
        [email, hashedPassword]
    );

    res.json({ message: 'User registered successfully' });
});
```

---

## Complete System Diagram

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                                     │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐  │
│  │  Web Browser     │  │  Mobile App      │  │  Admin Dashboard     │  │
│  └──────────────────┘  └──────────────────┘  └──────────────────────┘  │
└──────────────────────────────┬───────────────────────────────────────────┘
                               │ HTTPS
                               ↓
┌──────────────────────────────────────────────────────────────────────────┐
│                     API GATEWAY / LOAD BALANCER                          │
│  • Authentication                                                        │
│  • Rate Limiting                                                        │
│  • Request Routing                                                      │
│  • SSL/TLS Termination                                                  │
└──────────────────────────┬───────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┬──────────────────┐
        │                  │                  │                  │
        ↓                  ↓                  ↓                  ↓
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│  User Service    │ │ Product Service  │ │  Order Service   │ │ Payment Service  │
│                  │ │                  │ │                  │ │                  │
│ • Register       │ │ • Search         │ │ • Create order   │ │ • Process payment│
│ • Login          │ │ • Browse         │ │ • Order history  │ │ • Refund         │
│ • Profile        │ │ • Details        │ │ • Status track   │ │ • Reconciliation │
│ • Addresses      │ │ • Reviews        │ │ • Cancel order   │ │                  │
└──────────────────┘ └──────────────────┘ └──────────────────┘ └──────────────────┘
        │                  │                  │                  │
        ↓                  ↓                  ↓                  ↓
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Cart Service     │ │ Inventory Service│ │ Notification Svc │ │ Shipping Service │
│                  │ │                  │ │                  │ │                  │
│ • Add to cart    │ │ • Stock check    │ │ • Email          │ │ • Calculate cost │
│ • Update cart    │ │ • Reserve        │ │ • SMS            │ │ • Generate label │
│ • Checkout       │ │ • Release        │ │ • Push notif     │ │ • Track shipment │
│ • Apply coupon   │ │ • Movements      │ │ • In-app          │ │                  │
└──────────────────┘ └──────────────────┘ └──────────────────┘ └──────────────────┘
        │                  │                  │                  │
        └──────────────────┼──────────────────┴──────────────────┘
                           │
        ┌──────────────────┼──────────────────────┐
        │                  │                      │
        ↓                  ↓                      ↓
  ┌─────────────────────────────────────────────────────┐
  │          CACHING LAYER (Redis)                      │
  │  • Product cache        • User sessions            │
  │  • Cart data            • Auth tokens              │
  │  • Frequently accessed  • Rate limit counters      │
  └─────────────────────────────────────────────────────┘
        │
        ↓
  ┌─────────────────────────────────────────────────────┐
  │        MESSAGE QUEUE (Kafka)                        │
  │  • order-events                                    │
  │  • payment-events                                  │
  │  • inventory-events                                │
  │  • notification-events                             │
  │  • user-events                                     │
  └─────────────────────────────────────────────────────┘
        │
        ↓
  ┌─────────────────────────────────────────────────────┐
  │        DATA PERSISTENCE LAYER                       │
  ├─────────────────────────────────────────────────────┤
  │                                                     │
  │  PostgreSQL (Primary)      MongoDB (Catalog)       │
  │  ├─ Users                  ├─ Products             │
  │  ├─ Orders                 ├─ Reviews              │
  │  ├─ Payments               ├─ Catalog              │
  │  ├─ Inventory              └─ User Activity        │
  │  └─ Transactions                                   │
  │                            Elasticsearch           │
  │  Read Replicas             ├─ Full-text search    │
  │  ├─ Replica 1              ├─ Product index       │
  │  ├─ Replica 2              └─ Analytics           │
  │  └─ Replica 3                                      │
  │                                                     │
  └─────────────────────────────────────────────────────┘
        │
        ↓
  ┌─────────────────────────────────────────────────────┐
  │     EXTERNAL SERVICES & STORAGE                     │
  │  • AWS S3 (Images/Videos)                          │
  │  • CloudFront (CDN)                                │
  │  • Stripe/PayPal (Payment)                         │
  │  • SendGrid (Email)                                │
  │  • Twilio (SMS)                                    │
  │  • Firebase (Push)                                 │
  │  • BigQuery (Analytics)                            │
  └─────────────────────────────────────────────────────┘
        │
        ↓
  ┌─────────────────────────────────────────────────────┐
  │     MONITORING & LOGGING                            │
  │  • Prometheus (Metrics)                            │
  │  • ELK Stack (Logs)                                │
  │  • Jaeger (Tracing)                                │
  │  • Grafana (Dashboards)                            │
  │  • PagerDuty (Alerting)                            │
  └─────────────────────────────────────────────────────┘
```

---

## Interview Talking Points

### Start with Requirements Clarification
```
"Before designing, let me clarify the requirements:

1. Scale: How many DAU, QPS, peak traffic?
2. Consistency: Strong or eventual consistency?
3. Latency: Response time targets?
4. Availability: Uptime SLA required?
5. Regions: Global or single region?
6. Features: MVP vs. full platform?"
```

### Explain Your Choices
```
"I chose PostgreSQL for orders because:
- ACID compliance is critical for payments
- Consistency over availability is important
- I chose MongoDB for products because:
- Flexible schema for different product types
- High read throughput needed"
```

### Identify Trade-offs
```
"This design trades off:
- Consistency for availability in product catalog
- Storage for performance by caching
- Operational complexity for scalability"
```

### Discuss Scalability
```
"To handle 10x growth:
- Horizontal scaling: Add more service instances
- Database sharding by user_id
- Cache larger datasets
- Add CDN for static content
- Increase message queue partitions"
```

---

**Remember:** System design is about understanding your requirements, making informed trade-offs, and explaining your reasoning. There's no single "perfect" solution – it depends on your specific constraints and requirements.
