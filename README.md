# conway

Conway is an attempt at building a programming language that would fit into the systems development niche, hopefully making some common systems tasks easier than they currently are in the languages that are most often used for such tasks, such as Python, Perl, or writing bash scripts directly. This is heavily informed by my own day-to-day tasks, which often involve running sequences of system calls and getting data out of them and then doing something with the data to decide what to do next. I would like to combine the ease of parsing text and basic data types of Python, the type safety of Golang or Haskell, generics faculties along the lines of those provided by Scala, while making it easy to make systems calls including piping data into and out of calls.

## Design concept examples

Note: `%` for terminal in examples and `#` for comments

Assignment:

```
% 123 -> myNum
% "this" -> myStr
```

For system calls, can we make assignment also work as a pipe? (unashamedly stealing Python's string format syntax) So, the `->` operator is more like the "send into" operator.

```
% "this" -> myStr
% sys.call(f"echo {myStr}") -> sys.call("grep t") -> res
% print(res)
this
```

Need to work out when to use `fn(a)` and when to use `a -> fn`. I'm leaning toward `fn(a)` as application / partial application syntax (which *can* be used to fully apply a function, e.g. call it). So, reasoning that out:

```
% # non-generics function w/ 1 arg
% fn print(x[Str]) Str {
...  sys.stdout(x)
...  return x
... }
defined function print(x[Str]) Str
% # generic function w/ 1 arg
% fn print[T](x[T]) T {
...  sys.stdout(T.stringify(x))
...  return x
...  }
defined function print[T](x[T]) T
% print("this")
this
% # is shorthand for
% print[Str]("this")
this
% # and is also equivalent to
% "this" -> print
this
% # non-generic fn w/ 2 args
% fn sum(x[Int])(y[Int]) Int {
...  return x + y
...  }
defined function sum(x[Int])(y[Int]) Int
% sum(3)(4)  # would we allow `sum 3 4`?
7
% # partial application
% sum(1) -> increment
defined function increment(y[Int]) Int
% increment(5)
6
```

Would need to come up with a system for eliminating ambiguity in partial application. This *could* involve a type alias, or some kind of tags. Some options for specifying which variable to apply:
```
% fn div(x[Float])(y[Float]) Float {
...  return x/y
...  }
defined function div(x[Float])(y[Float]) Float
% # op 1
% div()(2.0) -> halve
defined function halve(x[Float]) Float
% halve(5.0)
2.5
% div(1.0) -> powNegOne
defined function powNegOne(y[Float]) Float
% powNegOne(8.0)
0.125
```

What if iteration works like this? (e.g.: `-->` is the "for each" operator)

```
% [1,2,3] -> myList   # assigns myList = [1,2,3]
% # assuming print is a fn that takes one arg
% [1,2,3] --> print
1
2
3
```

Generics example (a la Scala):

```
% Some(5) -> myNum0
% type(myNum0)
Option[Int]
% print(myNum2)
5
% # type-safe alternative to Null
% None[Int] -> myNum1
% type(myNum1)
Option[Int]
% print(myNum1)

% Some[Float](5) -> myNum2
% type(myNum2)
Option[Float]
% print(myNum2)
5.0
% Some(4) --> print
4
% None[Int] --> print

```

## General notes

Desired attributes:
- type safe (probably compiled)
- easy to make system calls and access the results
- easy to make something run in a separate thread, and to collect threads (coroutines and monitor pattern, probably, a la Go?)
- opinionated about how to do things like asynchrony (one scheduler used throughout, one async semantic [what if no async semantic, coroutines only?]) and error handling
- functional features would be nice, like partial application and generics -- considering making the language non-object-oriented even (no inheritence, no methods on data objects, types are more related to modules and have little or nothing to do with instances of data)
- how should memory management work? I want it to be easy, but does that mean trying to figure out an easier version of Rust's memory management syntax, or doing garbage collection, or something else? Programmers should definitely not be thinking about malloc/release. Does it get easier if you don't allow non-local variables (all variables are explicitly passed into a closure, and all instances get released at the end of the closure)?
- should it allow "truthy" and "falsey" values? Or should it instead force the user to either explicitly type cast to bool or make use of generics? I'm inclined not to use "truthy" and "falsey" values, as the implicit type conversions appear sloppy to me, but I may change my mind on this.
- can we not have a null value? I need to re-read the paper about null being a mistake and see if we can make a language where this doesn't happen.
