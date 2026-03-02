# Bit Manipulation

## Overview

At their core, computers operate on bits -- binary digits that are either 0 or 1.
Bit manipulation is the practice of using bitwise operators to read, set, clear, and
transform individual bits or groups of bits within a value.

Mastering bit manipulation unlocks algorithms that are remarkably fast and memory-
efficient. A single bitwise operation can often replace loops or conditionals. Bit
tricks appear throughout systems programming, cryptography, graphics, and network
protocols.

## Binary Number Representation

### Unsigned Integers

An unsigned integer with W bits represents values from 0 to 2^W - 1.

```
8-bit unsigned:       Position values (right to left):
  0  = 00000000         128  64  32  16   8   4   2   1
  42 = 00101010          2^7 2^6 2^5 2^4 2^3 2^2 2^1 2^0
 255 = 11111111
                       42 = 32 + 8 + 2 = 00101010
```

### Signed Integers (Two's Complement)

The MSB is the sign bit: 0 = positive, 1 = negative. To negate: invert all bits, add 1.

```
8-bit two's complement:                Range for W bits:
   1 = 00000001    -1  = 11111111        [-2^(W-1), 2^(W-1) - 1]
 127 = 01111111    -42 = 11010110        8-bit:  [-128, 127]
-128 = 10000000                          32-bit: [-2147483648, 2147483647]
```

## Bitwise Operators

### Truth Tables

```
AND (&)       OR (|)        XOR (^)       NOT (~)
A B | A&B     A B | A|B     A B | A^B     A | ~A
0 0 |  0      0 0 |  0      0 0 |  0      0 |  1
0 1 |  0      0 1 |  1      0 1 |  1      1 |  0
1 0 |  0      1 0 |  1      1 0 |  1
1 1 |  1      1 1 |  1      1 1 |  0
```

### Shift Operations

```
Left Shift (<<):   00001101 << 2 = 00110100   (13 * 4 = 52)
Right Shift (>>):  00110100 >> 2 = 00001101   (52 / 4 = 13)
Arithmetic >>:     11010110 >> 2 = 11110101   (sign bit preserved)

+----------+--------+---------------------------------------------+
| Operator | Symbol | Purpose                                     |
+----------+--------+---------------------------------------------+
| AND      |   &    | Mask bits, clear bits, test bits             |
| OR       |   |    | Set bits, combine flags                     |
| XOR      |   ^    | Toggle bits, swap values, find differences  |
| NOT      |   ~    | Invert all bits                             |
| L-Shift  |  <<    | Multiply by powers of 2, create masks       |
| R-Shift  |  >>    | Divide by powers of 2, extract fields       |
+----------+--------+---------------------------------------------+
```

## Common Bit Tricks

### Check Odd/Even

```
FUNCTION is_odd(n: Integer) -> Boolean:
    RETURN (n AND 1) = 1
END FUNCTION
                                     5 & 1 = 00000101 & 00000001 = 1 (odd)
                                     6 & 1 = 00000110 & 00000001 = 0 (even)
```

### Swap Without Temporary Variable

```
FUNCTION swap(a: Integer, b: Integer) -> (Integer, Integer):
    SET a = a XOR b
    SET b = a XOR b
    SET a = a XOR b
    RETURN (a, b)
END FUNCTION

Trace (a=5, b=3):  a = 0101 ^ 0011 = 0110
                    b = 0110 ^ 0011 = 0101  (original a)
                    a = 0110 ^ 0101 = 0011  (original b)
```

### Check Power of 2

```
FUNCTION is_power_of_two(n: Integer) -> Boolean:
    RETURN n > 0 AND (n AND (n - 1)) = 0
END FUNCTION

  8:  1000 & 0111 = 0000 -> TRUE       6:  0110 & 0101 = 0100 -> FALSE
```

### Count Set Bits (Brian Kernighan's Algorithm)

```
FUNCTION count_set_bits(n: Integer) -> Integer:
    SET count = 0
    WHILE n != 0 DO
        SET n = n AND (n - 1)
        SET count = count + 1
    END WHILE
    RETURN count
END FUNCTION

n=13 (1101): 1101 & 1100=1100, 1100 & 1011=1000, 1000 & 0111=0000 -> 3 bits
```

### Toggle, Set, Clear, and Check the Nth Bit

```
FUNCTION toggle_bit(value: Integer, n: Integer) -> Integer:
    RETURN value XOR (1 << n)
END FUNCTION

FUNCTION set_bit(value: Integer, n: Integer) -> Integer:
    RETURN value OR (1 << n)
END FUNCTION

FUNCTION clear_bit(value: Integer, n: Integer) -> Integer:
    RETURN value AND NOT(1 << n)
END FUNCTION

FUNCTION check_bit(value: Integer, n: Integer) -> Boolean:
    RETURN (value AND (1 << n)) != 0
END FUNCTION
```

### Clear and Isolate Lowest Set Bit

```
FUNCTION clear_lowest_set_bit(n: Integer) -> Integer:
    RETURN n AND (n - 1)
END FUNCTION

FUNCTION isolate_lowest_set_bit(n: Integer) -> Integer:
    RETURN n AND (-n)
END FUNCTION

  n = 10110100       clear lowest:  10110100 & 10110011 = 10110000
                     isolate lowest: 10110100 & 01001100 = 00000100
```

## Bit Masking Patterns

### Extract and Set a Field

```
FUNCTION extract_field(value: Integer, position: Integer, width: Integer) -> Integer:
    RETURN (value >> position) AND ((1 << width) - 1)
END FUNCTION

FUNCTION set_field(value: Integer, position: Integer, width: Integer, field: Integer) -> Integer:
    SET mask = ((1 << width) - 1) << position
    RETURN (value AND NOT(mask)) OR (field << position)
END FUNCTION

Extract 3 bits at position 2 from 10110100:
  76543210          >> 2 = 00101101, mask = 00000111, result = 00000101 (5)
       ^^^  bits 2-4 = 101
```

### Permission Flags

```
CONSTANT READ = 4, WRITE = 2, EXECUTE = 1     // 100, 010, 001

SET perms = READ OR WRITE                       // 110
IF (perms AND WRITE) != 0 THEN ...             // check write
SET perms = perms OR EXECUTE                    // 111 (add execute)
SET perms = perms AND NOT(WRITE)                // 101 (remove write)

+-------+------+-------+---------+
| Value | Read | Write | Execute |
+-------+------+-------+---------+
|   0   |  No  |  No   |   No    |
|   1   |  No  |  No   |   Yes   |
|   2   |  No  |  Yes  |   No    |
|   3   |  No  |  Yes  |   Yes   |
|   4   |  Yes |  No   |   No    |
|   5   |  Yes |  No   |   Yes   |
|   6   |  Yes |  Yes  |   No    |
|   7   |  Yes |  Yes  |   Yes   |
+-------+------+-------+---------+
```

## XOR Properties and Applications

```
a XOR 0 = a           (identity)         a XOR a = 0     (self-inverse)
a XOR b = b XOR a     (commutative)      associative:  (a^b)^c = a^(b^c)
```

### Find the Unique Element

Given a list where every element appears twice except one, XOR all elements together.

```
FUNCTION find_unique(items: List of Integer) -> Integer:
    SET result = 0
    FOR EACH item IN items DO
        SET result = result XOR item
    END FOR
    RETURN result
END FUNCTION

[4, 1, 2, 1, 2]:  0^4=4, 4^1=5, 5^2=7, 7^1=6, 6^2=4 -> unique is 4
```

## Compact Storage

Bit fields pack multiple small values into a single integer.

```
Date packed into 21 bits: [Year: 12 bits | Month: 4 bits | Day: 5 bits]

FUNCTION pack_date(year: Integer, month: Integer, day: Integer) -> Integer:
    RETURN (year << 9) OR (month << 5) OR day
END FUNCTION

FUNCTION unpack_year(packed: Integer) -> Integer:
    RETURN (packed >> 9) AND ((1 << 12) - 1)
END FUNCTION

FUNCTION unpack_month(packed: Integer) -> Integer:
    RETURN (packed >> 5) AND ((1 << 4) - 1)
END FUNCTION

FUNCTION unpack_day(packed: Integer) -> Integer:
    RETURN packed AND ((1 << 5) - 1)
END FUNCTION
```

## Practice Problems

1. **Reverse Bits:** Reverse the bit order of an unsigned integer in O(log W) time.

2. **Single Number II:** Every element appears three times except one that appears
   once. Find it using bitwise counting modulo 3 (XOR alone is insufficient).

3. **Bitwise Addition:** Add two integers using only AND, XOR, and shift -- no
   arithmetic addition. Hint: XOR = sum without carry, AND << 1 = carry.

4. **Maximum XOR Pair:** Given a list of integers, find the pair with maximum XOR.
   Solve in O(n * W) using a trie of bit prefixes instead of O(n^2) brute force.

5. **Power Set via Bitmask:** Generate all 2^N subsets of N elements using integers
   0 to 2^N - 1 where bit i indicates element i's inclusion.

6. **Hamming Distance:** Compute the number of differing bit positions between two
   integers. Then find the total Hamming distance over all pairs of N integers in
   O(N * W) instead of O(N^2).
