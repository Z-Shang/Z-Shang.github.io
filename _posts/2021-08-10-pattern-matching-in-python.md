---
layout: post
title:  "ReInventing Pattern Matching for Python"
date:   2021-08-10 01:25 +1000
categories: work
---

# `ReInventing` Pattern Matching for Python

If you are a fan of [PEP 622](https://www.python.org/dev/peps/pep-0622/), you can leave the page now.

As we all know that, [PEP 622](https://www.python.org/dev/peps/pep-0622/) has introduced the so-called "Structural Pattern Matching", and not surprisingly, they messed up scoping again.

## Why
Recently while doing some of my personal stuffs, I found that some little pattern matching could help a lot, and didn't want to swalow the sh*t of Mr. BDFL.

Coincidentally, I've just finished reading the paper *Compiling Pattern Matching* by Lennart Augustsson (which is a quite interesting paper that one can enojoy for a weekend), then my fingers were whispering to me again: "Bloody hell, you gotta re-invent this wheel".

## What
As what I've done before: [EDSL in Python](http://gilgamesh.me/work/2020/09/23/edsl-in-python.md), with some little help of [fpy](http://gilgamesh.me/work/2021/03/17/fpy.html), my little new toy [Pyttern](https://github.com/Z-Shang/pyttern) took its form.

Now behold some magic:
```python
from pyttern.pyttern import pyttern
from fpy.composable.collections import transN, apply
from fpy.data.either import Left, Right, isRight, fromLeft

inc = lambda x: (x + 1) % 256
end = lambda x: len(x) - 1

@pyttern
def interp(f, r, t, p, o): {
    ('<', _r, _t, 0, _o)        : Right((_r[0], _r[1:], [0] + _t, 0, _o)),
    ('<', _r, _t, _p, _o)       : Right((_r[0], _r[1:], _t, _p - 1, _o)),
    ('>', _r, _t, end(t), _o)   : Right((_r[0], _r[1:], _t + [0], p + 1, _o)),
    ('>', _r, _t, _p, _o)       : Right((_r[0], _r[1:], _t, _p + 1, _o)),
    ('+', _r, _t, _p, _o)       : Right((_r[0], _r[1:], transN(_p, inc, _t), _p, _o)),
    ('*', _r, _t, _p, _o)       : Right((_r[0], _r[1:], _t, _p, _o + chr(_t[_p]))),
    ('*', (), _t, _p, _o)       : Left(_o + chr(_t[_p])),
    (_f, (), _t, _p, _o)        : Left(_o),
    _                           : Left(None)
}

def tick(inp):
    if not inp:
        return ''
    nxt = interp(inp[0], inp[1:], [0], 0, '')
    while isRight(nxt):
        nxt = nxt >> apply(interp)
    return fromLeft("", nxt)

if __name__ == '__main__':
    # Hello World Taken from CodeWars
    test = '++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*>+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*>++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++**>+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*>++++++++++++++++++++++++++++++++*>+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*<<*>>>++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*<<<<*>>>>>++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*>+++++++++++++++++++++++++++++++++*'
    print(tick(tuple(test)))
```

## How
As usual, there are some bytecode hack happening within that `pyttern` decorator, to simplify, this decorator accepts a function that has nothing but a dict as its body.

Then it and transforms dict values into a lambda expression which takes the bound pattern variables as arguments, and return the value.

After this, the keys are  used as patterns, for example:
```python
{
  (1, 2, 3): 100
}
```
will be turned into:
```python
{
  1: {
    2: {
      3: 100
    }
  }
}
```

For the detailed solution, please refer to Augustsson's paper.

## What's Next
* Guards
* Conditions
* You tell me

## Why Pyttern?
From where I was born, the Chinglish accent doesn't distinguish `/a/` and `/ʌɪ/`, therefore the word "pattern" could be pronunced like "pyttern".
