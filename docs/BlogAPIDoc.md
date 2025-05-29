# BlogAPI - API and Project Documentation

**Version:** 1.0
**Last Updated:** May 26, 2025

## Table of Contents

1.  [Introduction](#1-introduction)
2.  [Technologies Used](#2-technologies-used)
3.  [Getting Started](#3-getting-started)
    *   [Prerequisites](#31-prerequisites)
    *   [Installation](#32-installation)
    *   [Configuration](#33-configuration)
    *   [Database Setup](#34-database-setup)
    *   [Running the API](#35-running-the-api)
4.  [Authentication](#4-authentication)
5.  [API Reference - REST API](#5-api-reference---rest-api)
    *   [Base URL](#51-base-url)
    *   [Authentication Endpoint](#52-authentication-endpoint)
    *   [Users Endpoints](#53-users-endpoints)
    *   [Posts Endpoints](#54-posts-endpoints)
    *   [Tags Endpoints](#55-tags-endpoints)
    *   [Comments Endpoints](#56-comments-endpoints)
6.  [API Reference - GraphQL API](#6-api-reference---graphql-api)
    *   [Endpoint](#61-endpoint)
    *   [Schema Overview](#62-schema-overview)
    *   [Example Query: Get Post](#63-example-query-get-post)
    *   [Example Mutation: Create Post](#64-example-mutation-create-post)
7.  [Error Handling](#7-error-handling)
    *   [REST API Errors](#71-rest-api-errors)
    *   [GraphQL API Errors](#72-graphql-api-errors)
8.  [Rate Limiting](#8-rate-limiting)
9.  [Further Information](#9-further-information)

## 1. Introduction

This document provides detailed information about the BlogAPI, including setup instructions, authentication mechanisms, and comprehensive API endpoint documentation for both REST and GraphQL interfaces. The BlogAPI backend serves a modern blogging platform, enabling content creation, management, and delivery.

Refer to [BlogAPI_PRD.md](BlogAPI_PRD.md) for product requirements and [TechStack.md](docs/TechStack.md) for technology choices.

## 2. Technologies Used

The BlogAPI is built using the following core technologies:

*   **Programming Language:** Node.js (with TypeScript)
*   **Backend Framework:** Express.js
*   **Database:** PostgreSQL
*   **ORM:** TypeORM
*   **GraphQL Server:** Apollo Server
*   **Authentication:** JSON Web Tokens (JWT) using `jsonwebtoken` for token generation/verification and `bcrypt` for password hashing.
*   **Markdown Parsing:** `markdown-it` for processing blog post content.
*   **Containerization (Recommended):** Docker

## 3. Getting Started

### 3.1. Prerequisites

*   Node.js (Version >=18.x recommended)
*   npm (Node Package Manager) or yarn
*   Docker (Recommended for running PostgreSQL and other services consistently)
*   PostgreSQL client tools (e.g., `psql`, pgAdmin) if interacting with the database directly.

### 3.2. Installation

1.  **Clone the repository:**
    ```bash
    git clone <repository_url> BlogAPI
    cd BlogAPI
    ```
2.  **Install dependencies:**
    ```bash
    npm install
    # or
    # yarn install
    ```

### 3.3. Configuration

Application configuration is managed via environment variables. A `.env.example` file is provided as a template.

1.  **Create a `.env` file:**
    ```bash
    cp .env.example .env
    ```
2.  **Update `.env` with your settings:**

    *   `DATABASE_URL`: Connection string for your PostgreSQL database.
        *   Example: `postgresql://user:password@localhost:5432/blogapi_dev`
    *   `JWT_SECRET`: A long, strong, random string used to sign and verify JWTs.
        *   Example: `your-very-secure-and-long-random-string`
    *   `API_PORT`: The port on which the API server will listen.
        *   Example: `3000` (or `8080`)
    *   `GRAPHQL_ENDPOINT`: The path for the GraphQL API.
        *   Example: `/graphql`
    *   `LOG_LEVEL`: The logging level for the application.
        *   Example: `info` (common levels: `error`, `warn`, `info`, `http`, `verbose`, `debug`, `silly`)
    *   *(Add other variables as needed, e.g., `CORS_ORIGIN`, `RATE_LIMIT_WINDOW_MS`, `RATE_LIMIT_MAX_REQUESTS`)*

### 3.4. Database Setup

*   **Migrations:** Apply database schema migrations.
    ```bash
    npm run migrate
    ```
    *(This command assumes a script `migrate` is defined in `package.json` that uses TypeORM CLI or a similar tool to run migrations.)*

*   **Seeding (Optional):** Populate the database with initial data for development or testing.
    ```bash
    npm run seed
    ```
    *(This command assumes a script `seed` is defined in `package.json`.)*

### 3.5. Running the API

*   **Development Mode:** Starts the server with features like hot-reloading.
    ```bash
    npm run dev
    ```
    Output should indicate the server is running, e.g., `API server listening on http://localhost:3000`.

*   **Production Mode:**
    1.  Build the TypeScript code:
        ```bash
        npm run build
        ```
    2.  Start the server:
        ```bash
        npm start
        ```

## 4. Authentication

Authentication is handled using JSON Web Tokens (JWT).

*   **Obtaining a Token:**
    Clients must send a `POST` request to the `/api/v1/auth/login` endpoint with valid user credentials (e.g., email and password). If authentication is successful, the API will return a JWT.

*   **Using the Token:**
    To access protected routes, the client must include the JWT in the `Authorization` header of subsequent requests, using the `Bearer` scheme:
    ```
    Authorization: Bearer <your_jwt_token>
    ```

*   **Token Expiry and Renewal:**
    JWTs are configured to expire after a certain period (e.g., 1 hour, specified in the `expiresIn` field of the login response). For continued access, clients may need to re-authenticate or, if implemented, use a refresh token mechanism to obtain a new access token. *(Note: Refresh token mechanism details are not covered in the current PRD and would require further specification.)*

## 5. API Reference - REST API

### 5.1. Base URL

The base URL for all v1 REST API endpoints is `/api/v1`.

### 5.2. Authentication Endpoint

#### `POST /api/v1/auth/login`

*   **Description:** Authenticates a user and returns a JWT.
*   **Request Body:**
    ```json
    {
      "email": "user@example.com",
      "password": "password123"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "tokenType": "Bearer",
      "expiresIn": 3600 // Example: token valid for 1 hour (3600 seconds)
    }
    ```
*   **Response (400 Bad Request):** Invalid input format or missing fields.
    ```json
    {
      "statusCode": 400,
      "message": "Validation failed",
      "error": "Bad Request",
      "details": [
        { "field": "email", "message": "Email must be a valid email address" }
      ]
    }
    ```
*   **Response (401 Unauthorized):** Invalid credentials.
    ```json
    {
      "statusCode": 401,
      "message": "Invalid email or password",
      "error": "Unauthorized"
    }
    ```

### 5.3. Users Endpoints

Base path: `/api/v1/users`

#### `GET /api/v1/users`

*   **Description:** List all users.
*   **Auth:** Required. Role: Admin.
*   **Query Parameters:**
    *   `page` (number, optional, default: 1): Page number for pagination.
    *   `limit` (number, optional, default: 10): Number of users per page.
    *   *(Other filter parameters like `role` could be added here)*
*   **Response (200 OK):**
    ```json
    {
      "data": [
        {
          "id": "user-uuid-1",
          "username": "adminuser",
          "email": "admin@example.com",
          "role": "admin",
          "createdAt": "2024-05-26T10:00:00.000Z"
        }
        // ... more users
      ],
      "pagination": {
        "totalItems": 50,
        "totalPages": 5,
        "currentPage": 1,
        "limit": 10
      }
    }
    ```
*   **Response (401 Unauthorized):** Missing or invalid JWT.
*   **Response (403 Forbidden):** User does not have Admin role.

#### `POST /api/v1/users`

*   **Description:** Create a new user.
*   **Auth:** Required. Role: Admin.
*   **Request Body:**
    ```json
    {
      "username": "neweditor",
      "email": "editor@example.com",
      "password": "securePassword123!",
      "role": "editor" // or "user"
    }
    ```
*   **Response (201 Created):**
    ```json
    {
      "id": "user-uuid-new",
      "username": "neweditor",
      "email": "editor@example.com",
      "role": "editor",
      "createdAt": "2024-05-26T10:05:00.000Z"
    }
    ```
*   **Response (400 Bad Request / 422 Unprocessable Entity):** Validation errors (e.g., duplicate email/username, weak password).
    ```json
    {
      "statusCode": 422,
      "message": "User creation failed due to validation errors",
      "error": "Unprocessable Entity",
      "details": [
        { "field": "email", "message": "Email already exists" }
      ]
    }
    ```

#### `GET /api/v1/users/me`

*   **Description:** Get the profile of the currently authenticated user.
*   **Auth:** Required (any valid user).
*   **Response (200 OK):**
    ```json
    {
      "id": "user-uuid-current",
      "username": "currentuser",
      "email": "current@example.com",
      "role": "user",
      "bio": "This is my bio.",
      "profilePictureUrl": "https://example.com/path/to/image.jpg",
      "createdAt": "2024-05-26T09:00:00.000Z"
    }
    ```
*   **Response (401 Unauthorized):** Missing or invalid JWT.

#### `GET /api/v1/users/{id}`

*   **Description:** Get a specific user by their ID.
*   **Auth:** Required. Admin role, or the user themselves requesting their own profile.
*   **Path Parameters:**
    *   `id` (string, UUID): The ID of the user to retrieve.
*   **Response (200 OK):**
    ```json
    {
      "id": "user-uuid-specific",
      "username": "specificuser",
      "email": "specific@example.com", // Email might be omitted for non-admin/non-owner requests
      "role": "user",
      "bio": "Another user's bio.",
      "createdAt": "2024-05-25T12:00:00.000Z"
    }
    ```
*   **Response (401 Unauthorized):** Missing or invalid JWT.
*   **Response (403 Forbidden):** User not authorized to view this profile.
*   **Response (404 Not Found):** User with the specified ID not found.

#### `PUT /api/v1/users/{id}`

*   **Description:** Update a user's profile information.
*   **Auth:** Required. Admin role, or the user themselves updating their own profile.
*   **Path Parameters:**
    *   `id` (string, UUID): The ID of the user to update.
*   **Request Body:** (Fields are optional; only provided fields will be updated)
    ```json
    {
      "username": "updatedUsername",
      "bio": "An updated bio.",
      "profilePictureUrl": "https://example.com/path/to/new_image.jpg"
      // Password updates should ideally be handled via a separate endpoint e.g., /api/v1/users/{id}/password
      // Role updates should only be allowed for Admins.
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "id": "user-uuid-specific",
      "username": "updatedUsername",
      "email": "specific@example.com",
      "role": "user",
      "bio": "An updated bio.",
      "profilePictureUrl": "https://example.com/path/to/new_image.jpg",
      "updatedAt": "2024-05-26T11:00:00.000Z"
    }
    ```
*   **Response (400 Bad Request / 422 Unprocessable Entity):** Validation errors.
*   **Response (401 Unauthorized):** Missing or invalid JWT.
*   **Response (403 Forbidden):** User not authorized to update this profile.
*   **Response (404 Not Found):** User with the specified ID not found.

#### `DELETE /api/v1/users/{id}`

*   **Description:** Delete a user.
*   **Auth:** Required. Role: Admin.
*   **Path Parameters:**
    *   `id` (string, UUID): The ID of the user to delete.
*   **Response (204 No Content):** Successfully deleted.
*   **Response (401 Unauthorized):** Missing or invalid JWT.
*   **Response (403 Forbidden):** User does not have Admin role.
*   **Response (404 Not Found):** User with the specified ID not found.

### 5.4. Posts Endpoints

Base path: `/api/v1/posts`

#### `GET /api/v1/posts`

*   **Description:** List all posts. Publicly accessible for published posts. Drafts/archived posts may require authentication and specific roles.
*   **Auth:** Optional. Admins/Editors can see all posts including drafts/archived. Authenticated users might see their own drafts. Unauthenticated users see only published posts.
*   **Query Parameters:**
    *   `page` (number, optional, default: 1): Page number for pagination.
    *   `limit` (number, optional, default: 10): Number of posts per page.
    *   `status` (string, optional): Filter posts by status (e.g., `published`, `draft`, `archived`). Access to non-published statuses may be role-restricted.
    *   `tag` (string, optional): Filter posts by a specific tag name or tag ID/slug.
    *   `authorId` (string, optional, UUID): Filter posts by author ID.
    *   `search` (string, optional): Search term to filter posts by title or content.
    *   `sortBy` (string, optional, default: `publishedAt` or `createdAt`): Field to sort by (e.g., `title`, `createdAt`, `updatedAt`, `publishedAt`).
    *   `sortOrder` (string, optional, default: `desc`): Sort order (`asc` or `desc`).
*   **Response (200 OK):**
    ```json
    {
      "data": [
        {
          "id": "post-uuid-1",
          "title": "My First Blog Post",
          "slug": "my-first-blog-post",
          "content_markdown_summary": "This is a short summary of the post...", // Summary for list view
          "author": { // Expanded author details
            "id": "user-uuid-author1",
            "username": "author1"
          },
          "tags": [ // Array of tag objects
            {"id": "tag-uuid-1", "name": "Technology", "slug": "technology"},
            {"id": "tag-uuid-2", "name": "Node.js", "slug": "nodejs"}
          ],
          "status": "published",
          "createdAt": "2024-05-20T10:00:00.000Z",
          "updatedAt": "2024-05-20T11:00:00.000Z",
          "publishedAt": "2024-05-20T12:00:00.000Z",
          "version": 2 // Current version number
        }
        // ... more posts
      ],
      "pagination": {
        "totalItems": 100,
        "totalPages": 10,
        "currentPage": 1,
        "limit": 10
      }
    }
    ```

#### `POST /api/v1/posts`

*   **Description:** Create a new post.
*   **Auth:** Required. Role: Editor or Admin.
*   **Request Body:**
    ```json
    {
      "title": "A New Post Title",
      "content_markdown": "## Content of the new post\n\nThis is written in Markdown.",
      "status": "draft", // "draft" or "published"
      "tags": ["new-tag", "existing-tag-slug"] // Array of tag names or slugs/IDs. New tags might be created.
      // slug might be auto-generated if not provided
      // publishedAt will be set automatically if status is 'published'
    }
    ```
*   **Response (201 Created):**
    ```json
    {
      "id": "post-uuid-new",
      "title": "A New Post Title",
      "slug": "a-new-post-title", // Auto-generated or provided
      "content_markdown": "## Content of the new post\n\nThis is written in Markdown.",
      "author": {
        "id": "current-user-uuid", // ID of the authenticated editor/admin
        "username": "currentUser"
      },
      "tags": [
        {"id": "tag-uuid-3", "name": "new-tag", "slug": "new-tag"},
        {"id": "tag-uuid-existing", "name": "Existing Tag", "slug": "existing-tag-slug"}
      ],
      "status": "draft",
      "version": 1,
      "createdAt": "2024-05-26T14:00:00.000Z",
      "updatedAt": "2024-05-26T14:00:00.000Z",
      "publishedAt": null
    }
    ```
*   **Response (400 Bad Request / 422 Unprocessable Entity):** Validation errors (e.g., missing title, invalid status).

#### `GET /api/v1/posts/{id_or_slug}`

*   **Description:** Get a specific post by its ID or slug.
*   **Auth:** Optional for published posts. Required for drafts/archived (Editor/Admin, or Author for their own draft).
*   **Path Parameters:**
    *   `id_or_slug` (string): The ID (UUID) or slug of the post.
*   **Response (200 OK):**
    ```json
    {
      "id": "post-uuid-example",
      "title": "Example Post Title",
      "slug": "example-post-title",
      "content_markdown": "Full content of the post in Markdown...",
      // "content_html": "<p>Full content of the post in HTML...</p>", // Optionally include rendered HTML
      "author": {
        "id": "user-uuid-author",
        "username": "postauthor"
      },
      "tags": [
        {"id": "tag-uuid-1", "name": "Technology", "slug": "technology"}
      ],
      "status": "published",
      "version": 1,
      "createdAt": "2024-05-22T10:00:00.000Z",
      "updatedAt": "2024-05-22T10:00:00.000Z",
      "publishedAt": "2024-05-22T12:00:00.000Z"
    }
    ```
*   **Response (404 Not Found):** Post not found.
*   **Response (403 Forbidden):** User not authorized to view the post (e.g., a draft by another user).

#### `PUT /api/v1/posts/{id_or_slug}`

*   **Description:** Update an existing post. Creates a new version if content changes significantly.
*   **Auth:** Required. Role: Editor/Admin, or Author updating their own draft/published post.
*   **Path Parameters:**
    *   `id_or_slug` (string): The ID (UUID) or slug of the post to update.
*   **Request Body:** (Fields are optional)
    ```json
    {
      "title": "Updated Post Title",
      "content_markdown": "Updated content of the post.",
      "status": "published", // Can change status
      "tags": ["technology", "updated-tag"]
      // slug might be updated if title changes and no custom slug is set
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "id": "post-uuid-example",
      "title": "Updated Post Title",
      "slug": "updated-post-title",
      "content_markdown": "Updated content of the post.",
      "author": { /* ... */ },
      "tags": [ /* ... */ ],
      "status": "published",
      "version": 2, // Incremented version
      "createdAt": "2024-05-22T10:00:00.000Z",
      "updatedAt": "2024-05-26T15:00:00.000Z",
      "publishedAt": "2024-05-26T15:00:00.000Z" // Updated if status changed to published
    }
    ```
*   **Response (404 Not Found):** Post not found.
*   **Response (403 Forbidden):** User not authorized to update the post.

#### `DELETE /api/v1/posts/{id_or_slug}`

*   **Description:** Delete a post. This might be a soft delete (changing status to 'archived' or 'deleted') or a hard delete, depending on implementation.
*   **Auth:** Required. Role: Editor/Admin, or Author deleting their own draft/post.
*   **Path Parameters:**
    *   `id_or_slug` (string): The ID (UUID) or slug of the post to delete.
*   **Response (204 No Content):** Successfully deleted.
*   **Response (404 Not Found):** Post not found.
*   **Response (403 Forbidden):** User not authorized to delete the post.

#### `GET /api/v1/posts/{id_or_slug}/versions`

*   **Description:** List all versions of a specific post.
*   **Auth:** Required. Role: Editor/Admin, or Author for their own post.
*   **Path Parameters:**
    *   `id_or_slug` (string): The ID (UUID) or slug of the post.
*   **Response (200 OK):**
    ```json
    {
      "data": [
        {
          "versionId": "version-uuid-2", // ID of the version entry
          "versionNumber": 2,
          "content_markdown_summary": "Summary of version 2 content...",
          "updatedAt": "2024-05-26T15:00:00.000Z",
          "editor": { // User who made this version
            "id": "user-uuid-editor",
            "username": "editorUser"
          }
        },
        {
          "versionId": "version-uuid-1",
          "versionNumber": 1,
          "content_markdown_summary": "Summary of version 1 content...",
          "updatedAt": "2024-05-22T10:00:00.000Z",
          "editor": {
            "id": "user-uuid-author",
            "username": "postauthor"
          }
        }
      ],
      "pagination": { // Optional if not too many versions expected per post
        "totalItems": 2,
        "totalPages": 1,
        "currentPage": 1,
        "limit": 10
      }
    }
    ```
*   **Response (404 Not Found):** Post not found.
*   **Response (403 Forbidden):** User not authorized.

#### `POST /api/v1/posts/{id_or_slug}/versions/{versionId}/revert`

*   **Description:** Revert a post to a specific version. This creates a new version based on the old content.
*   **Auth:** Required. Role: Editor/Admin.
*   **Path Parameters:**
    *   `id_or_slug` (string): The ID (UUID) or slug of the post.
    *   `versionId` (string, UUID): The ID of the version to revert to.
*   **Response (200 OK):** (Returns the post in its new state after reverting)
    ```json
    {
      "id": "post-uuid-example",
      "title": "Title from Reverted Version", // Content from the reverted version
      "slug": "example-post-title", // Slug may or may not change based on title
      "content_markdown": "Content from the reverted version...",
      "author": { /* ... */ },
      "tags": [ /* ... */ ], // Tags might also be reverted or kept as current
      "status": "draft", // Reverted posts might default to draft status
      "version": 3, // New version number after revert
      "createdAt": "2024-05-22T10:00:00.000Z", // Original creation date
      "updatedAt": "2024-05-26T16:00:00.000Z", // Timestamp of the revert action
      "publishedAt": null // Or original publishedAt if status is restored to published
    }
    ```
*   **Response (404 Not Found):** Post or Version not found.
*   **Response (403 Forbidden):** User not authorized.

### 5.5. Tags Endpoints

Base path: `/api/v1/tags`

#### `GET /api/v1/tags`

*   **Description:** List all tags. Publicly accessible.
*   **Auth:** Optional.
*   **Query Parameters:**
    *   `page` (number, optional, default: 1): Page number for pagination.
    *   `limit` (number, optional, default: 10): Number of tags per page.
    *   `search` (string, optional): Search term to filter tags by name.
    *   `sortBy` (string, optional, default: `name`): Field to sort by (e.g., `name`, `postCount` - if available).
    *   `sortOrder` (string, optional, default: `asc`): Sort order (`asc` or `desc`).
*   **Response (200 OK):**
    ```json
    {
      "data": [
        {
          "id": "tag-uuid-1",
          "name": "Technology",
          "slug": "technology"
          // "postCount": 25 // Optional: Number of posts associated with this tag
        },
        {
          "id": "tag-uuid-2",
          "name": "Node.js",
          "slug": "nodejs"
          // "postCount": 10
        }
        // ... more tags
      ],
      "pagination": {
        "totalItems": 50,
        "totalPages": 5,
        "currentPage": 1,
        "limit": 10
      }
    }
    ```

#### `POST /api/v1/tags`

*   **Description:** Create a new tag.
*   **Auth:** Required. Role: Editor or Admin.
*   **Request Body:**
    ```json
    {
      "name": "New Tag Name"
      // "slug": "new-tag-slug" // Optional: Slug can be auto-generated from name
    }
    ```
*   **Response (201 Created):**
    ```json
    {
      "id": "tag-uuid-new",
      "name": "New Tag Name",
      "slug": "new-tag-name" // Or "new-tag-slug" if provided
    }
    ```
*   **Response (400 Bad Request / 422 Unprocessable Entity):** Validation errors (e.g., missing name, duplicate name/slug).
    ```json
    {
      "statusCode": 422,
      "message": "Tag creation failed due to validation errors",
      "error": "Unprocessable Entity",
      "details": [
        { "field": "name", "message": "Tag name already exists" }
      ]
    }
    ```
*   **Response (401 Unauthorized):** Missing or invalid JWT.
*   **Response (403 Forbidden):** User does not have Editor or Admin role.

#### `GET /api/v1/tags/{id_or_slug}`

*   **Description:** Get a specific tag by its ID or slug.
*   **Auth:** Optional.
*   **Path Parameters:**
    *   `id_or_slug` (string): The ID (UUID) or slug of the tag.
*   **Response (200 OK):**
    ```json
    {
      "id": "tag-uuid-example",
      "name": "Example Tag",
      "slug": "example-tag"
      // "postCount": 15 // Optional
    }
    ```
*   **Response (404 Not Found):** Tag not found.

#### `PUT /api/v1/tags/{id_or_slug}`

*   **Description:** Update an existing tag.
*   **Auth:** Required. Role: Editor or Admin.
*   **Path Parameters:**
    *   `id_or_slug` (string): The ID (UUID) or slug of the tag to update.
*   **Request Body:**
    ```json
    {
      "name": "Updated Tag Name"
      // "slug": "updated-tag-slug" // Optional
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "id": "tag-uuid-example",
      "name": "Updated Tag Name",
      "slug": "updated-tag-name" // Or "updated-tag-slug"
    }
    ```
*   **Response (400 Bad Request / 422 Unprocessable Entity):** Validation errors.
*   **Response (401 Unauthorized):** Missing or invalid JWT.
*   **Response (403 Forbidden):** User does not have Editor or Admin role.
*   **Response (404 Not Found):** Tag not found.

#### `DELETE /api/v1/tags/{id_or_slug}`

*   **Description:** Delete a tag.
*   **Auth:** Required. Role: Editor or Admin.
*   **Path Parameters:**
    *   `id_or_slug` (string): The ID (UUID) or slug of the tag to delete.
*   **Response (204 No Content):** Successfully deleted.
*   **Response (401 Unauthorized):** Missing or invalid JWT.
*   **Response (403 Forbidden):** User does not have Editor or Admin role.
*   **Response (404 Not Found):** Tag not found.

### 5.6. Comments Endpoints

Base path for post-specific comments: `/api/v1/posts/{postId}/comments`
Base path for direct comment manipulation: `/api/v1/comments`

#### `GET /api/v1/posts/{postId}/comments`

*   **Description:** List all comments for a specific post. Publicly accessible.
*   **Auth:** Optional.
*   **Path Parameters:**
    *   `postId` (string, UUID): The ID of the post for which to retrieve comments.
*   **Query Parameters:**
    *   `page` (number, optional, default: 1): Page number for pagination.
    *   `limit` (number, optional, default: 10): Number of comments per page.
    *   `sortBy` (string, optional, default: `createdAt`): Field to sort by (e.g., `createdAt`).
    *   `sortOrder` (string, optional, default: `asc`): Sort order (`asc` or `desc`).
*   **Response (200 OK):**
    ```json
    {
      "data": [
        {
          "id": "comment-uuid-1",
          "postId": "post-uuid-example",
          "author": { // User who made the comment
            "id": "user-uuid-commenter1",
            "username": "commenter1"
          },
          "content": "This is the first comment on the post.",
          "createdAt": "2024-05-23T10:00:00.000Z",
          "updatedAt": "2024-05-23T10:00:00.000Z"
        },
        {
          "id": "comment-uuid-2",
          "postId": "post-uuid-example",
          "author": {
            "id": "user-uuid-commenter2",
            "username": "commenter2"
          },
          "content": "Another insightful comment.",
          "createdAt": "2024-05-23T11:00:00.000Z",
          "updatedAt": "2024-05-23T11:00:00.000Z"
        }
        // ... more comments
      ],
      "pagination": {
        "totalItems": 20,
        "totalPages": 2,
        "currentPage": 1,
        "limit": 10
      }
    }
    ```
*   **Response (404 Not Found):** If the post with `postId` does not exist.
    ```json
    {
      "statusCode": 404,
      "message": "Post not found",
      "error": "Not Found"
    }
    ```

#### `POST /api/v1/posts/{postId}/comments`

*   **Description:** Add a new comment to a specific post.
*   **Auth:** Required. Any authenticated user can comment.
*   **Path Parameters:**
    *   `postId` (string, UUID): The ID of the post to comment on.
*   **Request Body:**
    ```json
    {
      "content": "This is my thoughtful comment on this post."
    }
    ```
*   **Response (201 Created):**
    ```json
    {
      "id": "comment-uuid-new",
      "postId": "post-uuid-example",
      "author": { // Authenticated user who made the comment
        "id": "current-user-uuid",
        "username": "currentUser"
      },
      "content": "This is my thoughtful comment on this post.",
      "createdAt": "2024-05-26T17:00:00.000Z",
      "updatedAt": "2024-05-26T17:00:00.000Z"
    }
    ```
*   **Response (400 Bad Request / 422 Unprocessable Entity):** Validation errors (e.g., empty content).
    ```json
    {
      "statusCode": 400,
      "message": "Validation failed",
      "error": "Bad Request",
      "details": [
        { "field": "content", "message": "Content cannot be empty" }
      ]
    }
    ```
*   **Response (401 Unauthorized):** Missing or invalid JWT.
*   **Response (404 Not Found):** If the post with `postId` does not exist.

#### `PUT /api/v1/comments/{commentId}`

*   **Description:** Update an existing comment.
*   **Auth:** Required. Comment owner, Editor, or Admin.
*   **Path Parameters:**
    *   `commentId` (string, UUID): The ID of the comment to update.
*   **Request Body:**
    ```json
    {
      "content": "This is my updated comment."
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "id": "comment-uuid-example",
      "postId": "post-uuid-associated",
      "author": {
        "id": "user-uuid-owner",
        "username": "commentOwner"
      },
      "content": "This is my updated comment.",
      "createdAt": "2024-05-24T09:00:00.000Z", // Original creation date
      "updatedAt": "2024-05-26T17:30:00.000Z"  // Timestamp of the update
    }
    ```
*   **Response (400 Bad Request / 422 Unprocessable Entity):** Validation errors (e.g., empty content).
*   **Response (401 Unauthorized):** Missing or invalid JWT.
*   **Response (403 Forbidden):** User is not the owner of the comment and not an Editor/Admin.
*   **Response (404 Not Found):** Comment with `commentId` not found.

#### `DELETE /api/v1/comments/{commentId}`

*   **Description:** Delete a comment.
*   **Auth:** Required. Comment owner, Editor, or Admin.
*   **Path Parameters:**
    *   `commentId` (string, UUID): The ID of the comment to delete.
*   **Response (204 No Content):** Successfully deleted.
*   **Response (401 Unauthorized):** Missing or invalid JWT.
*   **Response (403 Forbidden):** User is not the owner of the comment and not an Editor/Admin.
*   **Response (404 Not Found):** Comment with `commentId` not found.

## 6. API Reference - GraphQL API

### 6.1. Endpoint

The GraphQL API is available at the path specified by the `GRAPHQL_ENDPOINT` environment variable (e.g., `/graphql`). It supports queries, mutations, and potentially subscriptions.

### 6.2. Schema Overview

The GraphQL schema defines all available types, queries, and mutations. Key types include:

*   `User`: Represents user accounts.
*   `Post`: Represents blog posts, including content, author, tags, and comments.
*   `Tag`: Represents tags that can be applied to posts.
*   `Comment`: Represents comments made on posts.
*   `PostVersion`: Represents a specific version of a post.

The full schema can be explored using GraphQL client tools like GraphQL Playground or GraphiQL, which are typically available at the GraphQL endpoint when the server is running in development mode.

### 6.3. Example Query: Get Post

Fetches a single post by its ID, along with related author and tags.

```graphql
query GetPost($id: ID!) {
  post(id: $id) {
    id
    title
    content # This could be raw Markdown or pre-rendered HTML based on resolver implementation
    # contentHtml # Alternatively, provide a separate field for rendered HTML
    status
    author {
      id
      username
    }
    tags {
      id
      name
    }
    comments { # Assuming comments are directly queryable under a post
        id
        content
        author {
            id
            username
        }
        createdAt
    }
    createdAt
    publishedAt
    updatedAt
    version # Current version number
  }
}
```

*   **Variables:**
    ```json
    {
      "id": "post-uuid-example"
    }
    ```
*   **Description:** Retrieves a specific post.
*   **Auth:** Behavior may vary. Published posts might be public, while drafts or archived posts may require specific permissions (editor/admin or author).

### 6.4. Example Mutation: Create Post

Creates a new blog post.

```graphql
mutation CreateNewPost($title: String!, $content: String!, $tags: [String!], $status: PostStatus!) {
  createPost(input: {
    title: $title,
    content: $content,
    tagNames: $tags, # Assuming tags can be created/linked by name
    status: $status  # e.g., DRAFT, PUBLISHED
  }) {
    id
    title
    status
    content
    author {
      id
      username
    }
    tags {
        id
        name
    }
    version
    createdAt
  }
}
```

*   **Variables:**
    ```json
    {
      "title": "My First GraphQL Post",
      "content": "## Hello GraphQL World!\n\nThis is exciting.",
      "tags": ["graphql", "nodejs", "api"],
      "status": "DRAFT" # Assuming PostStatus is an ENUM (DRAFT, PUBLISHED, ARCHIVED)
    }
    ```
*   **Description:** Allows authenticated users with appropriate roles (e.g., 'editor', 'admin') to create a new post.
*   **Auth:** Required. Typically 'editor' or 'admin' role.

## 7. Error Handling

### 7.1. REST API Errors

The REST API uses standard HTTP status codes to indicate the success or failure of a request. Error responses are generally in JSON format.

*   **General Error Format:**
    ```json
    {
      "statusCode": number,    // HTTP status code
      "message": string,       // Human-readable error message
      "error": "ErrorType",    // Category of error (e.g., "Bad Request", "Unauthorized")
      "details": [             // Optional: Array of specific validation errors or more details
        { "field": "fieldName", "message": "Specific error for this field" }
      ]
    }
    ```
*   **Common HTTP Status Codes:**
    *   `200 OK`: Request succeeded.
    *   `201 Created`: Resource successfully created.
    *   `204 No Content`: Request succeeded, but no content to return (e.g., after a DELETE).
    *   `400 Bad Request`: Invalid syntax or missing parameters.
    *   `401 Unauthorized`: Authentication failed or token not provided/invalid.
    *   `403 Forbidden`: Authenticated user does not have permission to access the resource.
    *   `404 Not Found`: Requested resource does not exist.
    *   `422 Unprocessable Entity`: Server understands the request but cannot process it due to validation errors.
    *   `500 Internal Server Error`: An unexpected error occurred on the server.

### 7.2. GraphQL API Errors

GraphQL API errors are returned in the `errors` array of the JSON response, even if the HTTP status code is 200.

*   **Standard GraphQL Error Format:**
    ```json
    {
      "errors": [
        {
          "message": "Error message describing the issue.",
          "locations": [{ "line": 2, "column": 3 }], // Location in the query
          "path": ["fieldName", "nestedField"],      // Path to the field that caused the error
          "extensions": {                            // Optional: Custom error details
            "code": "CUSTOM_ERROR_CODE",
            "timestamp": "2024-05-26T12:00:00.000Z"
            // ...other custom fields
          }
        }
      ],
      "data": null // Or partially resolved data
    }
    ```
    If custom error codes are used (via `extensions.code`), they will be documented here.

## 8. Rate Limiting

To ensure fair usage and protect the API from abuse, rate limiting is implemented.

*   **Example Limits (these are illustrative and subject to change):**
    *   **Anonymous users (by IP):** 60 requests per minute.
    *   **Authenticated users (by user ID):** 200 requests per minute.
*   **Response Headers:**
    When rate limits are applied, the following headers may be included in the response:
    *   `X-RateLimit-Limit`: The total number of requests allowed in the current window.
    *   `X-RateLimit-Remaining`: The number of requests remaining in the current window.
    *   `X-RateLimit-Reset`: The Unix timestamp (in seconds) when the rate limit will reset.
    *   `Retry-After`: (If a 429 Too Many Requests status is returned) The number of seconds to wait before making another request.
*   **Response Code:**
    *   `429 Too Many Requests`: Sent when a user exceeds the rate limit.

## 9. Further Information

This document provides a foundational overview. For more specific details on data models, business logic, and advanced configurations, refer to the source code, other documents in the `/docs` directory, and inline code comments.
The API will evolve; always refer to the latest version of this document or the API's self-documentation features (like GraphQL Playground) for the most current information.
