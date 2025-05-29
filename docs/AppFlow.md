# Application Flow Documentation

This document outlines various application flows within BlogAPI, detailing the sequence of operations from client request to database interaction and response.

## 1. Introduction

Understanding these flows is crucial for developers to grasp how different components of the system interact. Each flow will typically involve:
*   Client interaction
*   API layer (controllers, middlewares)
*   Service layer (business logic)
*   Repository layer (data persistence)
*   Database

## 2. Create New Blog Post Flow (Editor Role - REST API Example)

This flow describes how an authenticated user with an 'Editor' role creates a new blog post using the REST API.

### Sequence Diagram

```mermaid
sequenceDiagram
    actor Client
    participant APILayer as API Layer (Express.js Router)
    participant AuthMid as Auth Middleware
    participant RBACMid as RBAC Middleware
    participant ValidMid as Validation Middleware
    participant PostCtrl as PostController
    participant PostSvc as PostService
    participant TagSvc as TagService
    participant VersionSvc as VersioningService
    participant PostRepo as PostRepository (TypeORM)
    database Database as PostgreSQL

    Client->>APILayer: POST /api/v1/posts (payload, JWT)
    APILayer->>AuthMid: Verify JWT
    alt Invalid JWT
        AuthMid-->>Client: 401 Unauthorized
    else Valid JWT
        AuthMid-->>APILayer: User Info (req.user)
        APILayer->>RBACMid: Check Permissions (req.user.role)
        alt Insufficient Permissions
            RBACMid-->>Client: 403 Forbidden
        else Permissions OK
            RBACMid-->>APILayer: Permissions OK
            APILayer->>ValidMid: Validate Request Body
            alt Validation Failed
                ValidMid-->>Client: 400/422 Error
            else Validation OK
                ValidMid-->>APILayer: Validation OK
                APILayer->>PostCtrl: createPost(req)

                PostCtrl->>PostSvc: createPost(postData, userId)
                PostSvc->>TagSvc: processTags(tagStrings)
                TagSvc-->>PostSvc: Tag Entities
                PostSvc->>PostSvc: Generate Slug (ensure uniqueness)
                opt Post Versioning Active
                    PostSvc->>VersionSvc: createInitialVersion(content)
                    VersionSvc-->>PostSvc: Initial Version Data
                end
                PostSvc->>PostRepo: createAndSave(preparedPostData)
                PostRepo->>Database: Save Post Entity (SQL INSERT via TypeORM)
                Database-->>PostRepo: Saved Post Entity (with ID)
                PostRepo-->>PostSvc: Saved Post Entity

                PostSvc-->>PostCtrl: Created Post Entity
                PostCtrl-->>Client: 201 Created (JSON: New Post)
            end
        end
    end
```

### Textual Description

1.  **Client Request:**
    *   The client (e.g., a web frontend or a mobile app) constructs a `POST` request to the `/api/v1/posts` endpoint.
    *   The request header includes a valid JWT for authentication (`Authorization: Bearer <token>`).
    *   The request body contains the post data (e.g., `title`, `content_markdown`, desired `status` like 'draft' or 'published', and an array of `tags` as strings).
    ```json
    {
      "title": "My Awesome New Post",
      "content_markdown": "## Introduction\n\nThis is the content of my post.",
      "status": "draft",
      "tags": ["tech", "nodejs", "new-feature"]
    }
    ```

2.  **API Layer - Routing & Middleware (Express.js):**
    *   The request first hits the Express.js router, which matches the `POST /api/v1/posts` route to the appropriate handler in the `PostController` (e.g., `src/api/rest/v1/controllers/post.controller.ts`).
    *   **Authentication Middleware (`auth.middleware.ts`):**
        *   Verifies the JWT. If invalid or expired, it rejects the request with a `401 Unauthorized` error.
        *   If valid, it decodes the token and attaches the user's information (e.g., `userId`, `role`) to the request object (e.g., `req.user`).
    *   **RBAC Middleware (`rbac.middleware.ts`):**
        *   Checks if the authenticated user's role (from `req.user.role`) has the permission to create a post (e.g., 'editor' or 'admin').
        *   If not permitted, it rejects the request with a `403 Forbidden` error.
    *   **Validation Middleware (e.g., using `express-validator` or a custom Joi/class-validator based middleware):**
        *   Validates the request body against predefined rules (e.g., `title` is required and a string, `content_markdown` is required, `status` is one of the allowed values, `tags` is an array of strings).
        *   If validation fails, it rejects the request with a `400 Bad Request` or `422 Unprocessable Entity` error, including details of the validation errors.

3.  **API Layer - Controller (`post.controller.ts`):**
    *   If all middlewares pass, the `createPost` method in `PostController` is invoked.
    *   The controller extracts the validated post data from the request body and the authenticated user's ID from `req.user`.
    *   It calls the `PostService` (e.g., `src/core/services/post.service.ts`) to handle the business logic of creating the post, passing the post data and user ID.

4.  **Service Layer - `PostService` (`post.service.ts`):**
    *   The `createPost` method in `PostService` receives the data.
    *   **Tag Handling:**
        *   It may interact with a `TagService` (e.g., `src/core/services/tag.service.ts`).
        *   The `TagService` would be responsible for finding existing tags or creating new ones based on the provided tag strings. It would return an array of `Tag` entities.
    *   **Slug Generation:**
        *   It generates a URL-friendly slug for the post from the title (e.g., using a library like `slugify`). It ensures the slug is unique, possibly by querying the `PostRepository` if a similar slug exists and appending a suffix.
    *   **Versioning (Initial Version):**
        *   If post versioning is active, it may call a `VersioningService` (e.g., `src/core/services/versioning.service.ts`) or handle version creation logic internally.
        *   The initial content is saved as the first version of the post.
    *   **Data Preparation:**
        *   It assembles the `Post` entity data, including the author (linking to the `User` entity via `userId`), processed tags, generated slug, initial version details, and status.
    *   **Database Interaction (via Repository):**
        *   It calls the `createAndSave` (or similar) method on the `PostRepository` (e.g., `src/infrastructure/persistence/post.repository.ts`), passing the prepared `Post` entity data.

5.  **Repository Layer - `PostRepository` (`post.repository.ts` with TypeORM):**
    *   The `PostRepository` (a custom TypeORM repository or a class using TypeORM's `EntityManager` or `DataSource`) interacts with the database.
    *   It creates a new `Post` entity instance.
    *   It assigns values to the entity's properties.
    *   It uses TypeORM methods (e.g., `this.save(postEntity)` if it's a custom repository, or `entityManager.save(postEntity)`) to persist the new post and its relations (like tags, initial version) to the PostgreSQL database within a transaction.
    *   The repository returns the newly saved `Post` entity (now with an `id` and other database-generated fields like `createdAt`, `updatedAt`).

6.  **Service Layer - `PostService` (Return):**
    *   The `PostService` receives the saved `Post` entity from the repository.
    *   It might perform any final transformations or logging.
    *   It returns the created `Post` entity (or a DTO representation) to the `PostController`.

7.  **API Layer - Controller (Response):**
    *   The `PostController` receives the created `Post` entity from the `PostService`.
    *   It sends a `201 Created` HTTP response to the client.
    *   The response body includes a JSON representation of the newly created post.
    ```json
    {
      "id": "new-post-uuid",
      "title": "My Awesome New Post",
      "slug": "my-awesome-new-post",
      "content_markdown": "## Introduction\n\nThis is the content of my post.",
      "author": {
        "id": "user-uuid-editor",
        "username": "editorUser"
      },
      "tags": [
        {"id": "tag-uuid-1", "name": "tech", "slug": "tech"},
        {"id": "tag-uuid-2", "name": "nodejs", "slug": "nodejs"},
        {"id": "tag-uuid-3", "name": "new-feature", "slug": "new-feature"}
      ],
      "status": "draft",
      "version": 1,
      "createdAt": "YYYY-MM-DDTHH:mm:ss.sssZ",
      "updatedAt": "YYYY-MM-DDTHH:mm:ss.sssZ",
      "publishedAt": null
    }
    ```

8.  **Client Receives Response:**
    *   The client receives the `201 Created` response and the data of the new post.
    *   It can then update its UI accordingly (e.g., redirect to the new post or display a success message).

This flow ensures that authentication, authorization, validation, business logic, and data persistence are handled in a structured manner.

## 3. User Login Flow (REST API Example)

This flow describes how a user logs into the system using their email and password via the REST API, and receives a JWT upon successful authentication.

### Sequence Diagram

```mermaid
sequenceDiagram
    actor Client
    participant APILayer as API Layer (Express.js Router)
    participant ValidMid as Validation Middleware
    participant AuthCtrl as AuthController
    participant AuthSvc as AuthService
    participant UserRepo as UserRepository (TypeORM)
    database Database as PostgreSQL
    participant PwdHasher as PasswordHasher (bcrypt)
    participant JwtHandler as JwtHandler (jsonwebtoken)

    Client->>APILayer: POST /api/v1/auth/login (email, password)
    APILayer->>ValidMid: Validate Request Body (email, password)
    alt Validation Failed
        ValidMid-->>Client: 400/422 Error
    else Validation OK
        ValidMid-->>APILayer: Validation OK
        APILayer->>AuthCtrl: login(req)

        AuthCtrl->>AuthSvc: login(email, password)
        AuthSvc->>UserRepo: findByEmail(email)
        UserRepo->>Database: SELECT * FROM users WHERE email = ...
        alt User Not Found
            Database-->>UserRepo: null
            UserRepo-->>AuthSvc: null
            AuthSvc-->>AuthCtrl: Error (User not found)
            AuthCtrl-->>Client: 401 Unauthorized (Invalid credentials)
        else User Found
            Database-->>UserRepo: User Entity (with hashed password)
            UserRepo-->>AuthSvc: User Entity
            AuthSvc->>PwdHasher: compare(plainPassword, hashedPassword)
            alt Password Mismatch
                PwdHasher-->>AuthSvc: false
                AuthSvc-->>AuthCtrl: Error (Password mismatch)
                AuthCtrl-->>Client: 401 Unauthorized (Invalid credentials)
            else Password Match
                PwdHasher-->>AuthSvc: true
                AuthSvc->>JwtHandler: generateToken(userId, role, email)
                JwtHandler-->>AuthSvc: JWT (accessToken)
                AuthSvc-->>AuthCtrl: { accessToken, userDetails }
                AuthCtrl-->>Client: 200 OK (JSON: accessToken, user)
            end
        end
    end
```

### Textual Description

1.  **Client Request:**
    *   The client (e.g., a web frontend) sends a `POST` request to the `/api/v1/auth/login` endpoint.
    *   The request body contains the user's credentials: `email` and `password`.
    ```json
    {
      "email": "user@example.com",
      "password": "password123"
    }
    ```

2.  **API Layer - Routing & Middleware (Express.js):**
    *   The Express.js router matches the `POST /api/v1/auth/login` route to the `login` method in the `AuthController` (e.g., `src/api/rest/v1/controllers/auth.controller.ts`).
    *   **Validation Middleware (e.g., using `express-validator`):**
        *   Validates the request body (e.g., `email` is a valid email format and is required, `password` is a non-empty string and is required).
        *   If validation fails, it rejects the request with a `400 Bad Request` or `422 Unprocessable Entity` error, including details of the validation errors.

3.  **API Layer - Controller (`auth.controller.ts`):**
    *   If validation passes, the `login` method in `AuthController` is invoked.
    *   The controller extracts the validated `email` and `password` from the request body.
    *   It calls the `AuthService` (e.g., `src/core/services/auth.service.ts`) to handle the authentication logic, passing the credentials.

4.  **Service Layer - `AuthService` (`auth.service.ts`):**
    *   The `login` method in `AuthService` receives the `email` and `password`.
    *   **Fetch User:**
        *   It calls the `findByEmail` method on the `UserRepository` (e.g., `src/infrastructure/persistence/user.repository.ts`) to retrieve the user by their email address.
    *   **User Existence Check:**
        *   If the `UserRepository` returns no user (e.g., `null`), the `AuthService` concludes the user does not exist and prepares to signal an authentication failure (typically a generic "Invalid credentials" message to avoid leaking information about existing emails).

5.  **Repository Layer - `UserRepository` (`user.repository.ts` with TypeORM):**
    *   The `findByEmail` method queries the PostgreSQL database for a user with the given email.
    *   TypeORM translates this into a SQL `SELECT` query.
    *   It returns the `User` entity (including the stored hashed password) if found, otherwise `null`.

6.  **Service Layer - `AuthService` (Password Verification & JWT Generation):**
    *   **If User Found:**
        *   The `AuthService` receives the `User` entity from the repository.
        *   **Password Comparison:** It uses a `PasswordHasher` utility (e.g., `src/infrastructure/security/password.hasher.ts`, which internally uses `bcrypt`) to compare the provided plain-text `password` with the stored hashed password from the `User` entity.
        *   **If Password Mismatch:** If the passwords do not match, the `AuthService` signals an authentication failure ("Invalid credentials").
        *   **If Password Match:**
            *   Authentication is successful.
            *   The `AuthService` uses a `JwtHandler` utility (e.g., `src/infrastructure/security/jwt.handler.ts`, using `jsonwebtoken`) to generate a JWT (access token).
            *   The token payload typically includes the user's ID (`userId`), role (`user.role`), and potentially email or username, along with an expiration time.
            *   The `AuthService` prepares a successful authentication response, often containing the generated `accessToken` and some user details (e.g., user ID, username, email, role - carefully selected to not expose sensitive data).
    *   The `AuthService` returns the result (either success with token/user data, or an error indicating authentication failure) to the `AuthController`.

7.  **API Layer - Controller (Response):**
    *   **If Authentication Successful:**
        *   The `AuthController` receives the `accessToken` and user details from `AuthService`.
        *   It sends a `200 OK` HTTP response to the client.
        *   The response body includes the `accessToken` and relevant user information.
        ```json
        {
          "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
          "tokenType": "Bearer",
          "expiresIn": 3600, // Example: token valid for 1 hour
          "user": {
            "id": "user-uuid-123",
            "username": "exampleUser",
            "email": "user@example.com",
            "role": "user"
          }
        }
        ```
    *   **If Authentication Failed (User not found or Password mismatch):**
        *   The `AuthController` receives an error indication from `AuthService`.
        *   It sends a `401 Unauthorized` HTTP response to the client, usually with a generic error message to prevent user enumeration or password guessing attacks.
        ```json
        {
          "statusCode": 401,
          "message": "Invalid email or password",
          "error": "Unauthorized"
        }
        ```

8.  **Client Receives Response:**
    *   The client receives the HTTP response.
    *   If successful (`200 OK`), it typically stores the `accessToken` (e.g., in localStorage or a secure cookie) and uses it for subsequent authenticated requests. It may also update the UI to reflect the logged-in state.
    *   If failed (`401 Unauthorized`), it displays an appropriate error message to the user.

*(Placeholder for other application flows like Get Post, etc.)*
