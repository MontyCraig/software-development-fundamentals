# Database Fundamentals

## Overview

Databases are the foundation of nearly every software system. They provide organized,
persistent storage for data that must survive process restarts, be shared across users, and
be queried efficiently. Choosing the right database model, designing an effective schema,
understanding transactions, and optimizing queries are skills every developer needs.

The two broad families of databases are relational (structured tables with strict schemas,
queried with a declarative language) and non-relational/NoSQL (various models optimized for
specific access patterns). Neither is universally superior -- the choice depends on data
structure, query patterns, consistency requirements, and scale.

This chapter covers the relational model, query concepts (selection, projection, joins,
aggregation), NoSQL database types, ACID properties, indexing, normalization, transactions,
query optimization, and the ORM abstraction.

## Key Concepts

### Relational Model

Data is organized into tables (relations) with rows (tuples) and columns (attributes). Keys
establish relationships between tables.

```
  USERS Table                        ORDERS Table
  +----+--------+------------------+ +----------+--------+--------+-------+
  | id | name   | email            | | order_id | user_id| total  | date  |
  +----+--------+------------------+ +----------+--------+--------+-------+
  |  1 | Alice  | alice@mail.com   | |     101  |      1 | 59.99  | 01-15 |
  |  2 | Bob    | bob@mail.com     | |     102  |      2 | 124.50 | 01-16 |
  |  3 | Carol  | carol@mail.com   | |     103  |      1 | 35.00  | 01-17 |
  +----+--------+------------------+ |     104  |      3 | 89.99  | 01-18 |
                                     +----------+--------+--------+-------+
  Primary Key: id (USERS)                        Foreign Key: user_id -> USERS.id
  Primary Key: order_id (ORDERS)

  Relationship: One user has many orders (1:N)
```

```
  Relationship Types:
  +-----------+-------------------------------+
  | Type      | Description                   |
  +-----------+-------------------------------+
  | 1:1       | One user has one profile      |
  | 1:N       | One user has many orders      |
  | N:M       | Many students, many courses   |
  |           | (requires junction table)     |
  +-----------+-------------------------------+

  N:M with junction table:
  STUDENTS         ENROLLMENTS         COURSES
  +----+------+    +-----------+------+ +----+---------+
  | id | name |    |student_id |course| | id | title   |
  +----+------+    |           |_id   | +----+---------+
  |  1 | Alice|    |     1     |  10  | | 10 | Math    |
  |  2 | Bob  |    |     1     |  20  | | 20 | Science |
  +----+------+    |     2     |  10  | +----+---------+
                   +-----------+------+
```

### SQL Concepts

#### SELECT and Filtering

```
QUERY: Retrieve all orders over 50 for a specific user

  SELECT order_id, total, date
  FROM orders
  WHERE user_id = 1 AND total > 50.00
  ORDER BY date DESC

  Result:
  +----------+-------+-------+
  | order_id | total | date  |
  +----------+-------+-------+
  |     101  | 59.99 | 01-15 |
  +----------+-------+-------+
```

#### JOIN Types

```
  Table A          Table B            INNER JOIN (only matching rows)
  +----+-----+     +----+-----+      +----+-----+-----+
  | id | val |     | id | val |      | id | A   | B   |
  +----+-----+     +----+-----+      +----+-----+-----+
  |  1 | a   |     |  1 | x   |      |  1 | a   | x   |
  |  2 | b   |     |  3 | y   |      +----+-----+-----+
  |  3 | c   |     |  4 | z   |
  +----+-----+     +----+-----+

  Venn Diagram Representations:

  INNER JOIN:          LEFT JOIN:           RIGHT JOIN:         FULL OUTER JOIN:
  +-----+-----+       +-----+-----+        +-----+-----+      +-----+-----+
  |     |XXXXX|       |XXXXX|XXXXX|        |XXXXX|XXXXX|      |XXXXX|XXXXX|
  |  A  |  B  |       |  A  |  B  |        |  A  |  B  |      |  A  |  B  |
  |     |XXXXX|       |XXXXX|XXXXX|        |XXXXX|XXXXX|      |XXXXX|XXXXX|
  +-----+-----+       +-----+-----+        +-----+-----+      +-----+-----+
  (A intersect B)      (All A + match B)   (All B + match A)  (All A + All B)
```

```
  LEFT JOIN result:           RIGHT JOIN result:
  +----+-----+------+        +------+-----+----+
  | id | A   | B    |        | A    | B   | id |
  +----+-----+------+        +------+-----+----+
  |  1 | a   | x    |        | a    | x   |  1 |
  |  2 | b   | NULL |        | NULL | y   |  3 |
  |  3 | c   | y    |        | NULL | z   |  4 |
  +----+-----+------+        +------+-----+----+
```

#### GROUP BY and Aggregation

```
QUERY: Total spending per user

  SELECT user_id, COUNT(order_id) AS order_count, SUM(total) AS total_spent
  FROM orders
  GROUP BY user_id
  HAVING SUM(total) > 50

  Result:
  +---------+-------------+-------------+
  | user_id | order_count | total_spent |
  +---------+-------------+-------------+
  |       1 |           2 |       94.99 |
  |       2 |           1 |      124.50 |
  |       3 |           1 |       89.99 |
  +---------+-------------+-------------+
```

#### Subqueries

```
QUERY: Users who have placed orders above the average order total

  SELECT name FROM users
  WHERE id IN (
      SELECT user_id FROM orders
      WHERE total > (SELECT AVG(total) FROM orders)
  )
```

### NoSQL Database Types

```
  +-------------------+----------------------------------------------+
  | Type              | Structure and Use Case                       |
  +-------------------+----------------------------------------------+
  |                   |                                              |
  | DOCUMENT          |  { "id": 1,                                 |
  |                   |    "name": "Alice",                         |
  |                   |    "orders": [                               |
  |                   |      { "total": 59.99 },                    |
  |                   |      { "total": 35.00 }                     |
  |                   |    ]                                         |
  |                   |  }                                           |
  |                   |  Use: Content management, catalogs           |
  +-------------------+----------------------------------------------+
  |                   |                                              |
  | KEY-VALUE         |  "user:1" --> "{ serialized data }"         |
  |                   |  "session:abc" --> "{ session data }"       |
  |                   |  Use: Caching, sessions, simple lookups     |
  +-------------------+----------------------------------------------+
  |                   |                                              |
  | COLUMN-FAMILY     |  Row Key | cf:name | cf:email | cf:age     |
  |                   |  --------|---------|----------|--------     |
  |                   |  user1   | Alice   | a@m.com  | 30         |
  |                   |  user2   | Bob     |          | 25         |
  |                   |  Use: Analytics, time-series, wide-column   |
  +-------------------+----------------------------------------------+
  |                   |                                              |
  | GRAPH             |  (Alice)--[FRIENDS]->(Bob)                  |
  |                   |  (Alice)--[BOUGHT]->(Product1)              |
  |                   |  (Bob)--[REVIEWED]->(Product1)              |
  |                   |  Use: Social networks, recommendations      |
  +-------------------+----------------------------------------------+
```

### ACID Properties

```
  A - ATOMICITY:     All or nothing. Transaction fully completes or fully rolls back.
  C - CONSISTENCY:   Database moves from one valid state to another.
  I - ISOLATION:     Concurrent transactions do not interfere with each other.
  D - DURABILITY:    Committed data survives system failures.
```

```
FUNCTION transfer_funds(from_account: String, to_account: String, amount: Number):
    BEGIN TRANSACTION

    TRY
        SET from_balance = QUERY balance FROM accounts WHERE id = from_account
        IF from_balance < amount THEN
            ROLLBACK TRANSACTION
            RAISE InsufficientFundsError
        END IF

        UPDATE accounts SET balance = balance - amount WHERE id = from_account
        UPDATE accounts SET balance = balance + amount WHERE id = to_account

        COMMIT TRANSACTION         // Atomic: both updates or neither
    CATCH error
        ROLLBACK TRANSACTION       // Undo partial changes
        RAISE error
    END TRY
END FUNCTION
```

```
  Without ACID:                    With ACID:
  1. Deduct from A  ok            1. BEGIN TRANSACTION
  2. System crash!  fail          2. Deduct from A
  3. Credit to B never happens    3. Credit to B
  Result: Money vanished!         4. COMMIT (or ROLLBACK on failure)
                                  Result: Either both or neither
```

### Indexing

Indexes speed up queries by providing efficient lookup structures at the cost of extra
storage and slower writes.

#### B-Tree Index

```
  B-Tree Index on "name" column:

                        +-------+
                        | Maria |
                        +---+---+
                       /         \
               +-------+         +-------+
               | David |         | Sarah |
               +---+---+         +---+---+
              /    |    \        /    |    \
          +-----+ +-----+ +-----+ +-----+ +-----+
          |Alice| |David| |Maria| |Peter| |Sarah|
          |Bob  | |Eve  | |Nina | |Rob  | |Tom  |
          +-----+ +-----+ +-----+ +-----+ +-----+
              |       |       |       |       |
            [rows]  [rows]  [rows]  [rows]  [rows]

  Lookup: O(log n) -- follow tree from root to leaf
  Range query: O(log n + k) -- find start, scan leaves
```

#### Hash Index

```
  Hash Index on "id" column:

  hash(1) = 3  --> Bucket 3 --> { id: 1, name: "Alice" }
  hash(2) = 7  --> Bucket 7 --> { id: 2, name: "Bob" }
  hash(3) = 1  --> Bucket 1 --> { id: 3, name: "Carol" }

  +--------+---------------------------+
  | Bucket | Records                   |
  +--------+---------------------------+
  |   0    | (empty)                   |
  |   1    | { id: 3, "Carol" }        |
  |   2    | (empty)                   |
  |   3    | { id: 1, "Alice" }        |
  |   ...  | ...                       |
  |   7    | { id: 2, "Bob" }          |
  +--------+---------------------------+

  Exact lookup: O(1) average
  Range query:  NOT supported (hashes destroy ordering)
```

| Aspect        | B-Tree                    | Hash                      |
|---------------|---------------------------|---------------------------|
| Exact lookup  | O(log n)                  | O(1) average              |
| Range query   | Efficient                 | Not supported             |
| Ordering      | Maintained                | Not maintained            |
| Use case      | General purpose           | Exact-match only          |

### Normalization

Normalization reduces data redundancy and prevents update anomalies.

#### Unnormalized (problems)

```
  +----------+---------+------------+----------+---------+
  | order_id | product | cust_name  | cust_city| phone   |
  +----------+---------+------------+----------+---------+
  |     1    | Widget  | Alice      | Boston   | 555-0101|
  |     2    | Gadget  | Alice      | Boston   | 555-0101|
  |     3    | Widget  | Bob        | Denver   | 555-0202|
  +----------+---------+------------+----------+---------+

  Problems:
  - Update anomaly: Change Alice's city in one row but not the other
  - Insert anomaly: Cannot add a customer without an order
  - Delete anomaly: Deleting Bob's only order loses his information
```

#### First Normal Form (1NF): Atomic values, no repeating groups

```
  VIOLATES 1NF:                    SATISFIES 1NF:
  +----+-----------------+         +----+---------+
  | id | phones          |         | id | phone   |
  +----+-----------------+         +----+---------+
  |  1 | 555-01, 555-02  |         |  1 | 555-01  |
  +----+-----------------+         |  1 | 555-02  |
                                   +----+---------+
```

#### Second Normal Form (2NF): 1NF + no partial dependencies on composite key

```
  VIOLATES 2NF (composite key: student_id + course_id):
  +------------+-----------+---------------+
  | student_id | course_id | student_name  |  <-- Depends only on student_id
  +------------+-----------+---------------+

  SATISFIES 2NF (split into two tables):
  STUDENTS:                    ENROLLMENTS:
  +------------+-------------+ +------------+-----------+
  | student_id | student_name| | student_id | course_id |
  +------------+-------------+ +------------+-----------+
```

#### Third Normal Form (3NF): 2NF + no transitive dependencies

```
  VIOLATES 3NF:
  +----+--------+----------+-----------+
  | id | name   | zip_code | city      |  <-- city depends on zip_code,
  +----+--------+----------+-----------+      not directly on id

  SATISFIES 3NF:
  USERS:                      ZIP_CODES:
  +----+--------+----------+ +----------+-----------+
  | id | name   | zip_code | | zip_code | city      |
  +----+--------+----------+ +----------+-----------+
```

### Transactions and Isolation Levels

```
  +------------------+-------------+------------------+----------------+
  | Isolation Level  | Dirty Read  | Non-Repeatable   | Phantom Read   |
  |                  |             | Read             |                |
  +------------------+-------------+------------------+----------------+
  | Read Uncommitted | Possible    | Possible         | Possible       |
  | Read Committed   | Prevented   | Possible         | Possible       |
  | Repeatable Read  | Prevented   | Prevented        | Possible       |
  | Serializable     | Prevented   | Prevented        | Prevented      |
  +------------------+-------------+------------------+----------------+

  Dirty Read:          Reading data from an uncommitted transaction
  Non-Repeatable Read: Reading same row twice gets different values
  Phantom Read:        Re-executing a query returns new rows
```

```
FUNCTION demonstrate_isolation():
    // Transaction A                    // Transaction B
    BEGIN TRANSACTION A                 BEGIN TRANSACTION B

    QUERY balance FROM accounts         // (balance = 100)
    WHERE id = 1
    // reads 100

                                        UPDATE accounts
                                        SET balance = 200
                                        WHERE id = 1
                                        COMMIT TRANSACTION B

    QUERY balance FROM accounts
    WHERE id = 1
    // Repeatable Read:  reads 100 (snapshot)
    // Read Committed:   reads 200 (sees committed change)

    COMMIT TRANSACTION A
END FUNCTION
```

### Query Optimization

```
  Slow query:
  SELECT * FROM orders WHERE YEAR(date) = 2025

  Why slow: Function on column prevents index usage (full table scan)

  Optimized:
  SELECT * FROM orders WHERE date >= '2025-01-01' AND date < '2026-01-01'

  Why fast: Range query can use B-tree index on date column
```

Key optimization principles:
- Create indexes on columns used in WHERE, JOIN, and ORDER BY
- Avoid functions on indexed columns in WHERE clauses
- Use EXPLAIN to inspect query execution plans
- Avoid SELECT * -- request only needed columns
- Use JOIN instead of subqueries when possible
- Add composite indexes for multi-column queries

### ORM Concept

An Object-Relational Mapper maps between objects in application code and rows in database
tables, abstracting away direct query construction.

```
  Application Object              Database Table
  +------------------+            +----+--------+----------+
  | User             |    <--->   | id | name   | email    |
  |   id: Integer    |            +----+--------+----------+
  |   name: String   |            |  1 | Alice  | a@m.com  |
  |   email: String  |            |  2 | Bob    | b@m.com  |
  +------------------+            +----+--------+----------+

  ORM translates:
  user.find(1)                --> SELECT * FROM users WHERE id = 1
  user.name = "Alicia"        --> UPDATE users SET name = 'Alicia' WHERE id = 1
  user.save()                 --> INSERT INTO users (name, email) VALUES (...)
  user.delete()               --> DELETE FROM users WHERE id = 1
```

```
FUNCTION orm_example():
    // Instead of writing raw queries:
    SET results = EXECUTE("SELECT * FROM users WHERE age > 21")

    // ORM provides an object-oriented interface:
    SET results = User.where(age: GREATER_THAN(21)).all()

    // Relationships are navigated as object properties:
    SET user = User.find(1)
    SET orders = user.orders          // Lazy-loads related orders
    FOR EACH order IN orders DO
        PRINT order.total
    END FOR
END FUNCTION
```

## Common Pitfalls and Best Practices

| Pitfall                           | Best Practice                                   |
|-----------------------------------|--------------------------------------------------|
| No indexes on queried columns     | Index columns in WHERE, JOIN, ORDER BY            |
| Over-normalization                | Denormalize selectively for read-heavy workloads  |
| N+1 query problem with ORMs      | Use eager loading for known relationships          |
| Missing transactions for multi-   | Wrap related writes in transactions                |
| step operations                   |                                                    |
| Using wrong database type         | Match data model to access patterns                |
| Unbounded queries                 | Always paginate; add LIMIT to all queries          |
| Ignoring query execution plans    | Use EXPLAIN regularly to identify bottlenecks      |

## Real-World Applications

- **E-commerce**: Relational for orders and inventory, cache layer for product listings
- **Social media**: Graph database for connections, document store for posts
- **Analytics**: Column-family stores for aggregating large datasets
- **Session management**: Key-value stores for fast session lookups
- **Financial systems**: Relational with SERIALIZABLE isolation for transactions

## Related Topics

- [System Design Basics](system_design_basics.md) -- Replication, sharding, caching
- [API Design Principles](api_design_principles.md) -- Pagination, query patterns
- [Security Fundamentals](security_fundamentals.md) -- SQL injection prevention
- [Memory Management](memory_management.md) -- Buffer pool and memory-mapped I/O

## Practice Problems

1. Design a relational schema for a library system with books, authors (many-to-many),
   members, and loans. Normalize to 3NF.

2. Write queries to find: (a) all books by a specific author, (b) members with overdue
   loans, (c) the most borrowed book this month.

3. Explain the difference between a LEFT JOIN and an INNER JOIN using a concrete example
   with two tables. Show the result sets.

4. Given a slow query that scans 10 million rows, propose three indexing strategies and
   explain the trade-offs of each.

5. Design a NoSQL document structure for a blog platform with posts, comments, and tags.
   Compare it to the equivalent relational design.

6. Implement a bank transfer function that demonstrates all four ACID properties. Show what
   happens when a failure occurs mid-transaction.

7. Explain the N+1 query problem with an example and show how to solve it using eager
   loading.
