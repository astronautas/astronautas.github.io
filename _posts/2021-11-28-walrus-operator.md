The walrus operator, formally the assignment expressions, has been for some time around now in Python (since 3.8). I've grown fond of it :).

You can add more clarity to the code with it:

```python
last_visited_at = context.get("last_visited_at")

if last_visited_at is None:
    raise ValueError(f"last_visited_at should have value (is {last_visited_at})")
```

vs.

```python
if (last_visited_at := context.get("last_visited_at")) is None:
    raise ValueError(f"last_visited_at should have value (is {last_visited_at})")
```

The `last_visited_at` variable should not matter after the conditional block. The example with the walrus operator signals the intent better.

I haven't yet used the operator for anything fancier but as syntax sugar for better code clarity. RealPython illustrates more uses of the walrus operator e.g. [this one](https://realpython.com/python-walrus-operator/#list-comprehensions) of rewriting a double list comprehension into a single one!
