# Model-View-Controller (MVC)

## Overview and Intent

Model-View-Controller separates an application into three interconnected
components: the Model (data and business logic), the View (user interface and
presentation), and the Controller (input handling and coordination). Each
component has a distinct responsibility, and the separation enables independent
development, testing, and modification of each concern.

The intent of MVC is to decouple the internal representation of data from the
way it is presented to and manipulated by the user. This decoupling allows
multiple views to display the same data differently (e.g., a table view and a
chart view), and it allows the user interface to change without affecting the
underlying business logic.

MVC was originally conceived for desktop graphical user interfaces in the late
1970s, but its principles have been adapted for web applications, mobile apps,
and APIs. The specific data flow varies across implementations -- some use a
"push" model where the controller pushes data to the view, while others use
a "pull" model where the view observes the model directly.

## Structure and Components

### Component Diagram

```
                    User Interaction
                         |
                         v
                  +------+------+
                  | CONTROLLER  |
                  |             |
                  | - Handles   |
                  |   input     |
                  | - Updates   |
                  |   model     |
                  | - Selects   |
                  |   view      |
                  +--+-------+--+
                     |       |
          updates    |       |   selects
                     v       v
              +------+--+ +--+------+
              |  MODEL  | |  VIEW   |
              |         | |         |
              | - Data  | | - Render|
              | - Rules | | - Display
              | - State | | - Format|
              +----+----+ +----+----+
                   |            ^
                   |            |
                   +------------+
                   notifies / observes
```

### Key Components

1. **Model**: Manages application data, business rules, and state. It is
   independent of the user interface. When its state changes, it notifies
   observers (views or controllers).

2. **View**: Renders the model's data for the user. It reads from the model
   but does not modify it. A single model can have multiple views.

3. **Controller**: Receives user input (clicks, keystrokes, form submissions),
   interprets it, and invokes the appropriate operations on the model. It may
   also select which view to render.

## How It Works

```
  1. User submits form
       |
       v
  +----+------+
  | Controller|  2. Parse and validate input
  |           |  3. Call model operations
  +----+------+
       |
       v
  +----+------+
  | Model     |  4. Apply business logic
  |           |  5. Update internal state
  +----+------+  6. Notify observers
       |
       v
  +----+------+
  | View      |  7. Read updated model state
  |           |  8. Render response
  +----+------+
       |
       v
  Response sent to user
```

Detailed flow (web variant):

1. The user performs an action (submits a form, clicks a link).
2. The controller receives the request and extracts relevant data.
3. The controller validates input and calls the appropriate model method.
4. The model processes the request: applies rules, validates data, updates
   state, and persists changes.
5. The model may notify registered observers of the state change.
6. The controller selects the appropriate view and passes the model data.
7. The view reads the model (or a view model) and renders the output.
8. The rendered output is sent back to the user.

## Pseudocode Example

```
// === MODEL ===

STRUCTURE Task:
    id: String
    title: String
    description: String
    status: TaskStatus    // PENDING, IN_PROGRESS, COMPLETED
    assignedTo: String
    dueDate: DateTime
    createdAt: DateTime
END STRUCTURE

ENUMERATION TaskStatus:
    PENDING, IN_PROGRESS, COMPLETED
END ENUMERATION

STRUCTURE TaskModel:
    tasks: List of Task
    observers: List of Observer
END STRUCTURE

FUNCTION TaskModel.AddTask(title: String, description: String, dueDate: DateTime) -> Task:
    IF title IS EMPTY THEN
        RAISE ValidationError("Title is required")
    END IF
    IF dueDate < Now() THEN
        RAISE ValidationError("Due date must be in the future")
    END IF

    SET task = NEW Task
    SET task.id = GenerateId()
    SET task.title = title
    SET task.description = description
    SET task.status = PENDING
    SET task.createdAt = Now()
    SET task.dueDate = dueDate

    Append(this.tasks, task)
    TaskStore.Save(task)
    this.NotifyObservers("task_added", task)
    RETURN task
END FUNCTION

FUNCTION TaskModel.UpdateStatus(taskId: String, newStatus: TaskStatus):
    SET task = this.FindById(taskId)
    IF task IS NULL THEN
        RAISE NotFoundError("Task not found")
    END IF

    // Business rule: cannot reopen completed tasks
    IF task.status = COMPLETED AND newStatus != COMPLETED THEN
        RAISE BusinessRuleError("Cannot reopen completed tasks")
    END IF

    SET task.status = newStatus
    TaskStore.Update(task)
    this.NotifyObservers("task_updated", task)
END FUNCTION

FUNCTION TaskModel.GetTasksByStatus(status: TaskStatus) -> List of Task:
    RETURN Filter(this.tasks, FUNCTION(t): RETURN t.status = status END FUNCTION)
END FUNCTION

FUNCTION TaskModel.NotifyObservers(event: String, data: Any):
    FOR EACH observer IN this.observers DO
        observer.OnModelChanged(event, data)
    END FOR
END FUNCTION

// === CONTROLLER ===

STRUCTURE TaskController:
    model: TaskModel
    viewFactory: ViewFactory
END STRUCTURE

FUNCTION TaskController.HandleRequest(request: Request) -> Response:
    IF request.method = "GET" AND request.path = "/tasks" THEN
        RETURN this.ListTasks(request)
    ELSE IF request.method = "POST" AND request.path = "/tasks" THEN
        RETURN this.CreateTask(request)
    ELSE IF request.method = "PUT" AND StartsWith(request.path, "/tasks/") THEN
        RETURN this.UpdateTask(request)
    ELSE
        RETURN Response.NotFound("Unknown endpoint")
    END IF
END FUNCTION

FUNCTION TaskController.ListTasks(request: Request) -> Response:
    SET statusFilter = request.queryParams["status"]

    IF statusFilter IS NOT NULL THEN
        SET tasks = this.model.GetTasksByStatus(TaskStatus.FromString(statusFilter))
    ELSE
        SET tasks = this.model.tasks
    END IF

    // Select appropriate view based on client preference
    SET format = request.headers["Accept"]
    SET view = this.viewFactory.CreateView(format)

    RETURN view.RenderTaskList(tasks)
END FUNCTION

FUNCTION TaskController.CreateTask(request: Request) -> Response:
    SET body = ParseBody(request.body)

    TRY
        SET task = this.model.AddTask(
            title: body["title"],
            description: body["description"],
            dueDate: ParseDate(body["dueDate"])
        )

        SET view = this.viewFactory.CreateView(request.headers["Accept"])
        RETURN view.RenderTaskCreated(task)

    CATCH error AS ValidationError
        SET view = this.viewFactory.CreateView(request.headers["Accept"])
        RETURN view.RenderError(400, error.message)
    END TRY
END FUNCTION

FUNCTION TaskController.UpdateTask(request: Request) -> Response:
    SET taskId = ExtractId(request.path)
    SET body = ParseBody(request.body)

    TRY
        this.model.UpdateStatus(taskId, TaskStatus.FromString(body["status"]))
        SET task = this.model.FindById(taskId)

        SET view = this.viewFactory.CreateView(request.headers["Accept"])
        RETURN view.RenderTaskUpdated(task)

    CATCH error AS NotFoundError
        SET view = this.viewFactory.CreateView(request.headers["Accept"])
        RETURN view.RenderError(404, error.message)

    CATCH error AS BusinessRuleError
        SET view = this.viewFactory.CreateView(request.headers["Accept"])
        RETURN view.RenderError(422, error.message)
    END TRY
END FUNCTION

// === VIEWS ===

STRUCTURE HtmlTaskView:
    // Renders tasks as HTML pages
END STRUCTURE

FUNCTION HtmlTaskView.RenderTaskList(tasks: List of Task) -> Response:
    SET html = "<html><body>"
    SET html = html + "<h1>Task List</h1>"
    SET html = html + "<table><tr><th>Title</th><th>Status</th><th>Due</th></tr>"

    FOR EACH task IN tasks DO
        SET html = html + "<tr>"
        SET html = html + "<td>" + EscapeHtml(task.title) + "</td>"
        SET html = html + "<td>" + task.status.ToString() + "</td>"
        SET html = html + "<td>" + FormatDate(task.dueDate) + "</td>"
        SET html = html + "</tr>"
    END FOR

    SET html = html + "</table></body></html>"
    RETURN Response(200, html, contentType: "text/html")
END FUNCTION

STRUCTURE JsonTaskView:
    // Renders tasks as structured data
END STRUCTURE

FUNCTION JsonTaskView.RenderTaskList(tasks: List of Task) -> Response:
    SET result = NEW List
    FOR EACH task IN tasks DO
        Append(result, {
            id: task.id,
            title: task.title,
            status: task.status.ToString(),
            dueDate: FormatIsoDate(task.dueDate)
        })
    END FOR
    RETURN Response(200, Serialize(result), contentType: "application/json")
END FUNCTION

FUNCTION JsonTaskView.RenderError(code: Integer, message: String) -> Response:
    RETURN Response(code, Serialize({error: message}), contentType: "application/json")
END FUNCTION

// === VIEW FACTORY ===

FUNCTION ViewFactory.CreateView(acceptHeader: String) -> TaskView:
    IF Contains(acceptHeader, "application/json") THEN
        RETURN NEW JsonTaskView()
    ELSE
        RETURN NEW HtmlTaskView()
    END IF
END FUNCTION
```

## When to Use

- **Applications with user interfaces** where separating display logic from
  business logic improves maintainability.
- **Systems needing multiple representations** of the same data (HTML, JSON,
  charts, reports).
- **Teams with specialized roles** (UI designers and backend developers)
  who work on different concerns simultaneously.
- **Web applications** following the request-response cycle where controller
  routing maps naturally to URL patterns.
- **Rapid prototyping** where the pattern's simplicity enables quick iteration.

## When NOT to Use

- **Purely API-driven backends** with no user interface rendering, where
  simpler patterns (layered, hexagonal) suffice.
- **Real-time interactive applications** where the request-response cycle
  is too coarse (consider MVVM or reactive patterns).
- **Very simple applications** with a single view and minimal logic where
  MVC adds unnecessary separation.
- **Event-driven systems** where the input-process-output flow does not
  map to user actions.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Web applications | Blog platform | Clear separation of content model, template rendering, request handling |
| Desktop apps | Spreadsheet editor | Multiple views (sheet, chart, formula bar) of same data model |
| Mobile apps | Task management | Shared model across list view, detail view, calendar view |
| Enterprise | CRM dashboard | Multiple report views over shared customer model |
| Gaming | Turn-based game UI | Game state (model) separate from rendering (view) and input (controller) |

## Trade-offs

### Advantages

- **Separation of concerns**: Each component has a single responsibility.
- **Multiple views**: Same data can be displayed in different formats.
- **Parallel development**: View and model can be developed independently.
- **Testable model**: Business logic can be tested without UI.
- **Well-understood**: One of the most widely known patterns.

### Disadvantages

- **Controller bloat**: Controllers tend to accumulate logic over time.
- **View-model coupling**: Views often need knowledge of model structure.
- **Complexity for simple apps**: Overhead is not justified for trivial UIs.
- **Tight controller-view binding**: Changing one often requires changing
  the other.
- **Navigation logic**: Controller often mixes input handling with routing.

## Comparison with Related Patterns

| Dimension | MVC | MVVM | MVP |
|-----------|-----|------|-----|
| Data flow | Controller -> Model -> View | ViewModel <-> View (binding) | Presenter <-> View (interface) |
| View intelligence | Reads model directly | Binds to ViewModel | Passive, controlled by Presenter |
| Testability | Model testable, view harder | ViewModel easily testable | Presenter easily testable |
| Coupling | View knows model structure | View knows ViewModel | View knows Presenter interface |
| Best for | Web request/response | Rich client with data binding | Desktop/mobile with passive views |

## Evolution and Variations

- **MVC-1 (Original)**: View directly observes model. Controller handles
  input only. Suited for desktop applications with observer patterns.
- **MVC-2 (Web)**: Controller mediates all communication. View never
  contacts model directly. The dominant web application pattern.
- **Front Controller**: Single controller handles all requests and
  delegates to sub-controllers. Common in web frameworks.
- **Hierarchical MVC (HMVC)**: MVC triads can contain nested MVC triads.
  Useful for complex UIs with reusable widget components.

## Key Takeaways

1. MVC is a separation pattern, not an architecture. It is typically
   used within a larger architectural context (layered, hexagonal).
2. The model should contain all business logic. Fat controllers and
   smart views are signs of architectural drift.
3. The observer relationship between model and view is optional. In web
   applications, the controller typically mediates all data flow.
4. MVC works best for request-response interfaces. For rich interactive
   UIs with two-way data binding, consider MVVM.
5. The pattern's value scales with UI complexity. For a single-page API
   backend, MVC adds structure without clear benefit.
