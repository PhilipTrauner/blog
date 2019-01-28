# Python Tips: Generator unrolling

Generators are computed iterables, which only require a fraction of the space that would normally be necessary to store a fully populated collection in memory. 

While this property is generally beneficial, they also bring along some ergonomic problems:

* No index operator support
* Cloning is problematic (see [`itertools.tee`](https://docs.python.org/3/library/itertools.html#itertools.tee))
* Results have to be stored in an auxiliary collection if caching is desired
* Debugging misbehaving generators is non-trivial

The most common way to deal with these shortcomings is to just unroll the generator into a collection if the element count yielded by the generator is manageable. There are many ways to accomplish this, but all of them have different strengths and shortcomings.

<details><summary>Benchmark code</summary>

```python
from timeit import timeit
from functools import partial
from array import array
from sys import getsizeof

RUNS = 1000
RANGE = 10000

for exprs, desc in (
    (lambda gen: list(gen), "List constructor"),
    (
        lambda gen: (lambda gen, list: [list.append(num) for num in gen])(gen, list()),
        "List append",
    ),
    (lambda gen: [num for num in gen], "List comprehension"),
    (lambda gen: tuple(gen), "Tuple constructor"),
    (lambda gen: set(gen), "Set constructor"),
    (lambda gen: frozenset(gen), "Frozenset constructor"),
    (lambda gen: {num: None for num in gen}, "Dictionary nonsense"),
    (lambda gen: array("h", gen), "Array constructor"),
    (
        lambda gen: (lambda gen, arr: arr.extend(gen) or arr)(gen, array("h")),
        "Array extension",
    ),
    (
        lambda gen: (lambda gen, arr: [arr.append(num) for num in gen])(
            gen, array("h")
        ),
        "Array append",
    ),
):
    bound_expr = partial(exprs, (lambda: range(RANGE))())
    print(f"{timeit(bound_expr, number=RUNS):.5f}s: {desc} ({getsizeof(bound_expr())})")
```

</details>

|Method|Description|Speed (sec)|Size (byte)|
|------|-----------|:---------:|----------:|
|`list(gen)`|List constructor|0.27175|90112|
|`tuple(gen)`|Tuple constructor|0.35145|80048|
|`frozenset(gen)`|Frozenset constructor|0.47581|524512|
|`set(gen)`|Set constructor|0.47707|524512|
|`[num for num in gen]`|List comprehension|0.52155|87624|
|`{num: None for num in gen}`|Dictionary nonsense|0.74752|295008|
|`array("h", gen)`|Array constructor (`from array import array`)|0.90895|20234|
|`arr.extend(gen)`|Array extension|0.94037|20234|
|`[list.append(num) for num in gen]`|List append|1.19855|87624|
|`[arr.append(num) for num in gen]`|Array append|1.77882|87624|


## Conclusion
* The list constructor is the definitive all-rounder and the best choice in most cases. 
* Typed arrays shine when memory usage is key.
* The tuple constructor is about 20% slower than the list constructor, while taking up ~10% less space.
