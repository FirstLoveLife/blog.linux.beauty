---
weight: 2
title: "TLS done right(Chapter1~3)"
date: 2017-07-09 22:34:18

categories: [programming language]
tags: [lisp,scheme]
draft: false
author: "Li Chen"
images: ["https://mitpress.mit.edu/sites/default/files/9780262560993.jpg"]

categories: ["programming language"]

lightgallery: false

toc:
  auto: true
---
thumbnail: https://mitpress.mit.edu/sites/default/files/9780262560993.jpg

[toc]

>this article(list) is the record of studying [TLS](https://www.amazon.com/Little-Schemer-Daniel-P-Friedman/dp/0262560992). I try to implement all functions with describations by myself before reading author's implemention, which I think is the best way to learn  scheme as the second(or tree, four...) language with [TLS](https://www.amazon.com/Little-Schemer-Daniel-P-Friedman/dp/0262560992). 

# The Law of Car
> The primitive $car$ is defined only for non-empty lists.

$car$ is first S-expression of l. (include atom and list)

# The Law of Cdr
> The primitive $cdr$ is defined only for non-empty lists. the $cdr$ of any non-empty list is always another **list** 

$cdr$ is the list l without (car l)(so it is still list)

# The Law Of Cons
> The primitive $cons$ takes two arguments. The secone argument to $cons$ must be a list. The result is a list.

$cons$ takes two arguments:
the first one is any S-expression;
the second one is any list.

comment: racket can take atom as the second argument likely, just as the author marked in foot: 
> In practice, $(coms \alpha \ \beta$) works for values $\alpha \ and \ \beta$, and $(car (cons \  \alpha \ \beta)) = \alpha \\ (cdr (cons \  \alpha \ \beta)) = \beta$ 

# The Law of Null
> The primitive $null?$ is defined ony for lists.it is the list composed of zero S-expressions

author's comment: $(null? 'a)$ is also ok factly. 
So when author define $atom$, he used $...null? x...$

# The Law of Eq?
> The primitive $eq?$ takes two arguments. Each must be a non-numeric atom. (In pracice, some numbers  and lists may be arguments of eq?)


# $lat?$
> "$lat?$"looks at each S-expression in a list, in turn, and asks if each S-expression is an atom, until it runs out of S-expressions. If it runs out without encountering a list, the value is #t. If it finds a list, the value is #f--false. 

Before watching author's definition of $lat?$, I deduce it successfully, so in the following reading, I will always keep deducing until I am sure it is impossible for me inherently. Below is my codes(nearly the same as author's sample)

## My try:
```lisp
(define lat?
    (lambda (x)
      {cond ((null? x)
             #t)
            ((atom?(car x))
             (lat?(cdr x)))
            (else
             #f)}))
```

# $member?$
As I said above, when notice the concept of $member?$, I try deduce it on my own. Below is my trival codes(without $or?$)
```lisp
> (define member?
    (lambda (x list)
      (cond ((null? list)
            #f)
            ((eq? x (car list))
             #t)
            (else
             (member? x (cdr list))))))
> (define l '(coffee tea or milk) )
> (define x 'coffee)
> (define y 'haha)
> (member? x l)
#t
> (member? y l)
#f
```
then, there is author's sample with $or?$, which is much more elegent than mine. 
```lisp
(define member?
  (lambda (a lat)
    (cond
      ((null? lat) #f )
      (else (or ( eq? ( car lat) a)
        (member? a ( cdr lat)))))))
```

# The First Commandment
> (preliminary)
> Always ask $null?$ as the first question in expressing any function. 

I have realized this commandment when implement my own $lat?$ and $member?$. When I learn recursion in c/c++, setting terminals is important to terminate the recursion.

# $rember$
> $(rember a lat)$ takes an atom and a lat as its arguments, and makes a new lat with the first occurrence of the atom in the old lat removed. 

## My try

(while coding the last line, I used cond rather than cons uncanrefully -_-|| ): 

```lisp
 (define rember
    (lambda (x list)
      (cond ((null? list)
             '())
            ((eq? x (car list))
             (cdr list))
            (else
             (cons (car list) (rember x (cdr list)))))))
> x
'hh
> y
'(xu ba cd nn)
> (rember x y)
'(xu ba cd nn)
> (define x 'xu)
> (rember x y)
'(ba cd nn)
> (define y '(xu ba cd nn))
> (define x 'ba)
> (rember x y)
'(xu cd nn)
> (define y '(xu ba ba cd nn))
> (rember x y)
'(xu ba cd nn)
> (define x 'nn)
> (define x y)
> (define x 'nn)
> (rember x y)
'(xu ba ba cd)
```

Then, codes on the book(notice: the codes below is not correct. It is used to mention the second commandment about cons. But when I write my own $rember$, I didn't read these, so my implemention is all by myself)

```lisp
(define rember
  (lambda (a lat)
    (cond
      ( (null? lat) (quote ()))
      (else (cond
             ( ( eq ? ( car lat) a) ( cdr lat))
             (else ( rember a
                 ( cdr lat))))))))   
```

In the next page, author writes the right codes

```lisp
(define rember
(lambda (a lat)
(cond
((n·ull? lat) (quote ()))
(else (cond
(( eq ? ( car lat) a) ( cdr lat))
(else (cons ( car lat)
(rember a
( cdr lat)))))))))
```

# The Second Commandment
> Use $cons$ to build lists. 

$P_{56}$, author simplifies the codes as:
```lisp
(define rember
(lambda (a lat)
(cond
((null? lat) (quote ()))
((eq? ( car lat) a) ( cdr lat))
(else ( cons ( car lat)
(rember a ( cdr lat)))))))
```
the codes above is just the same as the manner I wrote at the beginning: )

# $first$
> "The function takes one argument, a list, which is either a null list or contains only non-empty lists. It builds another list composed of the first S-expression of each internal list."

## My try: 
```lisp
(define firsts
    (lambda (list)
      (cond ((null? list)
            '())
            (else
             (cons (car (car list)) (firsts (cdr list)))))))
```
(the same as author's)

# The Third Commandment

> When building a list, describe the first typical element, and then $cons$ it onto the natural recursion. 

# $insertR$
> It takes three arguments: the atoms new and old, and a lat. The function $insertR$ builds a lat with new inserted to the right of the first occurrence of old.

## My first try:

```lisp
(define insertR
    (lambda (new old list)
      (cond ((null? list)
             '())
            ((eq? old (car list))
             (cons new (cdr list)))
            (else
             (cons (car list) (insertR new old (cdr list)))))))
```

## My second try: 

```lisp
(define insertR
    (lambda (new old list)
      (cond ((null? list)
             '())
            ((eq? old (car list))
             (cons (car list)(cons new (cdr list))))
            (else
             (cons (car list) (insertR new old (cdr list)))))))
```

In the book, author guide readers with three steps while I try twice(both the final try are the same).

# $insertL$

## My first try(last line -____-|| ):
```lisp
(define insertL
    (lambda (new old list)
      (cond ((null? list)
             '())
            ((eq? old (car list))
             (cons new list))
            (else
             (cons (car list) (insertR new old (cdr list)))))))
```

## My second try:

```lisp
(define insertL
    (lambda (new old list)
      (cond ((null? list)
             '())
            ((eq? old (car list))
             (cons new list))
            (else
             (cons (car list) (insertL new old (cdr list)))))))
```

# $subst$
> ($subset new old lat$) replaces the first occurrence of old in the lat with new. 

## My try

```lisp
(define subst
    (lambda (new old list)
      (cond ((null? list)
             '())
            ((eq? old (car list))
             (cons new (cdr list)))
            (else
             (cons (car list) (subst new old (cdr list)))))))
```

# $subst2$

## My try

```lisp
(define subst2
    (lambda (new o1 o2 list)
           (cond ((null? list)
                  '())
                 ((or (eq? o1 (car list)) (eq? o2 (car list)))
                 (cons new (cdr list)))
                 (else
                  (cons (car list) (subst2 new o1 o2 (cdr list)))))))
```

# $multirember$
> gives all its final value the lat with all occurrences of $a removed$

## My try

```lisp
(define multirember
    (lambda (a list)
      (cond ((null? list)
             '())
            ((eq? a (car list))
              (multirember a (cdr list)))
            (else
             (cons (car list) (multirember a (cdr list)))))))
```



# $multiinsertR$

## My try:
```lisp
(define multiinsertR
    (lambda (new old list)
      (cond
        ((null? list)
         '())
        ((eq? old (car list))
         (cons (car list) (cons new (multiinsertR new old (cdr list)))))
        (else
         (cons (car list) (multiinsert new old (cdr list)))))))
```

## author's codes

```lisp
(define multiinsertR
(lambda (new old lat)
(cond
((null? lat) (quote ()))
(else
(cond
( ( eq? ( car lat) old)
( cons ( car lat)
( cons new
( multiinsertR new old
( cdr lat)))))
(else ( cons ( car lat)
( multiinsertR new old
( cdr lat)))))))))
```

# $multiinsertL$

## My first try:

```lisp
(define multiinsertL
    (lambda (new old list)
      (cond
        ((null? list)
         '())
        ((eq? old (car list))
         (cons new (multiinsertL new old (cdr list))))
        (else
         (cons (car list) (multiinsertL new old (cdr list)))))))
```

## My second try:

```lisp
(define multiinsertL
    (lambda (new old list)
      (cond
        ((null? list)
         '())
        ((eq? old (car list))
         (cons new (multiinsertL new old (cdr list))))
        (else
         (cons (car list) (multiinsertL new old (cdr list)))))))
```

author also showed one wrong version, which is about recuse endless while mime is about cons(situations of recusing endless I have met in the past)

# The Fourth Commandment

> preliminary
> Always change at least one argument while recurring. It must be changed to be closer to termination. The changing argument must be tested in the termination condition: when using $cdr$, test termination with $null?$

# $multisubst$

## My try:

```lisp

(define multisubst
    (lambda (new old list)
      (cond
        ((null? list)
         '())
        ((eq? old (car list))
         (cons new (multisubst new old (cdr list))))
        (else
         (cons (car list) (multisubst new old (cdr list)))))))
```
