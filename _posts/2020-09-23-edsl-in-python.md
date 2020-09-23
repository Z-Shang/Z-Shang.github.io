---
layout: post
title:  "Implementing EDSL in Python"
date:   2020-09-23 12:15 +1000
categories: work
---

# Implementing EDSL in Python

Recently, a colleague asked me to create a DSL for him to save them from the hell of hundreds lines of configuration in Python dicts.

I was like, sure, take this S-Exp.

## The Configuration
The configuration looks like this:
```
spec = {
  "operation_name" : <op>,
  "operation_args" : {
    "arg_1" : val_1,
    "arg_2" : val_2,
    ...
  }
  "nested_spec": [
    spec_1 = {
      ...
      "nested_spec": [
        spec_2 ...
      ]
    }
  ]
}
```

What I did first, is to flatten the spec, into:
```
spec = {
  "operation_name" : <op>,
  "operation_args" : {
    "arg_1" : val_1,
    "arg_2" : val_2,
    ...
  }
}

spec_1 = { ... }

sepc_2 = { ... }
...
```
then they will be folded into a nested structure during the codegen.


## First Approach

First things first, set up the AST nodes classes as usual, let's call it SPEC in the following content.

Then, by reusing the parser from [here](http://gilgamesh.me/work/2020/02/10/make-python-lisp-again.html), the entire codegen was done in about 30 min.

We ended up with something like this:
```
(<op> :arg_1 val_1 :arg_2 val_2 ...)
(spec_1)
(spec_2)
...
```

## What Next

He then asked me, is it possible for this to be consumed by the IDEs they use?

I was like, no, I'm not gonna write you a whole bunch of IDE plugins.

But it then came to my mind, what if I could embed this into Python, therefore I can rely on existing tools.

## Embedded DSL in Python

After several minutes biting my nails, I came up with the solution.
Which, somehow makes EDSL possible in Python.
(Well, maybe there was someone has done this before, but it is fun to do anyway)

Everything starts with a decorator `@config_spec`.

The pipeline:

```
@config_spec -> bytecode of function -> analyse bytecode to reconstruct syntax tree -> transform to SPEC AST -> reuse codegen -> done
```

## Bytecode Analysis

Firstly, clean the bytecode to keep only the instructions we care, such as `LOAD_GLOBAL`, `FUNCTION_CALL`.

Then I can parse the bytecode, and get the raw syntax tree, e.g.:
```
opname(arg_1 = val_1, arg_2 = val_2)
```
will be compiled into:
```
node = ('op',
  [ LOAD_GLOBAL(opname),
    LOAD_CONST(val_1),
    LOAD_CONST(val_2),
    LOAD_CONST(("arg_1", "arg_2",)),
    FUNCTION_CALL_WITH_KW(2) ])
```

## Transform to SPEC AST
The OP node in SPEC AST is something like this:
```
@dataclass
class OP(AST):
  name: str
  arg: dict
  ...
```
I initially thought about constructing the dictionary from the bytecode values manually, but then I realized that there is an instruction called `BUILD_CONST_KEY_MAP`.

The following is then simple and natural:
```
name = node[1][0]
rest = node[1][1:]
arg_dict = eval(rest[:-1] + [ BUILD_CONST_KEY_MAP, RETURN_VALUE ])
...
spec_node = OP(name, arg_dict, ...)
```
easy.

## Block
In our DSL, we have a block structure, and I used Python's `with` syntax, e.g.:
```
with OP1():
  ...
```
will be compiled into:
```
LOAD_GLOBAL(OP1)
FUNCTION_CALL()
SETUP_WITH(LABEL_1)
...
BEGIN_FINALLY
LABEL_1:
WITH_CLEANUP_START
WITH_CLEANUP_FINISH
END_FINALLY
```
I don't think that I need to explain how to parse something like this.

## What Next
Maybe I can get some LISP stuffs into Python as EDSL.

Just MAYBE.
