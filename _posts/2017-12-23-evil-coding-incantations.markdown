---
layout: post
title:  "Evil Coding Incantations"
date:   2017-12-23 7:42:15 -0500
categories: [random]
---

Ever since I watched the revered [Wat](https://www.destroyallsoftware.com/talks/wat) video by Gary
Bernhardt, I've been fascinated with the strange behavior of certain programming languages. Some
programming languages have more unexpected behaviors than others. Java, for example, has [a whole
book](http://amzn.to/2zoCGSn) dedicated to its edge cases and peculiarities. For C++'s equivalent,
you can refer to the [C++ specification itself](https://www.iso.org/standard/68564.html) for just
200 USD. :expressionless:


What follows is a collection of my favorite surprising, humorous, and yet valid incantations.
Generally speaking, taking advantage of these peculiar behaviors is considered *evil* since your
code should be anything but surprising. Thankfully, there are many linters that are primed and ready
to make fun of you if you try most of the following tomfoolery. All that being said, knowledge is
power, so let's begin.

### [Evil Reassignment of True]() in Python 2

It rhymes so you know it's poo.

```python
>>> True = False
>>> True
False
```

Thankfully, this yields a `SyntaxError` in Python 3 since [True, False, and None are now reserved
words](https://docs.python.org/3.0/whatsnew/3.0.html). This eccentricity is still far less evil than
the C++ prank of sneaking `#define true false` into a standard header file of your coworker's
development machine.

### [Spooky Action with My Instance]() in Java and Python

The semantics of `==` for beginning Java programmers is often perplexing, but the operator's
inconsistency in even trivial scenarios serves to complicate the situation, even if the performance
benefits are worth it.

```java
Integer a = 100;
Integer b = 100;
System.out.print(a == b); // prints true

Integer c = 200;
Integer d = 200;
System.out.print(c == d); // prints false
```

The JVM will use the same reference for values in the range `[-128, 127]`. What's even stranger is
the python equivalent behavior.

```python
>>> x = 256
>>> y = 256
>>> x is y
True

>>> x = 257
>>> y = 257
>>> x is y
False
```

Nothing *too* surprising so far.

```python
>>> x = -5
>>> y = -5
>>> x is y
True

>>> x = -6
>>> y = -6
>>> x is y
False
```

It seems the lower limit for the python interpreter to use the same instance is... `-5`. Integers in
the range of `[-5, 256]` get the same IDs. It somehow gets weirder still.

```python
>>> x = -10
>>> y = -10
>>> x is y
False
>>> x, y = [-10, -10]
>>> x is y
True
```

It seems that using destructured assignment changes the rules here. I'm not sure why this is and I
actually have [a Stack Overflow question](https://goo.gl/39FDVA) open to try to understand it.  My
guess is that repeating values in a list point to the same object to save memory.

### [The Reversed Subscript Notation]() in C
The reversed subscript notation gives any developer an instant headache.

```c
int x[1] = { 0xdeadbeef };

printf("%x\n", 0[x]); // prints deadbeef
```

The reason this works is that `array[index]` is really just syntactic sugar for `*(array + index)`.
Thanks to the commutative property of addition, we can swap the array and the index and get the same
result.

### [The "Goes Down To" Operator]() in C

When first seen, the `-->` operator appears to be a syntax error. When you realize it compiles, it
looks like an undocumented language feature. Fortunately, it's neither.

```c
for (x = 3; x --> 0;) {
    printf("%d ", x); // prints 2 1 0
}
```

The `-->` "operator," which is actually two operators, is parsed as `(x--) > 0` in this context. It
is known to cause [quite the
confusion](https://stackoverflow.com/questions/1642028/what-is-the-operator-in-c) when used in
production -- pure evil.

### [The `sizeof` Operator]() in C

The `sizeof` operator is a compile-time operator, which gives it interesting properties.

```c
int x = 0;
sizeof(x += 1);
if (x == 0) {
    printf("wtf?"); // this will be printed
}
```

Since instances of the `sizeof` operator are evaluated at compile time, the expression `(x += 1)`
never runs. Also interesting: studies revealed that `printf("wtf?")` is the most popular line of
code that never gets pushed.

### [Indexes Starting at 1]() in Lua, Smalltalk, MATLAB, and more...

[/r/programminghumor](https://www.reddit.com/r/ProgrammerHumor/) has been having fun with
"[indexing](https://goo.gl/QreBLU) [starts](https://goo.gl/bm8Akz) [at](https://goo.gl/joQ9gt)
[1](https://goo.gl/MM5BwP)" memes. Shockingly, there are plenty of programming languages that
actually sport 1-indexed arrays. A more comprehensive list can be found
[here](https://goo.gl/kASpE7).

### [0 Evaluates to `true`]() in Ruby

... and only Ruby. \*

```ruby
if 0 then print 'thanks, ruby' end # prints thanks, ruby
```

<small>
\* edit: It was pointed out on [reddit](https://goo.gl/gnm654) that this is true for Lua, Lisp, and
Erlang as well.
</small>

### [Trigraph, Digraphs, and Tokens]() in C

For historical reasons, there are alternatives to the non-alphanumeric symbols in C.

| Trigraph | Symbol | &nbsp;&nbsp; | Digraph | Symbol | &nbsp;&nbsp; | Token    | Symbol |
| ---------|:-------|--------------|---------|--------|--------------|----------|--------|
| `??=`    | `#`    |              | `<:`    | `]`    |              | `%:%:`   | `##`   |
| `??/`    | `\`    |              | `:>`    | `[`    |              | `compl`  | `~`    |
| `??'`    | `^`    |              | `<%`    | `{`    |              | `not`    | `!`    |
| `??(`    | `[`    |              | `%>`    | `}`    |              | `bitand` | `&`    |
| `??)`    | `]`    |              | `%:`    | `#`    |              | `bitor`  | `|`    |
| `??!`    | `|`    |              |         |        |              | `and`    | `&&`   |
| `??<`    | `{`    |              |         |        |              | `or`     | `||`   |
| `??>`    | `}`    |              |         |        |              | `xor`    | `^`    |
| `??-`    | `~`    |              |         |        |              | `and_eq` | `&=`   |
|          |        |              |         |        |              | `or_eq`  | `|=`   |
|          |        |              |         |        |              | `xor_eq` | `^=`   |
|          |        |              |         |        |              | `not_eq` | `!=`   |

<br />
```c++
if (true and true) { // same as if (true && true)
    printf("thanks, c");
}
```

Some foreign equipment, such as the IBM 3270, did not supply some of the commonly used symbols in
c/cpp, so digraphs, trigraphs, and tokens were supplied as to not discriminate against particular
character sets.

I hope this article was interesting. You can follow the discussion on reddit
[here](https://goo.gl/gnm654).
