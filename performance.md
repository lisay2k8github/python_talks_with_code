# Performance Optimization

## [Sebastian Witowski - Writing Faster Python 3](https://www.youtube.com/watch?v=5xArPgQMJls) [42 min, PyCon 2022, [code](https://github.com/switowski/writing-faster-python3)]

- Ways to speed up Python - hardware, switch interpreter, **update Python version**, and writing better **algorithms and data structures**.
- It is important to test out these improvements on your specific environment and code as their impact may vary.

### Source code improvements

- Use local variables where possible to reduce scope and increase access speed.
- Use built-in functions and modules (such as itertools and collections) for optimized performance.
- List comprehension instead of a loop
- Generator expression for lower memory usage
- **numpy** - dedicated library for scientific computing
- numba - JIT decorator for easy wins

```python
# slower
import time

# Initialize a global variable to store the total sum
total = 0

def compute_sum_of_powers():
    global total
    for x in range(1_000_001):
        # This operation is done iteratively in Python, which is slower
        # because each iteration requires Python's interpreter to handle
        # the arithmetic and variable update.
        total = total + x * x

start_time = time.time()
compute_sum_of_powers()
end_time = time.time()

# Calculate the elapsed time in milliseconds
elapsed_time = (end_time - start_time) * 1000

total, elapsed_time
```

```python
# faster
import time
import numpy as np

def compute_sum_of_powers():
    # NumPy arrays and operations are optimized for performance,
    # making this approach faster than the Python loop.
    numbers = np.arange(1_000_001, dtype=np.int64)
    powers = np.power(numbers, 2)
    return np.sum(powers)

start_time = time.time()
total = compute_sum_of_powers()
end_time = time.time()

# Calculate the elapsed time in milliseconds
elapsed_time = (end_time - start_time) * 1000

total, elapsed_time
```

### Further improvements

- Use mathematical formulas for direct computation where possible.

```py
import time


def formula(n):
    # Use integer division to avoid loss of precision
    return n * (n + 1) * (2 * n + 1) // 6


start_time = time.perf_counter()
total = formula(1_000_000)
end_time = time.perf_counter()

# Calculating elapsed time
elapsed_time = end_time - start_time

int(total), elapsed_time * 1000
```

### More improvements with benchmarking

Setup
```
Python 3.10.4
PYTHONDONTWRITEBYTECODE set to 1
python -m timeit -s "from my_module import function" "function()"
Machine: 14-inch Macbook Pro (2021) with 16GB of RAM, M1 with 10 CPU cores and 16 GPU cores
```

- Permission vs forgiveness - checks vs using try/except. Which is faster? It depends.
- Find element in collection
  - Use itertools
  - List comprehension is slower because a list had to be set up just to check the first few numbers
  - Generator (lazy list comprehension)
- Filter a list - list comprehension vs filter vs for loop
  - Why is list comprehension faster than filter in certain cases?
  - When to use loops?
- Membership testing - "in" operator vs for loop vs set
  - Why is set slower in certain cases?
- Dictionaries
  - Why is using functions slower than using literals, e.g., "dict()"  vs "{}"? See bytecode.
- Remove dupes - list comprehension, loop, set
  - List comprehensions are used to create lists, not for looping to create some side effect. In this case, loops are better as they show the intention of the code.
  
### Python versions

- Generally, the newer versions are faster. Python 3.11 has a lot of speed improvements. See benchmarking near the end of the video.

## [Alex Gaynor - Fast Python, Slow Python](https://www.youtube.com/watch?v=7eeEf_rAJds) [36 min, PyCon 2014]

- Performance is about specialization, which considers the unique attributes of your program and environment.
- Optimizing dynamic typed languages is different from optimizing static typed languages.
- Differentiating between whether an operation is inherently slow or that it is difficult to optimize.

```python
# slower
if [hex, bytes, bytes_le, fields, int].count(None) != 4:
    raise TypeError('need exactly one argument')
```

```python
# faster
if (
    (hex is None) + (bytes is None) + (bytes_le is None) +
    (fields is None) + (int is None)
) != 4:
    raise TypeError
```

- Dictionaries are not specialized, yet they are often used in place of objects
- Classes are specialized, dictionaries are not.


## [Nina Zakharenko - Memory Management in Python - The Basics](https://www.youtube.com/watch?v=URNdRl97q_0) [30 min, PyCon 2018]

- Python has names, not variables. Python variables are names that refer to objects via references. Note: Some developers nonetheless consider Python's names as variables.
- What is a name? Label for an object.
- What is a reference? A name or a container object that points to another object. The number of references to a particular object is called the **reference count**.
- The difference between C and Python style variables is that while the former creates a copy of the object upon assignment, Python adds a reference to the original object.
- How to increase or decrease reference count?
  - What exactly does "del" do? It removes the binding of a name to an object and does not directly affect the object unless it is the last reference to that object.
- Avoid putting large or complex objects in the global namespace to prevent them from staying in memory longer than necessary.

```python
def mem_test():
    x = 300
    y = 300
    print(id(x)) # Memory address of x
    print(id(y)) # Memory address of y
    print(x is y) # Do they refer to the same object?

mem_test()
```

- There are two types of garbage collection:
  - Reference counting: space + execution overhead, not thread safe, does not collect objects with cyclical references
  - Tracing: using mark and sweep
- Python uses Reference Counting and Generational (type of tracing)
- Weak references
- Why do we need GIL? So that only one thread can change the reference count on an object at any given point in time.

## [Brandon Rhodes - All Your Ducks In A Row: Data Structures in the Std Lib and Beyond](https://www.youtube.com/watch?v=fYlnfvKVDoM) [38 min, PyCon 2014]

- How python data structures are stored in memory
- What are intermediate objects and how they slow down math operations
- How container data structures "store" data of different types
- Moving an item in a general purpose data structure means merely copying the address
- Tuples vs dictionaries
- Why list is the most dangerous data structure in Python and why does it trade space for time
- Why lists can be slow
- Why an n slice result in n address copies, and how numpy circumvents this issue
- How are classes implemented in Python

## [Anthony Shaw - Write faster Python! Common performance anti patterns](https://www.youtube.com/watch?v=YY7yJHo0M5I) [30 min, PyCon 2022] [Anthony's GitHub](https://github.com/tonybaloney/anti-patterns)

- Performance profilers / benchmarking - austin, **scalene**, cprofile, pyinstrument, py-spy, yappi. 
  - Note: While not talked about in the video, I found that Scalene was useful as a first pass, but it was not able to profile the code in the imported modules. The output from using --profile-all did not capture all of the functions that were called. I used line_profiler as a second profiler to analyze the module functions.
- Common performance anti-patterns
  - Loop invariant
  - Missing list/dict/set comprehensions
  - Inefficient data structures
    - Example - which data container is faster? dataclass, named tuple, concrete class
    - Comparison with PyPy for various data structures
  - Too many tiny function calls
- Sample correctly and track regressions
- Catch issues in code-review to avoid performance regressions
- Match statements

## [Brandon Rhodes - The Dictionary Even Mightier](https://www.youtube.com/watch?v=66P5FMkWoVU) [47 min, PyCon 2017]

The video provides a detailed history of Python dictionary implementations, highlighting key changes and improvements over time. Brandon Rhodes explains various optimizations and features introduced in different Python versions, focusing particularly on how dictionaries handle data storage and retrieval. One particularly interesting topic discussed is why Python dictionaries were initially orderless and how recent updates have changed this behavior to maintain insertion order, aligning with user expectations and enhancing the overall efficiency of dictionaries.

## [Kavya Joshi - The Memory Chronicles A Tale of Two Pythons](https://www.youtube.com/watch?v=d7qEzpnkWaY) [28 min, PyCon 2017]

- Memory management differences between CPython and MicroPython.
- Illustrates how the way objects are stored impacts their memory usage.
- CPython:
  - Garbage Collection: Combines reference counting with a cyclic garbage collector to manage memory and handle circular references.
  - All objects, including integers and strings, have significant overhead, which increases overall memory usage.
- MicroPython:
  - Garbage Collection: Uses a simple mark-and-sweep garbage collector for all heap-allocated objects.
  - Object Model: Utilizes pointer tagging and stores small integers directly, minimizing memory usage.

## [Mike Müller - Faster Python Programs - Measure, don't Guess](https://www.youtube.com/watch?v=DGrS0uwMuHY) [3 hours 23 mins, PyCon 2018]

- Optimization brings tradeoffs
