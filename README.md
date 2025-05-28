# Technical Specification for Authorization Service Development

## Project Goal
Create an authorization service in Go for registration, authentication, token management, and providing user information using PostgreSQL and Redis.

## Requirements

### Technology Stack
- **Language**: Go (latest stable version)
- **Database**:
  - PostgreSQL for storing user information
  - Redis for storing access/refresh tokens and blacklisted tokens
- **Libraries and Frameworks**:
  - `gin` or `fiber` for HTTP server
  - `sqlx` or `pgx` for working with PostgreSQL
  - `go-redis` for working with Redis
  - `jwt` for token generation and validation
  - `zap` or `logrus` for logging
  - `testify` for testing
- **Tools**:
  - `golang-migrate` for database migrations
  - Docker for containerizing the service and dependencies
  - `godotenv` for configuration management

### Functional Requirements
1. **Endpoints**:
   - **/signup** (POST):
     - User registration.
     - Input data: email, password, name.
     - Password hashing (bcrypt).
     - Save to PostgreSQL.
     - Return: status 201, JSON with user ID.
   - **/signin** (POST):
     - User authentication.
     - Input data: email, password.
     - Password verification, generation of access (JWT, 15 min) and refresh (Redis, 7 days) tokens.
     - Return: JSON with access tokens.
   - **/refresh** (POST):
     - Refresh access token using refresh token.
     - Check refresh token in Redis.
     - Automatic logout if refresh token is expired (remove from Redis).
     - Return: new access token or error.
   - **/logout** (POST):
     - User logout.
     - Add access token to blacklist (Redis, TTL = token lifetime).
     - Remove refresh token from Redis.
     - Return: status 200.
   - **/echo** (GET):
     - Available only for authorized users.
     - Check access token (including blacklist).
     - Return: JSON with complete user information (ID, email, name).

### Non-functional Requirements
1. **Logging**:
   - Use `zap` or `logrus`.
   - Log all requests, errors, and key events (registration, login, logout, token refresh).
   - Format: JSON, with fields: timestamp, level, message, context (e.g., user_id, endpoint).
   - Output: stdout

2. **Error Handling**:
   - Unified error format: JSON `{ "error": "description" }`.
   - HTTP codes: 400 (invalid data), 401 (unauthorized), 403 (forbidden), 404 (not found), 500 (internal error).
   - Handle database errors, Redis errors, JWT errors.

3. **Testing**:
   - Unit tests for business logic (hashing, token generation, etc.).
   - Tests for error handling (invalid tokens, expired tokens, incorrect passwords, etc.).

4. **Migrations**:
   - Use `golang-migrate` or `goose`.
   - Migrations for creating `users` table.
   - Support for rollback (down migrations).
