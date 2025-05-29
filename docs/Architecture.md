# BlogAPI - Software Architecture Document

**Version:** 1.0
**Last Updated:** May 26, 2025

## 1. Introduction

This document provides a comprehensive overview of the BlogAPI's software architecture. It details the architectural style, layers, components, and design decisions that shape the system. The goal is to ensure a scalable, maintainable, and robust backend for the blogging platform.

## 2. Architectural Goals

*   **Scalability:** The architecture should support growth in users, data, and traffic.
*   **Maintainability:** The system should be easy to understand, modify, and extend.
*   **Performance:** Optimize for low latency and efficient resource utilization.
*   **Testability:** Components should be designed for easy unit and integration testing.
*   **Security:** Implement security best practices at all layers.
*   **Modularity:** Clear separation of concerns between different parts of the application.

## 3. System Architecture

### 3.1. Architectural Style & Diagram

BlogAPI primarily follows a **Layered Architecture** style, also incorporating aspects of **Domain-Driven Design (DDD)** for organizing the core business logic. This promotes separation of concerns and modularity.

The main layers are:
*   **Presentation/API Layer:** Handles incoming HTTP requests (REST and GraphQL) and user interactions.
*   **Service Layer (Application/Business Logic):** Contains the core business logic, orchestrates tasks, and acts as an intermediary between the API layer and the Data Access Layer.
*   **Data Access Layer (DAL):** Responsible for data persistence and retrieval.
*   **Infrastructure Layer:** Provides supporting technical capabilities like database connections, logging, and external service integrations.

```mermaid
graph TD
    subgraph UserFacing as "Clients (Web/Mobile/CLI)"
        Client[Browser/Mobile App/CLI Tool]
    end

    subgraph BlogAPISystem as "BlogAPI Backend (Node.js)"
        direction LR
        subgraph PresentationApiLayer as "Presentation/API Layer"
            direction TB
            RestApi["Express.js (REST API - /api/v1)"]
            GraphQLApi["Apollo Server (GraphQL API - /graphql)"]
            Middleware["Middleware (Auth, Validation, RBAC, Logging, Error Handling)"]
        end

        subgraph ServiceLayer as "Service Layer (Business Logic)"
            direction TB
            AuthService["AuthService"]
            PostService["PostService"]
            UserService["UserService"]
            TagService["TagService"]
            CommentService["CommentService"]
            VersioningService["VersioningService (for Posts)"]
        end

        subgraph DataAccessLayer as "Data Access Layer"
            direction TB
            Repositories["Repositories (e.g., PostRepository, UserRepository)"]
            Orm["TypeORM (ORM)"]
            Entities["TypeORM Entities (Data Models)"]
        end

        subgraph InfrastructureLayer as "Infrastructure Layer"
            direction TB
            DatabaseConn["Database Connection (PostgreSQL)"]
            SecurityUtils["Security Utilities (JWT, bcrypt)"]
            LoggingFramework["Logging Framework (e.g., Winston)"]
            ConfigMgmt["Configuration Management (.env)"]
        end

        Database[(PostgreSQL Database)]
    end

    Client --> RestApi
    Client --> GraphQLApi

    RestApi --> Middleware
    GraphQLApi --> Middleware
    Middleware --> ServiceLayer

    ServiceLayer --> DataAccessLayer
    ServiceLayer --> InfrastructureLayer

    DataAccessLayer --> Database
    InfrastructureLayer --> DatabaseConn
    DatabaseConn -.-> Database
```

**Key Interactions:**

*   Clients interact with the API Layer (REST via Express.js or GraphQL via Apollo Server).
*   The API Layer uses Middleware for cross-cutting concerns like authentication, authorization, and validation.
*   The API Layer (Controllers/Resolvers) delegates business logic execution to the Service Layer.
*   The Service Layer uses the Data Access Layer (Repositories using TypeORM) to interact with the PostgreSQL database.
*   The Infrastructure Layer provides support services like logging, configuration, and database connectivity.

### 3.2. Key Components and Responsibilities

#### Presentation/API Layer

*   **REST Controllers (Express.js):**
    *   Located in `src/api/rest/v1/controllers/`.
    *   Handle incoming HTTP requests for specific REST API routes (e.g., `/users`, `/posts`).
    *   Utilize Express.js routing (`app.get()`, `app.post()`, etc.) and request/response objects (`req`, `res`).
    *   Responsible for request parsing, basic input validation (often delegated to middleware), and calling appropriate methods in the Service Layer.
    *   Format and send HTTP responses (JSON).
*   **GraphQL Resolvers (Apollo Server):**
    *   Located in `src/api/graphql/resolvers/`.
    *   Define how data is fetched or mutated for each field in the GraphQL schema.
    *   Apollo Server integrates with Express.js to handle requests to the `/graphql` endpoint.
    *   Resolvers interact with the Service Layer to perform actions and retrieve data.
*   **Middleware (Express.js):**
    *   Located in `src/api/rest/middleware/` or integrated within framework modules.
    *   Functions that execute during the request-response cycle of Express.js.
    *   Used for:
        *   Authentication (verifying JWTs).
        *   Authorization (RBAC - Role-Based Access Control).
        *   Input Validation (validating request bodies, parameters, query strings).
        *   Logging HTTP requests.
        *   Error Handling (centralized error processing).
        *   CORS (Cross-Origin Resource Sharing).

#### Service Layer (Application/Business Logic)

*   Located in `src/core/services/`.
*   Encapsulates the core business rules and application logic.
*   Orchestrates operations by coordinating calls to repositories and other services.
*   Independent of the specific API technology (REST/GraphQL) and data persistence details.
*   Examples: `PostService`, `UserService`, `AuthService`.

#### Data Access Layer (DAL)

*   Located primarily in `src/infrastructure/persistence/`.
*   **ORM:** TypeORM is used as the Object-Relational Mapper.
*   **TypeORM Entities:** Data structures that map to database tables are defined as TypeORM entities (e.g., `User`, `Post` entities in `src/core/domain/entities/` or `src/infrastructure/persistence/entities/`). These are plain classes decorated with TypeORM decorators (`@Entity()`, `@Column()`, `@ManyToOne()`, etc.).
*   **Repositories:**
    *   Custom repositories (e.g., `PostRepository.ts`, `UserRepository.ts`) are implemented to encapsulate data access logic for specific entities.
    *   These repositories use TypeORM's `EntityManager` or `DataSource` (or extend `Repository` from TypeORM) to perform CRUD operations and complex queries against the PostgreSQL database.
    *   They work with TypeORM entities, abstracting the raw SQL.
*   **Database Migrations:** TypeORM migrations are used to manage database schema changes versionally.

#### Domain/Entities Layer

*   Located in `src/core/domain/entities/` (or directly used as TypeORM entities in the DAL).
*   Represents the core data structures and domain objects of the application (e.g., User, Post, Tag, Comment).
*   These are typically plain TypeScript classes or interfaces, which are then decorated as TypeORM entities for persistence.
*   May contain domain-specific validation logic or methods if following a richer domain model approach.

#### Infrastructure Layer

*   Located in `src/infrastructure/`.
*   Provides technical capabilities that support other layers.
*   **Database Connection:** Manages the connection to the PostgreSQL database, typically configured via TypeORM's `DataSource` options.
*   **Security Utilities:** Includes components for JWT handling (`jsonwebtoken`), password hashing (`bcrypt`), etc.
*   **Logging:** Configuration and integration of a logging framework (e.g., Winston, Pino).
*   **Configuration Management:** Loading and providing access to environment variables and configuration settings (e.g., using `dotenv`).
*   **External Service Integrations:** (If any) Clients for interacting with third-party APIs.

### 3.3. Data Flow

*(Placeholder for typical data flow examples)*

## 4. Design Decisions and Considerations

### 4.1. API Design

BlogAPI provides both RESTful and GraphQL APIs to cater to different client needs and use cases.

*   **REST API (Express.js):**
    *   Implemented using the Express.js framework.
    *   Follows standard REST principles with resource-based URLs (e.g., `/api/v1/users`, `/api/v1/posts`).
    *   Uses standard HTTP methods (GET, POST, PUT, DELETE) for CRUD operations.
    *   Stateless: Each request from a client contains all the information needed to process the request.
    *   Authentication is primarily JWT-based.
    *   **API Versioning:** The API is versioned via the URL path (e.g., `/api/v1/`). This allows for future iterations of the API without breaking existing client integrations.
    *   Request and response bodies are primarily JSON.
*   **GraphQL API (Apollo Server):**
    *   Implemented using Apollo Server, a comprehensive GraphQL server library.
    *   Provides a single endpoint (typically `/graphql`) for all GraphQL operations (queries, mutations, subscriptions).
    *   Apollo Server is integrated as middleware within the Express.js application, allowing both REST and GraphQL APIs to run on the same server instance.
    *   Clients can request exactly the data they need, reducing over-fetching and under-fetching.
    *   Strongly typed schema defines the API capabilities.
    *   Supports real-time updates via Subscriptions (if implemented).

### 4.2. Data Management

Effective data management is crucial for the integrity and performance of BlogAPI.

*   **Database:**
    *   **PostgreSQL** is the chosen relational database management system (RDBMS). It offers robustness, ACID compliance, advanced querying capabilities, and good support for JSON/JSONB data types, which can be beneficial for flexible data storage.
*   **Data Models (Entities):**
    *   Data models are defined as **TypeORM entities**. These are TypeScript classes decorated with TypeORM decorators (`@Entity`, `@Column`, `@PrimaryGeneratedColumn`, `@ManyToOne`, `@ManyToMany`, etc.) that map to tables and columns in the PostgreSQL database.
    *   Entities encapsulate the structure and relationships of the application's data (e.g., `User`, `Post`, `Tag`, `Comment`).
*   **Repository Pattern:**
    *   The Data Access Layer utilizes the Repository pattern to abstract data persistence logic.
    *   Custom repositories are created for each entity (e.g., `UserRepository`, `PostRepository`). These repositories typically extend TypeORM's `Repository` class or use TypeORM's `EntityManager` or `DataSource` directly to interact with the database.
    *   This pattern centralizes data access operations, making the service layer cleaner and data access logic more testable and maintainable.
*   **Migrations:**
    *   Database schema changes are managed using **TypeORM's migration system**.
    *   Migrations are written in TypeScript, allowing for version-controlled, programmatic updates to the database schema. This ensures consistency across different development, testing, and production environments.
    *   Commands like `typeorm migration:generate` and `typeorm migration:run` are used to create and apply migrations.
*   **Data Integrity & Validation:**
    *   Constraints (e.g., `NOT NULL`, `UNIQUE`) are defined at the entity/database level.
    *   Business rule validations are handled in the service layer before data persistence.

## 5. Cross-Cutting Concerns

These are aspects of the system that affect multiple architectural layers.

### 5.1. Security Architecture

*   **Authentication:**
    *   Authentication is primarily handled using **JSON Web Tokens (JWT)**.
    *   Upon successful login (credentials verified against stored hashes), a JWT is generated using the `jsonwebtoken` library.
    *   Passwords are securely stored in the database after being hashed using **`bcrypt`**, a strong adaptive hashing algorithm.
    *   Protected routes require a valid JWT in the `Authorization` header (Bearer scheme).
*   **Authorization (RBAC):**
    *   Role-Based Access Control is implemented to restrict access to certain API endpoints or operations based on user roles (e.g., admin, editor, user).
    *   This is typically enforced using custom middleware in the API layer that checks the user's role (extracted from the JWT) against the required permissions for the requested resource.
*   **Input Validation:**
    *   All incoming data from client requests (request bodies, query parameters, path parameters) is rigorously validated.
    *   Libraries like **`class-validator`** (often used with `class-transformer`) are employed, especially with DTOs (Data Transfer Objects) in the API layer (Express.js controllers), to ensure data integrity and prevent common vulnerabilities like injection attacks.
*   **HTTPS:**
    *   In production environments, all API communication must be over HTTPS to encrypt data in transit. This is typically handled by a reverse proxy (e.g., Nginx, Traefik) or load balancer.
*   **CORS (Cross-Origin Resource Sharing):**
    *   Appropriate CORS policies are configured in the Express.js application to control access from different domains, preventing unwanted cross-origin requests.
*   **Other Considerations:**
    *   Regular dependency updates and vulnerability scanning (e.g., `npm audit`).
    *   Security headers (e.g., Helmet.js middleware for Express.js).
    *   Rate limiting to prevent abuse.

### 5.2. Scalability and Performance

*   **Stateless Application:** The Node.js/Express.js backend is designed to be stateless, meaning each request can be handled independently by any instance of the application. This facilitates horizontal scaling (running multiple instances behind a load balancer).
*   **Asynchronous Operations:** Node.js's non-blocking I/O model is leveraged for efficient handling of concurrent requests. Promises and `async/await` are used extensively.
*   **Connection Pooling:** TypeORM manages database connection pooling to efficiently handle database connections to PostgreSQL.
*   **Query Optimization:**
    *   TypeORM provides tools for building efficient database queries. Careful selection of query methods, eager/lazy loading strategies, and indexing in PostgreSQL are important for performance.
    *   Analyzing slow queries and optimizing them is part of ongoing maintenance.
*   **Caching (Potential):** For frequently accessed, rarely changing data, a caching layer (e.g., Redis) could be introduced to reduce database load and improve response times. (This is a future consideration if needed).
*   **Load Balancing:** In a multi-instance deployment, a load balancer distributes traffic across instances.

### 5.3. Error Handling and Logging

*   **Centralized Error Handling (Express.js):**
    *   Custom error-handling middleware is implemented in Express.js. This middleware catches errors occurring during request processing (both synchronous and asynchronous) and standardizes the error response format sent to the client (e.g., JSON with appropriate HTTP status codes).
*   **Logging:**
    *   **Winston** is used as the primary logging library.
    *   **Structured logging** (e.g., JSON format) is implemented to make logs easily parsable, searchable, and analyzable by log management systems (e.g., ELK stack, Splunk).
    *   Logs include timestamps, log levels (error, warn, info, debug), context information (e.g., request ID, user ID), and error details (stack traces).
    *   Different log levels are used for development and production environments.

### 5.4. Configuration Management

*   Application configuration is managed using **environment variables**.
*   For local development, environment variables are loaded from **`.env` files** using the **`dotenv`** library.
*   In production environments, environment variables are typically injected by the deployment platform or container orchestration system.
*   Sensitive information (API keys, database credentials, JWT secrets) is never hardcoded into the codebase.

## 6. Deployment View

This section describes the typical deployment setup for the BlogAPI.

### 6.1. Containerization

*   The application is containerized using **Docker**.
*   A `Dockerfile` defines the image for the BlogAPI application, including the Node.js runtime, application code, dependencies, and necessary configurations.
*   Docker Compose might be used for local development to orchestrate the application container and a PostgreSQL database container.

### 6.2. CI/CD (Continuous Integration/Continuous Deployment)

*   **GitHub Actions** is used for automating the CI/CD pipeline.
*   The pipeline typically includes steps for:
    1.  **Linting:** Checking code quality.
    2.  **Unit & Integration Testing:** Running automated tests.
    3.  **Building:** Compiling TypeScript to JavaScript and creating a production build.
    4.  **Docker Image Building & Pushing:** Building the Docker image and pushing it to a container registry (e.g., Docker Hub, GitHub Container Registry, AWS ECR).
    5.  **Deployment:** Automatically deploying the new image to staging and production environments.

### 6.3. Production Environment Example

A typical production deployment might look like this:

```mermaid
graph TD
    subgraph Internet as "Internet Users"
        UserClient["User's Client (Browser/App)"]
    end

    subgraph CloudProvider as "Cloud Provider (e.g., AWS, GCP, Azure)"
        LB["Load Balancer (HTTPS Termination, SSL)"]

        subgraph AppServers as "Application Servers (Auto-Scaling Group)"
            direction LR
            AppContainer1["Docker Container: BlogAPI (Node.js/Express.js)"]
            AppContainer2["Docker Container: BlogAPI (Node.js/Express.js)"]
            AppContainer3["Docker Container: BlogAPI (Node.js/Express.js)"]
        end

        subgraph DatabaseService as "Managed Database Service"
            PostgresDB["PostgreSQL Database"]
        end

        subgraph LoggingMonitoring as "Logging & Monitoring"
            LogMgmt["Log Management (e.g., ELK, CloudWatch Logs)"]
            Monitor["Monitoring & Alerting (e.g., Prometheus, Grafana, CloudWatch)"]
        end
    end

    UserClient --> LB
    LB --> AppContainer1
    LB --> AppContainer2
    LB --> AppContainer3

    AppContainer1 --> PostgresDB
    AppContainer2 --> PostgresDB
    AppContainer3 --> PostgresDB

    AppContainer1 --> LogMgmt
    AppContainer2 --> LogMgmt
    AppContainer3 --> LogMgmt
    AppContainer1 --> Monitor
    AppContainer2 --> Monitor
    AppContainer3 --> Monitor
```
*   **Load Balancer:** Distributes incoming traffic to multiple instances of the application. Handles SSL termination.
*   **Application Instances:** Docker containers running the BlogAPI application. These can be scaled horizontally.
*   **Database:** A managed PostgreSQL instance (e.g., AWS RDS, Google Cloud SQL) for reliability and scalability.
*   **Logging/Monitoring:** Centralized logging and monitoring services.

## 6. Future Considerations

*(Placeholder: Potential future enhancements, areas for refactoring, technology roadmap)*
