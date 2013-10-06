---
layout: post
title: How Lisp Get Evaluated
---

> To the untutored eye, Lisp is a strange programming language.  
> <cite>— [Robert J. Chassell](http://bit.ly/1bIly9E)</cite>

In Lisp, both data and programs are lists of words, numbers, or other
lists, separeted by whitespace and surrounded by parenthese.

```scheme
'(rose violet daisy buttercup)
```

When Lisp interpreter evaluate a list, it simply returns the list if
the list is preceded with a quote:

```scheme
'(this list will get returned)
; => (this list will get returned)
```
Actually, `'` is an abbreviation for the special operator `quote`, which
just returns its single augument as written, be it a list, a symbol...

Otherwise, it will examine the first element, which alone determines
what kind of form the list is. A nonempty list is either a function
call, a macro call or a special operator.(sometimes called "special
forms")

If the first element is a Lisp function object, the defined
instructions will be carried out with the rest of lists as arguments.

```scheme
(+ 1 1)  ; => 2
```

If the list has another list inside it, the Lisp interpreter will work
on the innermost first, otherwise, it works left to right.

```scheme
(* 2 (+ 1 1))  ; => 4
```

If the first element is a macro object, unlike function call, the rest
of the list is not initially evaluated. In most cases, a macro call
finishes with expansion.

```scheme
(defmacro inc (var)
        (list 'setq var (list '1+ var)))

(inc x) ; => (setq x (1+ x))
```

"Special operators" (sometimes called "special forms") provide Lisp's
control structure, so its arguments are not all evaluated.

```scheme
(if (< x y)
    x
  y)
```

Strings and numbers evaluates to themselves, they are called
"self-evaluating form"

```scheme
123   ; => 123
"foo" ; => "foo"
```

Other than lists, the Lisp interpreter treat symbol not quoted and not
surrounded with parentheses as a variable:

```scheme
x ; => Symbol's value as variable is void: x
```

Note: A symbol can have both a function definition and a value attached to
it at the same time.

