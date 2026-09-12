# SpringEcom – E-Commerce REST API (Spring Boot)

A full-stack E-Commerce application: a **Java / Spring Boot** REST API backend paired with a **React** frontend, in a single repository. The backend implements product catalog management and order placement/tracking; the frontend (prebuilt, provided as part of the learning course) is used to interact with and demonstrate the API.

## 🚧 Project Status

This is a **learning project and still under active development** — not a production-ready, full-featured e-commerce app. Core backend product and order flows are implemented; the frontend is prebuilt (not authored by me) and used to demo the API. Features like authentication, payments, and a cart are not yet built (see [Future Improvements](#-future-improvements)).

## 🚀 About The Project

SpringEcom is being built to understand and apply real-world backend concepts — REST APIs, layered architecture, Spring Data JPA, entity relationships, and DTO-based data transfer.

The application is organized into clear layers: **Controller → DTO → Service → Repository → Entity**, with the DTO layer decoupling the API contract from the database schema.

> **Note:** This repository contains both the backend and frontend. The backend (this README's primary focus) was built by me as a learning exercise. The frontend is a **prebuilt React app** (provided as part of the learning course) included here for convenience so the whole project can be cloned and run from one place — it was **not built by the author**.

## ✨ Current Features

**Product Management**
- Add, view, update, and delete products
- Fetch a single product by ID
- Search products by keyword (matches name, description, brand, category)
- Upload and retrieve product images (stored as `byte[]` in the database)
- Track stock quantity and availability

**Order Management**
- Place a new order with one or more items
- Automatically compute item and order totals
- Retrieve all orders with their status, items, and totals
- Basic order status tracking (e.g. `PLACED`)

**Architecture**
- RESTful API with clear separation of concerns
- Database persistence via Spring Data JPA / Hibernate
- PostgreSQL integration
- DTO layer to keep API contracts independent from entity models
- CORS configuration for frontend integration

> Not yet implemented: authentication/authorization, shopping cart, payments, pagination, and other items listed under [Future Improvements](#-future-improvements).

## 🛠️ Tech Stack

**Backend:** Java, Spring Boot, Spring Web MVC, Spring Data JPA, Hibernate, Lombok, Maven

**Frontend:** React *(prebuilt, included in this repo under `/frontend`)*

**Database:** PostgreSQL

**Tools:** IntelliJ IDEA, Postman, Git, GitHub

## 🏗️ Project Architecture

```text
Client / Frontend
       │
       ▼
Controller Layer   (ProductController, OrderController)
       │
       ▼  (uses DTOs: OrderRequest / OrderResponse)
DTO Layer
       │
       ▼
Service Layer      (ProductService, OrderService)
       │
       ▼
Repository Layer   (ProductRepo, OrderRepo)
       │
       ▼
Spring Data JPA / Hibernate
       │
       ▼
PostgreSQL Database (Product, Order, OrderItem)
```

### Layer Breakdown

**1. Application Entry Point**
- `ProjectEcomApplication` — annotated with `@SpringBootApplication`, boots the app.

**2. Controller Layer** (`controller` package, `@RestController`)
- `ProductController`
  - `getProducts()` – get all products
  - `getProductById()` – get product by ID
  - `getProductImage()` / `getImageByProductId()` – get product image
  - `addProduct()` – add a new product
  - `updateProduct()` – update an existing product
  - `deleteProduct()` – delete a product
  - `searchProducts()` – search products by keyword
- `OrderController`
  - `placeOrder()` – place a new order
  - `getAllOrders()` – get all orders
- `HelloController` – simple health-check / greeting endpoint (`greet()`)

**3. Service Layer** (`service` package, `@Service`)
- `ProductService`
  - `getAllProducts()`, `getProductById()`
  - `addOrUpdateProduct()` – save/update product along with its image
  - `deleteProduct()`, `searchProducts()`
- `OrderService`
  - `placeOrder()` – converts `OrderRequest` into an `Order` entity, calculates totals, and saves it
  - `getAllOrderResponses()` – converts saved orders into `OrderResponse` DTOs

**4. Repository Layer** (`repository` package, `@Repository`)
- `ProductRepo extends JpaRepository<Product, Integer>`
  - `findAll()`, `findById()`, `save()`, `deleteById()`
  - `searchProducts()` – custom JPQL query on `name`, `description`, `brand`, `category`
- `OrderRepo extends JpaRepository<Order, Long>`
  - `findAll()`, `save()`, `findByOrderId()`, `findByUserEmail()`

**5. Entity / Model Layer** (`model` package, `@Entity`)
- `Product` — `int id` (`@Id @GeneratedValue`), `String name`, `String description`, `String brand`, `BigDecimal price`, `String category`, `Date releaseDate`, `boolean productAvailable`, `int stockQuantity`, `String imageName`, `String imageType`, `byte[] imageData` (`@Lob`). Uses Lombok `@Data`.
- `Order` (`@Entity(name = "orders")`) — `Long id`, `String orderId`, `String customerName`, `String email`, `String status`, `LocalDate orderDate`, `List<OrderItem> items`
- `OrderItem` — `Long id`, `Product product`, `int quantity`, `BigDecimal totalPrice`, `Order order`
- Relationship: one `Order` **has many** `OrderItem`s, each `OrderItem` references one `Product`.

**6. DTO Layer** (`model.dto` package, Java `record`s)
- `OrderRequest` – `customerName`, `email`, `List<OrderItemRequest> items` *(incoming)*
- `OrderItemRequest` – `productId`, `quantity`, `price` *(incoming)*
- `OrderResponse` – `orderId`, `customerName`, `email`, `status`, `orderDate`, `List<OrderItemResponse> items` *(outgoing)*
- `OrderItemResponse` – `productName`, `quantity`, `totalPrice` *(outgoing)*

## 🔗 API Endpoints

### Product Endpoints

| Method | Endpoint                        | Description                          |
| ------ | -------------------------------- | ------------------------------------- |
| GET    | `/api/products`                  | Get all products                      |
| GET    | `/api/products/{id}`             | Get product by ID                     |
| GET    | `/api/products/{id}/image`       | Get product image                     |
| POST   | `/api/products`                  | Add a new product (multipart/form-data) |
| PUT    | `/api/products/{id}`             | Update an existing product            |
| DELETE | `/api/products/{id}`             | Delete a product                      |
| GET    | `/api/products/search?keyword=`  | Search products by keyword            |

### Order Endpoints

| Method | Endpoint       | Description        |
| ------ | -------------- | ------------------- |
| POST   | `/api/orders`  | Place a new order    |
| GET    | `/api/orders`  | Get all orders        |

**Sample Order Request Payload** (`POST /api/orders`):

```json
{
  "customerName": "John Doe",
  "email": "john@example.com",
  "items": [
    {
      "productId": 1,
      "quantity": 2,
      "price": 99.99
    }
  ]
}
```

## 🗄️ Database

The application uses **PostgreSQL**. Spring Data JPA and Hibernate map the `Product`, `Order`, and `OrderItem` entities to database tables.

Configure your database connection in `backend/src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/springecom
spring.datasource.username=postgres
spring.datasource.password=your_password

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

**Do not commit your real database password or other sensitive credentials to GitHub.**

## 🖥️ Frontend

The `/frontend` folder contains a **prebuilt React app** (provided as part of the learning course this project is based on) used to visualize and interact with the backend API — product listing, order placement, and order management views. It was **not developed by the author**; it's included in this repo purely so the full project can be cloned and run from one place.

See [How To Run](#️-how-to-run) below for setup steps.

## ⚙️ How To Run

### Prerequisites
- Java JDK 17 or higher
- Maven 3.8+
- PostgreSQL running instance
- Node.js and npm (for the frontend)

### 1. Clone the repository

```bash
git clone https://github.com/abijeetomar/springecom.git
cd springecom
```

### 2. Run the backend

1. Configure PostgreSQL: make sure it's running, create the database, and update credentials in:
   ```text
   backend/src/main/resources/application.properties
   ```
2. Build and run:
   ```bash
   cd backend
   mvn clean install
   mvn spring-boot:run
   ```
   Or run `ProjectEcomApplication.java` directly from IntelliJ IDEA.

   By default the backend runs on `http://localhost:8080`.

### 3. Run the frontend

```bash
cd frontend
npm install
npm start
```

By default the React app runs on `http://localhost:5173` and calls the backend at `http://localhost:8080`. Update the API base URL in the frontend config if your backend runs elsewhere.

### 4. Test the APIs directly (optional)

Use Postman or cURL to hit the endpoints listed above without the frontend.

## 📂 Project Structure

```text
springecom/
├── backend/
│   └── src
│       └── main
│           ├── java
│           │   └── com.Abijeet.Project.e_com
│           │       ├── controller
│           │       │   ├── HelloController
│           │       │   ├── OrderController
│           │       │   └── ProductController
│           │       ├── service
│           │       │   ├── OrderService
│           │       │   └── ProductService
│           │       ├── repository
│           │       │   ├── OrderRepo
│           │       │   └── ProductRepo
│           │       ├── model
│           │       │   ├── Order
│           │       │   ├── OrderItem
│           │       │   ├── Product
│           │       │   └── dto
│           │       │       ├── OrderRequest
│           │       │       ├── OrderItemRequest
│           │       │       ├── OrderResponse
│           │       │       └── OrderItemResponse
│           │       └── ProjectEcomApplication
│           └── resources
│               ├── static
│               ├── templates
│               └── application.properties
│
└── frontend                (prebuilt React app — see Frontend section above)
    
```

> Update folder names to match your actual repo layout (e.g. if the frontend/backend live at the repo root instead of in subfolders).

## 🔮 Future Improvements

- User authentication and authorization (Spring Security + JWT)
- Shopping cart functionality
- Payment gateway integration (Stripe / Razorpay)
- Pagination and sorting for product and order listings
- Order status updates (e.g. shipped, delivered, cancelled)
- Cloud deployment and containerization (Docker)

## 📚 What I'm Learning

- Building REST APIs with Spring Boot across multiple resources (Products & Orders)
- Designing a layered backend with a dedicated DTO layer for clean API contracts
- Modeling one-to-many entity relationships with Spring Data JPA
- Handling file/image uploads and storage as `byte[]`
- Writing custom JPQL queries for multi-field search
- Transforming between entities and DTOs (request → entity → response)
- Testing REST APIs with Postman

## 👨‍💻 Author

**Abijeet Tomar**
Computer Engineering Student

GitHub: [github.com/abijeetomar](https://github.com/abijeetomar)

---

⭐ If you found this project useful, consider giving the repository a star.
