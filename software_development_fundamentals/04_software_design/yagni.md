# YAGNI - You Aren't Gonna Need It

## Definition

YAGNI states: do not implement functionality until it is actually needed.
Do not add features, abstractions, or infrastructure based on speculation
about future requirements.

Coined by Ron Jeffries as part of Extreme Programming (XP), YAGNI fights
the natural tendency of developers to build for imagined futures rather
than present realities.

```
+-------------------------------------------------------+
| "Always implement things when you actually need them,  |
|  never when you just foresee that you need them."      |
|                                    -- Ron Jeffries     |
+-------------------------------------------------------+
```

---

## The Cost of Speculation

Every feature has four costs, even if no one ever uses it:

```
+---------------------------------------------------+
| Cost Type       | Description                     |
+---------------------------------------------------+
| Build Cost      | Time to design, implement, test |
| Maintenance     | Ongoing updates, bug fixes      |
| Complexity      | Added cognitive load for team   |
| Opportunity     | Time not spent on real needs    |
+---------------------------------------------------+

Speculative Feature ROI:

  Cost:    [============================================]
  Value:   [====]  (if ever used)
  Value:   []      (if never used -- most common)
```

---

## Over-Engineering Examples

### Example 1: The Universal Data Processor

**Requirement:** "Read records from a file and compute totals."

**Over-engineered solution:**

```
MODULE DataPipeline:
    SET sources: List<DataSource>
    SET transforms: List<Transform>
    SET sinks: List<DataSink>
    SET errorHandler: ErrorStrategy
    SET retryPolicy: RetryPolicy
    SET metrics: MetricsCollector

    FUNCTION execute(config: PipelineConfig) -> PipelineResult:
        FOR EACH source IN sources DO
            SET data = CALL source.read(config)
            FOR EACH transform IN transforms DO
                SET data = CALL transform.apply(data)
            END FOR
            FOR EACH sink IN sinks DO
                CALL sink.write(data)
            END FOR
        END FOR
        RETURN metrics.summarize()
    END FUNCTION
END MODULE
```

**What was actually needed:**

```
FUNCTION computeTotals(filePath: Text) -> Map<Text, Number>:
    SET records = CALL readFile(filePath)
    SET totals = NEW EmptyMap()
    FOR EACH record IN records DO
        SET key = record.category
        SET totals[key] = totals.getOrDefault(key, 0) + record.amount
    END FOR
    RETURN totals
END FUNCTION
```

The pipeline framework adds 10x complexity for a problem solved in 10 lines.

### Example 2: Premature Multi-Tenancy

**Requirement:** "Build an internal tool for our team of 12 people."

**Over-engineered:**

```
MODULE TenantManager:
    FUNCTION resolveTenant(request: Request) -> Tenant
    FUNCTION isolateData(tenant: Tenant, query: Query) -> Query
    FUNCTION routeToShard(tenant: Tenant) -> DatabaseShard
    FUNCTION enforceQuota(tenant: Tenant, resource: Text) -> Boolean
END MODULE
```

**What was needed:**

```
// Single-tenant application with basic authentication
FUNCTION authenticate(token: Text) -> User:
    RETURN CALL userStore.findByToken(token)
END FUNCTION
```

---

## Feature Creep

Feature creep is the gradual expansion of scope beyond original requirements.
YAGNI is the antidote.

```
Feature Creep Timeline:

  Week 1: "Build a TODO list"
  Week 2: "Add categories and priorities"
  Week 3: "Add recurring tasks and reminders"
  Week 4: "Add team collaboration features"
  Week 5: "Add analytics dashboard"
  Week 6: "Add integration with 15 third-party services"

  Original requirement: A TODO list.
  Actual delivery: An incomplete project management suite.
```

---

## Decision Framework

```
FUNCTION shouldImplementFeature(feature: Feature) -> Decision:

    // Step 1: Is it needed RIGHT NOW?
    IF feature.isRequired BY currentIteration THEN
        RETURN "Implement it"
    END IF

    // Step 2: Is there a concrete, committed plan?
    IF feature.isScheduled IN nextIteration AND confirmed THEN
        RETURN "Design for it, but do not build yet"
    END IF

    // Step 3: Is it speculative?
    IF feature.isBasedOn = "might need" OR "just in case" THEN
        RETURN "Do NOT implement"
    END IF

    // Step 4: Reversibility check
    IF costToAddLater IS LOW THEN
        RETURN "Wait until needed"
    END IF

    // Step 5: Rare exception -- irreversibility
    IF costToAddLater IS VERY HIGH AND likelihood IS HIGH THEN
        RETURN "Consider minimal preparation (not full build)"
    END IF

    RETURN "Do NOT implement"
END FUNCTION
```

```
Decision Tree (ASCII):

  Is it needed NOW?
       |
    YES --> Build it.
    NO  --> Is there a committed plan for it?
                |
             YES --> Design hook points only. Do not build.
             NO  --> Is the cost of adding it later very high?
                        |
                     YES --> Evaluate carefully. Maybe add a seam.
                     NO  --> Do NOT build it. Period.
```

---

## Relationship to Agile

YAGNI is deeply rooted in Agile methodology:

```
+------------------------------------------------------+
| Agile Principle               | YAGNI Connection     |
+------------------------------------------------------+
| Working software over         | Build what works now, |
| comprehensive documentation   | not what might work   |
+------------------------------------------------------+
| Responding to change over     | Adapt when needs are  |
| following a plan              | real, not predicted   |
+------------------------------------------------------+
| Deliver frequently            | Ship small increments |
|                               | of real value         |
+------------------------------------------------------+
| Simplicity -- maximizing the  | YAGNI is this         |
| amount of work NOT done       | principle in action   |
+------------------------------------------------------+
```

### Iterative Approach

```
Iteration 1:  [Core Feature] -----------> Ship
Iteration 2:  [Core Feature + Feedback] -> Ship
Iteration 3:  [Add Feature B if needed] -> Ship
Iteration 4:  [Add Feature C if needed] -> Ship

NOT:
Big Bang:     [Core + B + C + D + E + F] -> Ship (maybe)
```

---

## When YAGNI Does NOT Apply

YAGNI is not absolute. There are legitimate exceptions:

```
+----------------------------------------------------------+
| Exception                  | Rationale                    |
+----------------------------------------------------------+
| Security fundamentals      | Retrofitting security is     |
|                            | extremely costly and risky   |
+----------------------------------------------------------+
| Data model extensibility   | Schema migrations on large   |
|                            | datasets are expensive        |
+----------------------------------------------------------+
| Public API contracts       | Breaking changes after       |
|                            | release are very costly      |
+----------------------------------------------------------+
| Accessibility              | Retrofitting is harder than  |
|                            | building in from the start   |
+----------------------------------------------------------+
| Logging and observability  | Cannot debug production      |
|                            | issues without them          |
+----------------------------------------------------------+
| Backup and recovery        | Needed before disaster, not  |
|                            | after                        |
+----------------------------------------------------------+
```

The key distinction: these are cross-cutting concerns that are genuinely
harder to add later, not speculative features.

---

## YAGNI vs Good Design

YAGNI does not mean "write sloppy code." It means:

```
+---------------------------------------------+
| YAGNI Says                | YAGNI Does NOT  |
|                           | Say              |
+---------------------------------------------+
| Do not build unused       | Write messy code |
| features                  |                  |
+---------------------------------------------+
| Do not add speculative    | Skip tests       |
| abstractions              |                  |
+---------------------------------------------+
| Do not over-architect     | Ignore design    |
| for unknown futures       | principles       |
+---------------------------------------------+
| Keep scope minimal        | Cut corners on   |
|                           | quality          |
+---------------------------------------------+
```

Write clean, well-tested code for the features you ARE building. Just do not
build features nobody asked for.

---

## Summary

```
Before writing any code, ask:

  1. Does someone need this RIGHT NOW?
  2. Am I building this because of a real requirement or a "what if"?
  3. What is the cost of adding this later versus now?
  4. Am I gold-plating, or solving a real problem?

If the answer to #1 is "no" and #2 is "what if" -- stop.
The best code is the code you never had to write.
```
