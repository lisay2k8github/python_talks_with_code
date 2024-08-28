# Refactoring and Code Quality

## [Jack Diederich - Stop Writing Classes](https://www.youtube.com/watch?v=o9pEzgHorH0) [27 min, PyCon 2012]

The video provides examples of overly complex code and demonstrates how to simplify it.

- **Simplify Code:** Avoid using classes when simpler solutions like functions will suffice. Many classes are unnecessary and add complexity without any real benefit.
- **Common Misuses:** Classes are often overused for tasks that don't require them, such as grouping constants or creating objects that are only used once. Instead, use functions or simpler data structures.
- **Avoid Premature Optimization:** Don't create complex structures in anticipation of future needs. Simplify first and add complexity only when it becomes necessary.

## [Brandon Rhodes - The Clean Architecture in Python](http://youtu.be/DJtef410XaM) [50 min, PyOhio 2014, [slides](http://rhodesmill.org/brandon/slides/2013-10-pyconie/)]

Applying Clean Architecture to your Python code, making it more functional, for easier testing and comprehension. Code examples in slides.

- **Separation of Concerns:** Clean Architecture promotes separating the core logic of an application from its input/output operations (e.g., database access, API calls). This separation improves code readability, testability, and maintainability.
- **Decoupling Logic from I/O:** Brandon illustrates the mistake of hiding I/O operations in subroutines without truly decoupling them from the core logic. He suggests rescuing the logic by separating it from the I/O, making it easier to test and modify.
- **Refactoring Approach:** He uses examples in Python to demonstrate how to refactor code to isolate pure logic from I/O. This involves creating smaller, pure functions that handle logic and only have data dependencies, not I/O or external state.
- **Testing Benefits:** By isolating pure functions, tests become simpler and more reliable as they do not require setting up complex environments or mocking I/O operations.
- **Applying Clean Architecture:** Brandon provides real-world examples from his projects, such as an astronomy API and a tax form generator, where he successfully applied these principles. The focus was on making core logic data-driven and isolating side effects.

## [Brett Slatkin - Refactoring Python: Why and how to restructure your code](https://www.youtube.com/watch?v=D_6ybDcU5gc&t=12s) [30 min, PyCon 2016]

- The goal of refactoring is to reorganize and rewrite the code until it is **obvious** to a new reader.
- How to refactor in practice?
  - rename, split, move
  - simplify
  - **redraw boundaries**
- Prerequisites to refactoring
- Strategies
  - Extract Variable & Function

    ```py
    # before - looks simple but hard to reason about
    if (month.lower().endswith('r') or
    month.lower().endswith('ary')):
    print('%s: oysters' % month)
    elif 8 > MONTHS.index(month) > 4:
        print('%s: tomatoes' % month)
    else:
        print('%s: asparagus' % month)
    ```

    ```py
    # extract variables - easier to reason about
    lowered = month.lower()
    ends_in_r = lowered.endswith('r')
    ends_in_ary = lowered.endswith('ary')
    index = MONTHS.index(month)
    summer = 8 > index > 4

    if ends_in_r or ends_in_ary:
        print('%s: oysters' % month)
    elif summer:
        print('%s: tomatoes' % month)
    else:
        print('%s: asparagus' % month)
    ```

    ```py
    # extract variables into functions
    def oysters_good(month):
        lowered = month.lower()
        return (
            lowered.endswith('r') or
            lowered.endswith('ary')
        )

    def tomatoes_good(month):
        index = MONTHS.index(month)
        return 8 > index > 4


    time_for_oysters = oysters_good(month)
    time_for_tomatoes = tomatoes_good(month)

    if time_for_oysters:
        print('%s: oysters' % month)
    elif time_for_tomatoes:
        print('%s: tomatoes' % month)
    else:
        print('%s: asparagus' % month)
    ```

    ```py
    # extract variables into classes
    class TomatoesGood:
        def __init__(self, month):
            self.index = MONTHS.index(month)
            self._result = 8 > index > 4

        # Use __bool__ to indicate a class is a paper trail
        def __bool__(self):  # aka __nonzero__
            return self._result

    
    time_for_oysters = OystersGood(month)
    time_for_tomatoes = TomatoesGood(month)

    if time_for_oysters:  # Calls __bool__
        print('%s: oysters' % month)
    elif time_for_tomatoes:  # Calls __bool__
        print('%s: tomatoes' % month)
    else:
        print('%s: asparagus' % month)
    ```

  - Extract Class & Move Fields
    - See video for examples of how to improve the backward compatibility of classes
  - Move Field gotchas
  - Extract Closure
- Turn warnings into exceptions

## [Anthony Shaw - Writing simpler and more maintainable Python](https://www.youtube.com/watch?v=dqdsNoApJ80) [29 min, PyCon 2019]

- Using radon for measuring the complexity and maintainability of your code base
- Wily for measuring complexity and maintainability over time by going through git history

## [Brandon Rhodes - When Python Practices Go Wrong](https://www.youtube.com/watch?v=S0No2zSJmks) [1 hour, code::dive 2019]

- Uses of eval() - Great for documentation, tests, usually not so great for source code.
- Object model
  - Programmers tend to be overly concise when it comes to comparisons, at the expense of code clarity.

    ```py
    # unsure what is the object type
    if users:
    ```

    ```py
    # clarifies whether the expression is a...
    if len(users):  # judgement on the size of the container
    if users > 0:   # or judgement on the value of the integer
    ```

  - Overuse of Mock encourages tests that are not *really* tests, because it is *too* flexible.

    ```text
    My complaints:

    Tests start to lose signal when Mock becomes routine instead of a reluctant workaround.

    Docs and blog posts too often focus on how to use rather than when.
    ```

  - Oversimplified use of \__call__ for brevity, at the expense of clarity.

- Mutability
  - Modules and classes are mutable, which means that they can change after runtime. This behavior is often unexpected to the C/C++ programmer.
    - Avoid using a global to prepresent conceptually separate objects, by mutating it as it is passed to functions.
  
    ```text
    - Data now enters a function from two directions
    - Tests will have to mutate a global
    - Threads would overwrite the one request object
    - Async framework will need special knowledge
    ```

    ```py
    # ok
    def index_view(request):
        ...

    def settings_view(request):
        ...

    def shop_view(request):
        ...
    ```

    ```py
    # bad
    from framework import request

    def index_view():
        ...

    def settings_view():
        ...

    def shop_view():
        ...
    ```

    ```py
    # worse
    # The framework loads your module.
    name = 'your_web_app'
    module = __import__(name)

    # Then mutates `request` with HTTP data.
    request.method = 'GET'
    request.url = '/shop/'
    response = getattr(module, 'shop_view')()
    ```

    - Valid uses of globals and mutability
    - Using Patch in testing to simulate functions with side effects
    - Example of linked functions with side effects. How many functions in this example can be tested without triggering a real HTTP request? (See video for a better picture)
  
    ```text
    The code *abstracts away* the I/O but fails to actually *decouple* it.

    get_books()            fetch()                  download()
        |                    |                           |
        v                    v                           v
    for isbn in books:    url = URL.format(isbn)    u = urlopen(url)
        fetch(isbn)           download(url)         data = u.read()
                                                    return data`
    ```

    ```text
    Best solution is Clean Architecture / Hexagonal

    get_books()             book_urls()                build_url()
        |                        |                           |
        v                        v                           v
    urls = book_urls(books)  for isbn in books:          u = URL.format(isbn)
    for url in urls             yield build_url(isbn)    return u
        urlopen(url)
        data = u.read()
    ```

    - An alternative is to use Patch on the HTTP request in the test.
    - Writing tests that require Patch and Mock all the time usually means that the objects need to be decoupled.
    - Import time side effects and how it can become problematic as architecture changes
    - Code in \__init__.py creates an obstable and cost and creates dependencies for every module in that folder. Provide a separate file for bulk imports.
- Object orientation
  - Using data to couple code instead of using behavior. E.g., using Pipeline instead of a Bridge pattern.

    ```text
    If you think of classes as structs for modeling the world, you'll create one class for each noun instead of for each interface = behavior.

    Separate the monolithic class into several conceptually modular classes.

    Server                Port                 Service
    ------                ----                 -------
    serve_forever()       bind()               service_actions()
    process_request()     activate()           verify_request()
    shutdown()            get()                handle_request()
                        close()              handle_error()
                                            handle_timeout()

    ```

    - Avoid routine use of mixins.

## [Conor Hoekstra - Beautiful Python Refactoring II](https://www.youtube.com/watch?v=nXZQfdxWgh0) [54 min, code::dive 2022]

- Refactor #1: @dataclass (introduced in Python 3.7)
- Refactor #2: f-strings (introduced in Python 3.6)
- Refactor #3: Enum (introduced in Python 3.4)
- Refactor #4: Optional
- Refactor #5: Use Algorithms
- Refactor #6: Avoid ITM

```py
# before
class Position():
    def __init__(self, dir, row, col):
        self.row = row
        self.col = col
        self.dir = dir

    def __lt__(self, other):
        return self.row < other.row

    def __eq__(self, other):
        return self.row == other.row and \
               self.col == other.col and \
               self.dir == other.dir

    def __repr__(self):
        return str(self.tuple())

    def tuple(self):
        return (self.row, self.col, self.dir)

```

```py
# after
@dataclass(frozen=True, order=True)
class Position():
    dir: Direction
    row: int
    col: int
```

```py
# before
class Board:
    def __init__(self, size):
        self._size = size
        self._tiles = []
        for _ in range(size):
            row = []
            for _ in range(size):
                row.append('.')
            self._tiles.append(row)

    def all_positions(self):
        result = []
        for row in range(self._size):
            for col in range(self._size):
                result.append((row, col))
        return result

    def copy(self):
        result = Board(self._size)
        for pos in self.all_positions():
            result.set_tile(pos, self.get_tile(pos))
        return result
# ...


```

```py
# after
import copy
import itertools as it

class Board:
    def __init__(self):
        self.size = 15
        self._tiles = [['.'] * self.size for _ in range(self.size)]

    def all_positions(self):
        return it.product(range(0, 15), range(0, 15))

    def copy(self):
        return copy.deepcopy(self)
```