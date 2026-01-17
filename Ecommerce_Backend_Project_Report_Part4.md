## Database Schema Design

The E-commerce Backend Microservices Platform employs a polyglot persistence strategy, utilizing different database technologies optimized for specific use cases. This approach ensures optimal performance, scalability, and flexibility across different service domains.

### Database Technology Selection

The platform strategically employs different database technologies based on the specific requirements of each microservice.

**Table 3.1: Database Technology Mapping**

| Database Type | Technology | Services | Justification |
|---------------|------------|----------|---------------|
| Relational | MySQL 8.0 | User Service, Order Service | ACID compliance for transactional data, Complex relationships, Strong consistency requirements |
| Document | MongoDB 6.0 | Product Service, Cart Service, Payment Service, Notification Service | Flexible schema for varied product attributes, High-performance reads, Horizontal scaling |
| Cache | Redis 7.0 | All Services | Session management, Response caching, Real-time data |
| Search | Elasticsearch 8.6 | Product Service | Full-text search, Faceted navigation, Auto-suggestions |

**Figure 3.1: Database Schema Relationship Diagram**

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Database Architecture Overview                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   MySQL Databases                    MongoDB Collections            │
│   ┌─────────────┐                    ┌─────────────┐              │
│   │ ecommerce_  │                    │  products   │              │
│   │    users    │                    └─────────────┘              │
│   │             │                    ┌─────────────┐              │
│   │ ┌─────────┐ │                    │    carts    │              │
│   │ │  users  │ │                    └─────────────┘              │
│   │ └─────────┘ │                    ┌─────────────┐              │
│   │ ┌─────────┐ │                    │  payments   │              │
│   │ │password_│ │                    └─────────────┘              │
│   │ │ reset_  │ │                    ┌─────────────┐              │
│   │ │ tokens  │ │                    │notifications│              │
│   │ └─────────┘ │                    └─────────────┘              │
│   └─────────────┘                                                  │
│   ┌─────────────┐                    Redis Caches                 │
│   │ ecommerce_  │                    ┌─────────────┐              │
│   │   orders    │                    │  sessions   │              │
│   │             │                    └─────────────┘              │
│   │ ┌─────────┐ │                    ┌─────────────┐              │
│   │ │ orders  │ │                    │  products   │              │
│   │ └─────────┘ │                    └─────────────┘              │
│   │ ┌─────────┐ │                    ┌─────────────┐              │
│   │ │ order_  │ │                    │    carts    │              │
│   │ │ items   │ │                    └─────────────┘              │
│   │ └─────────┘ │                                                  │
│   └─────────────┘                                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### User Service Database Schema

The User Service utilizes MySQL for storing user authentication and profile data with strong consistency guarantees.

**Table 3.2: User Database Schema**

```sql
-- Users Table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    phone_number VARCHAR(20),
    address_line1 VARCHAR(255),
    address_line2 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(50),
    zip_code VARCHAR(20),
    country VARCHAR(50),
    is_active BOOLEAN DEFAULT true,
    email_verified BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_email (email),
    INDEX idx_username (username)
);

-- Roles Table
CREATE TABLE roles (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT
);

-- User Roles Junction Table
CREATE TABLE user_roles (
    user_id BIGINT,
    role_id INT,
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
);

-- Password Reset Tokens Table
CREATE TABLE password_reset_tokens (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    token VARCHAR(255) UNIQUE NOT NULL,
    expiry_date TIMESTAMP NOT NULL,
    used BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_token (token)
);

-- Audit Log Table
CREATE TABLE user_audit_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT,
    action VARCHAR(50),
    details JSON,
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL,
    INDEX idx_user_action (user_id, action),
    INDEX idx_created_at (created_at)
);
```

**Foreign Key Relationships:**
- user_roles.user_id → users.id (Many-to-Many with roles)
- user_roles.role_id → roles.id
- password_reset_tokens.user_id → users.id (One-to-Many)
- user_audit_log.user_id → users.id (One-to-Many)

### Product Service Database Schema

The Product Service uses MongoDB for flexible product attributes and efficient document-based queries.

**Table 3.3: Product Database Schema**

```javascript
// Products Collection
{
  "_id": ObjectId,
  "sku": String (unique),
  "name": String (required),
  "description": String,
  "category": {
    "id": ObjectId,
    "name": String,
    "path": Array // ["Electronics", "Computers", "Laptops"]
  },
  "brand": String,
  "price": {
    "amount": Decimal128,
    "currency": String,
    "discount": {
      "percentage": Number,
      "validUntil": Date
    }
  },
  "inventory": {
    "quantity": Number,
    "reserved": Number,
    "warehouse": String,
    "lowStockThreshold": Number
  },
  "attributes": {
    // Flexible key-value pairs
    "color": String,
    "size": String,
    "weight": Number,
    "dimensions": {
      "length": Number,
      "width": Number,
      "height": Number
    }
  },
  "images": [
    {
      "url": String,
      "alt": String,
      "isPrimary": Boolean
    }
  ],
  "ratings": {
    "average": Number,
    "count": Number,
    "distribution": {
      "5": Number,
      "4": Number,
      "3": Number,
      "2": Number,
      "1": Number
    }
  },
  "tags": [String],
  "status": String, // "active", "inactive", "discontinued"
  "createdAt": Date,
  "updatedAt": Date,
  "createdBy": String,
  "lastModifiedBy": String
}

// Categories Collection
{
  "_id": ObjectId,
  "name": String,
  "slug": String (unique),
  "description": String,
  "parentId": ObjectId (nullable),
  "path": String, // "/electronics/computers/laptops"
  "level": Number,
  "attributes": [String], // Required attributes for products in this category
  "image": String,
  "isActive": Boolean,
  "sortOrder": Number,
  "createdAt": Date,
  "updatedAt": Date
}

// Product Reviews Collection
{
  "_id": ObjectId,
  "productId": ObjectId,
  "userId": String,
  "rating": Number,
  "title": String,
  "comment": String,
  "images": [String],
  "isVerifiedPurchase": Boolean,
  "helpful": {
    "yes": Number,
    "no": Number
  },
  "status": String, // "pending", "approved", "rejected"
  "createdAt": Date,
  "updatedAt": Date
}
```

**Indexes:**
- products.sku (unique)
- products.category.id
- products.status, products.createdAt (compound)
- products.name (text index for search)
- categories.slug (unique)
- categories.parentId
- reviews.productId, reviews.status (compound)

### Order Service Database Schema

The Order Service uses MySQL to maintain transactional integrity for order processing and fulfillment.

**Table 3.4: Order Database Schema**

```sql
-- Orders Table
CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    user_id VARCHAR(50) NOT NULL,
    status ENUM('CREATED', 'PAYMENT_PENDING', 'PAYMENT_COMPLETED', 
                'PROCESSING', 'SHIPPED', 'DELIVERED', 'CANCELLED', 
                'REFUNDED') NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    
    -- Shipping Address
    shipping_street VARCHAR(255),
    shipping_city VARCHAR(100),
    shipping_state VARCHAR(50),
    shipping_zip_code VARCHAR(20),
    shipping_country VARCHAR(50),
    shipping_phone VARCHAR(20),
    
    -- Billing Address
    billing_street VARCHAR(255),
    billing_city VARCHAR(100),
    billing_state VARCHAR(50),
    billing_zip_code VARCHAR(20),
    billing_country VARCHAR(50),
    billing_phone VARCHAR(20),
    
    payment_id VARCHAR(100),
    payment_method VARCHAR(50),
    tracking_number VARCHAR(100),
    notes TEXT,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    shipped_at TIMESTAMP NULL,
    delivered_at TIMESTAMP NULL,
    
    INDEX idx_order_number (order_number),
    INDEX idx_user_id (user_id),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at)
);

-- Order Items Table
CREATE TABLE order_items (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT NOT NULL,
    product_id VARCHAR(50) NOT NULL,
    product_name VARCHAR(255) NOT NULL,
    product_sku VARCHAR(100),
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    subtotal DECIMAL(10, 2) NOT NULL,
    discount_amount DECIMAL(10, 2) DEFAULT 0,
    tax_amount DECIMAL(10, 2) DEFAULT 0,
    image_url VARCHAR(500),
    
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    INDEX idx_order_id (order_id),
    INDEX idx_product_id (product_id)
);

-- Order Status History Table
CREATE TABLE order_status_history (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT NOT NULL,
    status VARCHAR(50) NOT NULL,
    notes TEXT,
    created_by VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    INDEX idx_order_status (order_id, created_at)
);

-- Shipments Table
CREATE TABLE shipments (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT NOT NULL,
    carrier VARCHAR(100),
    tracking_number VARCHAR(100) UNIQUE,
    status VARCHAR(50),
    estimated_delivery DATE,
    actual_delivery DATE,
    shipping_cost DECIMAL(10, 2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    INDEX idx_tracking (tracking_number)
);
```

**Foreign Key Relationships:**
- order_items.order_id → orders.id (One-to-Many)
- order_status_history.order_id → orders.id (One-to-Many)
- shipments.order_id → orders.id (One-to-One)

**Cardinality of Relations:**
- Orders to Order Items: 1:M (One order can have multiple items)
- Orders to Status History: 1:M (One order can have multiple status changes)
- Orders to Shipments: 1:1 (One order has one shipment record)
- Users to Orders: 1:M (One user can have multiple orders)

**Figure 3.2: MongoDB Document Structure**

```javascript
// Cart Collection Structure
{
  "_id": ObjectId,
  "userId": String (indexed),
  "status": "active" | "abandoned" | "converted",
  "items": [
    {
      "productId": String,
      "productName": String,
      "quantity": Number,
      "price": Decimal128,
      "subtotal": Decimal128,
      "imageUrl": String,
      "addedAt": Date
    }
  ],
  "totalPrice": Decimal128,
  "appliedCoupons": [
    {
      "code": String,
      "discount": Number,
      "type": "percentage" | "fixed"
    }
  ],
  "createdAt": Date,
  "updatedAt": Date,
  "expiresAt": Date // For session-based carts
}

// Payment Collection Structure
{
  "_id": ObjectId,
  "orderId": String,
  "orderNumber": String (indexed),
  "userId": String,
  "amount": Decimal128,
  "currency": String,
  "status": "pending" | "processing" | "completed" | "failed" | "refunded",
  "paymentMethod": "credit_card" | "paypal" | "bank_transfer",
  "transactionId": String (indexed),
  "gatewayResponse": {
    "code": String,
    "message": String,
    "details": Object
  },
  "refunds": [
    {
      "amount": Decimal128,
      "reason": String,
      "processedAt": Date,
      "transactionId": String
    }
  ],
  "metadata": Object, // Flexible additional data
  "createdAt": Date,
  "updatedAt": Date
}
```

The database schema design demonstrates:

1. **Appropriate Technology Selection**: Each database technology is chosen based on specific service requirements
2. **Normalization vs. Denormalization**: Relational databases are properly normalized while document stores use denormalization for performance
3. **Indexing Strategy**: Strategic indexes on frequently queried fields ensure optimal query performance
4. **Data Integrity**: Foreign key constraints and validation rules maintain data consistency
5. **Scalability Considerations**: Schema design supports horizontal scaling and sharding where appropriate
6. **Audit and History**: Comprehensive tracking of changes and historical data for compliance and debugging
