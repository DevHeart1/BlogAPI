# BlogAPI - Database Schema Design

**Version:** 1.0
**Last Updated:** May 26, 2025

## 1. Introduction

This document details the database schema for the BlogAPI project. It includes an Entity-Relationship Diagram (ERD), detailed table designs, data type justifications, and indexing strategies. The schema is designed for PostgreSQL and will be managed using TypeORM.

## 2. Guiding Principles

*   **Normalization:** Aim for a reasonable level of normalization (typically 3NF) to reduce data redundancy and improve data integrity.
*   **Performance:** Design for efficient querying by including appropriate indexes and considering common access patterns.
*   **Scalability:** The schema should be able to accommodate future growth in data volume and complexity.
*   **Clarity:** Table and column names should be clear, consistent, and self-explanatory.
*   **TypeORM Compatibility:** Design choices should align well with TypeORM capabilities and conventions.

## 3. Entity-Relationship Diagram (ERD)

```mermaid
erDiagram
    users {
        SERIAL id PK
        VARCHAR(255) username UK
        VARCHAR(255) email UK
        VARCHAR(255) password_hash
        USER_ROLE_ENUM role DEFAULT 'user'
        TEXT bio
        VARCHAR(255) profile_picture_url
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    posts {
        SERIAL id PK
        INTEGER author_id FK "REFERENCES users(id) ON DELETE SET NULL"
        VARCHAR(255) title
        VARCHAR(255) slug UK
        TEXT content_markdown
        POST_STATUS_ENUM status DEFAULT 'draft'
        INTEGER current_version_id "Nullable, self-referential to post_versions"
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
        TIMESTAMPTZ published_at
    }

    tags {
        SERIAL id PK
        VARCHAR(255) name UK
        VARCHAR(255) slug UK
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    comments {
        SERIAL id PK
        INTEGER post_id FK "REFERENCES posts(id) ON DELETE CASCADE"
        INTEGER user_id FK "REFERENCES users(id) ON DELETE SET NULL"
        TEXT content
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    post_tags {
        INTEGER post_id PK FK "REFERENCES posts(id) ON DELETE CASCADE"
        INTEGER tag_id PK FK "REFERENCES tags(id) ON DELETE CASCADE"
    }

    post_versions {
        SERIAL id PK
        INTEGER post_id FK "REFERENCES posts(id) ON DELETE CASCADE"
        INTEGER version_number
        TEXT content_markdown
        INTEGER editor_id FK "REFERENCES users(id) ON DELETE SET NULL"
        TIMESTAMPTZ created_at
        VARCHAR(255) change_summary "Optional summary of changes for this version"
    }

    users ||--o{ posts : "authored by (optional)"
    users ||--o{ comments : "commented by (optional)"
    users ||--o{ post_versions : "edited by (optional)"

    posts ||--o{ comments : "has comments"
    posts ||--o{ post_tags : "has tags"
    posts ||--o{ post_versions : "has versions"

    tags ||--o{ post_tags : "used in posts"

    posts }o--|| post_versions : "current version (circular, conceptual)"
```

*(Note: The `current_version_id` in `posts` pointing to `post_versions` creates a conceptual link. This might be handled at the application layer or via triggers if strict FK is problematic for bootstrapping. For simplicity, it's shown here conceptually. TypeORM might manage this relation without a direct FK if preferred.)*

## 4. Detailed Table Designs

### 4.1. `users` Table

Stores information about registered users.

| Column Name           | Data Type        | Constraints & Properties                                       | Notes                                     |
|-----------------------|------------------|----------------------------------------------------------------|-------------------------------------------|
| `id`                  | `SERIAL`         | `PRIMARY KEY`                                                  | Auto-incrementing integer ID.             |
| `username`            | `VARCHAR(255)`   | `NOT NULL`, `UNIQUE`                                           | User's chosen username.                   |
| `email`               | `VARCHAR(255)`   | `NOT NULL`, `UNIQUE`                                           | User's email address.                     |
| `password_hash`       | `VARCHAR(255)`   | `NOT NULL`                                                     | Hashed password (e.g., using bcrypt).     |
| `role`                | `VARCHAR(50)`    | `NOT NULL`, `DEFAULT 'user'`, `CHECK (role IN ('user', 'editor', 'admin'))` | User role (custom ENUM type `USER_ROLE_ENUM` preferred via TypeORM). |
| `bio`                 | `TEXT`           | `NULLABLE`                                                     | User's biography.                         |
| `profile_picture_url` | `VARCHAR(255)`   | `NULLABLE`                                                     | URL to the user's profile picture.        |
| `created_at`          | `TIMESTAMPTZ`    | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP`                        | Timestamp of user creation.               |
| `updated_at`          | `TIMESTAMPTZ`    | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP`                        | Timestamp of last user update.            |

**Indexes:**
*   `PRIMARY KEY` on `(id)`
*   `UNIQUE` constraint on `(username)` creates an index.
*   `UNIQUE` constraint on `(email)` creates an index.
*   Index on `(role)` for filtering by role.

### 4.2. `posts` Table

Stores blog post information.

| Column Name        | Data Type        | Constraints & Properties                                               | Notes                                     |
|--------------------|------------------|------------------------------------------------------------------------|-------------------------------------------|
| `id`               | `SERIAL`         | `PRIMARY KEY`                                                          | Auto-incrementing integer ID.             |
| `author_id`        | `INTEGER`        | `NULLABLE`, `REFERENCES users(id) ON DELETE SET NULL ON UPDATE CASCADE`  | Foreign key to `users` table.             |
| `title`            | `VARCHAR(255)`   | `NOT NULL`                                                             | Title of the post.                        |
| `slug`             | `VARCHAR(255)`   | `NOT NULL`, `UNIQUE`                                                   | URL-friendly slug for the post.           |
| `content_markdown` | `TEXT`           | `NOT NULL`                                                             | Post content in Markdown format.          |
| `status`           | `VARCHAR(50)`    | `NOT NULL`, `DEFAULT 'draft'`, `CHECK (status IN ('draft', 'published', 'archived'))` | Post status (custom ENUM `POST_STATUS_ENUM` preferred). |
| `current_version_id`| `INTEGER`       | `NULLABLE`                                                             | FK to `post_versions(id)`, conceptually.  |
| `created_at`       | `TIMESTAMPTZ`    | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP`                                | Timestamp of post creation.               |
| `updated_at`       | `TIMESTAMPTZ`    | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP`                                | Timestamp of last post update.            |
| `published_at`     | `TIMESTAMPTZ`    | `NULLABLE`                                                             | Timestamp when the post was published.    |

**Indexes:**
*   `PRIMARY KEY` on `(id)`
*   `UNIQUE` constraint on `(slug)` creates an index.
*   Index on `(author_id)` for querying posts by author.
*   Index on `(status)` for filtering posts by status.
*   Index on `(published_at)` for sorting/filtering by publication date.
*   GIN index on `(title, content_markdown)` for full-text search capabilities (e.g., using `to_tsvector`).

### 4.3. `tags` Table

Stores tags that can be applied to posts.

| Column Name  | Data Type        | Constraints & Properties                        | Notes                                     |
|--------------|------------------|-------------------------------------------------|-------------------------------------------|
| `id`         | `SERIAL`         | `PRIMARY KEY`                                   | Auto-incrementing integer ID.             |
| `name`       | `VARCHAR(255)`   | `NOT NULL`, `UNIQUE`                            | Tag name (e.g., "Technology").            |
| `slug`       | `VARCHAR(255)`   | `NOT NULL`, `UNIQUE`                            | URL-friendly slug for the tag.            |
| `created_at` | `TIMESTAMPTZ`    | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP`         | Timestamp of tag creation.                |
| `updated_at` | `TIMESTAMPTZ`    | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP`         | Timestamp of last tag update.             |

**Indexes:**
*   `PRIMARY KEY` on `(id)`
*   `UNIQUE` constraint on `(name)` creates an index.
*   `UNIQUE` constraint on `(slug)` creates an index.

### 4.4. `comments` Table

Stores comments made on posts.

| Column Name  | Data Type     | Constraints & Properties                                               | Notes                                     |
|--------------|---------------|------------------------------------------------------------------------|-------------------------------------------|
| `id`         | `SERIAL`      | `PRIMARY KEY`                                                          | Auto-incrementing integer ID.             |
| `post_id`    | `INTEGER`     | `NOT NULL`, `REFERENCES posts(id) ON DELETE CASCADE ON UPDATE CASCADE`   | Foreign key to `posts` table.             |
| `user_id`    | `INTEGER`     | `NULLABLE`, `REFERENCES users(id) ON DELETE SET NULL ON UPDATE CASCADE`  | Foreign key to `users` table (commenter). |
| `content`    | `TEXT`        | `NOT NULL`                                                             | Content of the comment.                   |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP`                                | Timestamp of comment creation.            |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP`                                | Timestamp of last comment update.         |

**Indexes:**
*   `PRIMARY KEY` on `(id)`
*   Index on `(post_id)` for retrieving comments for a post.
*   Index on `(user_id)` for retrieving comments by a user.

### 4.5. `post_tags` Table (Junction Table)

Links posts with tags (many-to-many relationship).

| Column Name | Data Type | Constraints & Properties                                             | Notes                         |
|-------------|-----------|----------------------------------------------------------------------|-------------------------------|
| `post_id`   | `INTEGER` | `PRIMARY KEY`, `REFERENCES posts(id) ON DELETE CASCADE ON UPDATE CASCADE` | Foreign key to `posts` table. |
| `tag_id`    | `INTEGER` | `PRIMARY KEY`, `REFERENCES tags(id) ON DELETE CASCADE ON UPDATE CASCADE`  | Foreign key to `tags` table.  |

**Indexes:**
*   `PRIMARY KEY` on `(post_id, tag_id)`.
*   Additional index on `(tag_id, post_id)` for efficiently finding posts associated with a specific tag.

### 4.6. `post_versions` Table

Stores historical versions of posts.

| Column Name        | Data Type        | Constraints & Properties                                               | Notes                                     |
|--------------------|------------------|------------------------------------------------------------------------|-------------------------------------------|
| `id`               | `SERIAL`         | `PRIMARY KEY`                                                          | Auto-incrementing integer ID.             |
| `post_id`          | `INTEGER`        | `NOT NULL`, `REFERENCES posts(id) ON DELETE CASCADE ON UPDATE CASCADE`   | Foreign key to the parent `posts` table.  |
| `version_number`   | `INTEGER`        | `NOT NULL`                                                             | Sequential version number for the post.   |
| `content_markdown` | `TEXT`           | `NOT NULL`                                                             | Post content for this version.            |
| `editor_id`        | `INTEGER`        | `NULLABLE`, `REFERENCES users(id) ON DELETE SET NULL ON UPDATE CASCADE`  | User who created this version.            |
| `created_at`       | `TIMESTAMPTZ`    | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP`                                | Timestamp of when this version was saved. |
| `change_summary`   | `VARCHAR(255)`   | `NULLABLE`                                                             | Optional summary of changes.              |

**Indexes:**
*   `PRIMARY KEY` on `(id)`
*   Index on `(post_id, version_number)` for unique identification and ordering of versions for a post (can be a `UNIQUE` constraint).
*   Index on `(post_id, created_at)` for retrieving versions in chronological order.
*   Index on `(editor_id)`.

## 5. Data Types Rationale

*   **`SERIAL` / `BIGSERIAL`:** Used for primary keys (`id` columns). PostgreSQL's `SERIAL` automatically creates an auto-incrementing integer sequence, ensuring unique identifiers for new rows. `BIGSERIAL` can be used if a larger range of IDs is anticipated.
*   **`VARCHAR(n)`:** Used for strings with a known maximum length (e.g., `username`, `email`, `slug`, `title`). This can offer minor optimization and data validation at the database level. For TypeORM, this typically maps to `string`.
*   **`TEXT`:** Used for strings of variable and potentially long length (e.g., `content_markdown`, `bio`). PostgreSQL's `TEXT` type is efficient for this purpose and has no practical length limit beyond system memory. For TypeORM, this also maps to `string`.
*   **`INTEGER`:** Standard integer type used for foreign keys and numerical fields like `version_number`.
*   **`TIMESTAMPTZ` (Timestamp with Time Zone):** Used for all date/time fields (`created_at`, `updated_at`, `published_at`). Storing timestamps with time zone information is crucial for applications that may operate across different time zones or have users in various locations. It ensures that timestamps are unambiguous and can be correctly converted to local time zones when displayed. TypeORM typically maps JavaScript `Date` objects to this type.
*   **Custom ENUM Types (e.g., `USER_ROLE_ENUM`, `POST_STATUS_ENUM`):** While shown as `VARCHAR` with `CHECK` constraints in the SQL-like definitions above for clarity, PostgreSQL supports native ENUM types. TypeORM allows defining and using these ENUMs, providing better type safety and data integrity than plain strings with check constraints. Example: `CREATE TYPE USER_ROLE_ENUM AS ENUM ('user', 'editor', 'admin');`
*   **`BOOLEAN`:** (Not explicitly used in the main tables above but available) For true/false values.

## 6. Relationships and Constraints

*   **Foreign Keys:** Enforce referential integrity between tables.
*   **Cascading Behavior:**
    *   `ON DELETE SET NULL`: Used for relationships where the child record can exist without the parent (e.g., a post's author is deleted, the post might become "anonymous" or assigned to a default author).
    *   `ON DELETE CASCADE`: Used when child records are intrinsically linked to the parent and should not exist independently (e.g., comments of a deleted post, entries in `post_tags` junction table).
    *   `ON UPDATE CASCADE`: Generally a safe default, ensuring that if a parent key changes, the child keys are updated to match.
*   **`UNIQUE` Constraints:** Ensure uniqueness for fields like `username`, `email`, `slugs`.
*   **`NOT NULL` Constraints:** Enforce mandatory fields.
*   **`DEFAULT` Values:** Provide default values for fields like `role`, `status`, `created_at`.
*   **`CHECK` Constraints:** Enforce specific value ranges or formats (e.g., for `role` and `status` if not using native ENUMs).

## 7. Future Considerations

*   **Partitioning:** For very large tables like `posts` or `comments`, partitioning strategies might be considered in the future to improve performance and manageability.
*   **JSONB for flexible fields:** If certain entities require highly flexible or unstructured data (e.g., custom fields for posts), PostgreSQL's `JSONB` type could be utilized.
```
