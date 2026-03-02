# Client-Server Architecture

## Overview and Intent

The Client-Server pattern divides a system into two distinct roles: clients that
initiate requests and servers that process those requests and return responses.
This fundamental division of labor underpins the vast majority of networked
applications, from web browsing to database access to email.

The pattern's intent is to centralize shared resources, business logic, or data
on a server while distributing the user interface and interaction logic to
clients. This centralization simplifies data management, enforces consistent
business rules, and enables multiple clients to share a single authoritative
data source.

Client-server systems exist on a spectrum from "thin clients" (where the server
does most of the work and the client is a simple display terminal) to "thick
clients" (where significant processing occurs on the client side, and the server
primarily manages data and coordination). The choice between thin and thick
determines bandwidth usage, offline capability, server load, and user experience.

## Structure and Components

### Component Diagram

```
+-----------+    +-----------+    +-----------+
|  Client   |    |  Client   |    |  Client   |
|  (Thin)   |    |  (Thick)  |    |  (Mobile) |
+-----+-----+    +-----+-----+    +-----+-----+
      |                |                |
      |    Request     |    Request     |    Request
      +-------+--------+-------+-------+--------+
              |                |                 |
              v                v                 v
      +-----------------------------------------------+
      |              NETWORK (Protocol)                |
      +----------------------+------------------------+
                             |
                             v
      +-----------------------------------------------+
      |                   SERVER                       |
      |  +------------+  +----------+  +----------+   |
      |  | Request    |  | Business |  | Data     |   |
      |  | Handler    |  | Logic    |  | Store    |   |
      |  +------------+  +----------+  +----------+   |
      +-----------------------------------------------+
```

### Key Components

1. **Client**: Initiates communication. Handles user interaction, input
   validation, and display. May cache data locally for performance.

2. **Server**: Listens for and processes requests. Manages shared state,
   enforces business rules, and controls access to resources.

3. **Communication Protocol**: Defines the message format, transport
   mechanism, and interaction semantics (synchronous or asynchronous).

4. **Request Handler**: Server-side component that parses incoming requests,
   routes them to the appropriate logic, and formats responses.

5. **Session Manager**: Tracks client state across multiple requests when
   the protocol itself is stateless.

## How It Works

```
  Client                           Server
    |                                |
    |  1. Establish connection       |
    |------------------------------->|
    |                                |
    |  2. Send request               |
    |------------------------------->|
    |                                | 3. Parse request
    |                                | 4. Authenticate/authorize
    |                                | 5. Execute business logic
    |                                | 6. Access data store
    |                                | 7. Format response
    |  8. Receive response           |
    |<-------------------------------|
    |                                |
    |  9. Render/display result      |
    |                                |
    | 10. Send next request          |
    |------------------------------->|
    |           ...                  |
```

Step-by-step:

1. The client establishes a connection to the server using a known address
   and protocol.
2. The client serializes a request and sends it over the network.
3. The server's request handler deserializes and validates the request.
4. Authentication and authorization checks confirm the client's identity
   and permissions.
5. The server executes business logic associated with the request.
6. The server reads from or writes to its data store as needed.
7. The server serializes the result into a response and sends it back.
8. The client receives and deserializes the response.
9. The client renders the result for the user.
10. The cycle repeats for subsequent interactions.

## Pseudocode Example

```
// --- Server Side ---

FUNCTION StartServer(port: Integer):
    SET listener = NetworkListener.Bind(port)
    LOG "Server listening on port " + port

    LOOP FOREVER DO
        SET connection = listener.AcceptConnection()
        SpawnHandler(connection)
    END LOOP
END FUNCTION

FUNCTION SpawnHandler(connection: Connection):
    // Handle in a separate execution context for concurrency
    ASYNC DO
        SET request = ReadRequest(connection)
        SET response = ProcessRequest(request)
        WriteResponse(connection, response)
        connection.Close()
    END ASYNC
END FUNCTION

FUNCTION ProcessRequest(request: Request) -> Response:
    // Authentication
    SET session = SessionManager.Validate(request.authToken)
    IF session IS NULL THEN
        RETURN Response.Unauthorized("Invalid or expired token")
    END IF

    // Routing
    IF request.method = "GET" AND request.path = "/users" THEN
        RETURN HandleGetUsers(session)
    ELSE IF request.method = "POST" AND request.path = "/users" THEN
        RETURN HandleCreateUser(session, request.body)
    ELSE IF request.method = "GET" AND StartsWith(request.path, "/users/") THEN
        SET userId = ExtractId(request.path)
        RETURN HandleGetUser(session, userId)
    ELSE
        RETURN Response.NotFound("Unknown endpoint")
    END IF
END FUNCTION

FUNCTION HandleGetUsers(session: Session) -> Response:
    IF NOT session.HasPermission("users.read") THEN
        RETURN Response.Forbidden("Insufficient permissions")
    END IF

    SET users = UserStore.FindAll(limit: 100, offset: 0)
    SET viewList = NEW List
    FOR EACH user IN users DO
        Append(viewList, {
            id: user.id,
            name: user.name,
            email: user.email
        })
    END FOR
    RETURN Response.OK(viewList)
END FUNCTION

FUNCTION HandleCreateUser(session: Session, body: Map) -> Response:
    IF NOT session.HasPermission("users.create") THEN
        RETURN Response.Forbidden("Insufficient permissions")
    END IF

    // Validate input
    SET errors = ValidateUserInput(body)
    IF Length(errors) > 0 THEN
        RETURN Response.BadRequest(errors)
    END IF

    SET user = NEW User
    SET user.name = body["name"]
    SET user.email = body["email"]
    SET user.createdBy = session.userId

    SET saved = UserStore.Save(user)
    RETURN Response.Created(saved)
END FUNCTION

// --- Client Side ---

FUNCTION FetchUsers(serverUrl: String, authToken: String) -> List:
    SET request = NEW Request
    SET request.method = "GET"
    SET request.url = serverUrl + "/users"
    SET request.authToken = authToken

    SET response = NetworkClient.Send(request, timeout: 30000)

    IF response.status = 200 THEN
        RETURN ParseList(response.body)
    ELSE IF response.status = 401 THEN
        RAISE AuthenticationError("Session expired")
    ELSE
        RAISE ServerError("Server returned " + response.status)
    END IF
END FUNCTION

FUNCTION DisplayUserList(users: List):
    ClearScreen()
    PrintHeader("User Directory")
    FOR EACH user IN users DO
        Print(FormatRow(user.id, user.name, user.email))
    END FOR
    PrintFooter("Total: " + Length(users) + " users")
END FUNCTION
```

## When to Use

- **Centralized data management** where multiple clients need a single source
  of truth for shared data.
- **Multi-platform access** where different client types (desktop, mobile,
  terminal) need the same backend services.
- **Resource-intensive processing** that benefits from powerful server hardware
  rather than running on constrained client devices.
- **Security-sensitive systems** where business rules and data access must be
  enforced on a trusted server, not on client devices that users control.
- **Systems requiring centralized administration** such as user management,
  configuration, and monitoring.

## When NOT to Use

- **Offline-first applications** where clients must function without network
  connectivity. Consider peer-to-peer or local-first architectures instead.
- **Extremely latency-sensitive interactions** where network round-trips are
  unacceptable (e.g., real-time gaming physics).
- **Highly distributed systems** with no natural centralization point.
  Consider peer-to-peer or event-driven patterns instead.
- **Simple standalone tools** where adding a server would be unnecessary
  complexity.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Web applications | Online banking portal | Centralized security, shared account data |
| Email | Mail client and mail server | Centralized mail storage, multi-device access |
| Databases | Application querying a data store | Centralized data, concurrent access control |
| File sharing | Network file server | Centralized storage, access control |
| Gaming | Multiplayer game lobby server | Authoritative state, anti-cheat enforcement |

## Trade-offs

### Advantages

- **Centralized control**: Single point for business rules, security, updates.
- **Data consistency**: One authoritative data source prevents conflicts.
- **Resource sharing**: Expensive resources (databases, GPUs) are centralized.
- **Simplified clients**: Thin clients require minimal local resources.
- **Scalability of server**: Server can be upgraded or replicated independently.

### Disadvantages

- **Single point of failure**: Server outage affects all clients.
- **Network dependency**: Clients cannot function without connectivity.
- **Bottleneck risk**: All traffic funnels through the server.
- **Latency**: Every operation requires a network round-trip.
- **Server cost**: High-performance servers are expensive to provision.

## Comparison with Related Patterns

| Dimension | Client-Server | Peer-to-Peer | Broker |
|-----------|--------------|--------------|--------|
| Topology | Centralized | Decentralized | Centralized routing, distributed processing |
| Single point of failure | Server | None (distributed) | Broker (mitigated by clustering) |
| Data consistency | Strong (single source) | Eventual | Varies by implementation |
| Scalability model | Scale server vertically/replicate | Scale by adding peers | Scale by adding services |
| Offline capability | Limited | Good | Limited |

## Evolution and Variations

- **Two-Tier**: Client communicates directly with a database server. Simple
  but mixes presentation and business logic on the client.
- **Three-Tier**: Introduces a middle tier (application server) between
  client and database. The most common modern variant.
- **N-Tier**: Additional tiers for caching, messaging, or specialized
  processing. Increases flexibility at the cost of complexity.
- **Thin Client**: Minimal client logic; the server renders full UI (e.g.,
  server-rendered web pages). Simplifies client but increases server load.
- **Thick/Rich Client**: Significant logic on the client side with the
  server providing data APIs. Reduces server load but complicates updates.
- **Hybrid**: Clients run rich local logic but synchronize with the server
  when connected. Supports offline operation.

## Key Takeaways

1. Client-server is the foundational pattern for networked applications.
   Most other distributed patterns are specializations of it.
2. The thin-vs-thick client decision is a core architectural choice that
   affects bandwidth, latency, offline capability, and update strategy.
3. Server availability is critical. Plan for redundancy, load balancing,
   and graceful degradation from the start.
4. Stateless server design simplifies scaling. Push session state to clients
   or a shared session store rather than keeping it in server memory.
5. Client-server naturally evolves into more sophisticated patterns
   (microservices, event-driven) as systems grow in complexity.
