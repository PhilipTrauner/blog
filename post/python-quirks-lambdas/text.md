# Python Quirks: Lambdas

> The crippled nature of Python's lambdas are a big weakness of Python. They sit in a no-mans land of providing half a solution but being too crippled to actually ~~to~~ *improve* clarity / ~~improve~~ programming style.
>
> — [zmmmmm](https://news.ycombinator.com/item?id=18089917)

Comments like this one about Python's lambdas are [very](http://treyhunner.com/2018/09/stop-writing-lambda-expressions/), [very](http://web.archive.org/web/20140711172717/http://symbo1ics.com/blog/?p=1292), [very](https://news.ycombinator.com/item?id=18088758) common. Even Guido himself isn't particularly fond of lambdas, as demonstrated by his suggestion to [remove them](https://www.artima.com/weblogs/viewpost.jsp?thread=98196) outright.

With all the negativity surrounding this admittedly underpowered language feature it is worth exploring why the capabilities of lambdas are so limited, and how to make use of them.

## History lesson
> About 12 years ago, Python ~~aquired~~ acquired lambda, reduce(), filter() and map(), courtesy of (I believe) a Lisp hacker who missed them and submitted working patches. 
>
> — [Guido van Rossum](https://www.artima.com/weblogs/viewpost.jsp?thread=98196) (2005)

`lambda_input` was first introduced into the grammar on [October 26th 1993](https://github.com/python/cpython/blob/12d12c5faf4d770160b7975b54e8f9b12694e012/Grammar/Grammar#L88). At this point `lambda` wasn't a keyword yet, but instead a function that would consume a `vararglist` (parameter list) and `testlist` (basically an expression) as a string. 

```python
lambd​a('x: x + 1')
```

About [one month](https://github.com/python/cpython/blob/590baa4a7a43b596119b47f605e3e570c2b3b0ee/Grammar/Grammar#L141) later it finally became a keyword and has remained largely unchanged to this day.

## Pythonic?
> For example, today someone claimed to have solved the problem of the multi-statement lambda.
> But such solutions often lack "Pythonicity" [...]  
> 
> — [Guido van Rossum](https://www.artima.com/weblogs/viewpost.jsp?thread=147358) (2006)

Technical feasibility is not really a concern when it comes to possible lambda improvements. The main issue is that Guido will not accept a solution that introduces indentation within expressions on the grounds of lacking "Pythonicity". This effectively rules out multi-statement lambdas.

```python
foo(
    (
        lambda a, b:
            print(a)
            print(b)
    ), 
    "bar",
) 
```
<figcaption>Possible sparse syntax for multi-statement lambdas. Statements are enclosed by parentheses and indented one level further than the declaration.</figcaption>

```python
foo((
    lambda a, b:
        print(a)
        print(b)
    ), 
    "bar",
) 
```
<figcaption>Denser syntax for multi-statement lambdas.</figcaption>

If both variants were to be deemed valid, the concept of significant whitespace for indentation would be defeated, because the sparse syntax would require an optional extra level of indentation. If only one variant was to be allowed, there would be endless bike-shedding over which one should be chosen.

## Patterns

### Function binding 
```python
from functools import partial

for fun in (partial(lambda number, exp: number ** exp, number) for number in range(RUNS)):
    print(fun(2))
```

(Nested) lambdas can be used instead of `partial` for binding values to functions.

```python
for fun in ((lambda number: lambda exp: number ** exp)(number) for number in range(RUNS)):
    print(fun(2))
```

Readability is questionable is both cases, but benchmarking (10^6*5 runs) reveals some noticeable speed improvements when using list comprehensions, which are eagerly evaluated. Generator expressions, which evaluate lazily, exhibit a smaller performance gain. The overhead of lambda binding is generally smaller, because a wrapper function is created instead of a [`partial` object](https://docs.python.org/3.7/library/functools.html#partial-objects).

<table style="display: table !important;">
<tr>
    <th>Eager</th>
    <th>Lazy</th>
</tr>
<tr>
<td>
    <table style="display: table !important;">
        <thead>
            <tr>
                <th><strong>Partial</strong></th>
                <th><strong>Nested</strong></th>
                <th><strong>Improvement</strong></th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>12.097278s</td>
                <td>11.102859s</td>
                <td>8,22%</td>
            </tr>
            <tr>
                <td>12.075038s</td>
                <td>10.530266s</td>
                <td>12,80%</td>
            </tr>
            <tr>
                <td>11.61805s</td>
                <td>10.08913s</td>
                <td>13,16%</td>
            </tr>
        </tbody>
    </table>
</td>
<td>
    <table style="display: table !important;">
        <thead>
            <tr>
                <th><strong>Partial</strong></th>
                <th><strong>Nested</strong></th>
                <th><strong>Improvement</strong></th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>5.131468s</td>
                <td>4.854471s</td>
                <td>5,40%</td>
            </tr>
            <tr>
                <td>4.223810s</td>
                <td>4.122884s</td>
                <td>2,39%</td>
            </tr>
            <tr>
                <td>4.248887s</td>
                <td>4.158348s</td>
                <td>2,13%</td>
            </tr>
        </tbody>
    </table>
</td>
</table>

<details><summary>Benchmark code</summary>

```python
from functools import partial
from time import time

ENTRIES = 5000000
RUNS = 2
FUNCTION_ARG = (2,)


def bench(generator_expression, expand):
    start_bind = time()
    callables_ = list(generator_expression) if expand else generator_expression
    end_bind = time()

    start_exec = time()

    for fun in callables_:
        fun(*FUNCTION_ARG)

    end_exec = time()

    return (end_bind - start_bind) if expand else None, (end_exec - start_exec)


def format_result(result, expand):
    return "%sExecuting time: %fs\nOverall time: %fs" % (
        "Binding time: %fs\n" % result[0] if expand else "",
        result[1],
        (result[0] + result[1]) if expand else result[1],
    )


for expand in (True, False):
    print((" eager " if expand else " lazy ").center(20, "-"))
    for _ in range(RUNS):
        print(" partial ".center(20, "-"))
        print(
            format_result(
                bench(
                    (
                        partial(lambda number, exp: number ** exp, number)
                        for number in range(ENTRIES)
                    ),
                    expand,
                ),
                expand,
            )
        )

        print(" lambda ".center(20, "-"))
        print(
            format_result(
                bench(
                    (
                        (lambda number: lambda exp: number ** exp)(number)
                        for number in range(ENTRIES)
                    ),
                    expand,
                ),
                expand,
            )
        )
```

</details>

### Computed sort key

```pycon
>>> from math import tan
>>> sorted(range(10, 20), key=lambda number: tan(number))
[11, 18, 15, 12, 19, 16, 13, 10, 17, 14]
```



## Conclusion
* List comprehensions are often faster and always more "Pythonic" than `map()`, `filter()`, and `reduce()`. 
* Improving upon lambdas while preserving "Pythonicity" is hard.
* There are some legitimate use-cases for lambdas... just not that many. 