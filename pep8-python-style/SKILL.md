---
name: pep8-python-style
description: "PEP 8 Python style guide enforcement. Use this skill whenever writing, editing, or reviewing Python code — including new files, modifications to existing files, code reviews, and style checks.
---

# PEP 8 Python Style Guide

This skill ensures all Python code follows PEP 8 conventions. It operates in two modes depending on context:

- **Writing/editing code**: Apply PEP 8 silently — just write correct code without calling out each rule.
- **Reviewing code**: List specific violations with locations and suggested fixes.

The guiding philosophy of PEP 8 is that code is read far more often than it is written, so readability and consistency matter. That said, know when to be inconsistent — breaking a rule is fine when applying it would make the code less readable, when it conflicts with surrounding code that predates this guide, or when the code predates the rule and there's no reason to modify it.

## Code Layout

### Indentation
Use 4 spaces per indentation level. Never use tabs (unless maintaining consistency with existing tab-indented code).

Continuation lines align wrapped elements using one of these approaches:

```python
# Aligned with opening delimiter
foo = long_function_name(var_one, var_two,
                         var_three, var_four)

# Hanging indent — add an extra 4 spaces to distinguish from the body
def long_function_name(
        var_one, var_two,
        var_three, var_four):
    print(var_one)

# Hanging indent in a data structure
my_list = [
    1, 2, 3,
    4, 5, 6,
]
```

When using hanging indents, no arguments go on the first line.

### Maximum Line Length
- Code: 79 characters
- Docstrings and comments: 72 characters
- Teams may agree on up to 99 for code, but docstrings/comments stay at 72

Use Python's implied line continuation inside parentheses, brackets, and braces. Backslash continuation is acceptable for constructs that can't use implied continuation (e.g., `with` statements, `assert`).

### Line Breaks Around Binary Operators
Break **before** the operator (Knuth's style):

```python
# Correct
income = (gross_wages
          + taxable_interest
          + (dividends - qualified_dividends))

# Wrong
income = (gross_wages +
          taxable_interest +
          (dividends - qualified_dividends))
```

### Blank Lines
- **Two blank lines** before and after top-level function and class definitions
- **One blank line** between methods inside a class
- Extra blank lines may separate logical sections within a function (sparingly)

### Imports
One import per line (except multiple names from the same module via `from`):

```python
# Correct
import os
import sys
from subprocess import Popen, PIPE

# Wrong
import os, sys
```

Order import groups with a blank line between each:
1. Standard library
2. Third-party packages
3. Local application/library

Always use absolute imports. Avoid wildcard imports (`from module import *`).

### Module-Level Dunder Names
Place `__all__`, `__author__`, `__version__` etc. after the module docstring but before imports (except `from __future__` imports, which come first).

### Source File Encoding
Use UTF-8. Omit the encoding declaration unless necessary for backwards compatibility.

## String Quotes
Pick single or double quotes and be consistent. Use the opposite style to avoid backslash escapes when a string contains quotes. Triple-quoted strings always use double quotes (`"""`).

## Whitespace

### Avoid Extraneous Whitespace

```python
# Correct                    # Wrong
spam(ham[1], {eggs: 2})      spam( ham[ 1 ], { eggs: 2 } )
foo = (0,)                   bar = (0, )
spam(1)                      spam (1)
dct['key']                   dct ['key']
x = 1                        x             = 1
```

### Operator Spacing
Surround binary operators with a single space on each side: `=`, `+=`, `==`, `<`, `>`, `!=`, `in`, `not in`, `is`, `is not`, `and`, `or`, `not`.

For expressions with mixed priority, consider spacing only around the lowest-priority operator:

```python
# OK
hypot2 = x*x + y*y
c = (a+b) * (a-b)
```

No spaces around `=` in keyword arguments or default values (when unannotated):

```python
def complex(real, imag=0.0):
    return magic(r=real, i=imag)
```

But when combining with an annotation, use spaces:

```python
def munge(sep: AnyStr = None): ...
def munge(input: AnyStr, sep: AnyStr = None, limit=1000): ...
```

### Slices
Treat the colon like a binary operator — equal amounts of space on each side (or none):

```python
ham[1:9], ham[1:9:3], ham[:9:3], ham[1::3], ham[1:9:]
ham[lower::2], ham[:upper], ham[lower+offset : upper+offset]
```

### Trailing Whitespace
Never leave trailing whitespace on any line.

## Comments

### Block Comments
Start each line with `# ` (hash and a space). Indent to match the code they describe. Separate paragraphs with a line containing only `#`.

### Inline Comments
Use sparingly. Separate from the statement by at least two spaces. Don't state the obvious.

### Docstrings
Write docstrings for all public modules, functions, classes, and methods. The closing `"""` goes on the same line for one-liners, on its own line for multi-line docstrings. Follow PEP 257 conventions.

## Naming Conventions

### The Rules

| What | Style | Example |
|------|-------|---------|
| Packages | short lowercase, avoid underscores | `mypackage` |
| Modules | lowercase with underscores | `my_module` |
| Classes | CapWords | `MyClass` |
| Exceptions | CapWords with `Error` suffix | `ValueError` |
| Functions | lowercase_with_underscores | `calculate_total` |
| Variables | lowercase_with_underscores | `item_count` |
| Constants | UPPER_CASE_WITH_UNDERSCORES | `MAX_OVERFLOW` |
| Method arguments | `self` (instance), `cls` (class) | |
| Type variables | Short CapWords | `T`, `AnyStr` |

### Names to Avoid
Never use `l` (lowercase L), `O` (uppercase O), or `I` (uppercase I) as single-character variable names — they are too easily confused with 1 and 0.

### Leading Underscores
- `_single`: internal/non-public indicator
- `__double`: triggers name mangling in classes (use only to avoid name collisions with subclasses)
- `__dunder__`: reserved for Python special methods — never invent these

### Trailing Underscore
Use a single trailing underscore to avoid conflicts with Python keywords: `class_`, `type_`.

### Designing for Inheritance
- Public attributes: no leading underscores
- Non-public attributes: single leading underscore
- Use `__all__` to declare the public API of a module

## Programming Recommendations

### Singletons
Compare with `is` / `is not`, never `==`:

```python
if foo is not None:    # Correct
if foo != None:        # Wrong
```

### Boolean Tests
Use truthiness directly — don't compare to `True` or `False`:

```python
if greeting:           # Correct
if greeting == True:   # Wrong
if greeting is True:   # Worst
```

### Empty Sequences
Test with truthiness, not length:

```python
if not seq:            # Correct (empty)
if seq:                # Correct (non-empty)
if len(seq) == 0:      # Wrong
```

### String Methods
Use `.startswith()` and `.endswith()` instead of slicing:

```python
if foo.startswith('bar'):   # Correct
if foo[:3] == 'bar':        # Wrong
```

### Type Checking
Use `isinstance()`, not direct type comparison:

```python
if isinstance(obj, int):    # Correct
if type(obj) is int:        # Wrong
```

### Functions
Use `def` instead of assigning lambdas to names:

```python
# Correct
def double(x):
    return x * 2

# Wrong
double = lambda x: x * 2
```

### Return Statements
Be consistent — if any return in a function returns a value, all returns should be explicit:

```python
# Correct
def foo(x):
    if x >= 0:
        return math.sqrt(x)
    return None

# Wrong
def foo(x):
    if x >= 0:
        return math.sqrt(x)
```

### Exception Handling
- Derive from `Exception`, not `BaseException`
- Catch specific exceptions — avoid bare `except:`
- Minimize code inside `try` blocks
- Use `raise X from Y` for explicit exception chaining
- Use `with` statements for resource management

### String Concatenation
Use `''.join()` for building strings in loops rather than `+=` concatenation, because join is efficient across all Python implementations.

### Avoid `return`/`break`/`continue` in `finally` Blocks
These silently cancel any active exception being propagated.

## Function and Variable Annotations

Follow PEP 484 syntax:
```python
def greeting(name: str) -> str:
    return 'Hello ' + name

code: int
primes: list[int] = []
```

- Space after colon, no space before
- Spaces around `=` when assignment is present
- Space around `->` in return annotations
