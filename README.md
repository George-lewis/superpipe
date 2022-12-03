## Superpipe: Elixir-style Pipes for Python

In the Elixir programming language the `|>` pipe operator allows you to chain 
together multiple function calls to avoid nesting. It allows for this:

```elixir 
c(b(a(1, 2), 3, 4))
```

to be writen as this:

```elixir
1 |> a(2) |> b(3, 4) |> c()
```

Superpipe implements a more powerful version of this operator in Python, the superpipe operator (henceforth, just "the pipe operator").

Superpipe allows you to turn heavily-nested code into easily-readable flows of data from left to right.

For example:

```python
# Take all the numbers [0, 100) that have an odd-number bit length, square them, and print the result
print("The numbers:", map(lambda x: x * x, filter(lambda x: x.bit_length() % 2 != 0, range(100))))
```

Using superpipe:

```python
range(100) >> filter(lambda x: x.bit_length() % 2 != 0) >> map(lambda x: x * x) >> print("The numbers:", _)
```

## Install

Get it on PyPi: https://pypi.org/project/superpipe/

## Tutorial

```py
# To begin using the pipe operator, you need to import the decorator

from superpipe import pipes

# And then apply it to a function, method, or class

@pipes
def foo():
    5 >> print

@pipes
class Bar:
    def __init__(self):
        self.qux = 5 >> _ * 2

class Loq:
    @pipes
    def vaz():
        "superpipes!" >> print
```

```py
# The @pipes decorator transforms the right-shift operator >> into the pipe operator
# You insert it between two expressions to inject the value of the lefthand side into the right

# The simplest example is to pass the lefthand side as a function argument

5 >> print
# The same as print(5)

5 >> print()
# Also the same as print(5)

# The pipe operator always adds the argument to the end of argument list

5 >> print("abc,")
# This prints "abc, 5"

range(3) >> map(lambda x: x)
# The operator really shines with functional constructs
# This is the same as map(lambda x: x, range(3))
```

```py
# Q: I don't want it to be the last argument
# A: You can use the special implicit substitution identifier "_" (an underscore)
# Superpipe will substitute the lefthand side into wherever it finds it

5 >> print(_, "abc")
# This prints "5 abc"

5 >> print(_)
# This is the same as earlier

5 >> print(_, _)
# Prints "5 5"
```

```py
# Q: I want to call a method of the lefthand side
# A: You can do this via the _ identifier. In fact, you can use the underscore almost anywhere

"superpipe" >> _.title()
# This is the same as "superpipe".title()

5 >> _ + 1
# The same as 5 + 1

"xyz" >> f"look at this!: {_}. wow."
# f-strings

[1, 2, 3] >> _[0]
# This gets the first element out of the list

def foo(a, b, c):
    pass

{"c": 1, "a": 2, "b": 3} >> foo(**_)
# You can use star-expansion on _

for x in range(3) >> map(lambda x: x * x):
    pass
# It works in loops, too

print(
    5
    >> _ + 1
    >> _ * _
)
# In some contexts Python syntax allows for spreading over multiple lines, like in a function call
# In other contexts, you can use a backslash (\) to split the line over multiple
# This prints 36

# You may be surprised at the number of things superpipe can do
# Try things out, and check out `tests/test.py` for a demonstration of everything it can do
```

## How it Works and Performance Considerations

When the pipe decorator is applied to a function it grabs the source code using the inspect module, parses it using the ast module, performs recursive transformations on the tree, and then substitutes the original function with the result.

Generally speaking, code written using superpipe will perform the same as writing the nested code explicitly, with two major caveats:

1. The first time Python evaluates your function and the decorator runs, there is a small overhead due to the AST transformations. This overhead should be relatively low and a one-time cost, happening only the first time the function is seen.
2. The pipe operator does not optimize for multiple _ substitutions in the same expression. When the decorator encounters the substitution identifier it substitutes all of the code to the left into the expression. When it has to do multiple such substitutions the lefthand side will be evaluated multiple times. For example, `5*5 >> print(_)` becomes `print(5*5)`, however `5*5 >> print(5*5, 5*5)`, thus performing the calculation twice, whereas one could write it more efficiently as `twentyfive = 5*5; print(twentyfive, twentyfive)`. This example is trivial, but gets worse the more nesting there is. This can be avoided by carefully considering where you perform multiple substitutions, and breaking up long chains into multiple.

### Feedback, Comments, Improvements?

Please open an issue on the repository, I would be happy to discuss with you.

---

Thank you to Robin Hilliard for the original inspiration for this project, the one from which this one was forked. Please consider visiting his implementation here: [robinhilliard/pipes](https://github.com/robinhilliard/pipes).
