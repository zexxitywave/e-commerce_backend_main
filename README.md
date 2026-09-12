# E-Commerce Backend Application

A robust and secure e-commerce backend REST API built with Java and Spring Boot. It handles user authentication (JWT + HTTP Basic), product catalog management, shopping cart operations, order processing, and order history tracking.

## Tech Stack

- **Language:** Java 21
- **Framework:** Spring Boot 4.1.1, Spring Data JPA (Hibernate)
- **Security:** Spring Security, JWT, HTTP Basic
- **Database:** MySQL 8
- **Build:** Maven

## Features

- User registration, login, and stateless authentication using JWT
- HTTP Basic auth fallback with a fixed admin user
- Product catalog CRUD operations
- Shopping cart management (add, view, update, remove)
- Order checkout and order history per user
- Global exception handling returning standardized JSON errors

## Getting Started

### Prerequisites

- Java 21
- Maven 3.9+
- MySQL 8 running on `localhost:3306`

### 1. Start MySQL

```bash
docker run -d --name ecommerce-mysql \
  -e MYSQL_ROOT_PASSWORD=Thamizh@123 \
  -e MYSQL_DATABASE=ecommerce_db \
  -p 3306:3306 mysql:8.0
```

Tables are created automatically via `spring.jpa.hibernate.ddl-auto=update`.

### 2. Configure Database

Edit `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce_db?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=your_password
```

### 3. Run

```bash
mvnw.cmd spring-boot:run
# or
mvn spring-boot:run
```

The app starts at http://localhost:8080.

### 4. Verify

```bash
# Health check (use admin / admin123 when prompted)
curl http://localhost:8080/test
```

## Authentication

Two ways to access protected endpoints:

1. **JWT (recommended):** call `/api/users/login`, copy the returned token, send it as:
   ```
   Authorization: Bearer <jwt-token>
   ```
2. **HTTP Basic:** `admin` / `admin123` (fixed in-memory user, works for any protected endpoint)

Public endpoints: `POST /api/users/register`, `POST /api/users/login`.

## API Reference

All endpoints are under `http://localhost:8080`.

### Auth

**POST /api/users/register** — register a user
```json
{
  "name": "Rahul",
  "email": "rahul@test.com",
  "password": "pass123",
  "role": "CUSTOMER"
}
```
`role`: `CUSTOMER` or `ADMIN`.

**POST /api/users/login** — returns a JWT token
```json
{
  "email": "rahul@test.com",
  "password": "pass123"
}
```

**GET /api/users/all** — list all users (auth)

### Products (auth)

**GET /api/products/all** — list all products

**POST /api/products/add**
```json
{
  "productName": "Laptop",
  "description": "16GB RAM, 512GB SSD",
  "price": 99999.99,
  "stock": 10
}
```

**PUT /api/products/{id}**
```json
{
  "productName": "Laptop Pro",
  "description": "32GB RAM, 1TB SSD",
  "price": 129999.00,
  "stock": 5
}
```

**DELETE /api/products/{id}** — delete a product

### Cart (auth)

**GET /api/cart/{email}** — get cart for user, e.g. `/api/cart/rahul@test.com`

**POST /api/cart/add/{email}**
```json
{
  "productId": 1,
  "productName": "Laptop",
  "price": 99999.99,
  "quantity": 2
}
```

**DELETE /api/cart/remove/{email}/{productId}** — remove an item

**PUT /api/cart/update/{email}/{productId}/{quantity}** — update quantity

### Orders (auth)

**POST /api/orders/place** — place an ad-hoc order
```json
{
  "userEmail": "rahul@test.com",
  "totalAmount": 199999.98,
  "orderStatus": "SUCCESS",
  "productsSummary": ["Laptop (Qty: 2)"]
}
```

**POST /api/orders/checkout/{email}** — build an order from the cart and clear it

**GET /api/orders/user/{email}** — order history for a user

### Health

**GET /test** — health check, returns a success message

## Project Structure

```text
ecommerce-backend/
├── src/
│   ├── main/
│   │   ├── java/com/ecommerce/ecommerce_backend/
│   │   │   ├── controller/      # REST endpoints
│   │   │   ├── exception/       # global exception handler
│   │   │   ├── model/           # JPA entities
│   │   │   ├── repository/      # Spring Data JPA repositories
│   │   │   ├── security/        # JWT filter, security config, utils
│   │   │   └── service/         # business logic
│   │   └── resources/
│   │       └── application.properties
│   └── test/
└── pom.xml
```

## Note

Update the credentials in `application.properties` (and the hardcoded JWT secret in `JwtUtil.java`) before deploying to production.