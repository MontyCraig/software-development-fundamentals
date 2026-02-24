# Networking Fundamentals

## Overview

Networking is the backbone of modern software. Nearly every application -- from web browsers
to mobile apps to distributed microservices -- communicates over a network. Understanding
how data travels from one machine to another, how protocols ensure reliable delivery, and
how applications interact with network services is essential for building correct, performant,
and secure systems.

The networking stack is organized in layers, each providing specific services to the layer
above it. The OSI (Open Systems Interconnection) model defines seven layers, while the
practical internet uses a simplified four-layer model. Developers most frequently interact
with the application layer (HTTP, DNS, WebSocket) and the transport layer (TCP, UDP), but
understanding the full stack helps diagnose issues and make informed architectural decisions.

This chapter covers the OSI model, transport protocols, HTTP, DNS, socket programming,
TLS/SSL, WebSockets, and a comparison of modern API transport mechanisms.

## Key Concepts

### OSI Model (7 Layers)

```
  +-----+------------------------------+---------------------------+
  | #   | Layer          | Function                                |
  +-----+------------------------------+---------------------------+
  |     |                |                                         |
  |  7  | APPLICATION    | User-facing protocols (HTTP, FTP, SMTP) |
  |     |                |                                         |
  +-----+----------------+-----------------------------------------+
  |     |                |                                         |
  |  6  | PRESENTATION   | Data format, encryption, compression    |
  |     |                |                                         |
  +-----+----------------+-----------------------------------------+
  |     |                |                                         |
  |  5  | SESSION        | Connection management, sessions         |
  |     |                |                                         |
  +-----+----------------+-----------------------------------------+
  |     |                |                                         |
  |  4  | TRANSPORT      | Reliable delivery (TCP) or fast (UDP)   |
  |     |                |                                         |
  +-----+----------------+-----------------------------------------+
  |     |                |                                         |
  |  3  | NETWORK        | Routing and addressing (IP)             |
  |     |                |                                         |
  +-----+----------------+-----------------------------------------+
  |     |                |                                         |
  |  2  | DATA LINK      | Frame delivery on local network (MAC)   |
  |     |                |                                         |
  +-----+----------------+-----------------------------------------+
  |     |                |                                         |
  |  1  | PHYSICAL       | Bits on the wire (electrical, optical)  |
  |     |                |                                         |
  +-----+----------------+-----------------------------------------+

  Data encapsulation (sending):
  Application data
    --> Segment (Transport header + data)
      --> Packet (Network header + segment)
        --> Frame (Data Link header + packet + trailer)
          --> Bits (Physical signal)
```

```
  Sending side:                    Receiving side:
  +-------------+                  +-------------+
  | Application |  <-- data -->    | Application |
  +------+------+                  +------+------+
         |                                |
  +------+------+                  +------+------+
  | Transport   |  <-- segment --> | Transport   |
  +------+------+                  +------+------+
         |                                |
  +------+------+                  +------+------+
  | Network     |  <-- packet -->  | Network     |
  +------+------+                  +------+------+
         |                                |
  +------+------+                  +------+------+
  | Data Link   |  <-- frame -->   | Data Link   |
  +------+------+                  +------+------+
         |                                |
  +------+------+                  +------+------+
  | Physical    |  <-- bits -->    | Physical    |
  +-------------+                  +-------------+
```

### TCP vs UDP

| Aspect           | TCP                          | UDP                          |
|------------------|------------------------------|------------------------------|
| Connection       | Connection-oriented          | Connectionless               |
| Reliability      | Guaranteed delivery, ordered | Best-effort, no guarantees   |
| Flow control     | Yes (sliding window)         | No                           |
| Overhead         | Higher (headers, handshake)  | Lower (minimal headers)      |
| Speed            | Slower (reliability checks)  | Faster (fire and forget)     |
| Use cases        | Web, email, file transfer    | Video streaming, gaming, DNS |

### TCP Three-Way Handshake

```
  Client                          Server
    |                                |
    |  ---- SYN (seq=100) ------->  |   Step 1: Client initiates
    |                                |
    |  <-- SYN-ACK (seq=300,  ----  |   Step 2: Server acknowledges
    |       ack=101)                 |            and sends own SYN
    |                                |
    |  ---- ACK (seq=101,  ------> |   Step 3: Client acknowledges
    |       ack=301)                 |
    |                                |
    |  === CONNECTION ESTABLISHED == |
    |                                |
    |  ---- Data transfer -------->  |
    |  <--- Data transfer ---------  |
    |                                |
    |  ---- FIN ------------------>  |   Connection teardown
    |  <--- ACK -------------------  |
    |  <--- FIN -------------------  |
    |  ---- ACK ------------------>  |
    |                                |
```

### HTTP Request/Response Cycle

```
  Client (Browser)                    Server
    |                                    |
    | -- HTTP Request ----------------> |
    |    METHOD /path HTTP/1.1          |
    |    Host: example.com              |
    |    Accept: text/html              |
    |    Authorization: Bearer token    |
    |    [blank line]                   |
    |    [optional body]               |
    |                                    |
    |                                    | (process request)
    |                                    |
    | <-- HTTP Response ---------------- |
    |    HTTP/1.1 200 OK                |
    |    Content-Type: text/html        |
    |    Content-Length: 1234           |
    |    Cache-Control: max-age=3600   |
    |    [blank line]                   |
    |    [response body]               |
    |                                    |
```

### HTTP Methods and Status Codes

```
  METHODS:
  +--------+-------------+------------+-------------+
  | Method | Purpose     | Idempotent | Has Body    |
  +--------+-------------+------------+-------------+
  | GET    | Read        | Yes        | No          |
  | POST   | Create      | No         | Yes         |
  | PUT    | Replace     | Yes        | Yes         |
  | PATCH  | Partial upd | No*        | Yes         |
  | DELETE | Remove      | Yes        | Optional    |
  | HEAD   | Headers only| Yes        | No          |
  | OPTIONS| Capabilities| Yes        | No          |
  +--------+-------------+------------+-------------+
  (* PATCH can be made idempotent by design)

  STATUS CODE RANGES:
  +-------+-------------------+---------------------------+
  | Range | Category          | Examples                  |
  +-------+-------------------+---------------------------+
  | 1xx   | Informational     | 100 Continue              |
  | 2xx   | Success           | 200 OK, 201 Created       |
  | 3xx   | Redirection       | 301 Moved, 304 Not Mod.  |
  | 4xx   | Client Error      | 400 Bad Req, 404 Not Found|
  | 5xx   | Server Error      | 500 Internal, 503 Unavail.|
  +-------+-------------------+---------------------------+
```

### DNS Resolution

```
  User types "www.example.com" in browser

  +--------+                  +-----------+
  | Client | -- 1. Query ---> | Recursive |
  +--------+                  | Resolver  |
       ^                      +-----------+
       |                           |
       |                    2. Query root server
       |                           |
       |                    +-------------+
       |                    | Root Server |
       |                    | "Go ask .com|
       |                    |  nameserver"|
       |                    +-------------+
       |                           |
       |                    3. Query .com TLD server
       |                           |
       |                    +-----------+
       |                    | .com TLD  |
       |                    | "Go ask   |
       |                    | ns1.      |
       |                    | example.  |
       |                    | com"      |
       |                    +-----------+
       |                           |
       |                    4. Query authoritative server
       |                           |
       |                    +----------------+
       |                    | ns1.example.com|
       |                    | "IP: 93.184.   |
       |                    |  216.34"       |
       |                    +----------------+
       |                           |
       +--- 5. Response: ----------+
            93.184.216.34
            (cached for TTL)
```

### Socket Programming

Sockets provide the low-level interface for network communication.

```
FUNCTION run_server(port: Integer):
    SET server_socket = CREATE Socket(type: TCP)
    BIND server_socket TO address("0.0.0.0", port)
    LISTEN server_socket WITH backlog = 128

    LOOP
        SET client_socket, client_address = ACCEPT(server_socket)
        SPAWN THREAD handle_client(client_socket)
    END LOOP
END FUNCTION

FUNCTION handle_client(socket: Socket):
    SET request = READ FROM socket
    SET response = process_request(request)
    WRITE response TO socket
    CLOSE socket
END FUNCTION

FUNCTION run_client(host: String, port: Integer, message: String):
    SET socket = CREATE Socket(type: TCP)
    CONNECT socket TO address(host, port)
    WRITE message TO socket
    SET response = READ FROM socket
    CLOSE socket
    RETURN response
END FUNCTION
```

```
  Server                              Client
  +---------------------+            +---------------------+
  | CREATE socket       |            | CREATE socket       |
  | BIND to port        |            |                     |
  | LISTEN              |            |                     |
  |        <------- CONNECT ---------|                     |
  | ACCEPT              |            |                     |
  |        <------- SEND ----------- | SEND request        |
  | RECEIVE request     |            |                     |
  | Process             |            |                     |
  | SEND response ------|--------->  | RECEIVE response    |
  | CLOSE               |            | CLOSE               |
  +---------------------+            +---------------------+
```

### TLS/SSL Handshake

TLS (Transport Layer Security) encrypts communication between client and server.

```
  Client                                Server
    |                                      |
    | -- ClientHello ------------------>   |  (supported ciphers,
    |    (TLS version, random bytes,       |   random bytes)
    |     cipher suites)                   |
    |                                      |
    | <-- ServerHello -------------------  |  (chosen cipher,
    |    (TLS version, random bytes,       |   random bytes)
    |     chosen cipher suite)             |
    |                                      |
    | <-- Certificate ------------------   |  (server's public key
    |                                      |   in X.509 certificate)
    |                                      |
    | <-- ServerHelloDone --------------   |
    |                                      |
    | -- ClientKeyExchange ------------>   |  (pre-master secret
    |    (encrypted with server's          |   encrypted with
    |     public key)                      |   server's public key)
    |                                      |
    |  [Both derive session keys from      |
    |   pre-master secret + randoms]       |
    |                                      |
    | -- ChangeCipherSpec ------------->   |
    | -- Finished (encrypted) -------->    |
    |                                      |
    | <-- ChangeCipherSpec ---------------  |
    | <-- Finished (encrypted) ----------  |
    |                                      |
    | == ENCRYPTED DATA EXCHANGE ========  |
```

### WebSocket Concept

WebSocket provides full-duplex, persistent communication over a single TCP connection.

```
  HTTP (Request-Response):            WebSocket (Full-Duplex):
  Client     Server                   Client     Server
    | --Req-->  |                       | --Upgrade-> |
    | <--Res--  |                       | <--101---   |
    | --Req-->  |                       |  <======>   |  Bidirectional
    | <--Res--  |                       |  <======>   |  persistent
    | --Req-->  |                       |  <======>   |  connection
    | <--Res--  |                       |  <======>   |
```

```
  WebSocket Handshake (HTTP Upgrade):

  Client --> GET /chat HTTP/1.1
             Host: server.example.com
             Upgrade: websocket
             Connection: Upgrade
             Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==

  Server <-- HTTP/1.1 101 Switching Protocols
             Upgrade: websocket
             Connection: Upgrade
             Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

  [Connection upgraded -- now bidirectional frames]
```

```
FUNCTION websocket_server(port: Integer):
    SET server = CREATE WebSocketServer(port)

    ON server.connection(client):
        ON client.message(data):
            // Echo to all connected clients (chat example)
            FOR EACH connected_client IN server.clients DO
                SEND data TO connected_client
            END FOR
        END ON

        ON client.close():
            REMOVE client FROM server.clients
        END ON
    END ON
END FUNCTION
```

### REST vs gRPC vs GraphQL

```
  +------------+------------------+-------------------+------------------+
  | Aspect     | REST             | gRPC              | GraphQL          |
  +------------+------------------+-------------------+------------------+
  | Protocol   | HTTP/1.1 or 2    | HTTP/2             | HTTP/1.1 or 2   |
  | Format     | JSON (text)      | Protocol Buffers   | JSON (text)     |
  |            |                  | (binary)           |                 |
  | Schema     | Optional         | Required (.proto)  | Required (SDL)  |
  | Streaming  | Limited          | Bidirectional      | Subscriptions   |
  | Caching    | HTTP caching     | Complex            | Complex         |
  | Discovery  | HATEOAS          | Reflection         | Introspection   |
  | Best for   | Public APIs,     | Microservices,     | Flexible client |
  |            | CRUD operations  | low-latency, inter-| queries, mobile |
  |            |                  | service calls      | backends        |
  +------------+------------------+-------------------+------------------+
```

```
  Data Fetching Comparison:

  REST (over-fetching):
  GET /users/1       -> { id, name, email, address, phone, ... }
  GET /users/1/posts -> { [all post fields for all posts] }
  (2 requests, lots of unused data)

  GraphQL (exact data):
  QUERY { user(id: 1) { name, posts { title } } }
  (1 request, only name and post titles)

  gRPC (binary, typed):
  GetUser(UserRequest { id: 1 }) -> UserResponse { name, email }
  (1 request, binary format, very fast)
```

## Common Pitfalls and Best Practices

| Pitfall                           | Best Practice                                   |
|-----------------------------------|--------------------------------------------------|
| Ignoring connection timeouts      | Always set connect, read, and write timeouts     |
| No retry logic                    | Implement retries with exponential backoff        |
| DNS caching too long              | Respect TTL; handle DNS changes gracefully        |
| Plaintext communication           | Use TLS for all production traffic                |
| Blocking on network I/O           | Use async I/O or thread pools for concurrency     |
| Large payload transfers           | Compress responses; use pagination                |
| Ignoring HTTP caching headers     | Set Cache-Control, ETag, and Last-Modified         |

## Real-World Applications

- **Web applications**: HTTP, TLS, WebSocket for real-time features
- **Microservices**: gRPC for internal communication, REST for public APIs
- **Content delivery**: DNS for global routing, CDN edge servers
- **IoT devices**: UDP for sensor data, MQTT over TCP for commands
- **Gaming**: UDP for real-time state, TCP for chat and authentication

## Related Topics

- [System Design Basics](system_design_basics.md) -- Load balancing, CDN, distributed systems
- [Security Fundamentals](security_fundamentals.md) -- TLS, HTTPS, CORS
- [API Design Principles](api_design_principles.md) -- HTTP methods, REST, GraphQL

## Practice Problems

1. Trace the full journey of an HTTP request from a browser to a server, identifying which
   OSI layer each step operates at.

2. Write pseudocode for a simple TCP echo server that handles multiple clients concurrently.

3. Explain why UDP is preferred over TCP for real-time video streaming, and describe how
   the application layer compensates for lost packets.

4. Walk through a complete TLS handshake. At what point does the connection become encrypted?
   What happens if the certificate is invalid?

5. Design a chat application using WebSockets. Include connection management, message
   broadcasting, and handling client disconnects.

6. Compare REST, gRPC, and GraphQL for an e-commerce platform with a web frontend, mobile
   app, and backend microservices. Which would you use for each interaction and why?
