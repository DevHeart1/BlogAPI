# Application Flow Document: BlogAPI Backend

**Version:** 1.0
**Date:** May 24, 2025
**Status:** Proposed

## 1. Introduction

This document describes the typical application flows for key features of the BlogAPI backend. It illustrates the sequence of interactions between different architectural layers (API, Service, Data Access) when processing user requests. These flows are based on the requirements in `BlogAPI_PRD.md` and the architecture defined in `Architecture.md` and `BackendStructure.md`.

Understanding these flows helps in visualizing how data moves through the system and how business logic is applied.

## 2. Core Architectural Layers Involved

*   **API Layer (Presentation):** Receives requests (REST or GraphQL), handles authentication/authorization, validates input, and calls the Service Layer. Formats and sends responses.
*   **Service Layer (Business Logic):** Contains the core application logic, orchestrates operations, and interacts with the Data Access Layer.
*   **Data Access Layer (DAL):** Interacts with the database through repositories and ORM/ODM models.
*   **Infrastructure Layer:** Provides supporting services like security utilities (JWT, hashing) and database connections.

## 3. Key Application Flows

### 3.1. User Login Flow (REST API Example)

*   **Actor:** A registered user attempting to log in.
*   **Trigger:** Client sends a `POST` request to `/api/v1/auth/login` with username/email and password.
*   **Pre-conditions:** User account exists and is active.
*   **Flow Steps:**
    1.  **API Layer (Router):** The request is routed to `auth.routes.[ext]` and then to `auth.controller.[ext]`.
    2.  **API Layer (Middleware - Validation):** Request body (username, password) is validated (e.g., presence, format) by `auth.validator.[ext]` or a validation middleware.
    3.  **API Layer (Controller):** `auth.controller` calls the `login` method in `AuthService` (`src/core/services/auth.service.[ext]`).
    4.  **Service Layer (AuthService):**
        a.  Retrieves the user from the database by username/email via `UserRepository` (`src/infrastructure/persistence/user.repository.[ext]`).
        b.  If user not found, returns an error (e.g., "Invalid credentials").
        c.  Compares the provided password with the stored hashed password using a utility from `src/infrastructure/security/password.hasher.[ext]`.
        d.  If password mismatch, returns an error (e.g., "Invalid credentials").
        e.  If credentials are valid, generates a JWT using `src/infrastructure/security/jwt.handler.[ext]`, including user ID and role in the payload.
        f.  Returns the JWT (and potentially user information) to the `AuthService` caller.
    5.  **API Layer (Controller):** Receives the JWT from `AuthService`.
    6.  **API Layer (Controller):** Constructs a success response (e.g., 200 OK) containing the JWT and sends it back to the client.
*   **Post-conditions:** Client receives a JWT which can be used for authenticating subsequent requests.

### 3.2. Create New Blog Post Flow (Editor Role - REST API Example)

*   **Actor:** An authenticated user with the 'editor' or 'admin' role.
*   **Trigger:** Client sends a `POST` request to `/api/v1/posts` with post data (title, content, tags, etc.) and a valid JWT in the Authorization header.
*   **Pre-conditions:** User is authenticated and has the required role.
*   **Flow Steps:**
    1.  **API Layer (Router & Middleware - Auth):** Request is routed. `auth.middleware.[ext]` validates the JWT. If invalid or missing, returns a 401 Unauthorized error.
    2.  **API Layer (Middleware - RBAC):** `rbac.middleware.[ext]` checks if the authenticated user's role (from JWT payload) is 'editor' or 'admin'. If not authorized, returns a 403 Forbidden error.
    3.  **API Layer (Middleware - Validation):** Post data in the request body is validated by `post.validator.[ext]` or a validation middleware.
    4.  **API Layer (Controller):** `post.controller.[ext]` calls the `createPost` method in `PostService` (`src/core/services/post.service.[ext]`), passing the validated post data and the authenticated user's ID (from JWT).
    5.  **Service Layer (PostService):**
        a.  Performs any business logic (e.g., generating a slug from the title, setting default status to 'draft').
        b.  Interacts with `TagService` if tags need to be created or fetched.
        c.  Calls `PostRepository` (`src/infrastructure/persistence/post.repository.[ext]`) to save the new post data to the database, associating it with the author (user ID) and tags.
        d.  (If post versioning is active) Calls `VersioningService` to create an initial version of the post.
        e.  Returns the created post data (or a success indicator) to the `PostController`.
    6.  **API Layer (Controller):** Receives the created post data from `PostService`.
    7.  **API Layer (Controller):** Constructs a success response (e.g., 201 Created) with the new post data and sends it to the client.
*   **Post-conditions:** A new blog post is created in the database. The client receives the representation of the newly created post.

### 3.3. Retrieve a Single Blog Post Flow (Any User - GraphQL Example)

*   **Actor:** Any user (authenticated or anonymous, depending on post visibility settings).
*   **Trigger:** Client sends a GraphQL query to `/graphql` requesting a specific post by its ID or slug.
    ```graphql
    query GetPost($id: ID!) {
      post(id: $id) {
        id
        title
        content
        author {
          username
        }
        tags {
          name
        }
      }
    }
    ```
*   **Pre-conditions:** The post exists and is published (or user has rights to view drafts).
*   **Flow Steps:**
    1.  **API Layer (GraphQL Server):** Receives the GraphQL query.
    2.  **API Layer (GraphQL Middleware - Auth - Optional):** If authentication is required for certain fields or post types, `auth.middleware` (adapted for GraphQL) would run.
    3.  **API Layer (GraphQL Resolvers):** The query is parsed, and the `post` resolver in `src/api/graphql/resolvers/post.resolver.[ext]` is invoked.
    4.  **API Layer (Resolver):** The `post` resolver calls the `getPostByIdOrSlug` method in `PostService` (`src/core/services/post.service.[ext]`), passing the post ID/slug.
    5.  **Service Layer (PostService):**
        a.  Calls `PostRepository` to fetch the post data from the database.
        b.  If the post is not found, returns an error or null.
        c.  Checks if the post is published or if the current user (if authenticated) has permission to view it (e.g., if it's their own draft).
        d.  If Markdown parsing is handled server-side, calls `MarkdownService` to convert Markdown content to HTML.
        e.  Returns the post data (including author and tag details, potentially fetched via their respective services or through relations in the ORM) to the resolver.
    6.  **API Layer (Resolver & GraphQL Server):** The resolver provides the data for the requested fields. If nested resolvers are needed (e.g., for `author` or `tags`), they are invoked. DataLoaders (`src/api/graphql/dataloaders/`) might be used here to optimize fetching related data.
    7.  **API Layer (GraphQL Server):** Constructs the GraphQL JSON response according to the query structure and sends it to the client.
*   **Post-conditions:** Client receives the requested blog post data in JSON format.

### 3.4. Add Comment to a Post Flow (Authenticated User - REST API Example)

*   **Actor:** An authenticated user.
*   **Trigger:** Client sends a `POST` request to `/api/v1/posts/{postId}/comments` with comment content and a valid JWT.
*   **Pre-conditions:** User is authenticated. The target post exists.
*   **Flow Steps:**
    1.  **API Layer (Router & Middleware - Auth):** Request is routed. `auth.middleware.[ext]` validates JWT.
    2.  **API Layer (Middleware - Validation):** Comment data is validated.
    3.  **API Layer (Controller):** `comment.controller.[ext]` calls `addComment` in `CommentService` (`src/core/services/comment.service.[ext]`), passing `postId`, comment data, and authenticated user ID.
    4.  **Service Layer (CommentService):**
        a.  Verifies the target post (`postId`) exists by calling `PostService` or `PostRepository`.
        b.  Performs any business logic (e.g., profanity filter if implemented).
        c.  Calls `CommentRepository` to save the new comment, associating it with the post and user.
        d.  Returns the created comment data.
    5.  **API Layer (Controller):** Receives the created comment.
    6.  **API Layer (Controller):** Sends a success response (e.g., 201 Created) with the new comment data.
*   **Post-conditions:** A new comment is added to the specified post. Client receives the new comment data.

### 3.5. Admin Deleting a User Flow (Admin Role - REST API Example)

*   **Actor:** An authenticated user with the 'admin' role.
*   **Trigger:** Client sends a `DELETE` request to `/api/v1/users/{userId}` with a valid JWT.
*   **Pre-conditions:** User is authenticated and has 'admin' role. Target user (`userId`) exists.
*   **Flow Steps:**
    1.  **API Layer (Router & Middleware - Auth):** Request is routed. `auth.middleware.[ext]` validates JWT.
    2.  **API Layer (Middleware - RBAC):** `rbac.middleware.[ext]` checks if the authenticated user's role is 'admin'. If not, returns 403 Forbidden.
    3.  **API Layer (Controller):** `user.controller.[ext]` calls `deleteUser` in `UserService` (`src/core/services/user.service.[ext]`), passing `userId`.
    4.  **Service Layer (UserService):**
        a.  Verifies the target user (`userId`) exists via `UserRepository`.
        b.  (Optional: Business logic, e.g., cannot delete the last admin user, or what happens to content owned by the deleted user).
        c.  Calls `UserRepository` to delete the user from the database.
        d.  Returns a success indicator.
    5.  **API Layer (Controller):** Receives success confirmation.
    6.  **API Layer (Controller):** Sends a success response (e.g., 204 No Content or 200 OK with a message).
*   **Post-conditions:** The specified user account is deleted from the system.

## 4. Conclusion

These flows provide a snapshot of how different parts of the BlogAPI backend interact. While not exhaustive, they cover common operations and demonstrate the layered approach to handling requests, applying business logic, and managing data. The actual implementation will involve more detailed error handling, logging, and specific interactions based on the chosen `TechStack`.
