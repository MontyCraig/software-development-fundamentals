# Hash Tables

## Overview
Data structure that implements an associative array abstract data type, mapping keys to values using a hash function.

## Core Concepts

### 1. Hash Functions
- Properties of good hash functions
- Collision handling
- Universal hashing
- Perfect hashing
- Common hash functions

### 2. Collision Resolution
- Chaining (Separate Chaining)
  - Linked list implementation
  - Time complexity analysis
  - Load factor considerations
- Open Addressing
  - Linear probing
  - Quadratic probing
  - Double hashing
  - Performance implications

### 3. Dynamic Resizing
- Load factor monitoring
- Rehashing process
- Growth strategies
- Performance implications

## Time Complexities
- Average Case:
  - Insert: O(1)
  - Delete: O(1)
  - Search: O(1)
- Worst Case (with collisions):
  - All operations: O(n)

## Implementation Files
- `hash_table.py`: Base implementation
- `chaining_hash_table.py`: Separate chaining implementation
- `open_addressing_hash_table.py`: Open addressing implementation
- `hash_functions.py`: Various hash function implementations
- `hash_utils.py`: Utility functions

## Testing
- `test_hash_table.py`
- `test_chaining_hash_table.py`
- `test_open_addressing_hash_table.py`
- `test_hash_functions.py`

## Applications
1. Database Indexing
2. Caching
3. Symbol Tables in Compilers
4. Blockchain Mining
5. Password Verification 