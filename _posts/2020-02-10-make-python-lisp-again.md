---
layout: post
title:  "On Making Python Lisp Again"
date:   2020-02-10 20:07 +1000
categories: work
---

# On Making Python Lisp Again

Once upon a time, there was a voice claims that "Python is the new Lisp", which is obviously, not true. With the rapid developement of the buzz-word industry, Python has gained far more attention than its surface language deserves. While having an OK-ish VM, Python the language is recklessly wasting its natural advantage. 

I, a Lisp programmer like you can find in any neighborhood, tried to escape from the stupidity of Python while can still write codes that runs in my working envrionment, have decided to write a simple compiler that compiles a small Lisp into Python's bytecode, so that I can write whatever I want to and produce a binary file that my collages can use.


# Naming

I named this small Lisp dialect as LyPPS, after the idol group [LiPPS](https://www.project-imas.com/wiki/LiPPS) from THE iDOLM@STER, hope there is no legal issue on this lol.

# Components

## Parser
I have written a simple and straightforward combinatoric parser library in Python, hence the parsing is as easy as:

```python
id_char = lambda c: c.isdigit() or c.isalpha() or c in ".?!$%^&*_+/|~-"

lead = lambda c: c in "#@:'`,"


@parser
def atom(s):
    return p(any, id_char)(s) >> (
        lambda pair: ([("atom", chars2str(pair[0]))], pair[1])
    )


@parser
def string(s):
    return (p(one, __ == '"') >> p(pany, __ != '"') << p(one, __ == '"'))(s) >> (
        lambda pair: ([("string", chars2str(pair[0]))], pair[1])
    )


@parser
def cons(s):
    return (p(one, __ == "(") >> many(prefix) << p(one, __ == ")"))(s) >> (
        lambda pair: ([("cons", *pair[0])], pair[1])
    )


@parser
def sexp(s):
    return (spc >> (atom | string | cons) << spc)(skip_comment(s))


@parser
def prefix(s):
    pref = p(one, lead)(s)
    if not pref:
        return sexp(s)
    lead_char, rest = pref.from_right(([], []))
    val, rest = prefix(rest).from_right(([], []))
    if not val:
        return left([], s)
    return right([("prefix", lead_char[0], val[0])], rest)
```

With some simple and stupid preprocessing, the input sexp will be turned into your familiar tuples / lists.

## Lambda and Scoping

The scoping rule of Python is daft, there are two kinds of scopes: `CellVar` and `FreeVar`, to explain it with some kindergarten level C++ code:
```c++
auto a = bla;

auto f = [a] (auto b) {
    auto g = [a, b]() { return a + b; };
    return g;
};
```
For `f`, `a` is `FreeVar`, which is variable captured by a lambda expression, and b is a `CellVar`, which is captured by an inner closure `g`, and for `g`, both `a` and `b` are `FreeVar`s.

In LyPPS, to support currying by default, I designed the lambda expressions to take at most one argument, with an explicit "capture list" to fit my muscles trained by C++. Your familiar Scheme code will be turned into:
```scheme
; Scheme
(define (foo (a b c)
    (+ a b c)))

; LyPPS
(def foo
    (lambda () (a)
;           ^ capture list
        (lambda (a) (b)
;               ^ capture list
            (lambda (a b) (c)
;                   ^ capture list
                (+ a b c)))))
```

## AST Transformation

Once the code is parsed, we need to identify the special forms and transform the tuples and lists into something that our backend likes, and some desugaring such as turning a lambda with multiple arguments into the curried form, and turning a function apply into a curried apply. E.g.:
```scheme
; surface lang
(def foo (lambda () (a b c) (+ a b c)))

(foo 1 2)

; compiled
(def foo
    (lambda () (a)
        (lambda (a) (b)
            (lambda (a b) (c)
                (+ a b c)))))

((foo 1) 2)
```

## Codegen

There is not much to say about the codegen, just going down the AST and generate the corresponding bytecodes (thanks to the [bytecode library](https://github.com/vstinner/bytecode)), this should be as easy as breathe for any programmer who dares to put Lisp on her LinkedIn profile.

## Generating `.pyc`

Since my workplace forced us to used Python 3.6, some handy features such as `source_hash` are not available, and I was just too lazy to backport the stuffs from Python 3.7's C code to Python 3.6, I did a little research on Python 3.6's code object generation, the core implementation looks like this:
```python
with open(filename, 'wb') as fout:
    data = bytearray(MAGIC_NUMBER)
    loader = importlib.machinery.SourceFileLoader("<lps>", filename)
    stats = loader.path_stats(filename)
    mtime = stats["mtime"]
    source_size = stats["size"]
    data.extend(_w_long(mtime))
    data.extend(_w_long(source_size))
    data.extend(marshal.dumps(co))

    fout.write(data)
```

# What are Left Behind
## Macro
A Lisp dialect without a macro system is just an S-expression translater that an elementary school kid can finish within a weekend.

The approach that LyPPS used is to combine an interpreter into the compiler, so that during the AST transformation phase, macros can be evaluated.

## Graph
The current implementation is simple tree traverse, it might help future developement if I make a graph out of the ast, so that more checkings can be done at compile time.

## Python FFI
This wouldn't be a big deal, since  `import` is native for bytecode.

## Inline Bytecode
Just like your favourite `__asm` blocks in C codes, nothing special, I might be able to finish it on my way to office tomorrow morning.

## Open Source
Yea, since LyPPS was initially developed for my project at my company, I need to get through some process to get it published, hopefully I can do GPL.

## Why GPL?
Remember:
> FREE SOURCE, FREE MIND