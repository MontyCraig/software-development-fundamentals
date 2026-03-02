# DRY - Don't Repeat Yourself

## Definition

"Every piece of knowledge must have a single, unambiguous, authoritative
representation within a system." -- Andy Hunt and Dave Thomas, The Pragmatic
Programmer (1999).

DRY is not merely about avoiding duplicate lines of code. It is about ensuring
that every concept, business rule, or piece of domain knowledge exists in
exactly one place. When a change is needed, you change it in one spot.

---

## Types of Duplication

### 1. Code Duplication

The most obvious form. Identical or near-identical blocks of code appear in
multiple locations.

**Before (duplicated validation):**

```
FUNCTION validateUserEmail(email: Text) -> Boolean:
    IF email IS EMPTY THEN RETURN false END IF
    IF email DOES NOT CONTAIN "@" THEN RETURN false END IF
    IF LENGTH(email) > 254 THEN RETURN false END IF
    RETURN true
END FUNCTION

FUNCTION validateContactEmail(email: Text) -> Boolean:
    IF email IS EMPTY THEN RETURN false END IF
    IF email DOES NOT CONTAIN "@" THEN RETURN false END IF
    IF LENGTH(email) > 254 THEN RETURN false END IF
    RETURN true
END FUNCTION
```

**After (single source of truth):**

```
FUNCTION validateEmail(email: Text) -> Boolean:
    IF email IS EMPTY THEN RETURN false END IF
    IF email DOES NOT CONTAIN "@" THEN RETURN false END IF
    IF LENGTH(email) > 254 THEN RETURN false END IF
    RETURN true
END FUNCTION
```

### 2. Logic Duplication

The code looks different, but the underlying logic is the same.

**Before (same business rule, different expressions):**

```
// In OrderModule:
FUNCTION isEligibleForFreeShipping(order: Order) -> Boolean:
    RETURN order.total >= 50 AND order.destination = "domestic"
END FUNCTION

// In CartModule:
FUNCTION showFreeShippingBanner(cart: Cart) -> Boolean:
    SET sum = 0
    FOR EACH item IN cart.items DO
        SET sum = sum + item.price * item.quantity
    END FOR
    RETURN sum >= 50 AND cart.shippingType <> "international"
END FUNCTION
```

Both encode the same business rule: "free shipping for domestic orders over 50."
If the threshold changes to 75, both must be updated -- and you will forget one.

**After:**

```
MODULE ShippingPolicy:
    CONSTANT FREE_SHIPPING_THRESHOLD = 50

    FUNCTION qualifiesForFreeShipping(total: Number, isDomestic: Boolean) -> Boolean:
        RETURN total >= FREE_SHIPPING_THRESHOLD AND isDomestic
    END FUNCTION
END MODULE
```

### 3. Knowledge Duplication

Domain knowledge is encoded in multiple places: code, configuration, database
schemas, documentation, or comments.

```
Danger Zone:

  [Code]        says: "Max password length is 128"
  [Database]    says: VARCHAR(128) for password column
  [Docs]        says: "Passwords can be up to 128 characters"
  [Validation]  says: IF LENGTH(pwd) > 128 THEN reject

  If the limit changes, ALL FOUR locations must be updated.
```

**Solution: Single source of truth**

```
MODULE SecurityConfig:
    CONSTANT MAX_PASSWORD_LENGTH = 128
END MODULE

// Database schema references SecurityConfig.MAX_PASSWORD_LENGTH
// Validation uses SecurityConfig.MAX_PASSWORD_LENGTH
// Documentation is auto-generated from SecurityConfig
```

### 4. Data Duplication

The same data stored or derived in multiple places.

**Before:**

```
MODULE Order:
    SET items: List<OrderItem>
    SET subtotal: Number          // Stored separately
    SET taxAmount: Number         // Stored separately
    SET total: Number             // Stored separately
END MODULE
```

If items change, subtotal, taxAmount, and total can become stale.

**After:**

```
MODULE Order:
    SET items: List<OrderItem>
    SET taxRate: Number

    FUNCTION subtotal() -> Number:
        SET sum = 0
        FOR EACH item IN items DO
            SET sum = sum + item.price * item.quantity
        END FOR
        RETURN sum
    END FUNCTION

    FUNCTION taxAmount() -> Number:
        RETURN subtotal() * taxRate
    END FUNCTION

    FUNCTION total() -> Number:
        RETURN subtotal() + taxAmount()
    END FUNCTION
END MODULE
```

Derived values are calculated, not stored, so they never go stale.

---

## Abstraction Techniques for Eliminating Duplication

```
+-------------------------------------------+
| Technique          | When to Use           |
+-------------------------------------------+
| Extract Function   | Repeated code blocks  |
| Extract Module     | Repeated across files |
| Parameterize       | Similar with small    |
|                    | variations            |
| Template Method    | Same skeleton,        |
|                    | different steps       |
| Configuration      | Repeated magic values |
| Code Generation    | Repeated structural   |
|                    | patterns              |
+-------------------------------------------+
```

### Extract and Parameterize Example

**Before (three near-identical functions):**

```
FUNCTION formatCurrency_USD(amount: Number) -> Text:
    RETURN "$" + ROUND(amount, 2)
END FUNCTION

FUNCTION formatCurrency_EUR(amount: Number) -> Text:
    RETURN ROUND(amount, 2) + " EUR"
END FUNCTION

FUNCTION formatCurrency_GBP(amount: Number) -> Text:
    RETURN "GBP " + ROUND(amount, 2)
END FUNCTION
```

**After (parameterized):**

```
MODULE CurrencyFormats:
    SET formats = {
        "USD": { prefix: "$",   suffix: "" },
        "EUR": { prefix: "",    suffix: " EUR" },
        "GBP": { prefix: "GBP ", suffix: "" }
    }
END MODULE

FUNCTION formatCurrency(amount: Number, currencyCode: Text) -> Text:
    SET fmt = CurrencyFormats.formats[currencyCode]
    RETURN fmt.prefix + ROUND(amount, 2) + fmt.suffix
END FUNCTION
```

---

## DRY vs WET (Write Everything Twice)

Premature abstraction can be worse than duplication. The WET principle suggests
that duplication is acceptable until you see the pattern three times.

```
Decision Framework:

  Is this duplicated?
       |
       YES
       |
  How many times? -----> Once: Keep it (coincidence, not pattern)
       |
       +--> Twice: Note it, but tolerate (WET is OK)
       |
       +--> Three or more: Abstract it (Rule of Three)
       |
  Is the duplication across the same concern?
       |
       YES --> Abstract immediately
       NO  --> May be coincidental; do not force a shared abstraction
```

### When DRY Hurts

```
ANTI-PATTERN: Forced Shared Abstraction

  Module A needs: validate, transform, log
  Module B needs: validate, transform, notify

  Tempting but wrong:
  [SharedProcessor] -- does validate + transform for both
                    -- now A and B are coupled through SharedProcessor
                    -- changes for A's validation break B

  Better:
  [ValidationRules] -- shared validation (truly same rules)
  [Module A]        -- own transform + log
  [Module B]        -- own transform + notify
```

### Signs of Over-DRYing

1. A shared function has many boolean parameters controlling behavior
2. A "utility" module is imported by every module in the system
3. Changing one feature breaks an unrelated feature through shared code
4. The abstraction is harder to understand than the duplication was

---

## Relationship to Other Principles

```
+---------------------------------------------------+
| Principle   | Relationship to DRY                  |
+---------------------------------------------------+
| SRP         | DRY helps enforce SRP by             |
|             | centralizing each responsibility     |
+---------------------------------------------------+
| OCP         | DRY abstractions become extension    |
|             | points for OCP                       |
+---------------------------------------------------+
| KISS        | Tension: DRY can add complexity;     |
|             | balance with simplicity              |
+---------------------------------------------------+
| YAGNI       | Do not DRY up code to prepare for    |
|             | hypothetical future duplication      |
+---------------------------------------------------+
```

---

## Practical Checklist

```
+----------------------------------------------------------+
| DRY Checklist                                            |
+----------------------------------------------------------+
| [ ] Business rules exist in exactly one module           |
| [ ] Constants are defined once, referenced everywhere    |
| [ ] Validation logic is not duplicated across layers     |
| [ ] Derived data is computed, not stored redundantly     |
| [ ] Configuration lives in one place                     |
| [ ] No copy-paste code blocks exist                      |
| [ ] Abstractions represent real shared concepts          |
| [ ] The Rule of Three is respected (not premature DRY)   |
+----------------------------------------------------------+
```

DRY is a principle about knowledge, not just text. Two identical lines of code
that represent different concepts should remain separate. Two different-looking
blocks that encode the same business rule must be unified. Always ask: "If this
knowledge changes, how many places must I edit?"
