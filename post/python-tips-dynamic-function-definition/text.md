# Python Tips: Dynamic function definition

In Python, there isn't any syntactic sugar that eases function definition during runtime. However, that doesn't mean that its impossible or even difficult.

```python
from types import FunctionType

foo_code = compile('def foo(): return "bar"', "<string>", "exec")
foo_func = FunctionType(foo_code.co_consts[0], globals(), "foo")

print(foo_func())
```

<details open><summary>Output</summary>
```
bar
```
</details>

## Dissection
Stepping through the code line-by-line reveals how thin the language / interpreter barrier really is.

```pycon
>>> from types import FunctionType
```

The Python docs often do not list the signatures of classes that aren't meant to be created manually (which is completely reasonable). 
There are three ways to work around this: [`help()`](https://docs.python.org/3.7/library/functions.html#help), [`inspect`](https://docs.python.org/3/library/inspect.html) (which can't examine built-in functions), and the fallback solution of looking at the [CPython](https://github.com/python/cpython/) source code. 
In this instance both `help()` and `inspect` do the job, but looking at the actual [source code](https://github.com/python/cpython/blob/5bb146aaea1484bcc117ab6cb38dda39ceb5df0f/Objects/funcobject.c#L458) reveals additional details in regards to data-types.

```pycon
>>> from inspect import signature
>>> signature(FunctionType)
<Signature (code, globals, name=None, argdefs=None, closure=None)>
```

1. `code`  
	Internally a [`PyCodeObject`](https://github.com/python/cpython/blob/master/Objects/codeobject.c), which is exposed as a [`types.CodeType`](https://docs.python.org/3.7/library/types.html#types.CodeType). 
	Non-built-in functions have a `__code__` attribute which holds their corresponding code object. 
	[`types.CodeType`](https://docs.python.org/3.7/library/types.html#types.CodeType) objects can be created at runtime by utilizing the [`compile()`](https://docs.python.org/3/library/functions.html#compile) built-in.
2. `globals`  
	If a variable that is being referenced in a function isn't defined locally, passed in as a parameter, provided by a default argument value, or supplied through a closure context, it is looked up in the `globals` dictionary. 
	The `globals()` built-in function returns a **reference** to the global symbol table of the current module, and can therefor be used to supply a dictionary that is always up-to-date with the current state of said table. Passing in any other dictionary works as well (`FunctionType((lambda: bar).__code__, {"bar" : "baz"}, "foo")() == "baz"`).
3. `name` (optional)  
	Controls the `__name__` attribute of the returned function. Only really useful for lambdas (due to their anonymous nature they normally don't have names), and renaming functions.
4. `argdefs` (optional)  
	Provides a way to supply default argument values (`def foo(bar="baz")`) by passing in a [`tuple`](https://docs.python.org/3.7/library/stdtypes.html#tuple) that contains objects of arbitrary type. (`FunctionType((lambda bar: bar).__code__, {}, "foo", (10,))() == 10`).  
5. `closure` (optional)  
	(Probably shouldn't be touched if execution in any Python VM other than CPython (PyPy, Jython, ...) is desired, as it heavily relies on implementation details)
	A `tuple` of [`cell`](https://github.com/python/cpython/blob/master/Objects/cellobject.c) objects. [Creating](https://github.com/PhilipTrauner/exalt/blob/846763f24c9e09e578ea24216f08e4268eb71bc0/exalt/__init__.py#L43) `cell` objects isn't exactly straight forward, because calling into [CPython internals](https://github.com/python/cpython/blob/5bb146aaea1484bcc117ab6cb38dda39ceb5df0f/Objects/cellobject.c#L9) is required, but there is a library that makes this more convenient: [exalt](https://github.com/PhilipTrauner/exalt) (shameless advertisement). 

```pycon
>>> foo_code = compile('def foo(): return "bar"', "<string>", "exec")
```

`compile()` is a built-in function, and therefor also well [documented](https://docs.python.org/3.7/library/functions.html#compile). 

`exec` mode is utilized, because multiple statements are necessary to define a function.

```pycon
>>> foo_func = FunctionType(foo_code.co_consts[0], globals(), "foo")
```

Bringing it all together and assigning the dynamically created function to a variable. 
The function that was compiled in the previous statement becomes the first constant of the generated code object, therefor just referring to `foo_code` is insufficient. This is a direct consequence of `exec` mode, because the resulting code object can contain multiple constants.  

```pycon
>>> print(foo_func())
```

The dynamically created function can be called just like any other function.

## Conclusion
* There are very few use-cases for dynamic function creation outside of experimentation.
* Toying around with Python internals is a great way to learn more about the language.
* The interpreter / language boundary can be crossed rather effortlessly if desired.

And as always: **Don’t abuse the language** (well, a little can’t hurt, right?)
