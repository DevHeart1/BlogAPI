# Tech Stack Document: BlogAPI Backend

**Version:** 1.0
**Date:** May 26, 2025
**Status:** Finalized

## 1. Introduction

This document details the proposed technology stack for the BlogAPI backend project. The choices outlined here are based on the requirements specified in the Product Requirements Document (PRD) and aim to create a robust, scalable, and maintainable system.

As BlogAPI is a backend-only project, this document focuses exclusively on server-side technologies, databases, APIs, and related development and deployment tools.

## 2. Guiding Principles for Technology Selection

*   **Scalability:** Technologies should support horizontal and vertical scaling to handle growing data and user load.
*   **Performance:** Chosen stack should enable efficient request processing and low latency.
*   **Maintainability:** Prefer technologies with strong community support, good documentation, and established best practices.
*   **Developer Productivity:** Tools and frameworks that enhance developer workflow and speed up development.
*   **Security:** Prioritize technologies with built-in security features and a good track record.
*   **Ecosystem & Integration:** Consider how well the technologies integrate with each other and with potential third-party services.
*   **Alignment with PRD:** Ensure the stack can deliver all functional and non-functional requirements (e.g., REST & GraphQL support, RBAC, Post Versioning).

## 3. Core Backend Technologies

### 3.1. Programming Language

*   **Decision:** Node.js (TypeScript)
*   **Rationale:**
    *   TypeScript enhances JavaScript with static typing, improving code quality, maintainability, and developer productivity, which is crucial for a project like BlogAPI.
    *   Node.js offers a vast ecosystem (npm) providing a rich set of libraries for various functionalities.
    *   Its non-blocking I/O model is well-suited for building scalable and performant APIs, handling concurrent user requests efficiently.
    *   Strong support for both REST and GraphQL API development, aligning with PRD requirements.
    *   Team familiarity with JavaScript/TypeScript can accelerate development.

### 3.2. Backend Framework

*   **Decision:** NestJS (Node.js)
*   **Rationale:**
    *   NestJS is a progressive Node.js framework that uses TypeScript out-of-the-box, providing a structured and maintainable architecture inspired by Angular.
    *   It offers built-in support for microservices, WebSockets, and GraphQL, making it versatile for future expansions.
    *   Dependency injection, modularity, and decorators simplify development and testing.
    *   Excellent documentation and a growing community.
    *   Integrates well with TypeORM for database interactions and Apollo Server for GraphQL.

### 3.3. Database

*   **Type:** Relational (SQL)
*   **Decision (SQL):** PostgreSQL
    *   **Rationale:** PostgreSQL is a powerful, open-source object-relational database system known for its reliability, feature robustness, and data integrity. It handles complex queries and relationships effectively, which is essential for features like post versioning, tags, and user roles. Its support for JSONB can also offer flexibility.
*   **ORM (Object-Relational Mapper):**
    *   **Decision:** TypeORM
    *   **Rationale:** TypeORM is a mature ORM for TypeScript and JavaScript, supporting both Data Mapper and ActiveRecord patterns. It integrates seamlessly with NestJS and PostgreSQL, allowing developers to work with databases using TypeScript classes and decorators. It supports migrations, transactions, and a wide range of database features.

## 4. API Design & Implementation

### 4.1. REST API

*   **Specification:** OpenAPI (formerly Swagger) version 3.x.
*   **Implementation:** Using chosen backend framework's capabilities.
*   **Tools:** Swagger Editor/UI for documentation and testing.

### 4.2. GraphQL API

*   **Decision (Core Library):** Apollo Server
    *   **Rationale:** Apollo Server is a widely-adopted, feature-rich GraphQL server for Node.js. It integrates seamlessly with NestJS (via `@nestjs/graphql` module) and TypeORM. It supports subscriptions, caching, and has excellent tooling (Apollo Studio, GraphQL Playground).
*   **Schema Definition Language (SDL):** Standard GraphQL SDL will be used.
*   **Tools:** GraphQL Playground or GraphiQL for schema exploration and testing (often bundled with Apollo Server).

## 5. Authentication & Authorization

*   **Mechanism:** JSON Web Tokens (JWT).
    *   **Rationale:** JWTs are a standard, stateless, and secure method for API authentication. They allow for easy integration with various clients and can carry user role information for RBAC.
*   **Libraries:**
    *   **JWT Handling:** `jsonwebtoken` (Node.js)
        *   **Rationale:** A popular and well-maintained library for signing and verifying JWTs in Node.js.
    *   **Password Hashing:** `bcrypt`
        *   **Rationale:** `bcrypt` is a strong, adaptive hashing algorithm specifically designed for passwords, providing protection against brute-force attacks.
*   **RBAC Implementation:** Custom logic within the application, mapping roles to permissions, potentially leveraging NestJS Guards and decorators.
    *   **Rationale:** Provides fine-grained control over access to different API resources and operations based on user roles.

## 6. Key Functionality Support

*   **Markdown Parsing:**
    *   **Decision (Library):** `marked`
    *   **Rationale:** `marked` is a popular, fast, and lightweight Markdown parser for JavaScript. It's easy to integrate and customize if needed.
*   **Search:**
    *   **Initial:** Database-specific search capabilities (e.g., `LIKE` queries, Full-Text Search in PostgreSQL).
    *   **Advanced (Future):** Dedicated search engine like Elasticsearch or Algolia if needed.
*   **Pagination:** Implemented via API parameters (e.g., `limit`/`offset` or `page`/`pageSize`).
*   **Post Versioning:** Custom logic to store and retrieve different versions of posts, likely in a separate table linked to posts.

## 7. Development & Operations (DevOps)

### 7.1. Version Control

*   **System:** Git
    *   **Rationale:** Git is the de-facto standard for version control, offering robust branching, merging, and history tracking capabilities.
*   **Hosting:** GitHub
    *   **Rationale:** GitHub provides excellent collaboration features, issue tracking, and integrates well with CI/CD tools like GitHub Actions. It's widely used and has a large community.

### 7.2. Containerization (Recommended)

*   **Technology:** Docker
*   **Orchestration (Optional, for larger deployments):** Kubernetes, Docker Swarm
*   **Rationale:** Consistent development and deployment environments, simplifies dependency management.

### 7.3. CI/CD (Continuous Integration/Continuous Deployment)

*   **Decision (Tools):** GitHub Actions
    *   **Rationale:** GitHub Actions offers a convenient way to automate software workflows directly within the GitHub repository. It's easy to set up for CI/CD pipelines, including linting, testing, building, and deploying applications.
*   **Pipeline Stages:** Linting, testing (unit, integration), building, deploying.

### 7.4. Testing

*   **Decision (Unit Testing Framework):** Jest
    *   **Rationale:** Jest is a popular testing framework for JavaScript/TypeScript, particularly within the Node.js ecosystem. It offers a simple API, built-in mocking capabilities, code coverage reports, and parallel test execution. It integrates well with NestJS.
*   **Integration Testing:** Using Jest with live or mocked dependencies (e.g., using `supertest` for API endpoint testing).
*   **API Testing Tools:** Postman, Newman, or custom scripts using libraries like `axios` or `fetch` within Jest.

### 7.5. Logging

*   **Decision (Library):** Winston
    *   **Rationale:** Winston is a versatile and widely used logging library for Node.js. It supports multiple transport options (console, file, databases, etc.), customizable log formats (including JSON), and different log levels.
*   **Format:** Structured logging (JSON) for easier parsing by log management systems.
*   **Log Management (Optional):** ELK Stack (Elasticsearch, Logstash, Kibana), Grafana Loki, Datadog.

### 7.6. Monitoring & Alerting

*   **Tools (Optional):** Prometheus & Grafana, Datadog, New Relic.
*   **Metrics:** API response times, error rates, system resource usage.

## 8. Environment Configuration

*   **Method:** Environment variables (e.g., using `.env` files for local development, platform-provided environment variables in production).
    *   **Rationale:** Using environment variables is a standard practice for managing application configuration, ensuring that sensitive information (like API keys or database credentials) is not hardcoded into the codebase.
*   **Libraries:** `dotenv` (Node.js)
    *   **Rationale:** `dotenv` is a zero-dependency module that loads environment variables from a `.env` file into `process.env`, simplifying local development configuration. NestJS also has its own `@nestjs/config` module which often uses `dotenv` under the hood.

## 9. Security Considerations

*   **Input Validation:** `class-validator` (often used with `class-transformer`)
    *   **Rationale:** `class-validator` works seamlessly with NestJS and TypeScript. It uses decorator-based validation, making it easy to define validation rules directly on DTO (Data Transfer Object) classes, ensuring data integrity and security.
*   **HTTPS:** Enforced for all API communication (handled by reverse proxy or load balancer in production).
*   **Dependency Management:** Regular scanning for vulnerabilities in third-party libraries (e.g., `npm audit`, Snyk).
*   **Rate Limiting:** Implemented at the API gateway level or within the application using libraries like `express-rate-limit` (Node.js).

## 10. Decision Summary & Justification

This section summarizes the finalized technology choices for the BlogAPI backend project.

*   **Programming Language:** Node.js (TypeScript)
    *   **Justification:** TypeScript enhances JavaScript with static typing, improving code quality and maintainability. Node.js offers a vast ecosystem (npm), non-blocking I/O for performance, and strong support for both REST and GraphQL, aligning with PRD requirements and team familiarity.
*   **Backend Framework:** NestJS
    *   **Justification:** NestJS provides a structured, TypeScript-first architecture, built-in support for microservices, GraphQL, and WebSockets. Its modularity, dependency injection, and integration with TypeORM and Apollo Server enhance developer productivity and maintainability.
*   **Database:** PostgreSQL
    *   **Justification:** PostgreSQL is a robust, open-source relational database known for reliability, data integrity, and handling complex queries, suitable for features like post versioning and RBAC.
*   **ORM:** TypeORM
    *   **Justification:** TypeORM is a mature ORM for TypeScript, integrating seamlessly with NestJS and PostgreSQL. It simplifies database interactions using TypeScript classes and supports migrations and transactions.
*   **GraphQL Library:** Apollo Server
    *   **Justification:** Apollo Server is a feature-rich GraphQL server for Node.js, integrating well with NestJS. It supports subscriptions, caching, and provides excellent tooling.
*   **Authentication Libraries:**
    *   **JWT Handling:** `jsonwebtoken`
        *   **Justification:** A popular and well-maintained library for signing and verifying JWTs in Node.js.
    *   **Password Hashing:** `bcrypt`
        *   **Justification:** A strong, adaptive hashing algorithm for secure password storage.
*   **Markdown Parsing Library:** `marked`
    *   **Justification:** A popular, fast, and lightweight Markdown parser for JavaScript, easy to integrate.
*   **Version Control Hosting:** GitHub
    *   **Justification:** GitHub offers excellent collaboration features, issue tracking, and seamless integration with CI/CD tools like GitHub Actions.
*   **CI/CD Tools:** GitHub Actions
    *   **Justification:** Provides convenient automation for CI/CD pipelines directly within the GitHub repository.
*   **Unit Testing Framework:** Jest
    *   **Justification:** A popular testing framework for JavaScript/TypeScript, offering a simple API, mocking, code coverage, and good integration with NestJS.
*   **Logging Library:** Winston
    *   **Justification:** A versatile logging library for Node.js supporting multiple transports and customizable log formats.
*   **Environment Configuration Library:** `dotenv`
    *   **Justification:** Simplifies loading of environment variables from `.env` files for local development.
*   **Input Validation Library:** `class-validator`
    *   **Justification:** Works seamlessly with NestJS and TypeScript, using decorator-based validation for DTOs.

---

This document reflects the finalized technology stack for the BlogAPI project as of the date specified.
