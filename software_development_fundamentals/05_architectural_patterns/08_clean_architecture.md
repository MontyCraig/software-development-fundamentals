# Clean Architecture

## Overview and Intent

Clean Architecture organizes a system into concentric circles of dependency,
with the most stable and abstract elements at the center and the most volatile
and concrete elements at the outer edges. The fundamental rule is that source
code dependencies may only point inward -- outer circles may depend on inner
circles, but inner circles must know nothing about outer circles.

The intent is to create systems where business rules are the central organizing
principle, completely independent of frameworks, databases, and delivery
mechanisms. This makes the system testable without infrastructure, portable
across platforms, and resilient to technology changes.

Clean Architecture synthesizes ideas from Hexagonal Architecture, Onion
Architecture, and other dependency-inversion approaches into a unified model
with four primary layers: Entities, Use Cases, Interface Adapters, and
Frameworks/Drivers. It was formalized by Robert C. Martin and has become one
of the most widely referenced architectural patterns for domain-centric design.

## Structure and Components

### Component Diagram

```
+================================================================+
|                                                                  |
|  FRAMEWORKS & DRIVERS (outermost)                               |
|  +-----------------------------------------------------------+  |
|  |                                                             | |
|  |  INTERFACE ADAPTERS                                         | |
|  |  +------------------------------------------------------+  | |
|  |  |                                                        | | |
|  |  |  USE CASES (Application Business Rules)               | | |
|  |  |  +--------------------------------------------------+ | | |
|  |  |  |                                                    | | | |
|  |  |  |  ENTITIES (Enterprise Business Rules)             | | | |
|  |  |  |                                                    | | | |
|  |  |  |  - Domain objects                                  | | | |
|  |  |  |  - Business rules                                  | | | |
|  |  |  |  - Value objects                                   | | | |
|  |  |  |                                                    | | | |
|  |  |  +--------------------------------------------------+ | | |
|  |  |                                                        | | |
|  |  |  - Application-specific logic                          | | |
|  |  |  - Orchestrates entity interactions                    | | |
|  |  |  - Defines input/output boundaries                     | | |
|  |  |                                                        | | |
|  |  +------------------------------------------------------+  | |
|  |                                                             | |
|  |  - Controllers, Presenters, Gateways                       | |
|  |  - Maps between use case and external formats               | |
|  |                                                             | |
|  +-----------------------------------------------------------+  |
|                                                                  |
|  - Web framework, Database driver, External APIs                |
|  - UI rendering, Message queue client                            |
|                                                                  |
+================================================================+

   Dependencies point INWARD only:
   Frameworks -> Adapters -> Use Cases -> Entities
```

### Key Components

1. **Entities**: Enterprise-wide business rules and objects. The most stable
   layer, changing only when fundamental business rules change.

2. **Use Cases**: Application-specific business rules. Each use case
   orchestrates entity interactions for a single application operation.

3. **Interface Adapters**: Translators between the use case layer and the
   outside world. Controllers convert input; presenters convert output;
   gateways abstract data access.

4. **Frameworks and Drivers**: The outermost layer containing concrete
   implementations of infrastructure: web frameworks, database drivers,
   UI toolkits, and external service clients.

## How It Works

```
  External Request (HTTP, CLI, Event)
       |
       v
  +----+--------+
  | Framework   |  1. Receive raw input from external source
  | (Web server)|
  +----+--------+
       |
       v
  +----+--------+
  | Controller  |  2. Parse input, create Request Model
  | (Adapter)   |  3. Call use case with Request Model
  +----+--------+
       |
       v
  +----+--------+
  | Use Case    |  4. Validate business rules
  | Interactor  |  5. Orchestrate entities
  +----+--------+  6. Call gateway interfaces for data
       |
       v
  +----+--------+
  | Entity      |  7. Apply core business rules
  |             |  8. Return results
  +----+--------+
       |
  (back up through layers)
       |
  +----+--------+
  | Presenter   |  9. Format output into ViewModel
  | (Adapter)   |
  +----+--------+
       |
       v
  +----+--------+
  | View        | 10. Render for the user
  | (Framework) |
  +----+--------+
```

## Pseudocode Example

```
// === ENTITIES (Innermost Circle) ===

STRUCTURE Account:
    id: String
    ownerId: String
    balance: Decimal
    currency: String
    status: AccountStatus
    createdAt: DateTime
END STRUCTURE

ENUMERATION AccountStatus:
    ACTIVE, FROZEN, CLOSED
END ENUMERATION

FUNCTION Account.Withdraw(amount: Decimal) -> WithdrawalResult:
    IF this.status != ACTIVE THEN
        RETURN WithdrawalResult.Failure("Account is not active")
    END IF
    IF amount <= 0 THEN
        RETURN WithdrawalResult.Failure("Amount must be positive")
    END IF
    IF amount > this.balance THEN
        RETURN WithdrawalResult.Failure("Insufficient funds")
    END IF
    SET this.balance = this.balance - amount
    RETURN WithdrawalResult.Success(this.balance)
END FUNCTION

FUNCTION Account.Deposit(amount: Decimal) -> DepositResult:
    IF this.status != ACTIVE THEN
        RETURN DepositResult.Failure("Account is not active")
    END IF
    IF amount <= 0 THEN
        RETURN DepositResult.Failure("Amount must be positive")
    END IF
    SET this.balance = this.balance + amount
    RETURN DepositResult.Success(this.balance)
END FUNCTION

// === USE CASES (Application Business Rules) ===

// Input boundary (interface the adapter calls)
INTERFACE TransferMoneyInputPort:
    FUNCTION Execute(request: TransferRequest) -> TransferResponse
END INTERFACE

// Output boundary (interface the use case calls to present results)
INTERFACE TransferMoneyOutputPort:
    FUNCTION PresentSuccess(response: TransferResponse)
    FUNCTION PresentFailure(error: String)
END INTERFACE

// Data access boundary (interface the use case calls for persistence)
INTERFACE AccountGateway:
    FUNCTION FindById(accountId: String) -> Account
    FUNCTION Save(account: Account)
END INTERFACE

STRUCTURE TransferRequest:
    sourceAccountId: String
    targetAccountId: String
    amount: Decimal
    description: String
END STRUCTURE

STRUCTURE TransferResponse:
    transferId: String
    sourceBalance: Decimal
    targetBalance: Decimal
    timestamp: DateTime
END STRUCTURE

// Use Case Interactor (implements the input port)
STRUCTURE TransferMoneyUseCase IMPLEMENTS TransferMoneyInputPort:
    accountGateway: AccountGateway
    outputPort: TransferMoneyOutputPort
END STRUCTURE

FUNCTION TransferMoneyUseCase.Execute(request: TransferRequest) -> TransferResponse:
    // Load entities through the gateway port
    SET sourceAccount = this.accountGateway.FindById(request.sourceAccountId)
    IF sourceAccount IS NULL THEN
        this.outputPort.PresentFailure("Source account not found")
        RETURN NULL
    END IF

    SET targetAccount = this.accountGateway.FindById(request.targetAccountId)
    IF targetAccount IS NULL THEN
        this.outputPort.PresentFailure("Target account not found")
        RETURN NULL
    END IF

    IF sourceAccount.currency != targetAccount.currency THEN
        this.outputPort.PresentFailure("Currency mismatch")
        RETURN NULL
    END IF

    // Business rule: use entity methods
    SET withdrawResult = sourceAccount.Withdraw(request.amount)
    IF NOT withdrawResult.isSuccess THEN
        this.outputPort.PresentFailure(withdrawResult.message)
        RETURN NULL
    END IF

    SET depositResult = targetAccount.Deposit(request.amount)
    IF NOT depositResult.isSuccess THEN
        // Compensate: put money back
        sourceAccount.Deposit(request.amount)
        this.outputPort.PresentFailure(depositResult.message)
        RETURN NULL
    END IF

    // Persist through the gateway port
    this.accountGateway.Save(sourceAccount)
    this.accountGateway.Save(targetAccount)

    // Build response
    SET response = NEW TransferResponse
    SET response.transferId = GenerateId()
    SET response.sourceBalance = sourceAccount.balance
    SET response.targetBalance = targetAccount.balance
    SET response.timestamp = Now()

    this.outputPort.PresentSuccess(response)
    RETURN response
END FUNCTION

// === INTERFACE ADAPTERS ===

// Controller (driving adapter -- converts external input to use case input)
STRUCTURE TransferController:
    useCase: TransferMoneyInputPort
END STRUCTURE

FUNCTION TransferController.HandleTransfer(httpRequest: HttpRequest) -> HttpResponse:
    SET body = ParseBody(httpRequest)

    SET request = NEW TransferRequest
    SET request.sourceAccountId = body["from"]
    SET request.targetAccountId = body["to"]
    SET request.amount = ParseDecimal(body["amount"])
    SET request.description = body["description"]

    SET response = this.useCase.Execute(request)
    IF response IS NULL THEN
        RETURN HttpResponse(400, {error: "Transfer failed"})
    END IF
    RETURN HttpResponse(200, Serialize(response))
END FUNCTION

// Presenter (formats use case output for display)
STRUCTURE TransferPresenter IMPLEMENTS TransferMoneyOutputPort:
    viewModel: TransferViewModel
END STRUCTURE

FUNCTION TransferPresenter.PresentSuccess(response: TransferResponse):
    SET this.viewModel = NEW TransferViewModel
    SET this.viewModel.message = "Transfer successful"
    SET this.viewModel.transferId = response.transferId
    SET this.viewModel.newBalance = FormatCurrency(response.sourceBalance)
    SET this.viewModel.timestamp = FormatDate(response.timestamp)
END FUNCTION

FUNCTION TransferPresenter.PresentFailure(error: String):
    SET this.viewModel = NEW TransferViewModel
    SET this.viewModel.message = "Transfer failed: " + error
    SET this.viewModel.isError = TRUE
END FUNCTION

// Gateway Implementation (driven adapter -- connects to database)
STRUCTURE SqlAccountGateway IMPLEMENTS AccountGateway:
    database: DatabaseConnection
END STRUCTURE

FUNCTION SqlAccountGateway.FindById(accountId: String) -> Account:
    SET row = this.database.QueryOne(
        "SELECT * FROM accounts WHERE id = :id", {id: accountId}
    )
    IF row IS NULL THEN RETURN NULL END IF
    SET account = NEW Account
    SET account.id = row["id"]
    SET account.ownerId = row["owner_id"]
    SET account.balance = row["balance"]
    SET account.currency = row["currency"]
    SET account.status = AccountStatus.FromString(row["status"])
    RETURN account
END FUNCTION

FUNCTION SqlAccountGateway.Save(account: Account):
    this.database.Execute(
        "UPDATE accounts SET balance = :bal, status = :st WHERE id = :id",
        {bal: account.balance, st: account.status.ToString(), id: account.id}
    )
END FUNCTION

// === COMPOSITION ROOT (Wiring) ===

FUNCTION ConfigureApplication() -> Application:
    SET db = ConnectDatabase(LoadConfig("db.url"))

    SET accountGateway = NEW SqlAccountGateway(db)
    SET presenter = NEW TransferPresenter()
    SET useCase = NEW TransferMoneyUseCase(accountGateway, presenter)
    SET controller = NEW TransferController(useCase)

    SET app = NEW Application
    app.Route("POST", "/transfers", controller.HandleTransfer)
    RETURN app
END FUNCTION
```

## When to Use

- **Complex domain logic** that benefits from isolation and testability.
- **Long-lived systems** where technology decisions will change over the
  system's lifetime.
- **Multiple delivery mechanisms** (web, mobile, CLI, batch) sharing the
  same business logic.
- **Teams practicing TDD** who need fast unit tests without infrastructure.
- **Systems with regulatory requirements** where business rules must be
  clearly separated and auditable.

## When NOT to Use

- **Simple CRUD applications** where the overhead of multiple boundaries
  outweighs the benefits.
- **Short-lived prototypes** where speed matters more than architecture.
- **Performance-critical hot paths** where abstraction layers add
  measurable latency.
- **Very small teams** who may find the ceremony burdensome.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Banking | Transaction processing system | Complex rules, regulatory isolation, long lifespan |
| Healthcare | Clinical workflow engine | Critical business logic must be infrastructure-independent |
| Insurance | Claims adjudication | Business rules change independently of technology |
| Government | Tax calculation system | Rules must be testable and auditable independently |
| SaaS | Multi-tenant platform | Same core logic, different infrastructure per tier |

## Trade-offs

### Advantages

- **Framework independence**: Business logic does not depend on any framework.
- **Testability**: Core logic tested with simple in-memory implementations.
- **Technology flexibility**: Infrastructure can be swapped via adapters.
- **Explicit boundaries**: Clear separation between concerns.
- **Longevity**: Architecture survives framework and database changes.

### Disadvantages

- **Boilerplate**: Many interfaces, data transfer objects, and mappers.
- **Learning curve**: The concentric model requires conceptual training.
- **Over-engineering risk**: Excessive for simple applications.
- **Mapping overhead**: Data must be translated at each boundary crossing.
- **Team discipline**: Requires consistent enforcement of dependency rules.

## Comparison with Related Patterns

| Dimension | Clean | Hexagonal | Layered |
|-----------|-------|-----------|---------|
| Layers | 4 concentric circles | 2 sides (driving/driven) | N horizontal layers |
| Dependency rule | Always inward | Always inward | Always downward |
| Output handling | Explicit output port/presenter | Part of driving adapter | Return through layers |
| Entity emphasis | Central (enterprise rules) | Part of core | Spread across layers |
| Formality | High (strict boundaries) | Medium | Low |

## Evolution and Variations

- **Screaming Architecture**: Clean architecture where the top-level
  directory structure reflects the domain (e.g., "accounts/", "transfers/")
  rather than technical layers (e.g., "controllers/", "models/").
- **Vertical Slicing**: Organizing by feature rather than by layer, with
  each slice containing its own use case, adapter, and entity components.
- **Modular Clean Architecture**: Combining clean architecture with module
  boundaries for large monoliths or bounded contexts.
- **Functional Clean Architecture**: Replacing object-oriented interactors
  with pure functions that take dependencies as parameters.

## Key Takeaways

1. The Dependency Rule is the single most important concept: source code
   dependencies must always point inward toward higher-level policies.
2. Entities represent enterprise-wide business rules; use cases represent
   application-specific rules. The distinction matters for reuse.
3. Interface adapters are translators. They do not contain business logic;
   they convert data between the format convenient for the use case and
   the format convenient for external agents.
4. The outermost circle is where all the "messy details" live. Frameworks,
   databases, and UIs are plugins to the application, not its foundation.
5. The architecture's value scales with domain complexity. For simple CRUD
   operations, the overhead is not justified. For complex business systems,
   it is essential.
