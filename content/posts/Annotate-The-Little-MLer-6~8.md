---
title: Annotate The Little MLer 6~8
date: 2017-10-24 17:57:16
tags: [standard ml]
categories: [programming language]
author: "Li Chen"
---

> 记些比较典型又有趣的例子吧. 留着以后自己复习看看. 

# Oh My, It's Full of Stars!
``` sml
fun height(Bud) 
  = 0
 |height(Flat(f, t))
  = 1 + height(t)
 |height(Split(s, t))
  = 1 + large_of(height(s), large_of(t))

datatype
  'a slist = 
    Empty
  | Scons of (('a sexp) * ('a slist))
and
  'a sexp = 
    An_atom of 'a
  | A_slist of ('a slist)

fun 
    occurs_in_slist(a, Empty)
    = 0
  | occurs_in_slist(a, Scons(s, y))
    = occurs_in_sexp(a, s) + 
      occurs_in_slist(a, y)
and 
    occurs_in_sexp(a, An_atom(b))
    = if eq_fruit(b, a)
      then 1
      else 0
  | occurs_in_sexp(a, A_slist(A_slist(y)))
    = occurs_in_slist(a, y)

fun 
    rem_from_slist(a, Empty)
    = Empty
  | rem_from_slist(a, Scons)
    = if eq_fruit(a, b)
      then rem_from_slist(a, y)
      else Scons(An_atom(b), rem_from_slist(a, y))
  | rem_from_slist(a, Scons(A_slist(x), y))
    = Scons(rem_from_slist(a, x), rem_from_slist(a, y))
```



# Functions Are People, Too

## It's time for high-order function

```sml
datatype chain =
    Link of (int * (int -> chain));
```
