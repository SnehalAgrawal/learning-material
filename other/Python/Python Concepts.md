1. **Pythonic Constructs and Idioms**
    - Prefer list comprehensions over loops, use `enumerate()` instead of manual counters, and follow PEP8.
    - Embrace the Zen of Python (`import this`) for readability and simplicity.
2. **Data Structures: list, dict, set, tuple**
    - `list`: Ordered, mutable collection.
    - `dict`: Key-value mapping with O(1) access time.
    - `set`: Unordered collection of unique items.
    - `tuple`: Immutable and hashable sequences.
3. **Comprehensions and Generators**
    - Comprehensions (`[x for x in range(10)]`) for concise syntax.
    - Generators (`(x for x in range(10))`) for memory-efficient iteration.
    - Use `yield` to build generator functions.
4. **AsyncIO and Multiprocessing**
    - `asyncio` handles I/O-bound tasks with coroutines and `await`.
    - `multiprocessing` is for CPU-bound tasks using multiple processes.
    - Avoid using threading for CPU-bound workloads due to GIL.
5. **Flask and FastAPI Basics**
    - Flask: Lightweight WSGI web framework, ideal for small apps.
    - FastAPI: Modern, async-ready, type-safe framework with automatic docs (Swagger, ReDoc).
    - Use FastAPI for building performant REST APIs with type hints and dependency injection.
6. **Scripting and Automation Patterns**
    - Automate tasks like file I/O, data cleanup, and API calls with scripts.
    - Use `argparse` or `click` for CLI tools.
    - Use `os`, `pathlib`, and `subprocess` to interface with the system.