# Cat code (3 points)

Ahoy, officer,

due to the lack of developers on board, the development of the access code generator for the satellite connection was
entrusted to the cat of the chief officer. Your task is to analyze the cat's creation and find out the code.

May you have fair winds and following seas!

Download the [cat code](cat_code.zip).
(MD5 checksum: `aac150b3f24e5b047ee99e25ad263f56`)

## Hints

* Cats are cute and all, but studies have shown that they really aren't good developers.

## Solution

After extracting the files with python source code we see (in `meoemeow.py`) that the program runs in a loop asking for
user input until the user acknowledges that kittens rule the world. Then two imported functions from `meow.py` are
executed.

After closer examination of `meow.py` we come across the following code:

```python
UNITE = 1
UNITED = 2


def meow(kittens_of_the_world):
    """
    meowwwwww meow
    """
    print('meowwww ', end='')
    if kittens_of_the_world < UNITED:
        return kittens_of_the_world
    return meow(kittens_of_the_world - UNITE) + meow(kittens_of_the_world - UNITED)
```

This is a naive implementation of calculating n-th member of the [Fibonacci sequence] which is used as an example of
poorly used recursion in almost all textbooks. Let's replace it by a non-recursive version, e.g. like this:

```python
def meow(kittens_of_the_world):
    if kittens_of_the_world < UNITED:
        return kittens_of_the_world
    prev = 0
    cur = 1
    n = 1
    while n < kittens_of_the_world:
        nxt = cur + prev
        prev = cur
        cur = nxt
        n += 1
    return cur
```

If we don't mind the meowing we can just run it to get the flag. If we do, we can also remove the line containing
`print('meowwww ', end='')` in `meowmeow(meow)` function.

```console
$ python meowmeow.py
Who rules the world? kittens
FLAG{YcbS-IAbQ-KHRE-BTNR}
```

[Fibonacci sequence]: https://en.wikipedia.org/wiki/Fibonacci_sequence
