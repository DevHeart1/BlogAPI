# BlogAPI

BlogAPI is a feature-rich backend system for a modern blogging platform. It offers robust support for both RESTful and GraphQL APIs, making it a versatile solution for various frontend applications, including headless CMS setups, custom blog interfaces, and content-driven platforms. The system is built with a focus on scalability, maintainability, and developer productivity.

## Features

*   **Dual API Support:** Comprehensive RESTful and GraphQL APIs for flexible data access.
*   **Role-Based Access Control (RBAC):** Predefined roles (admin, editor, user) with distinct permissions.
*   **Content Management:** Full CRUD (Create, Read, Update, Delete) operations for:
    *   Posts (with Markdown support and versioning)
    *   Tags
    *   Comments
*   **User Management:** CRUD operations for user accounts.
*   **Advanced Post Features:**
    *   **Markdown Parsing:** Store and manage blog content in Markdown.
    *   **Post Versioning:** Keep track of changes to posts and revert to previous versions.
*   **Efficient Data Retrieval:**
    *   **Pagination:** For handling large datasets efficiently.
    *   **Search Functionality:** To find posts and other content.
*   **Authentication:** Secure JWT-based authentication.

## Technology Stack

The BlogAPI is built with a modern, robust technology stack:

*   **Programming Language:** Node.js (with TypeScript)
*   **Backend Framework:** Express.js
*   **Database:** PostgreSQL
*   **ORM:** TypeORM
*   **GraphQL:** Apollo Server
*   **Authentication:** JSON Web Tokens (JWT) using `jsonwebtoken` and `bcrypt` for password hashing.
*   **Containerization:** Docker support for consistent development and deployment environments.
*   **CI/CD:** Automated workflows using GitHub Actions.

For a more detailed breakdown of the technology choices and rationale, please see the [Tech Stack Document](docs/TechStack.md).

## Getting Started

Follow these instructions to get a local development environment up and running.

### Prerequisites

*   Node.js (LTS version recommended)
*   npm or yarn package manager
*   Docker (for running PostgreSQL or the full application in a container)
*   A running instance of PostgreSQL (can be local or a Docker container)

### Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/your-username/BlogAPI.git
    cd BlogAPI
    ```
2.  **Install dependencies:**
    ```bash
    npm install
    # or
    # yarn install
    ```

### Configuration

1.  Create a `.env` file in the root of the project by copying the example file:
    ```bash
    cp .env.example .env
    ```
2.  Update the `.env` file with your local development settings, especially database connection details (DATABASE_URL or individual DB_HOST, DB_PORT, DB_USERNAME, DB_PASSWORD, DB_DATABASE variables), JWT secret (JWT_SECRET), and any other required environment variables.

    Example `DATABASE_URL` format:
    `POSTGRES_URL="postgresql://user:password@localhost:5432/blogapi_dev"`

### Running the Application (Development Mode)

```bash
npm run start:dev
# or
# yarn start:dev
```
This will start the Express.js development server, typically on `http://localhost:3000` (or as configured in your `.env` file, e.g., 8080). The server will automatically reload if you make changes to the source code (if using a tool like `nodemon`).

For more detailed setup instructions, including running with Docker, database migrations, and seeding, please refer to the [API and Project Documentation (BlogAPIDoc.md)](docs/BlogAPIDoc.md).

## Project Structure

The project follows a standard application structure suitable for Express.js with TypeScript:

*   `src/`: Contains the core application code, including modules, controllers, services, entities, etc.
*   `tests/`: Includes unit tests and end-to-end (e2e) tests.
*   `docs/`: Contains project documentation files like this README, PRD, Tech Stack, API documentation, etc.

For a detailed explanation of the backend architecture and directory layout, please see the [Backend Structure Document](docs/BackendStructure.md).

## API Documentation

Detailed API documentation, including endpoint descriptions, request/response examples, and authentication details for both REST and GraphQL APIs, can be found in:

*   **[API and Project Documentation (BlogAPIDoc.md)](docs/BlogAPIDoc.md)**

The GraphQL API also supports schema exploration via tools like GraphQL Playground, typically available at `/graphql` when the server is running.

## Contributing

Contributions are welcome! If you'd like to contribute to BlogAPI, please follow these steps:

1.  Fork the repository.
2.  Create a new branch for your feature or bug fix (`git checkout -b feature/your-feature-name` or `bugfix/issue-number`).
3.  Make your changes and commit them with clear, descriptive messages.
4.  Push your changes to your forked repository.
5.  Create a Pull Request (PR) against the `main` branch of the original repository.

Please ensure your code adheres to the project's coding standards and all tests pass.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
