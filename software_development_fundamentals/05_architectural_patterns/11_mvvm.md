# Model-View-ViewModel (MVVM)

## Overview and Intent

Model-View-ViewModel separates an application into three components: the Model
(data and business logic), the View (visual presentation), and the ViewModel
(an abstraction of the view that exposes data and commands in a binding-friendly
format). The key innovation is the data binding mechanism that automatically
synchronizes the View and ViewModel, eliminating manual UI update code.

The intent of MVVM is to maximize testability and maintainability of
presentation logic. The ViewModel contains all the logic for preparing data
for display and handling user actions, but it has no direct reference to the
View. This means the ViewModel can be fully tested without any UI framework,
using simple assertions on its properties and command outputs.

MVVM emerged from the desktop application world where rich data binding
frameworks made two-way synchronization practical. It has since been adopted
in mobile development and modern frontend frameworks that support reactive
data binding. The pattern is particularly effective when the UI is complex,
interactive, and data-driven.

## Structure and Components

### Component Diagram

```
+-------------------------------------------------------------------+
|                                                                     |
|  +------------------+    Data Binding    +-------------------+     |
|  |      VIEW        |<=================>|    VIEWMODEL       |     |
|  |                   |                    |                   |     |
|  | - Visual layout   |    Properties      | - Observable      |     |
|  | - Input controls  |<--- (one-way) ----|   properties      |     |
|  | - Display widgets |                    | - Commands        |     |
|  | - Animations      |--- (user input) ->| - Validation      |     |
|  |                   |    Commands        | - State mgmt      |     |
|  +------------------+                    +--------+----------+     |
|                                                    |               |
|                                           calls    |               |
|                                                    v               |
|                                          +--------+----------+     |
|                                          |      MODEL        |     |
|                                          |                   |     |
|                                          | - Domain entities |     |
|                                          | - Business rules  |     |
|                                          | - Data access     |     |
|                                          +-------------------+     |
|                                                                     |
+-------------------------------------------------------------------+

  Data Binding Types:
  ================>  One-Way (Model to View)
  <===============>  Two-Way (Bidirectional)
  --------------->   Command Binding (View to ViewModel)
```

### Key Components

1. **Model**: Business logic, domain objects, and data access. Identical to
   the Model in MVC. Unaware of the ViewModel or View.

2. **View**: The visual UI. In MVVM, the View is declarative and "dumb" --
   it defines layout and binds to ViewModel properties. Contains minimal
   or no code-behind.

3. **ViewModel**: The bridge between View and Model. Exposes observable
   properties that the View binds to, and commands that the View invokes.
   Contains all presentation logic (formatting, visibility rules, validation).

4. **Data Binding Engine**: The framework mechanism that automatically
   synchronizes View elements with ViewModel properties.

5. **Commands**: Objects that encapsulate user actions. The ViewModel exposes
   commands; the View binds to them.

## How It Works

```
  User types in a text field
       |
       v
  +----+-----+
  | View     |  1. Binding engine detects input change
  | (UI)     |  2. Updates bound ViewModel property
  +----+-----+
       |
       | two-way binding
       v
  +----+--------+
  | ViewModel   |  3. Property setter fires
  |             |  4. Validation runs
  +----+--------+  5. Dependent properties recalculate
       |           6. Property change notifications fire
       |
       | notifications
       v
  +----+-----+
  | View     |  7. Binding engine updates all bound elements
  | (UI)     |  8. Error messages appear if validation failed
  +----+-----+  9. Submit button enables/disables based on validity

  User clicks Submit button
       |
       v
  +----+-----+
  | View     |  10. Command binding invokes ViewModel command
  +----+-----+
       |
       v
  +----+--------+
  | ViewModel   |  11. Command executes, calls Model methods
  +----+--------+  12. Updates properties with results
       |
       v
  +----+-----+
  | Model    |  13. Processes business logic, returns results
  +----+-----+
       |
  (ViewModel updates properties -> binding engine -> View refreshes)
```

## Pseudocode Example

```
// === MODEL ===

STRUCTURE Contact:
    id: String
    firstName: String
    lastName: String
    email: String
    phone: String
END STRUCTURE

INTERFACE ContactRepository:
    FUNCTION FindAll() -> List of Contact
    FUNCTION FindById(id: String) -> Contact
    FUNCTION Save(contact: Contact) -> Contact
    FUNCTION Delete(id: String) -> Boolean
END INTERFACE

// === VIEWMODEL ===

STRUCTURE ContactEditViewModel:
    // Observable properties (View binds to these)
    firstName: ObservableProperty of String
    lastName: ObservableProperty of String
    email: ObservableProperty of String
    phone: ObservableProperty of String
    fullName: ComputedProperty of String
    isValid: ComputedProperty of Boolean
    errorMessages: ObservableProperty of Map
    isSaving: ObservableProperty of Boolean
    statusMessage: ObservableProperty of String

    // Commands (View binds buttons to these)
    saveCommand: Command
    cancelCommand: Command

    // Dependencies
    repository: ContactRepository
    contactId: String
END STRUCTURE

FUNCTION ContactEditViewModel.Initialize(contactId: String, repo: ContactRepository):
    SET this.repository = repo
    SET this.contactId = contactId
    SET this.errorMessages = NEW ObservableProperty(EMPTY MAP)
    SET this.isSaving = NEW ObservableProperty(FALSE)
    SET this.statusMessage = NEW ObservableProperty("")

    // Set up observable properties with change handlers
    SET this.firstName = NEW ObservableProperty("", OnChange: this.Validate)
    SET this.lastName = NEW ObservableProperty("", OnChange: this.Validate)
    SET this.email = NEW ObservableProperty("", OnChange: this.Validate)
    SET this.phone = NEW ObservableProperty("", OnChange: this.Validate)

    // Computed property: automatically recalculates when dependencies change
    SET this.fullName = NEW ComputedProperty(
        dependencies: [this.firstName, this.lastName],
        compute: FUNCTION():
            RETURN Trim(this.firstName.value + " " + this.lastName.value)
        END FUNCTION
    )

    SET this.isValid = NEW ComputedProperty(
        dependencies: [this.errorMessages],
        compute: FUNCTION():
            RETURN Length(this.errorMessages.value) = 0
        END FUNCTION
    )

    // Commands
    SET this.saveCommand = NEW Command(
        execute: this.Save,
        canExecute: FUNCTION(): RETURN this.isValid.value AND NOT this.isSaving.value END FUNCTION
    )
    SET this.cancelCommand = NEW Command(
        execute: this.Cancel,
        canExecute: FUNCTION(): RETURN TRUE END FUNCTION
    )

    IF contactId IS NOT NULL THEN
        this.LoadContact(contactId)
    END IF
END FUNCTION

FUNCTION ContactEditViewModel.LoadContact(contactId: String):
    SET contact = this.repository.FindById(contactId)
    IF contact IS NOT NULL THEN
        SET this.firstName.value = contact.firstName
        SET this.lastName.value = contact.lastName
        SET this.email.value = contact.email
        SET this.phone.value = contact.phone
    END IF
END FUNCTION

FUNCTION ContactEditViewModel.Validate():
    SET errors = NEW Map

    IF Length(Trim(this.firstName.value)) = 0 THEN
        SET errors["firstName"] = "First name is required"
    END IF
    IF Length(Trim(this.lastName.value)) = 0 THEN
        SET errors["lastName"] = "Last name is required"
    END IF
    IF NOT IsValidEmail(this.email.value) THEN
        SET errors["email"] = "Please enter a valid email address"
    END IF
    IF Length(this.phone.value) > 0 AND NOT IsValidPhone(this.phone.value) THEN
        SET errors["phone"] = "Please enter a valid phone number"
    END IF

    SET this.errorMessages.value = errors
END FUNCTION

FUNCTION ContactEditViewModel.Save():
    IF NOT this.isValid.value THEN RETURN END IF
    SET this.isSaving.value = TRUE
    SET this.statusMessage.value = "Saving..."

    TRY
        SET contact = NEW Contact
        SET contact.id = this.contactId
        SET contact.firstName = Trim(this.firstName.value)
        SET contact.lastName = Trim(this.lastName.value)
        SET contact.email = Trim(this.email.value)
        SET contact.phone = Trim(this.phone.value)

        SET saved = this.repository.Save(contact)
        SET this.contactId = saved.id
        SET this.statusMessage.value = "Contact saved successfully"
    CATCH error
        SET this.statusMessage.value = "Error saving: " + error.message
    END TRY
    SET this.isSaving.value = FALSE
END FUNCTION

// === CONTACT LIST VIEWMODEL ===

STRUCTURE ContactListViewModel:
    contacts: ObservableProperty of List
    searchQuery: ObservableProperty of String
    filteredContacts: ComputedProperty of List
    selectedContact: ObservableProperty of Contact
    isLoading: ObservableProperty of Boolean
    deleteCommand: Command
    refreshCommand: Command
    repository: ContactRepository
END STRUCTURE

FUNCTION ContactListViewModel.Initialize(repo: ContactRepository):
    SET this.repository = repo
    SET this.contacts = NEW ObservableProperty(EMPTY LIST)
    SET this.searchQuery = NEW ObservableProperty("")
    SET this.isLoading = NEW ObservableProperty(FALSE)
    SET this.selectedContact = NEW ObservableProperty(NULL)

    SET this.filteredContacts = NEW ComputedProperty(
        dependencies: [this.contacts, this.searchQuery],
        compute: FUNCTION():
            IF Length(this.searchQuery.value) = 0 THEN
                RETURN this.contacts.value
            END IF
            SET query = LowerCase(this.searchQuery.value)
            RETURN Filter(this.contacts.value, FUNCTION(c):
                RETURN Contains(LowerCase(c.firstName), query) OR
                       Contains(LowerCase(c.lastName), query) OR
                       Contains(LowerCase(c.email), query)
            END FUNCTION)
        END FUNCTION
    )

    SET this.deleteCommand = NEW Command(
        execute: this.DeleteSelected,
        canExecute: FUNCTION(): RETURN this.selectedContact.value IS NOT NULL END FUNCTION
    )
    this.LoadContacts()
END FUNCTION

FUNCTION ContactListViewModel.LoadContacts():
    SET this.isLoading.value = TRUE
    TRY
        SET this.contacts.value = this.repository.FindAll()
    CATCH error
        LOG "Failed to load contacts: " + error.message
    END TRY
    SET this.isLoading.value = FALSE
END FUNCTION

// === VIEW (Declarative Binding Specification) ===

VIEW ContactEditView BINDS TO ContactEditViewModel:
    TextInput("First Name")  BIND value TO firstName
                              BIND errorText TO errorMessages["firstName"]
    TextInput("Last Name")   BIND value TO lastName
                              BIND errorText TO errorMessages["lastName"]
    TextInput("Email")       BIND value TO email
    TextInput("Phone")       BIND value TO phone
    Label("Full Name")       BIND text TO fullName
    Button("Save")           BIND command TO saveCommand
                              BIND enabled TO saveCommand.canExecute
    Button("Cancel")         BIND command TO cancelCommand
    ProgressIndicator        BIND visible TO isSaving
    Label("Status")          BIND text TO statusMessage
END VIEW
```

## When to Use

- **Rich interactive UIs** with complex forms, real-time validation, and
  dynamic visibility rules.
- **Applications with data binding frameworks** that support automatic
  View-ViewModel synchronization.
- **Teams needing high testability** of presentation logic without
  running a UI framework in tests.
- **Desktop and mobile applications** with stateful, interactive screens.
- **Applications where UI designers and developers** work separately --
  designers own the View, developers own the ViewModel.

## When NOT to Use

- **Server-rendered web applications** where the request-response model
  makes data binding impractical. Use MVC instead.
- **Simple read-only displays** where the overhead of ViewModels and
  bindings is not justified.
- **Applications without binding support** where manual synchronization
  negates the pattern's benefits.
- **API-only services** with no user interface.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Desktop software | Accounting application | Complex forms with real-time validation and computed fields |
| Mobile apps | Banking app | Interactive screens with offline capability and state management |
| Dashboard | Analytics platform | Multiple widgets binding to shared data models |
| POS system | Retail checkout | Real-time price calculation, inventory status |
| Design tools | Drawing application | Properties panel binding to selected object attributes |

## Trade-offs

### Advantages

- **Testable presentation logic**: ViewModel tested without UI framework.
- **Clean separation**: View is purely declarative; logic lives in ViewModel.
- **Automatic synchronization**: Data binding eliminates manual UI updates.
- **Designer-developer workflow**: Clear boundary between visual and logic.
- **Reusable ViewModels**: Same ViewModel can power different Views.

### Disadvantages

- **Binding complexity**: Debugging data binding issues can be difficult.
- **Memory overhead**: Observable properties and binding infrastructure
  consume additional memory.
- **Learning curve**: Reactive/observable patterns require training.
- **Overkill for simple views**: Simple displays do not benefit.
- **Binding performance**: Large numbers of bindings can cause lag.

## Comparison with Related Patterns

| Dimension | MVVM | MVC | MVP |
|-----------|------|-----|-----|
| View-Logic coupling | Low (binding) | Medium (controller selects view) | Low (presenter interface) |
| Testability | High (ViewModel) | Medium (controller) | High (presenter) |
| Data sync mechanism | Automatic binding | Manual push/pull | Manual through presenter |
| View intelligence | Declarative only | Reads model | Passive interface |
| Best suited for | Rich clients + binding | Web request/response | Desktop without binding |

## Evolution and Variations

- **MVVM with Coordinator**: Adds a navigation component that the
  ViewModel delegates to for screen transitions.
- **Redux/Flux Pattern**: Unidirectional data flow variant where state is
  centralized and views derive from a single state tree.
- **Reactive MVVM**: ViewModels expose reactive streams instead of
  observable properties, enabling composition of async data flows.
- **Presentation Model**: Martin Fowler's precursor to MVVM, with the
  same separation but without automatic data binding.

## Key Takeaways

1. The ViewModel is the pattern's core innovation. It contains all
   presentation logic without referencing the View directly.
2. Data binding is what makes MVVM practical. Without a binding framework,
   the pattern degenerates into manual synchronization code.
3. ViewModels should be testable with nothing but the standard testing
   framework -- no UI toolkit, no rendering engine.
4. Commands replace event handlers. Instead of the View calling methods,
   it binds buttons to command objects that the ViewModel exposes.
5. MVVM shines in rich, interactive applications. For server-rendered
   pages or simple displays, MVC or simpler patterns are more appropriate.
