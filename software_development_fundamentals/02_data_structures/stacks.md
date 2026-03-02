# Stacks

## Overview

A stack is a linear data structure that follows the **Last-In, First-Out (LIFO)**
principle. The last element added to the stack is the first one to be removed, much
like a stack of plates where you can only add or remove from the top. This simple
constraint makes stacks extraordinarily useful for managing nested contexts,
reversing sequences, and tracking state in recursive algorithms.

Stacks support two primary operations: **push** (add an element to the top) and
**pop** (remove the element from the top). A **peek** (or top) operation returns the
top element without removing it. All three operations run in O(1) time regardless of
implementation.

Stacks can be implemented using either an array or a linked list as the backing
store. The array-based approach offers better cache performance and lower memory
overhead. The linked-list approach avoids capacity limits and resizing costs. In
practice, the array-based implementation is more common due to its simplicity and
efficiency.

## Key Concepts

### LIFO Ordering

LIFO means that the most recently pushed element is always the first to be popped.
This property naturally models any situation where you need to "undo" the most recent
action or "return" to the most recent context.

### Stack Overflow and Underflow

A **stack overflow** occurs when attempting to push onto a full stack (relevant for
fixed-capacity array-based stacks). A **stack underflow** occurs when attempting to
pop from an empty stack. Both are error conditions that must be handled.

### Call Stack

Virtually every program execution environment uses a call stack to manage function
invocations. When a function is called, a new **stack frame** containing local
variables, parameters, and the return address is pushed. When the function returns,
its frame is popped. This is why deeply recursive functions can cause a stack
overflow.

### Two Implementation Strategies

**Array-based**: Use a dynamic array where push appends to the end and pop removes
from the end. The top of the stack is the last occupied index.

**Linked-list-based**: Use a singly linked list where push inserts at the head and
pop removes from the head. The top of the stack is the head node.

## Visual Representation

### Stack Operations

```
PUSH A:     PUSH B:     PUSH C:     POP:        POP:
+---+       +---+       +---+       +---+       +---+
|   |       |   |       | C | <-top |   |       |   |
+---+       +---+       +---+       +---+       +---+
|   |       | B | <-top | B |       | B | <-top |   |
+---+       +---+       +---+       +---+       +---+
| A | <-top | A |       | A |       | A |       | A | <-top
+---+       +---+       +---+       +---+       +---+

Returned: -   -   -   C   B
```

### Array-Based Stack

```
Backing array:
Index:  0     1     2     3     4     5     6     7
      +-----+-----+-----+-----+-----+-----+-----+-----+
      | 42  | 17  | 93  |     |     |     |     |     |
      +-----+-----+-----+-----+-----+-----+-----+-----+
                     ^
                    top (index = 2)
                    count = 3, capacity = 8
```

### Linked-List-Based Stack

```
  top (head)
    |
    v
+------+---+     +------+---+     +------+------+
|  93  | o-+---->|  17  | o-+---->|  42  | NULL |
+------+---+     +------+---+     +------+------+

Push adds at head, Pop removes from head.
```

### Call Stack Visualization

```
FUNCTION main() calls foo(), foo() calls bar():

+-------------------+
|   bar() frame     |  <-- current execution
|   local vars...   |
+-------------------+
|   foo() frame     |
|   local vars...   |
+-------------------+
|   main() frame    |
|   local vars...   |
+-------------------+

When bar() returns, its frame is popped.
When foo() returns, its frame is popped.
```

## Operations and Pseudocode

### Array-Based Stack

```
STRUCTURE ArrayStack:
    data: Array
    top: Integer        // index of top element (-1 when empty)
    capacity: Integer
END STRUCTURE

FUNCTION createStack(capacity: Integer) -> ArrayStack:
    SET stack = NEW ArrayStack
    SET stack.data = ALLOCATE Array of size capacity
    SET stack.top = -1
    SET stack.capacity = capacity
    RETURN stack
END FUNCTION

FUNCTION push(stack: ArrayStack, value: Element) -> Void:
    IF stack.top == stack.capacity - 1 THEN
        // Resize: double the capacity
        SET newData = ALLOCATE Array of size stack.capacity * 2
        FOR i FROM 0 TO stack.top DO
            SET newData[i] = stack.data[i]
        END FOR
        FREE stack.data
        SET stack.data = newData
        SET stack.capacity = stack.capacity * 2
    END IF
    SET stack.top = stack.top + 1
    SET stack.data[stack.top] = value
END FUNCTION

FUNCTION pop(stack: ArrayStack) -> Element:
    IF stack.top == -1 THEN
        ERROR "Stack underflow"
    END IF
    SET value = stack.data[stack.top]
    SET stack.top = stack.top - 1
    RETURN value
END FUNCTION

FUNCTION peek(stack: ArrayStack) -> Element:
    IF stack.top == -1 THEN
        ERROR "Stack is empty"
    END IF
    RETURN stack.data[stack.top]
END FUNCTION

FUNCTION isEmpty(stack: ArrayStack) -> Boolean:
    RETURN stack.top == -1
END FUNCTION

FUNCTION size(stack: ArrayStack) -> Integer:
    RETURN stack.top + 1
END FUNCTION
```

### Linked-List-Based Stack

```
FUNCTION push(stack: LinkedStack, value: Element) -> Void:
    SET newNode = CALL createNode(value)
    SET newNode.next = stack.top
    SET stack.top = newNode
    SET stack.size = stack.size + 1
END FUNCTION

FUNCTION pop(stack: LinkedStack) -> Element:
    IF stack.top == NULL THEN
        ERROR "Stack underflow"
    END IF
    SET value = stack.top.data
    SET oldTop = stack.top
    SET stack.top = stack.top.next
    FREE oldTop
    SET stack.size = stack.size - 1
    RETURN value
END FUNCTION

FUNCTION peek(stack: LinkedStack) -> Element:
    IF stack.top == NULL THEN
        ERROR "Stack is empty"
    END IF
    RETURN stack.top.data
END FUNCTION
```

## Expression Evaluation Walkthrough

### Infix to Postfix Conversion (Shunting-Yard Algorithm)

Convert `3 + 4 * 2 - ( 1 + 5 )` to postfix: `3 4 2 * + 1 5 + -`

```
FUNCTION infixToPostfix(tokens: List) -> List:
    SET output = EMPTY List
    SET operators = EMPTY Stack

    FOR EACH token IN tokens DO
        IF token is a number THEN
            APPEND token TO output
        ELSE IF token is an operator THEN
            WHILE operators is not empty
              AND peek(operators) is not "("
              AND precedence(peek(operators)) >= precedence(token) DO
                APPEND pop(operators) TO output
            END WHILE
            CALL push(operators, token)
        ELSE IF token == "(" THEN
            CALL push(operators, token)
        ELSE IF token == ")" THEN
            WHILE peek(operators) != "(" DO
                APPEND pop(operators) TO output
            END WHILE
            CALL pop(operators)   // discard the "("
        END IF
    END FOR

    WHILE operators is not empty DO
        APPEND pop(operators) TO output
    END WHILE

    RETURN output
END FUNCTION
```

#### Step-by-Step Trace

```
Token  | Output Queue       | Operator Stack  | Action
-------+--------------------+-----------------+------------------------
  3    | 3                  |                 | Number -> output
  +    | 3                  | +               | Push operator
  4    | 3 4                | +               | Number -> output
  *    | 3 4                | + *             | * > +, push
  2    | 3 4 2              | + *             | Number -> output
  -    | 3 4 2 * +          | -               | Pop *(>=), pop+(>=), push -
  (    | 3 4 2 * +          | - (             | Push (
  1    | 3 4 2 * + 1        | - (             | Number -> output
  +    | 3 4 2 * + 1        | - ( +           | Push + (after ()
  5    | 3 4 2 * + 1 5      | - ( +           | Number -> output
  )    | 3 4 2 * + 1 5 +    | -               | Pop until (, discard (
 END   | 3 4 2 * + 1 5 + -  |                 | Pop remaining
```

### Postfix Evaluation

```
FUNCTION evaluatePostfix(tokens: List) -> Number:
    SET operands = EMPTY Stack

    FOR EACH token IN tokens DO
        IF token is a number THEN
            CALL push(operands, token)
        ELSE
            SET right = CALL pop(operands)
            SET left = CALL pop(operands)
            SET result = APPLY token TO (left, right)
            CALL push(operands, result)
        END IF
    END FOR

    RETURN pop(operands)
END FUNCTION
```

#### Trace: Evaluate `3 4 2 * + 1 5 + -`

```
Token | Stack          | Action
------+----------------+----------------------------------
  3   | [3]            | Push 3
  4   | [3, 4]         | Push 4
  2   | [3, 4, 2]      | Push 2
  *   | [3, 8]         | Pop 2, 4; push 4*2=8
  +   | [11]           | Pop 8, 3; push 3+8=11
  1   | [11, 1]        | Push 1
  5   | [11, 1, 5]     | Push 5
  +   | [11, 6]        | Pop 5, 1; push 1+5=6
  -   | [5]            | Pop 6, 11; push 11-6=5

Result: 5
```

### Balanced Parentheses Checker

```
FUNCTION isBalanced(expression: String) -> Boolean:
    SET stack = EMPTY Stack
    SET pairs = { ")" : "(", "]" : "[", "}" : "{" }

    FOR EACH char IN expression DO
        IF char IN ["(", "[", "{"] THEN
            CALL push(stack, char)
        ELSE IF char IN [")", "]", "}"] THEN
            IF isEmpty(stack) THEN
                RETURN FALSE
            END IF
            IF pop(stack) != pairs[char] THEN
                RETURN FALSE
            END IF
        END IF
    END FOR

    RETURN isEmpty(stack)
END FUNCTION
```

## Complexity Analysis

| Operation       | Array-Based | Linked-List-Based | Notes                     |
|-----------------|-------------|-------------------|---------------------------|
| Push            | O(1)*       | O(1)              | *Amortized with resizing  |
| Pop             | O(1)        | O(1)              |                           |
| Peek / Top      | O(1)        | O(1)              |                           |
| isEmpty         | O(1)        | O(1)              |                           |
| Size            | O(1)        | O(1)              | If count is maintained    |
| Search          | O(n)        | O(n)              | Must pop to search        |
| Space           | O(n)        | O(n)              | Array may have slack      |

| Metric              | Array-Based          | Linked-List-Based     |
|----------------------|----------------------|-----------------------|
| Memory per element   | Element only         | Element + 1 pointer   |
| Cache performance    | Excellent            | Poor                  |
| Max capacity         | Fixed or resizable   | Unlimited (memory)    |
| Allocation overhead  | One block            | Per-node allocation   |

## When to Use / When NOT to Use

### Use Stacks When:
- You need LIFO access (most recent first).
- Parsing nested structures (parentheses, HTML tags, expressions).
- Implementing undo/redo functionality.
- Converting between expression notations (infix, postfix, prefix).
- Simulating recursion iteratively (DFS, backtracking).
- Managing function call contexts.

### Avoid Stacks When:
- You need FIFO access (use a queue).
- You need random access to arbitrary elements (use an array).
- You need to search or iterate frequently (the stack abstraction discourages this).
- You need both ends accessible (use a deque).

## Real-World Applications

- **Web browser back button**: Each visited page is pushed; pressing back pops.
- **Text editor undo**: Each action is pushed; undo pops the most recent action.
- **Compiler expression parsing**: Shunting-yard algorithm, syntax tree construction.
- **Depth-first search**: An explicit stack replaces recursion for DFS traversal.
- **Backtracking algorithms**: Maze solving, N-Queens, Sudoku solvers.
- **Memory management**: Call stacks track function invocations and local variables.
- **Syntax validation**: Matching brackets, parentheses, and tags.

## Common Pitfalls and Best Practices

1. **Stack underflow**: Always check `isEmpty()` before calling `pop()` or `peek()`.
2. **Unbounded growth**: If using an array stack without resizing, a burst of pushes
   causes overflow. Either resize dynamically or use a linked-list stack.
3. **Not clearing references**: After popping from an array-based stack, null out the
   vacated slot to prevent holding references to objects that should be garbage
   collected.
4. **Using a stack when a queue is needed**: LIFO vs FIFO confusion is a common
   source of bugs in BFS/DFS implementations.
5. **Deep recursion vs explicit stack**: Recursive algorithms with deep call trees
   can overflow the system call stack. Convert to an explicit stack when depth is
   unpredictable.
6. **Operator precedence errors**: When implementing expression parsers, incorrect
   precedence tables lead to wrong evaluation order. Test thoroughly with edge cases.

## Comparison with Related Structures

| Feature              | Stack      | Queue      | Deque      | Priority Queue |
|----------------------|------------|------------|------------|----------------|
| Ordering             | LIFO       | FIFO       | Both ends  | By priority    |
| Insert               | Top only   | Rear only  | Both ends  | Anywhere       |
| Remove               | Top only   | Front only | Both ends  | Min/Max        |
| Peek                 | Top        | Front      | Both ends  | Min/Max        |
| Random access        | No         | No         | No*        | No             |
| Implementation       | Array/List | Array/List | Array/List | Heap           |

\* Some deque implementations support O(1) indexed access.

## Practice Problems

1. **Min Stack**: Design a stack that supports push, pop, peek, and retrieving the
   minimum element, all in O(1) time. Hint: use an auxiliary stack.

2. **Valid Parentheses**: Given a string containing `()[]{}`, determine if the
   input is valid (every open bracket has a matching close bracket in correct order).

3. **Daily Temperatures**: Given an array of daily temperatures, for each day find
   how many days until a warmer temperature. Use a monotonic stack for O(n).

4. **Implement Queue Using Two Stacks**: Implement a FIFO queue using only two
   stacks. Analyze the amortized complexity.

5. **Largest Rectangle in Histogram**: Given an array of bar heights, find the
   largest rectangle that fits under the histogram. Use a stack-based O(n) approach.

6. **Iterative DFS**: Implement depth-first search for a graph using an explicit
   stack instead of recursion. Handle cycle detection.
