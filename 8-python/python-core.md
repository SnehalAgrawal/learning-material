# Python Core: Data Structures & Mutability

### 1. Overview
Python is a high-level, interpreted language known for its readability and "Batteries Included" philosophy. For senior engineers, understanding the nuances of Python's core data structures and how it handles object references is essential for writing memory-efficient and bug-free code.

### 2. Key Concepts
*   **Built-in Data Structures**:
    *   *Lists*: Dynamic arrays, O(1) append/access, O(n) insert/delete.
    *   *Sets*: Unordered collection of unique items, O(1) average lookup.
    *   *Dictionaries*: Hash maps, O(1) average lookup. Python 3.7+ preserves insertion order.
*   **Mutable vs. Immutable**: 
    *   *Mutable*: Objects that can be changed (Lists, Dicts, Sets).
    *   *Immutable*: Objects that cannot be changed (Strings, Tuples, Integers, Frozensets).
*   **Pass-by-Assignment**: Python passes references to objects. Whether you can change the object depends on its mutability.

### 3. Real-World Usage
*   **Fast Lookups**: Using `Sets` or `Dicts` instead of lists for membership checks to reduce complexity from O(n) to O(1).
*   **Performance Optimization**: Using `List Comprehensions` instead of `for` loops for faster execution (C-level optimization).
*   **Data Integrity**: Using `Tuples` to return multiple values from a function, ensuring they aren't accidentally modified by the caller.
*   **Memory Management**: Choosing `Generators` (iterators) over lists to process millions of rows from a database without loading them all into RAM.

### 4. Tradeoffs
*   **Flexibility vs. Safety**: Dynamic typing allows for rapid development but makes it easier to pass the "wrong object type" into a function, causing runtime errors.
*   **Lists vs. Tuples**: Lists are flexible but use more memory due to "Over-allocation" for future growth; Tuples are fixed but extremely memory-efficient.
*   **Dictionaries vs. NamedTuples/Dataclasses**: Dicts are easier to build on the fly; Dataclasses are more memory-efficient and provide better "Type Hinting" support.

### 5. When NOT to Use
*   **Mutable Default Arguments**: **NEVER** use a list or dict as a default argument in a function definition (e.g., `def append(item, my_list=[])`). The list is created once at definition time, not execution time, leading to "Shared state" bugs.
*   **Massive Mathematical Computations**: Pure Python lists are slow for heavy math. Use **NumPy** which uses C-based contiguous memory arrays.

### 6. Interview Focus
*   **Mutability Trap**: "What is the output of this function when called twice? `def add_to(item, storage=[]): storage.append(item); return storage`."
*   **Computational Complexity**: "Explain the Big-O complexity of searching for an item in a List vs. a Set. When is a List better?"
*   **Copying**: "What is the difference between a 'Shallow Copy' (`copy()`) and a 'Deep Copy' (`deepcopy()`) in Python?"

### 7. Common Mistakes
*   **Inefficient Concatenation**: Using `+=` to join large strings inside a loop (Strings are immutable, so this creates a new string every time). Use `''.join(list_of_strings)`.
*   **Modifying a List while Iterating**: Deleting items from a list as you loop through it, which skips items and causes index errors.
*   **Over-using global**: Using `global` variables instead of passing arguments, making the code untestable and prone to side-effects.
