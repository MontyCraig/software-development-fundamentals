# String Algorithms

## Overview

Strings are among the most commonly processed data structures in computing. Whether
searching for a word in a document, checking user input, or indexing a corpus of text,
string algorithms form the backbone of text processing.

This chapter covers pattern matching techniques ranging from brute-force to linear-time,
classic string operations, the trie data structure for prefix-based retrieval, edit
distance for measuring string similarity, and suffix arrays for advanced text indexing.

## Key Concepts

| Term            | Meaning                                                        |
|-----------------|----------------------------------------------------------------|
| Text (T)        | The string being searched within, length n                     |
| Pattern (P)     | The string being searched for, length m                        |
| Substring       | A contiguous sequence of characters within a string            |
| Prefix          | A substring that starts at position 0                          |
| Suffix          | A substring that ends at the last position                     |
| Alphabet        | The set of all possible characters                             |

```
String:    B A N A N A        Prefixes:  B, BA, BAN, BANA, BANAN, BANANA
           | | | | | |        Suffixes:  BANANA, ANANA, NANA, ANA, NA, A
Indices:   0 1 2 3 4 5
```

## Pattern Matching Algorithms

### 1. Naive (Brute-Force) Pattern Matching

Slide the pattern across the text one position at a time, comparing characters at
each alignment.

```
FUNCTION naive_search(text: Text, pattern: Text) -> List of Integer:
    SET n = LENGTH(text)
    SET m = LENGTH(pattern)
    SET matches = EMPTY LIST
    FOR i = 0 TO n - m DO
        SET j = 0
        WHILE j < m AND text[i + j] = pattern[j] DO
            SET j = j + 1
        END WHILE
        IF j = m THEN
            APPEND i TO matches
        END IF
    END FOR
    RETURN matches
END FUNCTION
```

**Trace:** Search for "ABA" in "ABABABA"

```
Text:    A B A B A B A       i=0: ABA matches -> FOUND at 0
         0 1 2 3 4 5 6       i=1: B != A -> skip
                              i=2: ABA matches -> FOUND at 2
                              i=3: B != A -> skip
                              i=4: ABA matches -> FOUND at 4
                              Result: [0, 2, 4]
```

**Complexity:** O(n * m) worst case, O(n) average case for random text.

### 2. KMP (Knuth-Morris-Pratt) Algorithm

KMP precomputes a failure function: when a mismatch occurs at position j, it gives
the longest proper prefix of pattern[0..j-1] that is also a suffix, allowing us to
skip ahead instead of restarting.

**Step 1: Build the Failure Function**

```
FUNCTION build_failure_function(pattern: Text) -> Array of Integer:
    SET m = LENGTH(pattern)
    SET failure = NEW Array[m] FILLED WITH 0
    SET length = 0
    SET i = 1
    WHILE i < m DO
        IF pattern[i] = pattern[length] THEN
            SET length = length + 1
            SET failure[i] = length
            SET i = i + 1
        ELSE
            IF length > 0 THEN
                SET length = failure[length - 1]
            ELSE
                SET failure[i] = 0
                SET i = i + 1
            END IF
        END IF
    END WHILE
    RETURN failure
END FUNCTION
```

**Failure function trace for "ABABAC":**

```
Pattern:  A  B  A  B  A  C         Failure: [0, 0, 1, 2, 3, 0]
Index:    0  1  2  3  4  5

i=1: B != A           -> failure[1]=0
i=2: A = A            -> failure[2]=1
i=3: B = B            -> failure[3]=2
i=4: A = A            -> failure[4]=3
i=5: C != B, len=f[1]=0, C != A -> failure[5]=0
```

**Step 2: Search Using the Failure Function**

```
FUNCTION kmp_search(text: Text, pattern: Text) -> List of Integer:
    SET n = LENGTH(text)
    SET m = LENGTH(pattern)
    SET failure = build_failure_function(pattern)
    SET matches = EMPTY LIST
    SET j = 0
    FOR i = 0 TO n - 1 DO
        WHILE j > 0 AND text[i] != pattern[j] DO
            SET j = failure[j - 1]
        END WHILE
        IF text[i] = pattern[j] THEN
            SET j = j + 1
        END IF
        IF j = m THEN
            APPEND (i - m + 1) TO matches
            SET j = failure[j - 1]
        END IF
    END FOR
    RETURN matches
END FUNCTION
```

**Complexity:** O(n + m) time, O(m) space.

### 3. Rabin-Karp Algorithm

Rabin-Karp computes a rolling hash of each m-length window in the text. Only when
hashes match does it verify character-by-character.

```
FUNCTION rabin_karp_search(text: Text, pattern: Text, base: Integer, modulus: Integer) -> List of Integer:
    SET n = LENGTH(text)
    SET m = LENGTH(pattern)
    SET matches = EMPTY LIST
    SET pattern_hash = 0
    SET window_hash = 0
    SET highest_power = 1
    FOR i = 0 TO m - 2 DO
        SET highest_power = (highest_power * base) MOD modulus
    END FOR
    FOR i = 0 TO m - 1 DO
        SET pattern_hash = (pattern_hash * base + CHAR_CODE(pattern[i])) MOD modulus
        SET window_hash = (window_hash * base + CHAR_CODE(text[i])) MOD modulus
    END FOR
    FOR i = 0 TO n - m DO
        IF window_hash = pattern_hash THEN
            IF SUBSTRING(text, i, m) = pattern THEN
                APPEND i TO matches
            END IF
        END IF
        IF i < n - m THEN
            SET window_hash = ((window_hash - CHAR_CODE(text[i]) * highest_power) * base
                              + CHAR_CODE(text[i + m])) MOD modulus
            IF window_hash < 0 THEN
                SET window_hash = window_hash + modulus
            END IF
        END IF
    END FOR
    RETURN matches
END FUNCTION
```

```
Rolling hash (base=10, window=3):   "314159"
  Hash("314") = 3*100 + 1*10 + 4 = 314
  Slide right:  (314 - 3*100)*10 + 1 = 141    ("141")
  Slide right:  (141 - 1*100)*10 + 5 = 415    ("415")
```

**Complexity:** O(n + m) expected, O(n * m) worst case (many collisions).

## String Operations

### Reverse, Palindrome Check, and Anagram Detection

```
FUNCTION reverse_string(s: Text) -> Text:
    SET left = 0
    SET right = LENGTH(s) - 1
    SET chars = CHARACTERS(s)
    WHILE left < right DO
        SWAP chars[left] WITH chars[right]
        SET left = left + 1
        SET right = right - 1
    END WHILE
    RETURN JOIN(chars)
END FUNCTION

FUNCTION is_palindrome(s: Text) -> Boolean:
    SET left = 0
    SET right = LENGTH(s) - 1
    WHILE left < right DO
        IF s[left] != s[right] THEN
            RETURN FALSE
        END IF
        SET left = left + 1
        SET right = right - 1
    END WHILE
    RETURN TRUE
END FUNCTION

FUNCTION are_anagrams(a: Text, b: Text) -> Boolean:
    IF LENGTH(a) != LENGTH(b) THEN
        RETURN FALSE
    END IF
    SET counts = EMPTY MAP
    FOR EACH char IN CHARACTERS(a) DO
        SET counts[char] = GET(counts, char, 0) + 1
    END FOR
    FOR EACH char IN CHARACTERS(b) DO
        SET counts[char] = GET(counts, char, 0) - 1
        IF counts[char] < 0 THEN
            RETURN FALSE
        END IF
    END FOR
    RETURN TRUE
END FUNCTION
```

## Trie (Prefix Tree)

A trie stores strings character by character in a tree. Paths from root to marked
nodes spell stored strings. Tries excel at prefix operations: autocomplete, spell
checking, and prefix counting.

```
                    (root)
                   /  |   \
                  A   B    C
                 / \  |    |
                N   P E    A
                |   |  \   |
                T   P   D  T
               (*)  |  (*)
                    L             Stored words: ANT, APPLE, BED, CAT
                    |             (*) = end-of-word marker
                    E
                   (*)
```

```
FUNCTION trie_insert(root: TrieNode, word: Text) -> Void:
    SET current = root
    FOR EACH char IN CHARACTERS(word) DO
        IF char NOT IN current.children THEN
            SET current.children[char] = NEW TrieNode
        END IF
        SET current = current.children[char]
    END FOR
    SET current.is_end_of_word = TRUE
END FUNCTION

FUNCTION trie_search(root: TrieNode, word: Text) -> Boolean:
    SET current = root
    FOR EACH char IN CHARACTERS(word) DO
        IF char NOT IN current.children THEN
            RETURN FALSE
        END IF
        SET current = current.children[char]
    END FOR
    RETURN current.is_end_of_word
END FUNCTION

FUNCTION trie_starts_with(root: TrieNode, prefix: Text) -> Boolean:
    SET current = root
    FOR EACH char IN CHARACTERS(prefix) DO
        IF char NOT IN current.children THEN
            RETURN FALSE
        END IF
        SET current = current.children[char]
    END FOR
    RETURN TRUE
END FUNCTION
```

**Complexity:** O(L) per operation where L is word/prefix length.

## Edit Distance (Levenshtein Distance)

Edit distance measures the minimum single-character edits (insert, delete, substitute)
to transform one string into another. (See also: [Dynamic Programming](dynamic_programming.md))

```
FUNCTION edit_distance(a: Text, b: Text) -> Integer:
    SET m = LENGTH(a)
    SET n = LENGTH(b)
    SET dp = NEW 2D Array[m + 1][n + 1]
    FOR i = 0 TO m DO
        SET dp[i][0] = i
    END FOR
    FOR j = 0 TO n DO
        SET dp[0][j] = j
    END FOR
    FOR i = 1 TO m DO
        FOR j = 1 TO n DO
            IF a[i - 1] = b[j - 1] THEN
                SET dp[i][j] = dp[i - 1][j - 1]
            ELSE
                SET dp[i][j] = 1 + MINIMUM(
                    dp[i - 1][j],       // deletion
                    dp[i][j - 1],       // insertion
                    dp[i - 1][j - 1]    // substitution
                )
            END IF
        END FOR
    END FOR
    RETURN dp[m][n]
END FUNCTION
```

**DP Table Trace:** edit_distance("CAT", "CARS")

```
        ""   C    A    R    S
   ""  [ 0]  1    2    3    4       Optimal path:
    C    1  [ 0]  1    2    3         CAT -> CAR (substitute T->R)
    A    2    1  [ 0]  1    2             -> CARS (insert S)
    T    3    2    1  [ 1] [ 2]       Total edits: 2
```

**Complexity:** O(m * n) time and space; reducible to O(min(m, n)) space.

## Suffix Arrays

A suffix array is a sorted array of all suffix starting indices. It enables
efficient pattern search via binary search on sorted suffixes.

```
String: BANANA          Sorted suffixes:         Suffix Array:
                          5: A                   [5, 3, 1, 0, 4, 2]
All suffixes:             3: ANA
  0: BANANA              1: ANANA
  1: ANANA               0: BANANA
  2: NANA                4: NA
  3: ANA                 2: NANA
  4: NA
  5: A
```

**Searching:** Binary search on the suffix array yields O(m * log n) search time.

## Complexity Comparison

| Algorithm              | Time (Best) | Time (Worst) | Space   | Preprocessing |
|------------------------|-------------|--------------|---------|---------------|
| Naive Search           | O(n)        | O(n * m)     | O(1)    | None          |
| KMP                    | O(n + m)    | O(n + m)     | O(m)    | O(m)          |
| Rabin-Karp             | O(n + m)    | O(n * m)     | O(1)    | O(m)          |
| Trie Insert/Search     | O(L)        | O(L)         | O(N*L)  | O(N*L)        |
| Edit Distance          | O(m * n)    | O(m * n)     | O(m*n)  | None          |
| Suffix Array Search    | O(m log n)  | O(m log n)   | O(n)    | O(n log n)    |

Where: n = text length, m = pattern length, L = word length, N = number of strings.

## Practice Problems

1. **Repeated Substring Check:** Given a string S, determine whether it can be
   constructed by repeating a smaller substring. Hint: use the KMP failure function.

2. **First Unique Character:** Given a string, find the index of the first character
   that appears exactly once. Design a solution scanning the string at most twice.

3. **Longest Common Prefix:** Given a list of N strings, find the longest prefix
   common to all. Consider using a trie for an efficient solution.

4. **Minimum Edit Operations:** Find not just the edit distance but the actual
   sequence of operations transforming A into B by backtracking through the DP
   table. (See also: [Dynamic Programming](dynamic_programming.md))

5. **Word Search in Grid:** Given a 2D grid of characters and a list of words, find
   all words formed by traversing adjacent cells without reusing a cell. Use a trie
   to prune the search.

6. **Longest Palindromic Substring:** Given a string of length n, find the longest
   palindromic substring. Can you solve it in O(n) time using Manacher's algorithm?
