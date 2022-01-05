# Python Code Style Guide

Python is the main dynamic language used at 42maru. This style guide is a list of 
dos and don’ts for Python programs. 

So, all contributors and maintainers of this project are subject to this guide.

This guides is from [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html).
So, didn't edit it, simply just moved the necessary parts.

<!-- TOC depthFrom:1 depthTo:3 withLinks:1 updateOnSave:1 orderedList:0 -->

- [1. Python Language Rules](#LanguageRule)
    - [1.1 import](#imports)
    - [1.2 Exceptions](#Exceptions)
    - [1.3 Global variables](#Globalvar)
    - [1.4 Default Argument Values](#DefaultArgs)
    - [1.5 True/False Evaluations](#TrueFalse)


- [2 Python Style Rules](#Style)
    - [2.1 Parentheses](#Parentheses)
    - [2.2 Indentation](#Indentation)
    - [2.3 Whitespace](#Whitespace)
    - [2.4 Comments and Docstrings](#CommentsDocstring)
    - [2.5 Strings](#Strings)
    - [2.6 Imports formatting](#ImportFormatting)
    - [2.7 Naming](#Naming)
    - [2.8 Function length](#FunctionLength)

<!-- /TOC -->

##  <a name="LanguageRule"></a> 1. Python Language Rules

### <a name="imports"></a>1.1 imports

Use import statements for packages and modules only, not for individual classes or functions. 

#### 1.1.1 Definition
Reusability mechanism for sharing code from one module to another.

#### 1.1.2 Pros
The namespace management convention is simple. The source of each identifier is indicated in a consistent way; x.Obj says that object Obj is defined in module x.

#### 1.1.3 Cons
Module names can still collide. Some module names are inconveniently long.

#### 1.1.4 Decision

Use import x for importing packages and modules.
Use from x import y where x is the package prefix and y is the module name with no prefix.
Use from x import y as z if two modules named y are to be imported or if y is an inconveniently long name.
Use import y as z only when z is a standard abbreviation (e.g., np for numpy).

#### 1.1.5 Examples

For example the module sound.effects.echo may be imported as follows:

```python
YES:
    import elasticsearch as es
    from sound.effects import echo
    ...
    es_client = es.Elasticsearch({"host":"localhost", "port":"8888"})
    
```

```python
NO:
    from elasticsearch import Elasticsearch    
...
    es_client = Elasticsearch({"host":"localhost", "port":"8888"})
    
    

```
Do not use relative names in imports. Even if the module is in the same package, use the full package name. This helps prevent unintentionally importing a package twice.


### <a name="Exceptions"></a> 1.2 Exceptions

Exceptions are allowed but must be used carefully.

#### 1.2.1 Definition
Exceptions are a means of breaking out of normal control flow to handle errors or other exceptional conditions.

#### 1.2.2 Pros

The control flow of normal operation code is not cluttered by error-handling code. It also allows the control flow to skip multiple frames when a certain condition occurs, e.g., returning from N nested functions in one step instead of having to plumb error codes through.

#### 1.2.3 Cons

May cause the control flow to be confusing. Easy to miss error cases when making library calls.

#### 1.2.4 Decision

Exceptions must follow certain conditions:

Make use of built-in exception classes when it makes sense. For example, 
raise a ValueError to indicate a programming mistake like a violated precondition 
(such as if you were passed a negative number but required a positive one). 
Do not use assert statements for validating argument values of a public API. assert 
is used to ensure internal correctness, not to enforce correct usage nor to indicate 
that some unexpected event occurred. If an exception is desired in the latter cases, 
use a raise statement. For example:
```python
Yes:
  def connect_to_next_port(self, minimum: int) -> int:
    """Connects to the next available port.

    Args:
      minimum: A port value greater or equal to 1024.

    Returns:
      The new minimum port.

    Raises:
      ConnectionError: If no available port is found.
    """
    if minimum < 1024:
      # Note that this raising of ValueError is not mentioned in the doc
      # string's "Raises:" section because it is not appropriate to
      # guarantee this specific behavioral reaction to API misuse.
      raise ValueError(f'Min. port must be at least 1024, not {minimum}.')
    port = self._find_next_open_port(minimum)
    if not port:
      raise ConnectionError(
          f'Could not connect to service on port {minimum} or higher.')
    assert port >= minimum, (
        f'Unexpected port {port} when minimum was {minimum}.')
    return port

```

```python
No:
  def connect_to_next_port(self, minimum: int) -> int:
    """Connects to the next available port.

    Args:
      minimum: A port value greater or equal to 1024.

    Returns:
      The new minimum port.
    """
    assert minimum >= 1024, 'Minimum port must be at least 1024.'
    port = self._find_next_open_port(minimum)
    assert port is not None
    return port
```

Libraries or packages may define their own exceptions. When doing so they must inherit 
from an existing exception class. Exception names should end in Error and should not 
introduce repetition (foo.FooError).

Never use catch-all except: statements, or catch Exception or StandardError, unless you are

- re-raising the exception, or
- creating an isolation point in the program where exceptions are not propagated but 
  are recorded and suppressed instead, such as protecting a thread from crashing by 
  guarding its outermost block.

Python is very tolerant in this regard and except: will really catch everything 
including misspelled names, sys.exit() calls, Ctrl+C interrupts, unittest failures 
and all kinds of other exceptions that you simply don’t want to catch.

Minimize the amount of code in a try/except block. The larger the body of the try, 
the more likely that an exception will be raised by a line of code that you didn’t 
expect to raise an exception. In those cases, the try/except block hides a real error.

Use the finally clause to execute code whether or not an exception is raised in 
the try block. This is often useful for cleanup, i.e., closing a file.



### <a name="Globalvar"></a> 1.3 Global variables
Avoid global variables.

#### 1.3.1 Definition

Variables that are declared at the module level or as class attributes.

#### 1.3.2 Pros

Occasionally useful.

#### 1.3.3 Cons

Has the potential to change module behavior during the import, because assignments 
to global variables are done when the module is first imported.

#### 1.3.4 Decision

Avoid global variables.

While they are technically variables, module-level constants are permitted and 
encouraged. For example: _MAX_HOLY_HANDGRENADE_COUNT = 3. Constants must be named 
using all caps with underscores. See Naming below.

If needed, globals should be declared at the module level and made internal to the 
module by prepending an _ to the name. External access must be done through public 
module-level functions. See Naming below.



### <a name="DefaultArgs"></a> 1.4 Default Argument Values

Okay in most cases.

#### 1.4.1 Definition

You can specify values for variables at the end of a function’s parameter list, 
e.g., def foo(a, b=0):. If foo is called with only one argument, b is set to 0. 
If it is called with two arguments, b has the value of the second argument.

#### 1.4.2 Pros

Often you have a function that uses lots of default values, but on rare occasions 
you want to override the defaults. Default argument values provide an easy way to do 
this, without having to define lots of functions for the rare exceptions. 
As Python does not support overloaded methods/functions, default arguments are 
an easy way of “faking” the overloading behavior.

#### 1.4.3 Cons

Default arguments are evaluated once at module load time. 
This may cause problems if the argument is a mutable object such as a list or a dictionary. 
If the function modifies the object (e.g., by appending an item to a list), the default 
value is modified.

#### 1.4.4 Decision

Okay to use with the following caveat:

Do not use mutable objects as default values in the function or method definition.

```python
Yes: def foo(a, b=None):
         if b is None:
             b = []
Yes: def foo(a, b: Optional[Sequence] = None):
         if b is None:
             b = []
Yes: def foo(a, b: Sequence = ()):  # Empty tuple OK since tuples are immutable
         ...
```

```python
No:  def foo(a, b=[]):
         ...
No:  def foo(a, b=time.time()):  # The time the module was loaded???
         ...
No:  def foo(a, b=FLAGS.my_thing):  # sys.argv has not yet been parsed...
         ...
No:  def foo(a, b: Mapping = {}):  # Could still get passed to unchecked code
         ...
```



### <a name="TrueFalse"></a> 1.5 True/False Evaluations

Use the “implicit” false if at all possible.

#### 1.5.1 Definition

Python evaluates certain values as False when in a boolean context. 
A quick “rule of thumb” is that all “empty” values are considered false, 
so 0, None, [], {}, '' all evaluate as false in a boolean context.

#### 1.5.2 Pros

Conditions using Python booleans are easier to read and less error-prone. 
In most cases, they’re also faster.

#### 1.5.3 Cons

May look strange to C/C++ developers.

#### 1.5.4 Decision

Use the “implicit” false if possible, e.g., if foo: rather than if foo != []:. 
There are a few caveats that you should keep in mind though:

Always use if foo is None: (or is not None) to check for a None value. E.g., 
when testing whether a variable or argument that defaults to None was set to some 
other value. The other value might be a value that’s false in a boolean context!

Never compare a boolean variable to False using ==. Use if not x: instead. 
If you need to distinguish False from None then chain the expressions, 
such as if not x and x is not None:.

For sequences (strings, lists, tuples), use the fact that empty sequences are false, 
so if seq: and if not seq: are preferable to if len(seq): and if not len(seq): respectively.

When handling integers, implicit false may involve more risk than benefit 
(i.e., accidentally handling None as 0). You may compare a value which is known 
to be an integer (and is not the result of len()) against the integer 0.


```python
Yes: if not users:
         print('no users')

     if i % 10 == 0:
         self.handle_multiple_of_ten()

     def f(x=None):
         if x is None:
             x = []

```

```python
No:  if len(users) == 0:
         print('no users')

     if not i % 10:
         self.handle_multiple_of_ten()

     def f(x=None):
         x = x or []
```

Note that '0' (i.e., 0 as string) evaluates to true.

Note that Numpy arrays may raise an exception in an implicit boolean context. 
Prefer the .size attribute when testing emptiness of a np.array (e.g. if not users.size).


## <a name="Style"></a> 2 Python Style Rules


### <a name="Parentheses"></a> 2.1 Parentheses
Use parentheses sparingly.

It is fine, though not required, to use parentheses around tuples. Do not use them in return statements or conditional statements unless using parentheses for implied line continuation or to indicate a tuple.
```python
Yes: if foo:
         bar()
     while x:
         x = bar()
     if x and y:
         bar()
     if not x:
         bar()
     # For a 1 item tuple the ()s are more visually obvious than the comma.
     onesie = (foo,)
     return foo
     return spam, beans
     return (spam, beans)
     for (x, y) in dict.items(): ...
```

```python
No:  if (x):
         bar()
     if not(x):
         bar()
     return (foo)

```
     
     

### <a name="Indentation"></a> 2.2 Indentation
Indent your code blocks with 4 spaces.

Never use tabs or mix tabs and spaces. In cases of implied line continuation, you should align wrapped elements either vertically, as per the examples in the line length section; or using a hanging indent of 4 spaces, in which case there should be nothing after the open parenthesis or bracket on the first line.
```python
Yes:   # Aligned with opening delimiter
       foo = long_function_name(var_one, var_two,
                                var_three, var_four)
       meal = (spam,
               beans)

       # Aligned with opening delimiter in a dictionary
       foo = {
           'long_dictionary_key': value1 +
                                  value2,
           ...
       }

       # 4-space hanging indent; nothing on first line
       foo = long_function_name(
           var_one, var_two, var_three,
           var_four)
       meal = (
           spam,
           beans)

       # 4-space hanging indent in a dictionary
       foo = {
           'long_dictionary_key':
               long_dictionary_value,
           ...
       }
```

```python
No:    # Stuff on first line forbidden
       foo = long_function_name(var_one, var_two,
           var_three, var_four)
       meal = (spam,
           beans)

       # 2-space hanging indent forbidden
       foo = long_function_name(
         var_one, var_two, var_three,
         var_four)

       # No hanging indent in a dictionary
       foo = {
           'long_dictionary_key':
           long_dictionary_value,
           ...
       }
```

       
### <a name="Whitespace"></a> 2.3 Whitespace
Follow standard typographic rules for the use of spaces around punctuation.

No whitespace inside parentheses, brackets or braces.

```python
Yes: spam(ham[1], {'eggs': 2}, [])
```
```python
No:  spam( ham[ 1 ], { 'eggs': 2 }, [ ] )
```

No whitespace before a comma, semicolon, or colon. Do use whitespace after a comma, 
semicolon, or colon, except at the end of the line.
```python
Yes: if x == 4:
         print(x, y)
     x, y = y, x
```

```python
No:  if x == 4 :
         print(x , y)
     x , y = y , x
```

No whitespace before the open paren/bracket that starts an argument list, indexing or slicing.
```python
Yes: spam(1)
```
```python
No:  spam (1)
```

```python
Yes: dict['key'] = list[index]
```
```
No:  dict ['key'] = list [index]
```



No trailing whitespace.

Surround binary operators with a single space on either side for assignment (=), 
comparisons (==, <, >, !=, <>, <=, >=, in, not in, is, is not), and Booleans (and, or, not). 
Use your better judgment for the insertion of spaces around arithmetic operators (+, -, *, /, //, %, **, @).

```python
Yes: x == 1
```

```python
No:  x<1
```

Never use spaces around = when passing keyword arguments or defining a default parameter 
value, with one exception: when a type annotation is present, 
do use spaces around the = for the default parameter value.
```python
Yes: def complex(real, imag=0.0): return Magic(r=real, i=imag)
Yes: def complex(real, imag: float = 0.0): return Magic(r=real, i=imag)
```

```python
No:  def complex(real, imag = 0.0): return Magic(r = real, i = imag)
No:  def complex(real, imag: float=0.0): return Magic(r = real, i = imag)
```

Don’t use spaces to vertically align tokens on consecutive lines, 
since it becomes a maintenance burden (applies to :, #, =, etc.):
```python
Yes:
  foo = 1000  # comment
  long_name = 2  # comment that should not be aligned

  dictionary = {
      'foo': 1,
      'long_name': 2,
  }
```

```python
No:
  foo       = 1000  # comment
  long_name = 2     # comment that should not be aligned

  dictionary = {
      'foo'      : 1,
      'long_name': 2,
  }

```
  
### <a name="CommentsDocstring"></a> 2.4 Comments and Docstrings
Be sure to use the right style for module, function, method docstrings and inline comments.

#### 2.4.1 Docstrings
Python uses docstrings to document code. A docstring is a string that is the first statement in a package, module, class or function. These strings can be extracted automatically through the __doc__ member of the object and are used by pydoc. (Try running pydoc on your module to see how it looks.) Always use the three double-quote """ format for docstrings (per PEP 257). A docstring should be organized as a summary line (one physical line not exceeding 80 characters) terminated by a period, question mark, or exclamation point. When writing more (encouraged), this must be followed by a blank line, followed by the rest of the docstring starting at the same cursor position as the first quote of the first line. There are more formatting guidelines for docstrings below.

#### 2.4.2 Modules
Every file should contain license boilerplate. 
Choose the appropriate boilerplate for the license used by the project (for example, Apache 2.0, BSD, LGPL, GPL)

Files should start with a docstring describing the contents and usage of the module.

```
"""A one line summary of the module or program, terminated by a period.

Leave one blank line.  The rest of this docstring should contain an
overall description of the module or program.  Optionally, it may also
contain a brief description of exported classes and functions and/or usage
examples.

  Typical usage example:

  foo = ClassFoo()
  bar = foo.FunctionBar()
"""
```

#### 2.4.3 Functions and Methods

In this section, “function” means a method, function, or generator.

A function must have a docstring, unless it meets all of the following criteria:

not externally visible
very short
obvious
A docstring should give enough information to write a call to the function without 
reading the function’s code. The docstring should describe the function’s calling 
syntax and its semantics, but generally not its implementation details, unless those 
details are relevant to how the function is to be used. For example, a function that 
mutates one of its arguments as a side effect should note that in its docstring. 
Otherwise, subtle but important details of a function’s implementation that are not 
relevant to the caller are better expressed as comments alongside the code than within 
the function’s docstring.

The docstring should be descriptive-style ("""Fetches rows from a Bigtable.""") 
rather than imperative-style ("""Fetch rows from a Bigtable."""). 
The docstring for a @property data descriptor should use the same style as the 
docstring for an attribute or a function argument ("""The Bigtable path.""", 
rather than """Returns the Bigtable path.""").

A method that overrides a method from a base class may have a simple docstring 
sending the reader to its overridden method’s docstring, such as """See base class.""". 
The rationale is that there is no need to repeat in many places documentation 
that is already present in the base method’s docstring. However, if the overriding 
method’s behavior is substantially different from the overridden method, or details 
need to be provided (e.g., documenting additional side effects), a docstring with 
at least those differences is required on the overriding method.

Certain aspects of a function should be documented in special sections, listed 
below. Each section begins with a heading line, which ends with a colon. 
All sections other than the heading should maintain a hanging indent of two 
or four spaces (be consistent within a file). These sections can be omitted in 
cases where the function’s name and signature are informative enough that it can 
be aptly described using a one-line docstring.

- Args:
  
  List each parameter by name. A description should follow the name, and be separated 
  by a colon followed by either a space or newline. If the description is too long to 
  fit on a single 80-character line, use a hanging indent of 2 or 4 spaces more than 
  the parameter name (be consistent with the rest of the docstrings in the file). 
  The description should include required type(s) if the code does not contain a 
  corresponding type annotation. If a function accepts *foo (variable length argument lists) 
  and/or **bar (arbitrary keyword arguments), they should be listed as *foo and **bar.


- Returns: (or Yields: for generators)

  Describe the type and semantics of the return value. 
  If the function only returns None, this section is not required. 
  It may also be omitted if the docstring starts with Returns or Yields 
  (e.g. """Returns row from Bigtable as a tuple of strings.""") and the opening 
  sentence is sufficient to describe the return value. Do not imitate ‘NumPy style’ 
  (example), which frequently documents a tuple return value as if it were multiple 
  return values with individual names (never mentioning the tuple). Instead, 
  describe such a return value as: “Returns a tuple (mat_a, mat_b), where mat_a is …, 
  and …”. The auxiliary names in the docstring need not necessarily correspond to 
  any internal names used in the function body (as those are not part of the API).


- Raises:
  List all exceptions that are relevant to the interface followed by a description. 
  Use a similar exception name + colon + space or newline and hanging indent style 
  as described in Args:. You should not document exceptions that get raised if the 
  API specified in the docstring is violated (because this would paradoxically make 
  behavior under violation of the API part of the API).
  
```python
def fetch_smalltable_rows(table_handle: smalltable.Table,
                          keys: Sequence[Union[bytes, str]],
                          require_all_keys: bool = False,
) -> Mapping[bytes, Tuple[str, ...]]:
    """Fetches rows from a Smalltable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by table_handle.  String keys will be UTF-8 encoded.

    Args:
        table_handle: An open smalltable.Table instance.
        keys: A sequence of strings representing the key of each table
          row to fetch.  String keys will be UTF-8 encoded.
        require_all_keys: If True only rows with values set for all keys will be
          returned.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {b'Serak': ('Rigel VII', 'Preparer'),
         b'Zim': ('Irk', 'Invader'),
         b'Lrrr': ('Omicron Persei 8', 'Emperor')}

        Returned keys are always bytes.  If a key from the keys argument is
        missing from the dictionary, then that row was not found in the
        table (and require_all_keys must have been False).

    Raises:
        IOError: An error occurred accessing the smalltable.
    """
```

Similarly, this variation on Args: with a line break is also allowed:

```python
def fetch_smalltable_rows(table_handle: smalltable.Table,
                          keys: Sequence[Union[bytes, str]],
                          require_all_keys: bool = False,
) -> Mapping[bytes, Tuple[str, ...]]:
    """Fetches rows from a Smalltable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by table_handle.  String keys will be UTF-8 encoded.

    Args:
      table_handle:
        An open smalltable.Table instance.
      keys:
        A sequence of strings representing the key of each table row to
        fetch.  String keys will be UTF-8 encoded.
      require_all_keys:
        If True only rows with values set for all keys will be returned.

    Returns:
      A dict mapping keys to the corresponding table row data
      fetched. Each row is represented as a tuple of strings. For
      example:

      {b'Serak': ('Rigel VII', 'Preparer'),
       b'Zim': ('Irk', 'Invader'),
       b'Lrrr': ('Omicron Persei 8', 'Emperor')}

      Returned keys are always bytes.  If a key from the keys argument is
      missing from the dictionary, then that row was not found in the
      table (and require_all_keys must have been False).

    Raises:
      IOError: An error occurred accessing the smalltable.
    """

```

#### 2.4.4 Classes
Classes should have a docstring below the class definition describing the class. 
If your class has public attributes, they should be documented here in an Attributes 
section and follow the same formatting as a function’s Args section.

```python
class SampleClass:
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam: bool = False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""

```


#### 2.4.5 Block and Inline Comments
The final place to have comments is in tricky parts of the code. 
If you’re going to have to explain it at the next code review, 
you should comment it now. Complicated operations get a few lines of comments before 
the operations commence. Non-obvious ones get comments at the end of the line.

```
# We use a weighted dictionary search to find out where i is in
# the array.  We extrapolate position based on the largest num
# in the array and the array size and then do binary search to
# get the exact number.

if i & (i-1) == 0:  # True if i is 0 or a power of 2.
To improve legibility, these comments should start at least 2 spaces away from the code with the comment character #, followed by at least one space before the text of the comment itself.

On the other hand, never describe the code. Assume the person reading the code knows Python (though not what you’re trying to do) better than you do.

# BAD COMMENT: Now go through the b array and make sure whenever i occurs
# the next element is i+1
```

   
### <a name="Strings"></a> 2.5 Strings

- Do not use % or the format method for pure concatenation.
- Avoid using the + and += operators to accumulate a string within a loop,  Instead, add each substring to a list and ''.join
- Prefer """ for multi-line strings rather than '''

### <a name="ImportFormatting"></a> 2.6 Imports formatting
Imports should be on separate lines; there are exceptions for typing imports.

E.g.:
```python
Yes: import os
     import sys
     from typing import Mapping, Sequence
```

```python
No:  import os, sys
```

Imports are always put at the top of the file, just after any module comments and 
docstrings and before module globals and constants. Imports should be grouped 
from most generic to least generic:


Python standard library imports. For example:

```python
import sys
```

third-party module or package imports. For example:
```python
import tensorflow as tf
```

Code repository sub-package imports. For example:
```python
from otherproject.ai import mind
```

Deprecated: application-specific imports that are part of the same top level 
sub-package as this file. For example:

```python
from myproject.backend.hgwells import time_machine
```

You may find older Google Python Style code doing this, but it is no longer required. 
New code is encouraged not to bother with this. Simply treat application-specific 
sub-package imports the same as other sub-package imports.

Within each grouping, imports should be sorted lexicographically, ignoring case, 
according to each module’s full package path (the path in from path import ...). 
Code may optionally place a blank line between import sections.

```python
import collections
import queue
import sys

from absl import app
from absl import flags
import bs4
import cryptography
import tensorflow as tf

from book.genres import scifi
from myproject.backend import huxley
from myproject.backend.hgwells import time_machine
from myproject.backend.state_machine import main_loop
from otherproject.ai import body
from otherproject.ai import mind
from otherproject.ai import soul
```


```
#Older style code may have these imports down here instead:
#from myproject.backend.hgwells import time_machine
#from myproject.backend.state_machine import main_loop
```


### <a name="Naming"></a> 2.7 Naming
module_name, package_name, ClassName, method_name, ExceptionName, function_name, 
GLOBAL_CONSTANT_NAME, global_var_name, instance_var_name, function_parameter_name, 
local_var_name.

Function names, variable names, and filenames should be descriptive; 
eschew abbreviation. In particular, do not use abbreviations that are ambiguous 
or unfamiliar to readers outside your project, and do not abbreviate by deleting 
letters within a word.

Always use a .py filename extension. Never use dashes.

#### 2.7.1 Names to Avoid
- single character names, except for specifically allowed cases:

  - counters or iterators (e.g. i, j, k, v, et al.)
  - e as an exception identifier in try/except statements.
  - f as a file handle in with statements

  Please be mindful not to abuse single-character naming. Generally speaking, descriptiveness 
  should be proportional to the name’s scope of visibility. 
  For example, i might be a fine name for 5-line code block but within multiple nested scopes, 
  it is likely too vague.

- dashes (-) in any package/module name

- __double_leading_and_trailing_underscore__ names (reserved by Python)

- offensive terms

- names that needlessly include the type of the variable (for example: id_to_name_dict)


#### 2.7.2 Naming Conventions
- “Internal” means internal to a module, or protected or private within a class.

- Prepending a single underscore (_) has some support for protecting module 
variables and functions (linters will flag protected member access).

- Prepending a double underscore (__ aka “dunder”) to an instance variable or method 
  effectively makes the variable or method private to its class (using name mangling); 
  we discourage its use as it impacts readability and testability, and isn’t really private. 
  Prefer a single underscore.

- Place related classes and top-level functions together in a module. Unlike Java, 
  there is no need to limit yourself to one class per module.

- Use CapWords for class names, but lower_with_under.py for module names. 
  Although there are some old modules named CapWords.py, this is now discouraged 
  because it’s confusing when the module happens to be named after a class. 
  (“wait – did I write import StringIO or from StringIO import StringIO?”)

- Underscores may appear in unittest method names starting with test to separate 
  logical components of the name, even if those components use CapWords. One possible 
  pattern is test<MethodUnderTest>_<state>; for example testPop_EmptyStack is okay. 
  There is no One Correct Way to name test methods.



#### 2.7.3 File Naming
Python filenames must have a .py extension and must not contain dashes (-). 
This allows them to be imported and unittested. 
If you want an executable to be accessible without the extension, use a symbolic 
link or a simple bash wrapper containing exec "$0.py" "$@".


#### <a name="FunctionLength"></a> 2.8 Function length
Prefer small and focused functions.

We recognize that long functions are sometimes appropriate, so no hard limit is 
placed on function length. If a function exceeds about 40 lines, think about whether 
it can be broken up without harming the structure of the program.

Even if your long function works perfectly now, someone modifying it in a few 
months may add new behavior. This could result in bugs that are hard to find. 
Keeping your functions short and simple makes it easier for other people to read 
and modify your code.

You could find long and complicated functions when working with some code. 
Do not be intimidated by modifying existing code: if working with such a function 
proves to be difficult, you find that errors are hard to debug, or you want to use 
a piece of it in several different contexts, consider breaking up the function 
into smaller and more manageable pieces.
