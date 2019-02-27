# Python Quirks: Implicit Return

In Python, functions **always** have to return something.

```pycon
>>> def foo():
...     pass
...
>>> print(foo())
None
```

To ensure that this is the case, instructions that are equivalent to a `return None` statement are [appended to the inner-most code block](https://github.com/python/cpython/blob/16323cb2c3d315e02637cebebdc5ff46be32ecdf/Python/compile.c#L5861) by the bytecode compiler if no return statement is present.

```pycon
>>> from dis import dis
>>> dis(foo)
  2           0 LOAD_CONST               0 (None)
              2 RETURN_VALUE
```
<figcaption>Disassembled <code>foo</code> function</figcaption>

Enforcing this requirement on an individual function level instead of inside the [evaluation main loop](https://github.com/python/cpython/blob/234531b4462b20d668762bd78406fd2ebab129c9/Python/ceval.c) necessitates that all functions follow this protocol. Let's explore what happens when they **don't**.

## 📚 Incomprehensive overview of Python bytecode 
The bytecode of existing functions can be examined by looking at their `__code__.co_code` attribute. 

```pycon
>>> foo.__code__.co_code
b'd\x00S\x00'
```

Each instruction is 2 bytes long (`BB`) and consists of an opcode and an optional argument value. [`struct.unpack`](https://docs.python.org/3/library/struct.html#struct.unpack) can be used to decode individual instructions, and a human readable representation can be obtained through [`dis.opmap`](https://docs.python.org/3.7/library/dis.html#dis.opmap) (re-export of the undocumented `opcode.opmap`), which maps the opcode names to their respective numerical values. Swapping key and value subsequently provides a numerical value to text mapping.

```python
from struct import unpack
from dis import opmap

reverse_opmap = {v: k for k, v in opmap.items}


def foo():
    pass


foo_code = foo.__code__.co_code
for pos in range(0, len(foo_code), 2):
    inst = unpack("BB", foo_code[pos : pos + 2])
    print(f"{reverse_opmap[inst[0]]}: {inst[1]}")
```
<details open><summary>Output</summary>
```
LOAD_CONST: 0
RETURN_VALUE: 0
```
</details>

The bytecode section that instructs the VM to return `None` is 4 bytes long:  

1. `LOAD_CONST`(0)
    
    Loads the constant stored at index 0 of `__code__.co_consts` (which is `None` in this case) into memory.  

2. `RETURN_VALUE`(0)
    
    Returns the previously loaded constant (argument-less).

## 🤞 Crossing fingers

Stripping out the 4 bytes, replacing the `__code__` attribute of the `foo` function, and executing the resulting function does not crash the VM (surprisingly). Instead, a [`SystemError` is raised](https://github.com/python/cpython/blob/11c79531655a4aa3f82c20ff562ac571f40040cc/Python/ceval.c#L3430) and an accompanying [error message is printed to `stderr`](https://github.com/python/cpython/blob/11c79531655a4aa3f82c20ff562ac571f40040cc/Python/ceval.c#L3426).

```python
from types import FunctionType, CodeType


def foo():
    pass


foo_code = foo.__code__
foo.__code__ = CodeType(
    foo_code.co_argcount,
    foo_code.co_kwonlyargcount,
    foo_code.co_nlocals,
    foo_code.co_stacksize,
    foo_code.co_flags,
    foo_code.co_code[:-4],
    foo_code.co_consts,
    foo_code.co_names,
    foo_code.co_varnames,
    "",
    foo_code.co_name,
    1,
    b"",
)

foo()
```
<details open><summary>Output</summary>
```
XXX lineno: 1, opcode: 0
Traceback (most recent call last):
  File "/Users/philip/Developer/testing/python-implicit-return.py", line 25, in <module>
    foo()
  File "", line 1, in foo
SystemError: unknown opcode
```
</details>

`SystemError` is a regular runtime error (despite its name), and can therefor be caught with a regular `try` / `except` clause.

```pycon
>>> try:
...     foo()
... except SystemError:
...     pass
```

The accompanying error message (`XXX lineno: 1, opcode: 0`) can't be silence from within Python, because the output stream to which it is written is hard-coded to `stderr`. That doesn't mean that its impossible, though.

```bash
python3 <file> 2> >(sed "/^XXX lineno: [^,]*, opcode: [^\n]*/d" >&2)
```

## 🏁 Conclusion
**Don't** do this.