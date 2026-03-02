# Technical Documentation

## Introduction

Documentation is how teams scale knowledge beyond the people who happen to be
in the room. Code tells the computer what to do; documentation tells humans why,
how, and what else they need to know. Poor documentation forces every new team
member to reverse-engineer understanding from source code. Good documentation
multiplies the effectiveness of every person who reads it.

The challenge is not writing documentation -- it is writing documentation that
stays accurate, serves its audience, and is worth the cost of maintaining it.

## The Diataxis Framework

The Diataxis framework (by Daniele Procida) organizes documentation into four
distinct types based on what the reader needs. Each type serves a different
purpose and should be written differently.

```
THE DIATAXIS FRAMEWORK

                        PRACTICAL                    THEORETICAL
                   (steps to follow)            (knowledge to absorb)
              +----------------------------+----------------------------+
              |                            |                            |
              |        TUTORIALS           |       EXPLANATION          |
   LEARNING   |                            |                            |
   (study)    |  "Learning-oriented"       |  "Understanding-oriented"  |
              |  Guide the reader through  |  Clarify and illuminate a  |
              |  a series of steps to      |  particular topic. Discuss |
              |  learn a skill.            |  context, background, and  |
              |                            |  design decisions.         |
              |  Example: "Getting         |                            |
              |  Started with the API"     |  Example: "How the cache   |
              |                            |  invalidation strategy     |
              |                            |  works"                    |
              +----------------------------+----------------------------+
              |                            |                            |
              |        HOW-TO GUIDES       |       REFERENCE            |
   WORKING    |                            |                            |
   (apply)    |  "Task-oriented"           |  "Information-oriented"    |
              |  Show how to solve a       |  Describe the system       |
              |  specific problem. Assume  |  accurately and completely.|
              |  competence, provide steps.|  Structured for lookup,    |
              |                            |  not reading.              |
              |  Example: "How to add a    |                            |
              |  new payment provider"     |  Example: "Configuration   |
              |                            |  options reference"        |
              +----------------------------+----------------------------+
```

### Key Principle

Do not mix types within a single document. A tutorial that suddenly becomes a
reference table confuses the reader. Each document should serve one purpose
clearly.

## README Anatomy

The README is often the first document a new contributor encounters. A good
README answers the most common questions in a predictable order.

```
README STRUCTURE

  +-------------------------------------------------------+
  |  PROJECT NAME                                         |
  |  One-sentence description of what this does           |
  +-------------------------------------------------------+
  |                                                       |
  |  ## Status                                            |
  |  Build/test status, maturity level                    |
  |                                                       |
  |  ## Quick Start                                       |
  |  Minimal steps to get running (under 5 minutes)       |
  |                                                       |
  |  ## Prerequisites                                     |
  |  What must be installed or configured beforehand       |
  |                                                       |
  |  ## Installation                                      |
  |  Step-by-step setup instructions                      |
  |                                                       |
  |  ## Usage                                             |
  |  Common operations with examples                      |
  |                                                       |
  |  ## Configuration                                     |
  |  Available settings and their effects                 |
  |                                                       |
  |  ## Architecture                                      |
  |  High-level structure (link to detailed docs)         |
  |                                                       |
  |  ## Contributing                                      |
  |  How to submit changes, coding standards              |
  |                                                       |
  |  ## License                                           |
  |  Legal terms                                          |
  +-------------------------------------------------------+
```

### README Anti-Patterns

- **The novel**: Pages of text with no structure or headings
- **The skeleton**: Section headings with "TODO" under each one
- **The fossil**: Instructions that were accurate two years ago
- **The sales pitch**: Marketing language instead of technical content

## API Documentation Patterns

API documentation is reference material. It must be precise, complete, and
structured for lookup rather than sequential reading.

```
API DOCUMENTATION TEMPLATE (per endpoint/function)

  +----------------------------------------------------------+
  |  NAME: CalculateShippingCost                             |
  +----------------------------------------------------------+
  |  PURPOSE: Compute shipping cost based on weight,         |
  |           destination, and shipping speed.                |
  +----------------------------------------------------------+
  |  PARAMETERS:                                             |
  |    weight      : Number  - Package weight in kilograms   |
  |    destination  : String  - Destination region code       |
  |    speed       : String  - "standard" or "express"       |
  +----------------------------------------------------------+
  |  RETURNS:                                                |
  |    Number - Cost in base currency units                  |
  +----------------------------------------------------------+
  |  ERRORS:                                                 |
  |    InvalidWeight      - weight <= 0 or > 500             |
  |    UnknownDestination - region code not recognized        |
  +----------------------------------------------------------+
  |  EXAMPLE:                                                |
  |    CalculateShippingCost(2.5, "NA-EAST", "express")      |
  |    => 14.50                                              |
  +----------------------------------------------------------+
  |  NOTES:                                                  |
  |    Rates are refreshed daily. Express adds 60% surcharge.|
  +----------------------------------------------------------+
```

### Documentation for Each Parameter

Every parameter should document:
1. **Name** -- What it is called
2. **Type** -- What kind of value it accepts
3. **Description** -- What it represents
4. **Constraints** -- Valid ranges, formats, or enumerated values
5. **Default** -- What happens if it is omitted (if optional)

## Architecture Decision Records (ADR)

ADRs capture the *why* behind significant technical decisions. They are
invaluable months or years later when someone asks "why did we do it this way?"

```
ADR TEMPLATE

  +----------------------------------------------------------+
  |  ADR-NNN: [Title of Decision]                            |
  +----------------------------------------------------------+
  |                                                          |
  |  STATUS: [Proposed | Accepted | Deprecated | Superseded] |
  |                                                          |
  |  CONTEXT:                                                |
  |    What is the situation? What forces are at play?        |
  |    What constraints exist?                               |
  |                                                          |
  |  DECISION:                                               |
  |    What did we decide to do?                             |
  |                                                          |
  |  ALTERNATIVES CONSIDERED:                                |
  |    What other options were evaluated?                     |
  |    Why were they rejected?                               |
  |                                                          |
  |  CONSEQUENCES:                                           |
  |    What are the positive outcomes?                       |
  |    What are the negative outcomes or trade-offs?          |
  |    What new constraints does this create?                 |
  |                                                          |
  |  DATE: YYYY-MM-DD                                        |
  |  AUTHORS: [names]                                        |
  +----------------------------------------------------------+
```

### ADR Best Practices

- Number ADRs sequentially (ADR-001, ADR-002, ...) for easy reference.
- Write the context section for someone who was not in the room.
- Never delete or modify accepted ADRs -- create a new one that supersedes the old.
- Store ADRs alongside the code they describe.

## Documentation as Code Workflow

Treat documentation with the same rigor as source code: version it, review it,
test it, and deploy it.

```
DOCUMENTATION AS CODE WORKFLOW

  +----------+    +----------+    +----------+    +----------+
  |  Author  |    |  Review  |    |  Build   |    |  Publish |
  |  writes  |--->|  via     |--->|  and     |--->|  to      |
  |  docs in |    |  same    |    |  validate|    |  internal |
  |  plain   |    |  review  |    |  links,  |    |  site    |
  |  text    |    |  process |    |  format  |    |          |
  +----------+    +----------+    +----------+    +----------+
       |                               |
       |   Same repository as code     |
       +-------------------------------+
```

### Benefits of Docs-as-Code

1. **Version history**: See when and why documentation changed.
2. **Review process**: Catch errors and improve clarity before publishing.
3. **Co-location**: Documentation lives next to the code it describes.
4. **Automation**: Broken links, formatting errors, and stale references can
   be caught automatically.
5. **Single source of truth**: No divergence between a wiki and a repository.

## Keeping Docs Current

Stale documentation is worse than no documentation because it misleads.

### The Docs-Near-Code Principle

Place documentation as close to the code it describes as possible.

```
DOCS-NEAR-CODE PLACEMENT

  project/
  +-- orders/
  |   +-- process_order.pseudo      <-- Code
  |   +-- process_order_test.pseudo <-- Tests
  |   +-- README.md                 <-- Module docs (HERE)
  |
  +-- payments/
  |   +-- charge_payment.pseudo
  |   +-- charge_payment_test.pseudo
  |   +-- README.md                 <-- Module docs (HERE)
  |
  +-- docs/
  |   +-- architecture/
  |   |   +-- ADR-001-queue-choice.md
  |   |   +-- ADR-002-auth-strategy.md
  |   +-- getting-started.md
  |   +-- how-to/
  |       +-- add-payment-provider.md
  |
  +-- README.md                     <-- Project-level docs
```

### Strategies for Freshness

1. **Review docs with code**: When code changes, require documentation updates
   in the same change request.
2. **Ownership**: Assign each document an owner responsible for its accuracy.
3. **Expiry dates**: Mark documents with a "review by" date. Stale documents
   get flagged automatically.
4. **Link checking**: Automate detection of broken internal and external links.
5. **Delete aggressively**: Removing outdated docs is better than leaving them
   to mislead future readers.

## Writing for Different Audiences

```
AUDIENCE MAPPING

  Audience               Needs                    Document Types
  +-------------------+------------------------+--------------------+
  | New team members  | Orientation, setup,    | Tutorials,         |
  |                   | "how do I get started" | getting started    |
  +-------------------+------------------------+--------------------+
  | Active developers | Answers to specific    | How-to guides,     |
  |                   | problems, lookup info  | reference docs     |
  +-------------------+------------------------+--------------------+
  | Future            | Why decisions were     | ADRs, explanation  |
  | maintainers       | made, system context   | documents          |
  +-------------------+------------------------+--------------------+
  | External users    | How to use the API,    | API reference,     |
  | or consumers      | integration guides     | tutorials          |
  +-------------------+------------------------+--------------------+
  | Stakeholders      | System capabilities,   | High-level         |
  |                   | status, roadmap        | overviews          |
  +-------------------+------------------------+--------------------+
```

### Adapting Your Writing

- **For beginners**: Explain every step. Do not assume prior knowledge. Prefer
  concrete examples over abstract descriptions.
- **For experts**: Be concise. Provide lookup tables. Skip the basics.
- **For decision makers**: Lead with the conclusion. Provide evidence. Quantify
  impact.

## Documentation Anti-Patterns

```
DOCUMENTATION ANTI-PATTERNS

  +---------------------------+----------------------------------------+
  | Anti-Pattern              | Description                            |
  +---------------------------+----------------------------------------+
  | Write-only docs           | Written once, never updated. Decays    |
  |                           | immediately.                           |
  +---------------------------+----------------------------------------+
  | Scattered knowledge       | Critical info spread across wikis,     |
  |                           | chat messages, emails, and code        |
  |                           | comments with no index.                |
  +---------------------------+----------------------------------------+
  | Screenshot-heavy docs     | Screenshots break when the UI changes. |
  |                           | Use text-based descriptions where      |
  |                           | possible.                              |
  +---------------------------+----------------------------------------+
  | "Self-documenting code"   | Using this phrase to justify writing    |
  |                           | zero documentation. Code shows *what*, |
  |                           | not *why*.                             |
  +---------------------------+----------------------------------------+
  | Documentation graveyard   | A docs folder full of files that no    |
  |                           | one reads or maintains.                |
  +---------------------------+----------------------------------------+
  | Duplicated truth          | The same information in two places,    |
  |                           | inevitably diverging.                  |
  +---------------------------+----------------------------------------+
```

## Documentation Quality Checklist

```
FUNCTION EvaluateDocumentation(document)
    SET score TO 0
    SET max_score TO 8

    IF document.has_clear_purpose THEN
        SET score TO score + 1    // Reader knows what this doc is for
    END IF

    IF document.identifies_audience THEN
        SET score TO score + 1    // Written for a specific reader
    END IF

    IF document.has_working_examples THEN
        SET score TO score + 1    // Examples are correct and runnable
    END IF

    IF document.is_findable THEN
        SET score TO score + 1    // Linked from relevant locations
    END IF

    IF document.last_reviewed_within_6_months THEN
        SET score TO score + 1    // Actively maintained
    END IF

    IF document.matches_current_code THEN
        SET score TO score + 1    // Consistent with implementation
    END IF

    IF document.has_no_broken_links THEN
        SET score TO score + 1    // All links resolve
    END IF

    IF document.follows_diataxis_type THEN
        SET score TO score + 1    // Serves one clear purpose
    END IF

    RETURN {
        "score": score,
        "max": max_score,
        "percentage": (score / max_score) * 100
    }
END FUNCTION
```

## Summary

```
DOCUMENTATION PRINCIPLES

  +---------------------------------------------------------+
  |                                                         |
  |  1. Separate tutorials, how-tos, reference, and         |
  |     explanation into distinct documents                 |
  |                                                         |
  |  2. Place docs near the code they describe              |
  |                                                         |
  |  3. Treat docs as code: review, version, test           |
  |                                                         |
  |  4. Write for your audience, not for yourself           |
  |                                                         |
  |  5. Delete stale docs rather than leaving them to       |
  |     mislead                                             |
  |                                                         |
  |  6. Record architectural decisions in ADRs              |
  |                                                         |
  |  7. A document nobody can find is a document that       |
  |     does not exist                                      |
  |                                                         |
  +---------------------------------------------------------+
```

---

## Practice Problems

1. **Diataxis classification**: Classify each of the following documents into
   the Diataxis quadrant (tutorial, how-to, reference, explanation):
   a. "How to configure single sign-on for your organization"
   b. "Understanding our event-driven architecture"
   c. "Build your first data pipeline in 30 minutes"
   d. "Configuration options: complete list with defaults"

2. **README critique**: Write a README for a hypothetical project called
   "TaskQueue" that processes background jobs. Include all sections from the
   anatomy described above. Keep each section to 2-3 sentences.

3. **ADR writing**: Write an ADR for the following decision: "We chose to use
   a message queue for communication between services instead of direct
   synchronous calls." Include context, the decision, two alternatives
   considered, and consequences.

4. **Audience adaptation**: Take the following technical statement and rewrite
   it for three audiences: (a) a new team member, (b) an experienced developer,
   and (c) a non-technical stakeholder.
   Statement: "The system uses eventual consistency with a 5-second propagation
   window for read replicas."

5. **Documentation audit**: Given the following project structure, identify what
   documentation is missing and where it should be placed:
   ```
   project/
   +-- auth/
   |   +-- login.pseudo
   |   +-- register.pseudo
   |   +-- password_reset.pseudo
   +-- billing/
   |   +-- invoice.pseudo
   |   +-- payment.pseudo
   +-- README.md (last updated 18 months ago)
   ```
