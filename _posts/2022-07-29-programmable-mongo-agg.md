---
layout: post
title:  "A Taste of Programmable Aggregation Pipeline in MongoDB"
date:   2022-07-29 16:15 +1000
categories: work
---

# A Taste of Programmable Aggregation Pipeline in MongoDB

> This was a project I worked on during the SkunkWorks time at MongoDB.

I assume that you have tried to play with MongoDB, or at least have heard about it.
There is a thing called aggregation pipeline in MongoDB if you are not yet aware of its existence,
it is basically, well, a pipeline for transforming data contained within your collection (table, if you are from the SQL world).

## What is an aggregation pipeline?

> Disclaimer: I'm no expert of aggregation pipeline, please feel free to point out my mistakes.

Basically, an aggregation pipeline consists of several stages, and each stage is capable of transforming a document (row, again, if you are from the SQL world)
in some way.

For example, consider that you have some data in a collection, namely `coll_foobar`:
```
{'key': 0, 'val': True}
{'key': 1, 'val': False}
{'key': 2, 'val': True}
{'key': 3, 'val': False}
{'key': 4, 'val': True}
{'key': 5, 'val': False}
{'key': 6, 'val': True}
{'key': 7, 'val': False}
{'key': 8, 'val': True}
{'key': 9, 'val': False}
{'key': 10, 'val': True}
{'key': 11, 'val': False}
{'key': 12, 'val': True}
{'key': 13, 'val': False}
{'key': 14, 'val': True}
{'key': 15, 'val': False}
{'key': 16, 'val': True}
{'key': 17, 'val': False}
{'key': 18, 'val': True}
{'key': 19, 'val': False}
...
```
(A trivial data set that `val = (key % 2 == 0)`)

Then, you can do something like (again, a trivial example):
```javascript
coll_foobar.aggregate([
  {
    $addFields : {
      foo: 2,
      bar: 1000
    }
  },
  {
    $set : {
        foobar: { $multiply: [ $foo, $bar ] }
    }
  }
])
```
Then this will return a cursor which gives you results like:
```
{'key': 0, 'val': True, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 1, 'val': False, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 2, 'val': True, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 3, 'val': False, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 4, 'val': True, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 5, 'val': False, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 6, 'val': True, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 7, 'val': False, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 8, 'val': True, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 9, 'val': False, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 10, 'val': True, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 11, 'val': False, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 12, 'val': True, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 13, 'val': False, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 14, 'val': True, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 15, 'val': False, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 16, 'val': True, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 17, 'val': False, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 18, 'val': True, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
{'key': 19, 'val': False, 'foo': 2, 'bar': 1000, 'foorbar': 2000}
...
```
If you have ever played with industrial functional programming languages such as Clojure / F#, you may find this is
, conceptually, similar to the threading macro in Clojure / Pipe-forward in F#.

You can read more about aggregation pipeline at [here](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/)

## `let`

In such pipelines, it will often be useful to have some variables across the entire pipeline.
This is currently impossible in MongoDB's aggregation pipeline.

An existing *operator* [`$let`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/let/) lets you
bind some variables into an expression, but unfortunately, a pipeline is not an expression.

An existing aggregation stage [`$lookup`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/lookup/) allows
you to define some variables for its subpipeline, but certain limitations apply to the usage of `$lookup` (as its name
suggests, it is not designed for binding variables into a subpipeline).

Following the spirit of the above parts, I've implemented a new aggregation stage `$letp` (`p` stands for pipeline),
with `$letp`, you can bind variables for a pipeline, consider the same collection used above, an example can be:

```javascript
coll_foobar.aggregate([
  {
    $letp: {
      vars: { foo: 2, bar: 1000 }
      pipeline: [
        {
          $set : {
            foobar: { $multiply: [ $foo, $bar ] }
          }
        }
      ]
    }
  }
])
```

Which gives you the same result as above sans fields `foo` and `bar`.

## `quote` And `eval`

For those who does not know Lisp, there are a pair of primitive operations in Lisp languages:
`quote` and `eval`.
Basically, `quote` keeps the quoted value unevaluated, and `eval` evaluates the value it takes.
For example:
```lisp
(quote (+ 1 1)) ; => '(+ 1 1)
(eval (quote (+ 1 1))) ; => 2
```

In MongoDB's aggregation pipeline, there is an existing operator called `$const` (or `$literal`, they are actually the same thing),
which basically works as a `quote`.
Then it came to me that, we should be able to create a pipeline from an array.

Introducing `$expand`, an aggregation stage that creates a pipeline from an expression that evaluates into an array.
What can we do with this?
We can achieve conditional pipeline by having different branches quoted by `$const` and expand them on demand.
I've even wrapped this into a stage `$cond`, you can then do something like:
```
{
    "$letp": {
        "vars": {"foo": 100},
        "pipeline": [
            {
                "$cond": {
                    "branches": [
                        {
                            "case": "$v",
                            "then": [
                                {"$addFields": {"foo": "foo"}},
                                {"$set": {"foobar": {"$multiply": ["$k", 2]}}},
                            ],
                        }
                    ],
                    "default": [
                        {"$addFields": {"bar": "bar"}},
                        {"$set": {"foobar": {"$multiply": ["$k", 3]}}},
                    ],
                }
            },
        ],
    }
}
```

## `subst`

Now we have a let binding, we have the ability to create a pipeline from array data, what's next?

**To have substitution of variables in an expression.**

With the expression `$subst`, you can specify a set of variables, and an expression,
then only the nominated variables will be evaluated and substituted into the result expression. E.g.:
```
{
    $let: {
        vars: {
            foo: 123,
            opname: {$const: $add}
          },
        in: {
            $subst:
              {
                  symbols: [opname],
                  in: {
                      $$opname: [$$foo, $$bar]
                    }
                }
          }
      }
}
=>
{
    $add: [$$foo, $$bar]
}
```

## Next Steps

A quiz, now we can construct nested scope with `$letp`, we can evaluate an expression with `$expand`,
and substitute variables within an expression, what can we create from these?

> 3

> 2

> 1

Yes, **FUNCTIONS** ;)

Wait a second, there is already a [`$function`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/function/)?

But it uses JavaScript (and WASM support is not yet implemented).

Also, what I have in mind is to have something like PL/pgSQL, not a general purpose programming language.
I think you can get the difference here.

Hope you enjoyed this.
