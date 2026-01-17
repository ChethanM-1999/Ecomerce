## Class Diagrams

The low-level design of the E-commerce Backend Microservices Platform is represented through detailed class diagrams that illustrate the object-oriented structure of each microservice. These diagrams showcase the relationships between entities, services, controllers, and repositories, following Domain-Driven Design (DDD) principles and clean architecture patterns.

### User Service Class Design

The User Service implements a layered architecture with clear separation between presentation, business logic, and data access layers.

**Figure 2.1: User Service Class Diagram**

```
┌─────────────────────────────────────────────────────────────────────┐
│                         User Service Classes                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐         ┌──────────────┐      ┌──────────────┐   │
│  │AuthController│◄────────┤ UserService  │─────►│UserRepository│   │
│  └──────┬───────┘         └──────┬───────┘      └──────────────┘   │
│         │                        │                                   │
│         │                        ▼                                   │
│         │               ┌──────────────────┐                        │
│         │               │PasswordEncoder  │                        │
│         │               └──────────────────┘                        │
│         │                                                           │
│         ▼                        ▼                                   │
│  ┌──────────────┐      ┌──────────────────┐    ┌──────────────┐   │
│  │   JwtUtils   │      │  SessionService  │───►│RedisTemplate │   │
│  └──────────────┘      └──────────────────┘    └──────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐      │
│  │                      Entities                             │      │
│  ├──────────────────────────────────────────────────────────┤      │
│  │ User                                                      │      │
│  │ - id: Long                                               │      │
│  │ - username: String                                       │      │
│  │ - email: String                                          │      │
│  │ - password: String                                       │      │
│  │ - roles: Set<Role>                                       │      │
│  │ - createdAt: LocalDateTime                              │      │
│  │ - updatedAt: LocalDateTime                              │      │
│  ├──────────────────────────────────────────────────────────┤      │
│  │ PasswordResetToken                                       │      │
│  │ - id: Long                                              │      │
│  │ - token: String                                         │      │
│  │ - user: User                                           │      │
│  │ - expiryDate: LocalDateTime                           │      │
│  │ - used: boolean                                        │      │
│  └──────────────────────────────────────────────────────────┘      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

The User Service architecture demonstrates:
- **Separation of Concerns**: Controllers handle HTTP requests, services contain business logic, and repositories manage data persistence
- **Security Layer**: JWT utilities and password encoding ensure secure authentication
- **Session Management**: Redis integration provides distributed session handling
- **Event Publishing**: Kafka producer service enables event-driven communication

**Table 2.1: User Service API Endpoints**

| Endpoint | Method | Description | Request Body | Response |
|----------|--------|-------------|--------------|----------|
| /api/users/auth/signup | POST | Register new user | SignupRequest | MessageResponse |
| /api/users/auth/signin | POST | User login | LoginRequest | JwtResponse |
| /api/users/auth/logout | POST | User logout | - | MessageResponse |
| /api/users/auth/refresh | POST | Refresh token | TokenRefreshRequest | TokenRefreshResponse |
| /api/users/auth/reset-password | POST | Request password reset | PasswordResetRequest | MessageResponse |
| /api/users/profile | GET | Get user profile | - | UserResponse |
| /api/users/profile | PUT | Update profile | UserUpdateRequest | UserResponse |

### Product Service Class Design

The Product Service implements a document-based design pattern optimized for flexible product attributes and efficient search operations.

**Figure 2.2: Product Service Class Diagram**

```
┌─────────────────────────────────────────────────────────────────────┐
│                       Product Service Classes                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────┐      ┌─────────────────┐   ┌────────────────┐  │
│  │ProductController│◄─────┤ ProductService  │──►│ProductRepository│ │
│  └────────────────┘      └────────┬────────┘   └────────────────┘  │
│                                   │                                  │
│                                   ▼                                  │
│                          ┌─────────────────┐                        │
│                          │ElasticsearchOps │                        │
│                          └─────────────────┘                        │
│                                   │                                  │
│  ┌────────────────┐              ▼                                  │
│  │CategoryController│   ┌─────────────────┐    ┌────────────────┐  │
│  └────────┬────────┘    │  CacheManager   │    │KafkaProducer   │  │
│           │             └─────────────────┘    └────────────────┘  │
│           ▼                                                          │
│  ┌─────────────────┐                                               │
│  │ CategoryService │                                                │
│  └─────────────────┘                                               │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐      │
│  │                    Document Models                        │      │
│  ├──────────────────────────────────────────────────────────┤      │
│  │ Product                                                   │      │
│  │ - id: String                                             │      │
│  │ - name: String                                           │      │
│  │ - description: String                                    │      │
│  │ - price: BigDecimal                                     │      │
│  │ - stockQuantity: Integer                                │      │
│  │ - category: Category                                    │      │
│  │ - images: List<String>                                  │      │
│  │ - attributes: Map<String, Object>                       │      │
│  │ - ratings: List<Rating>                                 │      │
│  ├──────────────────────────────────────────────────────────┤      │
│  │ Category                                                  │      │
│  │ - id: String                                             │      │
│  │ - name: String                                           │      │
│  │ - parentCategory: Category                               │      │
│  │ - attributes: List<String>                               │      │
│  └──────────────────────────────────────────────────────────┘      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Table 2.2: Product Service API Endpoints**

| Endpoint | Method | Description | Request Body | Response |
|----------|--------|-------------|--------------|----------|
| /api/products | GET | List all products | - | Page<ProductResponse> |
| /api/products/{id} | GET | Get product by ID | - | ProductResponse |
| /api/products | POST | Create product | ProductRequest | ProductResponse |
| /api/products/{id} | PUT | Update product | ProductRequest | ProductResponse |
| /api/products/{id} | DELETE | Delete product | - | MessageResponse |
| /api/products/search | GET | Search products | SearchParams | Page<ProductResponse> |
| /api/products/categories | GET | List categories | - | List<CategoryResponse> |

### Cart Service Class Design

The Cart Service manages shopping cart operations with optimized caching strategies for improved performance.

**Figure 2.3: Cart Service Class Diagram**

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Cart Service Classes                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────┐       ┌────────────────┐   ┌────────────────┐  │
│  │ CartController │◄──────┤  CartService   │──►│ CartRepository │  │
│  └────────────────┘       └────────┬───────┘   └────────────────┘  │
│                                    │                                 │
│                                    ▼                                 │
│                          ┌──────────────────┐                       │
│                          │ProductServiceClient│                      │
│                          └──────────────────┘                       │
│                                    │                                 │
│                                    ▼                                 │
│                          ┌──────────────────┐   ┌────────────────┐ │
│                          │  RedisTemplate   │   │ KafkaProducer  │ │
│                          └──────────────────┘   └────────────────┘ │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐      │
│  │                     Domain Models                         │      │
│  ├──────────────────────────────────────────────────────────┤      │
│  │ Cart                                                      │      │
│  │ - id: String                                             │      │
│  │ - userId: String                                         │      │
│  │ - items: List<CartItem>                                 │      │
│  │ - totalPrice: BigDecimal                                │      │
│  │ - createdAt: LocalDateTime                              │      │
│  │ - updatedAt: LocalDateTime                              │      │
│  ├──────────────────────────────────────────────────────────┤      │
│  │ CartItem                                                  │      │
│  │ - productId: String                                      │      │
│  │ - productName: String                                    │      │
│  │ - quantity: Integer                                      │      │
│  │ - price: BigDecimal                                      │      │
│  │ - subtotal: BigDecimal                                   │      │
│  └──────────────────────────────────────────────────────────┘      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Order Service Class Design

The Order Service orchestrates the order lifecycle from creation through fulfillment, integrating with payment and notification services.

**Figure 2.4: Order Service Class Diagram**

```
┌─────────────────────────────────────────────────────────────────────┐
│                       Order Service Classes                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────┐      ┌─────────────────┐   ┌────────────────┐  │
│  │OrderController │◄─────┤  OrderService   │──►│OrderRepository │  │
│  └────────────────┘      └────────┬────────┘   └────────────────┘  │
│                                   │                                  │
│                          ┌────────┴────────┐                        │
│                          │                 │                        │
│                 ┌────────▼────────┐ ┌──────▼──────┐                │
│                 │CartServiceClient│ │PaymentClient│                │
│                 └─────────────────┘ └─────────────┘                │
│                                   │                                  │
│                          ┌────────▼────────┐                        │
│                          │ KafkaProducer   │                        │
│                          └─────────────────┘                        │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐      │
│  │                      Entity Models                        │      │
│  ├──────────────────────────────────────────────────────────┤      │
│  │ Order                                                     │      │
│  │ - id: Long                                               │      │
│  │ - orderNumber: String                                    │      │
│  │ - userId: String                                         │      │
│  │ - status: OrderStatus                                    │      │
│  │ - totalAmount: BigDecimal                               │      │
│  │ - shippingAddress: Address                              │      │
│  │ - billingAddress: Address                               │      │
│  │ - items: List<OrderItem>                                │      │
│  │ - paymentId: String                                      │      │
│  │ - trackingNumber: String                                 │      │
│  │ - createdAt: LocalDateTime                              │      │
│  │ - shippedAt: LocalDateTime                              │      │
│  │ - deliveredAt: LocalDateTime                            │      │
│  ├──────────────────────────────────────────────────────────┤      │
│  │ OrderItem                                                 │      │
│  │ - id: Long                                               │      │
│  │ - productId: String                                      │      │
│  │ - productName: String                                    │      │
│  │ - quantity: Integer                                      │      │
│  │ - price: BigDecimal                                      │      │
│  │ - subtotal: BigDecimal                                   │      │
│  └──────────────────────────────────────────────────────────┘      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

The class diagrams illustrate the well-structured, modular design of each microservice, emphasizing:

1. **Clear Separation of Layers**: Each service follows the MVC pattern with distinct controller, service, and repository layers
2. **Domain-Driven Design**: Entities and value objects accurately model the business domain
3. **Dependency Injection**: Services are loosely coupled through interface-based design
4. **External Integration**: Services communicate through well-defined clients and message producers
5. **Cross-Cutting Concerns**: Caching, security, and messaging are consistently implemented across services
