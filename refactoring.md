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

## [Oleksii Pilkevych - Things not to do in Python](https://www.youtube.com/watch?v=tqfTrCNkHW8) [15 min, code::dive 2018]

- Python anti-patterns
- Exceptions - more subtle ways to mess up using try, except, finally. Avoid using returns in finally block
- Avoid setting global configs in modules (warnings, logging, basicConfig, setLevel, etc.)
- [More examples on Alex's GitHub](https://github.com/alexpilk/python-sandbox/tree/master/what_not_to_do)

## [Brandon Rhodes - Classic Design Patterns - Where Are They Now](https://www.youtube.com/watch?v=pGq7Cr2ekVM) [51 min, code:dive 2022]

- Outdated design patterns
- These patterns are rarely used but they illustrate the trend of going against the assumption that real-world objects map one-to-one with code objects
  - bridge
  - decorator
  - mediator
  - state
- Smaller, stand-alone patterns in wide-spread use today
  - builder
  - adapter
  - flyweight
- Framework patterns
  - composite
  - chain of responsibility
  - command
  - interpreter
- Facade

## [Trey Hunner - Readability Counts](https://www.youtube.com/watch?v=knMg6G9_XCg&list=PLCC_B16W4O8nwQdn0UppflyP56TrrhaUo&index=13) [27 min, PyCon 2017] [Trey's style guide](http://trey.io/style-guide)

Naming things

- Indexes vs named tuples

```py
# before
sc = {}
for i in csv_data:
    sc[i[0]] = i[1]
```

```py
# after
state_capitals = {}
for s, c, *_ in capitals_csv_data:
    state_capitals[s] = c
```

```py
# going even further
state_capitals = {}
for state, capital, *_ in capitals_csv_data:
    state_capitals[state] = capital
```

- Extracting functions

```py
# before
def detect_anagrams(word, candidates):
    anagrams = []
    for candidate in candidates:
        if (sorted(word.upper()) == sorted(candidate.upper())
            and word.upper() != candidate.upper()):
            anagrams.append(candidate)

```

```py
# after
def detect_anagrams(word, candidates):
    anagrams = []
    for candidate in candidates:
        if is_anagram(word, candidate):
            anagrams.append(candidate)

def is_anagram(word1, word2):
    word1, word2 = word1.upper(), word2.upper()
    return sorted(word1) == sorted(word2) and word1 != word2
```

```py
# added more names
def detect_anagrams(word, candidates):
    anagrams = []
    for candidate in candidates:
        if is_anagram(word, candidate):
            anagrams.append(candidate)
            
def is_anagram(word1, word2):
    word1, word2 = word1.upper(), word2.upper()
    are_different_words = word1 != word2
    have_same_letters = sorted(word1) == sorted(word2)
    return have_same_letters and are_different_words
```

- Consider embedding comments into variable names so that the code itself is more readable

```py
# before
def update_appointment_types(self):
    """Delete/make appt. types and set default appt. type"""
    
    # Delete appointment types for specialty besides current one
    self.appt_types.exclude(specialty=self.specialty).delete()
    
    # Create new appointment types based on specialty (if needed)
    new_types = self.specialty.appt_types.exclude(agent=self)
    self.appt_types.bulk_create(
        AppointmentType(agent=self, appointment_type=type_)
        for type_ in new_types
    )
    
    # Set default appointment type based on specialty
    old_default_id = self.default_appt_id
    self.default_appt_type = self.specialty.default_appt_type
    if self.default_appt_type.id != old_default_id:
        self.save(update_fields=['default_appt_type'])

```

```py
# after extarcting functions
def update_appointment_types(self):
    """Delete/make appt. types and set default appt. type"""
    
    self._delete_stale_appointment_types()
    self._create_new_appointment_types()
    self._update_default_appointment_type()
```

Special purpose constructs

- Write your own context managers instead of using try...finally

```py
# before
db = DBConnection("mydb")
try:
    records = db.query_all()
finally:
    db.close()
```

```py
# after
from contextlib import closing


class connect:
    def __init__(self, path):
        self.connection = DBConnection(path)
    
    def __enter__(self):
        return self.connection
    
    def __exit__(self):
        self.close()


with closing(DBConnection("mydb")) as db:
    db.query_all()
```

- Use list comprehension instead of a for loop when applicable
- Leverage built-in objects by using operator overloading

Shared data

- When to refactor into a class - when numerous functions share the same inputs

## [Jack Diederich - HOWTO Write a Function](https://www.youtube.com/watch?v=rrBJVMyD-Gs) [41 min, PyCon 2018]

Function structure

- input
- transform
- output

Helping the reader

- group objects by when they are used

```py
def loopem(records):
    results_lengths = []
    results_count = 0

    for record in records:
        result_lengths.append(len(record))

    for count in records_lengths:
        results_count += count
```

```py
# more clarity
def loopem(records):
    results_lengths = []
    for record in records:
        result_lengths.append(len(record))

    results_count = 0
    for count in records_lengths:
        results_count += count
```

- define inputs where they are used
- use custom exceptions sparringly, as they are typically defined far away from where they are used. Consider reusing existing errors.
- code structure reflects usage pattern.
  - what does the user care about?
  - what would you care about as the developer?

```py
raw = get_from_api(id=3)
user = deserialize_user(raw)
```

```py
# if the user does not care about raw
user = deserialize_user(get_from_api(id=3))
```

- return consistently

```py
def return_none_silently():
    x = 3

def return_none_quietly():
    x = 3
    return

def return_none_exlicitly():
    x = 3
    return None
```

- return the same types or return the same types + none

```py
def calc_union(list_a, list_b):
    if not list_a or not list_b:
        return None
    return list(set(list_a) | set(list_b))
```

- keep it together

```py
def config_exists(self):
    return get_config()

def has_owners(self):
    return get_config()['owners']

def validate(self):
    if not self.config_exists():
        return False
    if not self.has_owners():
        return False
    return True
```

```py
# group the funcs together if they will never be used separately
def validate(self):
    config = get_config()
    if not config:
        return False
    if not config['owners']:
        return False
    return True
```

Things that don't help

- using langauge features just because you learned about them
- problem with linters

## [Erik Rose - Constructive Code Review](https://www.youtube.com/watch?v=iNG1a--SIlk) [46 min, PyCon 2017]

- Coding is creative, and creative work is powered by enthusiasm. Therefore, constructive code reviews are an important part of the process to building excellent products.

How can we make code reviews more constructive?

- Comment with clarity. Explain "why". Do not rely on other people ability to remember what you said a week ago.
- Be language agnostic. Avoid using sarcasm or using language nuances as many people do not have English as their first language.
- Not all comments have to block the merge.
- Focus on the bigger picture, not the tiny details (unless they are important and would have an outsized impact on quality).
- Have a style guide.
- Provide quick turnarounds
  - Focus on improvements rather than perfection.
  - Comprehensiveness not required (personally, I think it depends on the nature of the patch being submitted.)
  - Respect working memory.
  - Quick "no"s.
- Notice the signs of insecurity. No one should lose in a code review.

Tact hacks - comments can come off as criticism to the other person, even if you did not mean it

- Use **questions** istead of statements, especially if you are not 100% sure that your statement is current or applies to the specific situation.
- Avoid pointing fingers inadvertantly by using "you" in a statement, because it could cause misunderstandings of intent.
- Comments do not have to be limited to negative feedback.

What if the pull request is too large? The reviewer can ask for...

- A meeting to walkthrough the change.
- A documentation of thought process and problem solving approach behind the change.
- More detailed commit messages and smaller commits.
- Comments, docstrings, naming