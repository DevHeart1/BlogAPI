# Database Schema Design: BlogAPI Backend

**Version:** 1.0
**Date:** May 24, 2025
**Status:** Proposed

## 1. Introduction

This document provides a detailed design of the database schema for the BlogAPI backend. It is based on the data models outlined in the Product Requirements Document (`BlogAPI_PRD.md`, section 5.3) and supports the functionalities described therein. The schema aims to be efficient, scalable, and maintainable.

This document is intended for developers and database administrators involved in the project.

## 2. Guiding Principles for Schema Design

*   **Normalization:** Aim for a reasonable level of normalization (e.g., 3NF) to reduce data redundancy and improve data integrity, while balancing query performance.
*   **Clarity & Consistency:** Use clear and consistent naming conventions for tables and columns.
*   **Performance:** Design with query performance in mind, including appropriate indexing strategies.
*   **Scalability:** The schema should be able to accommodate future growth in data volume and complexity.
*   **Integrity:** Utilize database constraints (Primary Keys, Foreign Keys, NOT NULL, UNIQUE) to enforce data integrity at the database level.
*   **Flexibility:** Allow for future extensions and modifications with minimal disruption.

## 3. Entity-Relationship Diagram (ERD)

Below is a conceptual ERD representing the main entities and their relationships. Specific data types and constraints are detailed in the subsequent sections.

```mermaid
erDiagram
    USERS {
        id INT PK
        username VARCHAR UNIQUE
        email VARCHAR UNIQUE
        password_hash VARCHAR
        role VARCHAR "(admin, editor, user)"
        bio TEXT
        created_at TIMESTAMP
        updated_at TIMESTAMP
    }

    POSTS {
        id INT PK
        title VARCHAR
        slug VARCHAR UNIQUE
        content_markdown TEXT
        author_id INT FK
        status VARCHAR "(draft, published, archived)"
        created_at TIMESTAMP
        updated_at TIMESTAMP
        published_at TIMESTAMP NULL
    }

    TAGS {
        id INT PK
        name VARCHAR UNIQUE
        slug VARCHAR UNIQUE
        created_at TIMESTAMP
        updated_at TIMESTAMP
    }

    COMMENTS {
        id INT PK
        post_id INT FK
        user_id INT FK "Can be NULL for anonymous comments if allowed"
        content TEXT
        created_at TIMESTAMP
        updated_at TIMESTAMP
    }

    POST_TAGS {
        post_id INT PK FK
        tag_id INT PK FK
    }

    POST_VERSIONS {
        id INT PK
        post_id INT FK
        content_markdown TEXT
        version_number INT
        created_at TIMESTAMP
    }

    USERS ||--o{ POSTS : "authors"
    USERS ||--o{ COMMENTS : "comments_by"
    POSTS ||--o{ COMMENTS : "has"
    POSTS ||--o{ POST_VERSIONS : "has_versions"
    POSTS }o--o{ POST_TAGS : "categorized_by"
    TAGS }o--o{ POST_TAGS : "categorizes"
```

## 4. Detailed Table Designs

Common conventions:
*   `id`: Typically an auto-incrementing integer primary key.
*   `created_at`: Timestamp of when the record was created (default to current time).
*   `updated_at`: Timestamp of when the record was last updated (auto-update on modification).
*   Data types are generic SQL types; specific RDBMS types (e.g., `SERIAL` for PostgreSQL auto-incrementing PK, `VARCHAR(255)`) will be chosen during implementation.

### 4.1. `users` Table

*   **Description:** Stores information about registered users.
*   **Columns:**
    | Column Name     | Data Type     | Constraints                                       | Notes                                     |
    |-----------------|---------------|---------------------------------------------------|-------------------------------------------|
    | `id`            | INT           | PRIMARY KEY, AUTO_INCREMENT                       | Unique identifier for the user            |
    | `username`      | VARCHAR       | NOT NULL, UNIQUE                                  | User's chosen username                    |
    | `email`         | VARCHAR       | NOT NULL, UNIQUE                                  | User's email address                      |
    | `password_hash` | VARCHAR       | NOT NULL                                          | Hashed password                           |
    | `role`          | VARCHAR       | NOT NULL, CHECK (`role` IN ('admin', 'editor', 'user')) | User's role (admin, editor, user)         |
    | `bio`           | TEXT          | NULL                                              | Optional user biography                   |
    | `created_at`    | TIMESTAMP     | NOT NULL, DEFAULT CURRENT_TIMESTAMP               | Record creation timestamp                 |
    | `updated_at`    | TIMESTAMP     | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Record last update timestamp              |
*   **Indexes:**
    *   PRIMARY KEY (`id`)
    *   UNIQUE (`username`)
    *   UNIQUE (`email`)
    *   INDEX (`role`) (if frequently queried by role)
*   **Relationships:**
    *   One-to-Many with `posts` (a user can author many posts).
    *   One-to-Many with `comments` (a user can write many comments).

### 4.2. `posts` Table

*   **Description:** Stores blog post content.
*   **Columns:**
    | Column Name        | Data Type     | Constraints                                       | Notes                                     |
    |--------------------|---------------|---------------------------------------------------|-------------------------------------------|
    | `id`               | INT           | PRIMARY KEY, AUTO_INCREMENT                       | Unique identifier for the post            |
    | `title`            | VARCHAR       | NOT NULL                                          | Title of the post                         |
    | `slug`             | VARCHAR       | NOT NULL, UNIQUE                                  | URL-friendly version of the title         |
    | `content_markdown` | TEXT          | NOT NULL                                          | Post content in Markdown format           |
    | `author_id`        | INT           | NOT NULL, FOREIGN KEY REFERENCES `users(id)`      | ID of the user who authored the post      |
    | `status`           | VARCHAR       | NOT NULL, CHECK (`status` IN ('draft', 'published', 'archived')) | Current status of the post              |
    | `created_at`       | TIMESTAMP     | NOT NULL, DEFAULT CURRENT_TIMESTAMP               | Record creation timestamp                 |
    | `updated_at`       | TIMESTAMP     | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Record last update timestamp              |
    | `published_at`     | TIMESTAMP     | NULL                                              | Timestamp when the post was published     |
*   **Indexes:**
    *   PRIMARY KEY (`id`)
    *   UNIQUE (`slug`)
    *   INDEX (`author_id`)
    *   INDEX (`status`)
    *   INDEX (`published_at`) (for querying by publication date)
*   **Relationships:**
    *   Many-to-One with `users` (a post is authored by one user).
    *   One-to-Many with `comments` (a post can have many comments).
    *   Many-to-Many with `tags` (via `post_tags` table).
    *   One-to-Many with `post_versions` (a post can have multiple versions).

### 4.3. `tags` Table

*   **Description:** Stores tags that can be applied to posts.
*   **Columns:**
    | Column Name  | Data Type | Constraints                                       | Notes                                     |
    |--------------|-----------|---------------------------------------------------|-------------------------------------------|
    | `id`         | INT       | PRIMARY KEY, AUTO_INCREMENT                       | Unique identifier for the tag             |
    | `name`       | VARCHAR   | NOT NULL, UNIQUE                                  | Name of the tag                           |
    | `slug`       | VARCHAR   | NOT NULL, UNIQUE                                  | URL-friendly version of the tag name      |
    | `created_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP               | Record creation timestamp                 |
    | `updated_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Record last update timestamp              |
*   **Indexes:**
    *   PRIMARY KEY (`id`)
    *   UNIQUE (`name`)
    *   UNIQUE (`slug`)
*   **Relationships:**
    *   Many-to-Many with `posts` (via `post_tags` table).

### 4.4. `comments` Table

*   **Description:** Stores comments made on posts.
*   **Columns:**
    | Column Name  | Data Type | Constraints                                       | Notes                                     |
    |--------------|-----------|---------------------------------------------------|-------------------------------------------|
    | `id`         | INT       | PRIMARY KEY, AUTO_INCREMENT                       | Unique identifier for the comment         |
    | `post_id`    | INT       | NOT NULL, FOREIGN KEY REFERENCES `posts(id)`      | ID of the post the comment belongs to     |
    | `user_id`    | INT       | NULL, FOREIGN KEY REFERENCES `users(id)`          | ID of the user who wrote the comment (can be NULL for anonymous comments if allowed) |
    | `content`    | TEXT      | NOT NULL                                          | Content of the comment                    |
    | `created_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP               | Record creation timestamp                 |
    | `updated_at` | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Record last update timestamp              |
*   **Indexes:**
    *   PRIMARY KEY (`id`)
    *   INDEX (`post_id`)
    *   INDEX (`user_id`)
*   **Relationships:**
    *   Many-to-One with `posts` (a comment belongs to one post).
    *   Many-to-One with `users` (a comment is written by one user, if not anonymous).

### 4.5. `post_tags` Table (Join Table)

*   **Description:** Links posts with their associated tags (Many-to-Many relationship).
*   **Columns:**
    | Column Name | Data Type | Constraints                                       | Notes                                     |
    |-------------|-----------|---------------------------------------------------|-------------------------------------------|
    | `post_id`   | INT       | NOT NULL, FOREIGN KEY REFERENCES `posts(id)`      | ID of the post                            |
    | `tag_id`    | INT       | NOT NULL, FOREIGN KEY REFERENCES `tags(id)`       | ID of the tag                             |
*   **Indexes:**
    *   PRIMARY KEY (`post_id`, `tag_id`)
    *   INDEX (`tag_id`) (to find all posts for a tag)
*   **Relationships:**
    *   Connects `posts` and `tags`.

### 4.6. `post_versions` Table

*   **Description:** Stores historical versions of post content.
*   **Columns:**
    | Column Name        | Data Type | Constraints                                       | Notes                                     |
    |--------------------|-----------|---------------------------------------------------|-------------------------------------------|
    | `id`               | INT       | PRIMARY KEY, AUTO_INCREMENT                       | Unique identifier for the version record  |
    | `post_id`          | INT       | NOT NULL, FOREIGN KEY REFERENCES `posts(id)` ON DELETE CASCADE | ID of the post this version belongs to    |
    | `content_markdown` | TEXT      | NOT NULL                                          | Post content in Markdown for this version |
    | `version_number`   | INT       | NOT NULL                                          | Sequential version number for the post    |
    | `created_at`       | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP               | Timestamp when this version was created   |
*   **Indexes:**
    *   PRIMARY KEY (`id`)
    *   UNIQUE (`post_id`, `version_number`)
    *   INDEX (`post_id`)
*   **Relationships:**
    *   Many-to-One with `posts` (a version belongs to one post).

## 5. Data Types Rationale

*   **INT/INTEGER:** Used for primary keys and foreign keys, offering good performance for joins and lookups.
*   **VARCHAR/STRING:** Used for text fields with a known or reasonable maximum length (e.g., usernames, emails, titles, slugs, roles, status). Lengths will be specified during implementation (e.g., `VARCHAR(255)`).
*   **TEXT:** Used for longer text content where the length is variable and can be substantial (e.g., post content, comments, user bios).
*   **TIMESTAMP/DATETIME:** Used for recording creation and update times, and specific event times like `published_at`.
*   **BOOLEAN:** Could be used for binary flags if needed, though not explicitly in the current PRD models (e.g., `is_active`).

## 6. Relationships Overview

*   **Users to Posts:** One-to-Many (An author can have many posts; a post has one author).
*   **Users to Comments:** One-to-Many (A user can write many comments; a comment is written by one user, or can be anonymous).
*   **Posts to Comments:** One-to-Many (A post can have many comments; a comment belongs to one post).
*   **Posts to Tags:** Many-to-Many (A post can have many tags; a tag can be on many posts). Implemented via the `post_tags` join table.
*   **Posts to PostVersions:** One-to-Many (A post can have many versions; a version belongs to one post).

## 7. Future Considerations / Potential Schema Evolution

*   **Comment Threading:** If threaded comments are required, a `parent_comment_id` (self-referencing FK) could be added to the `comments` table.
*   **Comment Moderation:** A `status` field (e.g., 'pending', 'approved', 'rejected') could be added to the `comments` table.
*   **User Profile Pictures/Avatars:** A `avatar_url` field in the `users` table.
*   **Media/Image Management:** If direct image/media uploads are supported (beyond Markdown links), a separate `media` table might be needed, linked to posts or users.
*   **Reactions/Likes:** A `reactions` table could be added to allow users to react to posts or comments.
*   **Audit Logs:** A more comprehensive audit logging table for tracking changes to critical data, if simple `updated_at` is insufficient.

This schema design provides a robust foundation for BlogAPI. It should be reviewed and refined as development progresses and specific database technology choices are made.
