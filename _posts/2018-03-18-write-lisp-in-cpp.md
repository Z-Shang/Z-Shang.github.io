---
layout: post
title: "Write Lisp in C++"
date: 2018-03-18 13:35:00 +0800
categories: lisp
---

# Write Lisp in C++

> This post is an advertisement (

For a long time I've been struggling to try to write my C++ code more elegant (in the Lisp's way). Recently, I've started a project: [**LICPP**](https://github.com/Z-Shang/LICPP), which is a Lisp flavoured DSL in C++ as a header library for C++ 14 and above.

The purpose of LICPP is *Not* to create something blazing fast to compete with others in the benchmark, but to provide the minority programmers like me who desire a more flexible C++ programming experience.

With LICPP, codes can be written in a way that all Lisp programmers should feel familiar while being able to utilise all C++ components naturally, for example:

```
var foo = list(1, 2, 3);
var bar = list("a", "b", "c");
cout << mapcar([](int n, string s){ return s + to_string(n); }, foo, bar) << endl;

=> (a1 . (b2 . (c3 . nil)))
```

There are many other features and more is coming soon.
