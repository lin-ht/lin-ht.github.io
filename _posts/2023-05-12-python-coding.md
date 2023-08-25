---
layout: post
title:  Python coding
date:   2023-05-12 13:40:00
description: Cheatsheet
tags: notes code
categories: note-posts
---
#### Python decorator usage
```python
# Example 1: logging.
import datetime

def log(func):
    def wrapper(*args, **kwargs):
        with open("logs.txt", "a") as f:
            f.write("Called func with " + " ".join([str(arg) for arg in args]) + " at " + str(datetime.datetime.now()) + "\n")
        val = func(*args, **kwargs)
        return val
    return wrapper

# Same as writing run = log(run)
@log
def run(a, b, c=9):
    print(a + b + c)

run(1, 3)

```
```python
# Example 2: class singleton instance.
def singleton(cls):
	instances = {}
	def getinstance():
		if cls not in instances:
            instances[cls] = cls()
        return instances[cls]
    return getinstance

@singleton
class MyClass:
    ...
```
```python
# Example 3: add attributes to a function.
def attrs(**kwds):
    def decorate(f):
        for k in kwds:
            setattr(f, k, kwds[k])
        return f
    return decorate

@attrs(versionadded="2.2",
       author="Guido van Rossum")
def mymethod(f):
    ...
```
<blockquote>
<p style="font-size:16px">
If the attribute is not found, setattr() creates a new attribute and assigns value to it. However, this is only possible if the object implements the __dict__() method.
</p>
</blockquote>

```python
# Example 4: enforce function argument and return types.
def accepts(*types):
    def check_accepts(f):
        assert len(types) == f.func_code.co_argcount
        def new_f(*args, **kwds):
            for (a, t) in zip(args, types):
                assert isinstance(a, t), \
                       "arg %r does not match %s" % (a,t)
            return f(*args, **kwds)
        new_f.func_name = f.func_name
        return new_f
    return check_accepts

def returns(rtype):
    def check_returns(f):
        def new_f(*args, **kwds):
            result = f(*args, **kwds)
            assert isinstance(result, rtype), \
                   "return value %r does not match %s" % (result,rtype)
            return result
        new_f.func_name = f.func_name
        return new_f
    return check_returns

@accepts(int, (int,float))
@returns((int,float))
def func(arg1, arg2):
    return arg1 * arg2
```
```python
# Example 5: Declare that a class implements a particular (set of) interface(s).
def provides(*interfaces):
     """
     An actual, working, implementation of provides for
     the current implementation of PyProtocols.  Not
     particularly important for the PEP text.
     """
     def provides(typ):
         declareImplementation(typ, instancesProvide=interfaces)
         return typ
     return provides

class IBar(Interface):
     """Declare something about IBar here"""

@provides(IBar)
class Foo(object):
        """Implement something here..."""
```
Commen decorators in python:
<ul>
    <li>@property</li>
    <li>@classmethod</li>
    <li>@staticmethod</li>
</ul>

```python
class Mass:
    def __init__(self, kilos):
        self.kilos = kilos
        
    @property
    def pounds(self):
        return self.kilos * 2.205

    @classmethod
    def from_pounds(cls, pounds):
        # convert pounds to kilos
        kilos = pounds / 2.205
        # cls is the same as Weight. calling cls(kilos) is the same as Weight(kilos)
        return cls(kilos)

    @staticmethod
    def conversion_info():
        print("Kilos are converted to pounds by multiplying by 2.205.")

```
<blockquote>
<p style="font-size:16px">
A static method is tied to the class, not to its instance. This may remind you of a class method but the key difference is that a static method doesn’t modify the class at all. In other words, a static method doesn’t take self or cls as its arguments. 
</p>
</blockquote>

#### Type annotation and protocol usage
```python
# Example 1: protocol
from typing import Protocol

class HasBirthYear(Protocol):
	# use ellipsis (...) as the function body.
    def get_birthyear(self) -> int: ...

class Person:
    def __init__(self, name, birthyear):
        self.name = name
        self.birthyear = birthyear

    def get_birthyear(self) -> int:
        return self.birthyear

def calc_age(current_year: int, data: HasBirthYear) -> int:
    return current_year - data.get_birthyear()

john = Person("john doe", 1996)
```

```python
# Example 2: Add type hints for Iterable class.
# Iterable type that implements the __iter__ method.
from collections.abc import Iterable

def double_elements(items: Iterable[int]) -> list[int]:
    return [item * 2 for item in items]

print(double_elements([2, 4, 6])) # list
print(double_elements((2, 4)))     # tuple
```

```python
# Example 3: Add type hints for Sequence class.
# Sequence type that have special methods: __getitem__ and __len__.
from collections.abc import Sequence

def get_last_element(data: Sequence[int]) -> int:
    return data[-1]

first_item = get_last_element((3, 4, 5))    # 5
second_item = get_last_element([3, 8]    # 8
```

```python
# Example 4: Add type hints for Mapping class.
# Mapping type that implements the following methods:
#   __getitem__: for accessing an element
#   __iter__: for iterating
#   __len__: computing the length
from collections.abc import Mapping

def get_full_name(student: Mapping[str, str]) -> str:
    return f'{student.get("first_name")} {student.get("last_name")}'

john = {
  "first_name": "John",
  "last_name": "Doe",
}

get_full_name(john)
```

```python
# Example 5: Add type hints for MutableMapping class.
# Mapping type that implements the following methods:
#   __getitem__: for accessing an element
#   __setitem__: for setting an element
#   __delitem__: for deleting an element
#   __iter__: for iterating
#   __len__: computing the length
from collections.abc import MutableMapping

def update_first_name(student: MutableMapping[str, str], first_name: str) -> None:
    student["first_name"] = first_name

john = {
    "first_name": "John",
    "last_name": "Doe",
}

update_first_name(john, "james")
```

```python
# Example 6: Add type hints to tuples
# Annotate a tuple with two elements
student: tuple[str, int] = ("John Doe", 18)
# Annotate a tuple with an unknown amount of elements of a similar type
letters: tuple[str, ...] = ('a', 'h', 'j', 'n', 'm', 'n', 'z')

# Annotate a tuple with a named type
from typing import NamedTuple

class StudentTuple(NamedTuple):
    name: str
    age: int

john = StudentTuple("John Doe", 33)
```

```python
# Example 7: Add type hints to TypedDict.
from typing import TypedDict

class StudentDict(TypedDict):
    first_name: str
    last_name: str
    age: int
    hobbies: list[str]

student1: StudentDict = {
    "first_name": "John",
    "last_name": "Doe",
    "age": 18,
    "hobbies": ["singing", "dancing"],
}
```

```python
# Example 8: Add type hints for a union type.
def show_type(num: str | int):
    if(isinstance(num, str)):
        print("You entered a string")
    elif (isinstance(num, int)):
        print("You entered an integer")

show_type('hello') # You entered a string
show_type(3)       # You entered an integer
```

```python
# Example 9: overloaded function
from typing import overload

# Decorator: @overload
@overload
def add_number(value: int, num: int) -> int: ...

@overload
def add_number(value: list, num: int) -> list: ...

def add_number(value, num):
    if isinstance(value, int):
        return value + num
    elif isinstance(value, list):
        return [i + num for i in value]

print(add_number(3, 4))
print(add_number([1, 2, 5], 4)
```

```python
# Example 10: Add type hints for optional parameters
def format_name(name: str, title: Optional[str] = None) -> str:
    if title:
        return f"Name: {title}. {name.title()}"
    else:
        return f"Name: {name.title()}"

format_name("john doe", "Mr")
```


#### Pytest
The <a href="https://docs.pytest.org/en/7.3.x/index.html">pytest framework</a> makes it easy to write small, readable tests, and can scale to support complex functional testing for applications and libraries. Pytest is equiped with metabuild ability so that it can compile the test files as desired.

What is <a href="https://docs.pytest.org/en/7.3.x/explanation/fixtures.html#about-fixtures">fixtures</a>? We can tell pytest that a particular function is a fixture by decorating it with `@pytest.fixture`. Fixtures are acquired by test funcitons by <a href="https://docs.pytest.org/en/7.3.x/how-to/fixtures.html#how-to-fixtures">declaring them as arguments</a>. Pytest has several useful <a href="https://docs.pytest.org/en/7.3.x/reference/fixtures.html">built-in fixtures</a>.


<a href="https://docs.pytest.org/en/7.3.x/how-to/parametrize.html">How to parametrize fixtures and test functions</a>

```bash
# run all the tests in a repo
pytest

# run a specific test
cd /path/to/code/
pytest -v test_file.py::test_case
```

```python
# content of test_expectation.py
import pytest

# pytest mark a function
@pytest.mark.parametrize("test_input,expected", [("3+5", 8), ("2+4", 6), ("6*9", 42)])
def test_eval(test_input, expected):
    assert eval(test_input) == expected

# pytest mark a class
@pytest.mark.parametrize("n,expected", [(1, 2), (3, 4)])
class TestClass:
    def test_simple_case(self, n, expected):
        assert n + 1 == expected

    def test_weird_simple_case(self, n, expected):
        assert (n * 1) + 1 == expected
```

```python
import pytest
# global pytest mark which parametrize all tests in a module
pytestmark = pytest.mark.parametrize("n,expected", [(1, 2), (3, 4)])

class TestClass:
    def test_simple_case(self, n, expected):
        assert n + 1 == expected

    def test_weird_simple_case(self, n, expected):
        assert (n * 1) + 1 == expected
```

```python
# mark individual test instances
import pytest

# use built-in xfail to set an expected failure test.
@pytest.mark.parametrize(
    "test_input,expected",
    [("3+5", 8), ("2+4", 6), pytest.param("6*9", 42, marks=pytest.mark.xfail)],
)
def test_eval(test_input, expected):
    assert eval(test_input) == expected
```

#### MISC

Ruff extension for VSCode as a quick linter:
```bash
ruff check --exclude clio-docker --ignore ANN,D . 
ruff check --exclude clio-docker --select UP . [--fix]

ruff rule S101
```

mypy for annotation check. (Check rules installed in `pyproject.toml`)
```bash
mypy /path/to/dir
```

Print Env var `LD_LIBRARY_PATH` in python:
```python
import os, sys
print(os.environ['LD_LIBRARY_PATH'])
```
#### References
<ul>
	<li><a href="https://peps.python.org/pep-0318/">PEP 318 – Decorators for Functions and Methods</a></li>
    <li>Decorators in python: <a href="https://builtin.com/software-engineering-perspectives/python-symbol">What Is the @ Symbol in Python and How Do I Use It?</a></li>
    <li>Packing&unpacking using asterisk(*):<a href="https://www.scaler.com/topics/asterisk-python/"> What is the Asterisk Operator in Python?</a></li>
    <li>Type annotation and protocol: <a href="https://blog.logrocket.com/understanding-type-annotation-python/">Understanding type annotation in Python</a></li>
    <li>Pytest <a href="https://docs.pytest.org/en/7.3.x/reference/reference.html">API reference</a></li>
</ul>