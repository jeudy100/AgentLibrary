# api-documenter

Generate API documentation from code analysis. Creates OpenAPI/Swagger specs, JSDoc, docstrings, and markdown documentation for REST APIs, GraphQL schemas, and library functions.

## Usage

```
/api-documenter <target> [--format <format>]
```

Targets:
- File path or directory
- `--all` for entire project

Formats:
- `openapi` - OpenAPI 3.0 specification (YAML/JSON)
- `swagger` - Swagger 2.0 specification
- `markdown` - Markdown documentation
- `jsdoc` - JSDoc comments
- `docstring` - Python docstrings
- `godoc` - Go documentation comments

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for API documentation. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: api, documentation, swagger, openapi, rest, graphql
> 3. **Existing Docs** - Find existing API docs, OpenAPI specs, swagger files
> 4. **API Routes** - Find route definitions, controllers, handlers
>
> Return: API framework, existing doc format, route patterns, authentication methods, base URL

From the Explore results, extract:
- API framework (Express, FastAPI, Gin, etc.)
- Existing documentation format
- Authentication methods
- API versioning strategy
- Base URL patterns

---

## Documentation Types

### OpenAPI/Swagger Generation

Analyze API routes and generate OpenAPI 3.0 specification.

**Route Detection Patterns:**

**Express.js:**
```javascript
app.get('/users/:id', handler)
router.post('/users', validate, createUser)
```

**FastAPI:**
```python
@app.get("/users/{user_id}")
@router.post("/users", response_model=User)
```

**Go (Gin/Echo/Chi):**
```go
r.GET("/users/:id", getUser)
r.POST("/users", createUser)
```

**Rails:**
```ruby
get '/users/:id', to: 'users#show'
resources :users
```

**Generated OpenAPI Structure:**
```yaml
openapi: 3.0.3
info:
  title: API Name
  description: API description from project analysis
  version: 1.0.0

servers:
  - url: http://localhost:3000
    description: Development server

paths:
  /users:
    get:
      summary: List all users
      operationId: listUsers
      tags:
        - Users
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
    post:
      summary: Create a user
      operationId: createUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: User created

  /users/{id}:
    get:
      summary: Get user by ID
      operationId: getUserById
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Successful response
        '404':
          description: User not found

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        email:
          type: string
          format: email
        name:
          type: string

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

---

### JSDoc Generation

Add JSDoc comments to JavaScript/TypeScript functions.

**Input:**
```javascript
async function getUserById(id, options) {
  const user = await db.users.findById(id);
  if (!user) throw new NotFoundError('User not found');
  return options.includeProfile ? { ...user, profile: await getProfile(id) } : user;
}
```

**Generated JSDoc:**
```javascript
/**
 * Retrieves a user by their unique identifier.
 *
 * @async
 * @param {string} id - The unique identifier of the user
 * @param {Object} options - Query options
 * @param {boolean} [options.includeProfile=false] - Whether to include user profile data
 * @returns {Promise<User>} The user object, optionally with profile
 * @throws {NotFoundError} When user with given ID does not exist
 *
 * @example
 * const user = await getUserById('123', { includeProfile: true });
 * console.log(user.name);
 */
async function getUserById(id, options) {
```

---

### Python Docstrings

Generate Google-style or NumPy-style docstrings.

**Input:**
```python
def calculate_discount(price, discount_percent, max_discount=None):
    if discount_percent < 0 or discount_percent > 100:
        raise ValueError("Discount must be between 0 and 100")
    discount = price * (discount_percent / 100)
    if max_discount:
        discount = min(discount, max_discount)
    return price - discount
```

**Generated (Google style):**
```python
def calculate_discount(price, discount_percent, max_discount=None):
    """Calculate the discounted price.

    Applies a percentage discount to the given price, optionally
    capped at a maximum discount amount.

    Args:
        price (float): The original price before discount.
        discount_percent (float): The discount percentage (0-100).
        max_discount (float, optional): Maximum discount amount.
            Defaults to None (no cap).

    Returns:
        float: The final price after applying the discount.

    Raises:
        ValueError: If discount_percent is not between 0 and 100.

    Examples:
        >>> calculate_discount(100, 20)
        80.0
        >>> calculate_discount(100, 50, max_discount=30)
        70.0
    """
```

---

### Go Documentation

Generate Go doc comments following conventions.

**Input:**
```go
func ParseConfig(path string, strict bool) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("reading config: %w", err)
    }
    // ... parsing logic
}
```

**Generated:**
```go
// ParseConfig reads and parses a configuration file from the given path.
//
// The strict parameter controls validation behavior:
//   - If true, unknown fields cause an error
//   - If false, unknown fields are ignored
//
// It returns the parsed Config and any error encountered.
// Common errors include file not found and invalid syntax.
//
// Example:
//
//	cfg, err := ParseConfig("/etc/myapp/config.yaml", true)
//	if err != nil {
//	    log.Fatal(err)
//	}
func ParseConfig(path string, strict bool) (*Config, error) {
```

---

### Markdown Documentation

Generate human-readable API reference in Markdown.

**Output Structure:**
```markdown
# API Reference

## Overview

Base URL: `https://api.example.com/v1`

Authentication: Bearer token in Authorization header

## Endpoints

### Users

#### List Users

`GET /users`

Returns a paginated list of users.

**Query Parameters**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| limit | integer | No | Max results (default: 10) |
| offset | integer | No | Pagination offset |

**Response**

```json
{
  "data": [
    {
      "id": "123",
      "email": "user@example.com",
      "name": "John Doe"
    }
  ],
  "total": 100,
  "limit": 10,
  "offset": 0
}
```

**Example**

```bash
curl -X GET "https://api.example.com/v1/users?limit=5" \
  -H "Authorization: Bearer <token>"
```

---

#### Create User

`POST /users`

Creates a new user account.

**Request Body**

```json
{
  "email": "user@example.com",
  "name": "John Doe",
  "password": "securepassword"
}
```

**Response** (201 Created)

```json
{
  "id": "123",
  "email": "user@example.com",
  "name": "John Doe",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

**Errors**

| Status | Description |
|--------|-------------|
| 400 | Invalid request body |
| 409 | Email already exists |
```

---

## Output Format

```
## API Documentation Generated

**Target**: [file or directory]
**Format**: [openapi/markdown/jsdoc/etc.]
**Output**: [output file path]

### Summary

| Metric | Count |
|--------|-------|
| Endpoints | 15 |
| Schemas | 8 |
| Operations | 23 |

### Endpoints Documented

| Method | Path | Summary |
|--------|------|---------|
| GET | /users | List users |
| POST | /users | Create user |
| GET | /users/{id} | Get user |

### Schemas Generated

| Name | Fields | Used In |
|------|--------|---------|
| User | 5 | GET /users, GET /users/{id} |
| CreateUserRequest | 3 | POST /users |

### Output File

[Path to generated documentation]

To view: [command to serve/view docs]
```

---

## Error Handling

### No API Routes Found

```
Question: "Could not detect API routes. What should I do?"
Options:
  - Specify the routes directory manually
  - Scan for common API patterns
  - Generate from type definitions only
  - Cancel
```

### Unknown Framework

```
Question: "Could not detect API framework. Which are you using?"
Options:
  - Express.js (Node.js)
  - FastAPI (Python)
  - Gin/Echo/Chi (Go)
  - Rails (Ruby)
  - Specify other
```

### Incomplete Type Information

```
Question: "Some endpoints lack type information. How should I proceed?"
Options:
  - Use 'any' type and add TODO comments
  - Infer types from usage
  - Skip undocumented endpoints
  - Cancel
```

### Existing Documentation

```
Question: "Existing documentation found at [path]. What should I do?"
Options:
  - Merge with existing (add missing)
  - Replace existing
  - Create new file with different name
  - Cancel
```

## Important Notes

- Preserve existing documentation when possible
- Include authentication requirements for each endpoint
- Document error responses, not just success
- Use consistent naming (operationId, schemas)
- Add examples for complex request/response bodies
- Consider versioning in API paths
- Link related endpoints and schemas
- Validate generated OpenAPI specs with linters
- Keep docs close to code (inline comments) when possible
