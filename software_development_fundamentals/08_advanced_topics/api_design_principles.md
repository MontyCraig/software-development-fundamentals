# API Design Principles

## Overview

An Application Programming Interface (API) is a contract between software components. A
well-designed API is intuitive, consistent, and evolvable. A poorly designed API creates
confusion, bugs, and maintenance nightmares. Whether you are building a public-facing web
API, an internal service interface, or a library, the principles of good API design apply.

The most common API paradigms for networked services are REST (Representational State
Transfer), RPC (Remote Procedure Call), and GraphQL. Each has strengths suited to different
use cases. Beyond choosing a paradigm, API designers must address cross-cutting concerns:
versioning, authentication, rate limiting, pagination, error handling, and documentation.

This chapter covers the fundamental principles and patterns for designing APIs that are
easy to use correctly and hard to use incorrectly. The examples use generic HTTP-style
conventions without reference to any specific implementation framework.

## Key Concepts

### REST Principles

REST models an API as a set of resources, each identified by a URL, manipulated through
standard operations.

```
  Resource-Oriented Design:

  /users                    --> Collection of users
  /users/42                 --> Specific user (id=42)
  /users/42/orders          --> Orders belonging to user 42
  /users/42/orders/7        --> Specific order for user 42

  Standard Operations (HTTP Methods):
  +--------+-------------------+----------------------------+
  | Method | URL               | Action                     |
  +--------+-------------------+----------------------------+
  | GET    | /users            | List all users             |
  | GET    | /users/42         | Get user 42                |
  | POST   | /users            | Create a new user          |
  | PUT    | /users/42         | Replace user 42 entirely   |
  | PATCH  | /users/42         | Update user 42 partially   |
  | DELETE | /users/42         | Delete user 42             |
  +--------+-------------------+----------------------------+
```

#### HTTP Status Codes

```
  +------+----------------------------------+
  | Code | Meaning                          |
  +------+----------------------------------+
  | 200  | OK -- Success                    |
  | 201  | Created -- Resource created      |
  | 204  | No Content -- Success, no body   |
  | 400  | Bad Request -- Client error      |
  | 401  | Unauthorized -- Auth required    |
  | 403  | Forbidden -- Insufficient perms  |
  | 404  | Not Found -- Resource missing    |
  | 409  | Conflict -- State conflict       |
  | 429  | Too Many Requests -- Rate limit  |
  | 500  | Internal Server Error            |
  | 503  | Service Unavailable              |
  +------+----------------------------------+
```

#### HATEOAS (Hypermedia as the Engine of Application State)

Responses include links to related actions and resources, making the API self-discoverable.

```
RESPONSE for GET /users/42:
{
    "id": 42,
    "name": "Alice",
    "links": [
        { "rel": "self",   "href": "/users/42" },
        { "rel": "orders", "href": "/users/42/orders" },
        { "rel": "edit",   "href": "/users/42", "method": "PATCH" },
        { "rel": "delete", "href": "/users/42", "method": "DELETE" }
    ]
}
```

### RPC Patterns

RPC-style APIs expose operations (procedures) rather than resources.

```
  REST:  GET /users/42/orders?status=active
  RPC:   POST /getActiveOrders  { "user_id": 42 }

  +-----------------------+--------------------------------+
  | REST (Resource-based) | RPC (Action-based)             |
  +-----------------------+--------------------------------+
  | GET /users/42         | getUser(42)                    |
  | POST /users           | createUser(data)               |
  | DELETE /users/42      | deleteUser(42)                 |
  | PATCH /orders/7       | updateOrder(7, changes)        |
  +-----------------------+--------------------------------+
```

### GraphQL Concepts

GraphQL lets clients request exactly the data they need in a single query.

```
  REST: Multiple round trips          GraphQL: Single request
  GET /users/42         --|           QUERY {
  GET /users/42/orders  --|--> 3        user(id: 42) {
  GET /users/42/profile --|             name
                                        orders { id, total }
                                        profile { avatar }
                                      }
                                     }  --> 1 round trip
```

### API Versioning Strategies

```
  Strategy 1: URL Path Versioning
  /v1/users/42
  /v2/users/42

  Strategy 2: Header Versioning
  GET /users/42
  Accept: application/vnd.myapi.v2+json

  Strategy 3: Query Parameter
  GET /users/42?version=2

  +------------------+-------------+-------------+
  | Strategy         | Discoverability | Caching  |
  +------------------+-------------+-------------+
  | URL Path         | High           | Easy      |
  | Header           | Low            | Hard      |
  | Query Parameter  | Medium         | Medium    |
  +------------------+-------------+-------------+
```

### Idempotency

An idempotent operation produces the same result regardless of how many times it is called.

```
  IDEMPOTENT:
    GET  /users/42      --> Always returns user 42
    PUT  /users/42      --> Always sets user 42 to exact state
    DELETE /users/42    --> User 42 is deleted (subsequent calls: 404)

  NOT IDEMPOTENT:
    POST /users         --> Creates new user each time (different IDs)
```

```
FUNCTION idempotent_payment(payment_id: String, amount: Number):
    SET existing = LOOKUP payment_id IN payments
    IF existing IS NOT NULL THEN
        RETURN existing.result       // Already processed -- return same result
    END IF

    SET result = process_payment(amount)
    STORE payment_id -> result IN payments
    RETURN result
END FUNCTION
```

### Pagination

#### Offset-Based Pagination

```
  GET /users?offset=20&limit=10

  +----+----+----+----+----+----+----+----+----+----+
  | 1  | 2  |... | 20 | 21 | 22 | 23 | 24 | 25 | 26 |...
  +----+----+----+----+----+----+----+----+----+----+
                  ^--- offset=20, limit=10 returns 21-30

  Problem: Inserting/deleting rows shifts offsets (missed or duplicate items)
```

#### Cursor-Based Pagination

```
  GET /users?cursor=abc123&limit=10

  Client receives:
  {
      "data": [...10 users...],
      "next_cursor": "def456",
      "has_more": true
  }

  Next request: GET /users?cursor=def456&limit=10
```

```
FUNCTION paginate_with_cursor(cursor: String, limit: Integer) -> Page:
    IF cursor IS NOT NULL THEN
        SET start_id = DECODE(cursor)
        SET results = QUERY items WHERE id > start_id ORDER BY id LIMIT limit + 1
    ELSE
        SET results = QUERY items ORDER BY id LIMIT limit + 1
    END IF

    SET has_more = LENGTH(results) > limit
    IF has_more THEN
        REMOVE LAST FROM results
    END IF

    SET next_cursor = ENCODE(LAST(results).id) IF has_more ELSE NULL
    RETURN CREATE Page(data: results, next_cursor: next_cursor, has_more: has_more)
END FUNCTION
```

### Rate Limiting (Token Bucket Algorithm)

```
  Token Bucket:
  +------------------+
  |  O O O O O O     |  <- Tokens (capacity = 10)
  |  O O O O         |
  +------------------+
    ^                ^
    |                |
  Tokens added     Tokens consumed
  at fixed rate    per request

  If bucket empty --> request rejected (429 Too Many Requests)
```

```
FUNCTION create_rate_limiter(capacity: Integer, refill_rate: Number) -> Limiter:
    SET limiter.tokens = capacity
    SET limiter.capacity = capacity
    SET limiter.refill_rate = refill_rate           // Tokens per second
    SET limiter.last_refill = CURRENT_TIME()
    RETURN limiter
END FUNCTION

FUNCTION allow_request(limiter: Limiter) -> Boolean:
    // Refill tokens based on elapsed time
    SET now = CURRENT_TIME()
    SET elapsed = now - limiter.last_refill
    SET new_tokens = elapsed * limiter.refill_rate
    SET limiter.tokens = MIN(limiter.capacity, limiter.tokens + new_tokens)
    SET limiter.last_refill = now

    IF limiter.tokens >= 1 THEN
        SET limiter.tokens = limiter.tokens - 1
        RETURN true
    ELSE
        RETURN false
    END IF
END FUNCTION
```

### Authentication Patterns

#### API Keys

```
  Client --> GET /data
             Header: X-API-Key: sk_live_abc123def456
                                   |
             Server validates key against database
                                   |
             200 OK (valid) or 401 Unauthorized (invalid)
```

#### OAuth 2.0 Flow

```
  +--------+                               +---------------+
  | Client | --(1) Authorization Request-->| Authorization |
  |        |                               |    Server     |
  |        | <-(2) Authorization Code------+               |
  |        |                               +---------------+
  |        | --(3) Exchange Code + Secret->|               |
  |        | <-(4) Access Token -----------+               |
  +--------+                               +---------------+
       |
       | --(5) API Request + Access Token-->+----------+
       |                                    | Resource |
       | <-(6) Protected Resource ----------| Server   |
       |                                    +----------+
```

```
FUNCTION handle_oauth_callback(auth_code: String) -> Token:
    SET response = HTTP_POST(token_endpoint, {
        grant_type: "authorization_code",
        code: auth_code,
        client_id: CLIENT_ID,
        client_secret: CLIENT_SECRET,
        redirect_uri: REDIRECT_URI
    })

    IF response.status == 200 THEN
        RETURN response.body.access_token
    ELSE
        RAISE AuthenticationError(response.body.error)
    END IF
END FUNCTION
```

### API Documentation

Good API documentation includes:

```
  For each endpoint:
  +--------------------------------------------------+
  | METHOD  URL           Description                 |
  | POST    /users        Create a new user           |
  |                                                    |
  | Request Body:                                      |
  |   name: String (required) -- User's display name  |
  |   email: String (required) -- Must be unique      |
  |                                                    |
  | Response (201 Created):                            |
  |   id: Integer -- Unique user identifier            |
  |   name: String                                     |
  |   created_at: Timestamp                            |
  |                                                    |
  | Errors:                                            |
  |   400 -- Missing required fields                   |
  |   409 -- Email already exists                      |
  +--------------------------------------------------+
```

### Backward Compatibility

Rules for evolving an API without breaking existing clients:

- Adding new fields to responses is safe
- Adding new optional request parameters is safe
- Removing or renaming fields is a breaking change
- Changing field types is a breaking change
- Adding required request parameters is a breaking change
- New endpoints are always safe

## Common Pitfalls and Best Practices

| Pitfall                           | Best Practice                                  |
|-----------------------------------|------------------------------------------------|
| Inconsistent naming conventions   | Use one style (snake_case or camelCase) always |
| Exposing internal implementation  | Abstract internal models from API responses     |
| No error detail in responses      | Return structured errors with codes and messages|
| Missing pagination                | Always paginate collection endpoints             |
| No rate limiting                  | Protect all endpoints with rate limits           |
| Breaking changes without notice   | Version the API; maintain deprecation period     |
| Ignoring idempotency              | Make PUT and DELETE idempotent by design         |

## Real-World Applications

- **Public APIs**: Payment processors, social media platforms, cloud services
- **Microservices**: Internal service-to-service communication
- **Mobile backends**: APIs optimized for bandwidth-constrained clients
- **Partner integrations**: B2B data exchange with strict contracts
- **Developer platforms**: APIs as products with SDKs and documentation

## Related Topics

- [Security Fundamentals](security_fundamentals.md) -- Authentication and authorization
- [Networking Fundamentals](networking_fundamentals.md) -- HTTP protocol details
- [System Design Basics](system_design_basics.md) -- API gateway patterns

## Practice Problems

1. Design a REST API for a library system with books, authors, and borrowers. Define all
   endpoints, methods, and status codes.

2. Implement cursor-based pagination for a collection of 10,000 items with a default page
   size of 25.

3. Write pseudocode for a token bucket rate limiter that supports different rate limits per
   API key.

4. Design an idempotent payment endpoint that handles duplicate submissions gracefully.

5. Compare REST, RPC, and GraphQL for a social media application where clients need flexible
   queries. Which would you choose and why?

6. Create a versioning strategy for an API that must support three concurrent versions while
   deprecating the oldest.
