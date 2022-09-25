---
weight: 2
title: "TLS done right(Chapter4~7)"
date: 2017-07-12 20:34:18
categories: [programming language]
draft: false
author: "Li Chen"
images: ["https://mitpress.mit.edu/sites/default/files/9780262560993.jpg"]

categories: [programming language]
tags: [lisp,scheme]


lightgallery: true

toc:
  auto: true
---

# $+$

## My try: 
```lisp
(define +
    (lambda (n m)
      (cond
        ((zeor? n)
         m)
        (else
         (add1 ( + (sub1 n) m))))))
```
At the beginnging, I have no solutions . Several seconds later, I noticed the hint given by book: 

> Hint: It uses zero ? add1 1 and sub1 1

Also, I find it is important to take $+$ just as a pure function. Finally I get the same solution with author's

# $-$

## My try:
```lisp
(define -
    (lambda (n m)
      (cond
        ((zero? m)
         n)
        (else
         (sub1 ( - n (sub1 m) ))))))
```
I find $+$ define above will create chaos because of the order of n and m. When define $-$, I address it carefully. 

# The First Commandment
> (first revision)
> When recurring on a list of atoms, $lat$, ask two questions about it: $(null? lat)$ and else. 
> When recurring on a number, $n$, ask two questions about it: $(zero? n)$ and else. 

# $addup$

> It builds number by totaling all the numbers in a tup. 

## My try
```lisp
(define addup
    (lambda (tup)
      (cond
        ((null? tup)
         0)
        (else
         (+ (car tup) (addup (cdr tup)))))))
```

# The Fourth Commandment

> $(first revision)$
> Always change at least one argument while recurring. It must be changed to be closer to termination. The changing argument must be tested in the termination condition:
> when using $cdr$, test termination with null? and 
> when using $sub1$, test termination with $zero?$. 

## $\times$

```lisp
> (define *
    (lambda (n m)
      (cond
        ((zero? m)
         0)
        ((zero? m)
         n)
        (else
         (+ n (* n (sub1 m)))))))
> (* 3 0)
0
> (* 0 9)
0
> (* 9 8)
```

# The Fifth Commandment

> When building a value with $+$, always use $0$ for the value of the terminating line, for adding $0$ does not change the value of an adition.
> Whe building a value with $\times$, always use 1 for the value of the terminating line, for multiplying by 1 does not change the value of a multiplication.
> When building a value with $cons$, always consider () for the value of the terminating line. 

# $tup+$

## My try(quite long-winded)

```lisp
> (define tup+
    (lambda (tup1 tup2)
      (cond
        ((and(null? tup1)(null? tup2))
        '())
        ((null? tup1)
         (cons (car tup2) (tup+ tup1 (cdr tup2))))
        ((null? tup2)
         (cons (car tup1) (tup+ (cdr tup1) tup2)))
        (else
         (cons (+ (car tup1) (car tup2)) (tup+ (cdr tup1) (cdr tup2)))))))
> (tup+ tup1 tup2)
'(11 5 12 2 3 5)
> tup1
'(4 2 8 1)
> tup2
'(7 3 4 1 3 5)
```

## author's implemenation

```lisp
(define tup+
(lambda (tup1 tup2)
(cond
((and (null? tup1) (null? tup2))
(quote ()))
((null? tup1) tup2)
((null? tup2) tup1)
(else
( cons ( -11- ( car tup1) ( car tup2))
(tup+
( cdr tup 1 ) ( cdr tup2)))))))
```

# $>$

```lisp
(define >
    (lambda (a b)
      (cond
        ((zero? a)
         #f)
        ((zero? b)
         #t)
        (else
         (> (sub1 a) (sub1 b))))))
```

# $<$

```lisp
(define <
    (lambda (a b)
      (cond
        ((zero? b)
         #f)
        ((zero? a)
         #t)
        (else
         (< (sub1 a) (sub1 b))))))
```

# $=$

```lisp
(define =
    (lambda (a b)
      (cond
        ((or(< a b) (> a b))
         #f)
      (else
       #t))))
```

# $^$

# My try:
```lisp
> (define ^
    (lambda (a b)
      (cond
        ((zero? a)
         0)
        ((zero? b)
         1)
        (else
         (* a (^ a (sub1 b)))))))
> (^ 4 2)
16
> (^ 1 99)
1
> (^ 0 344)
0
> (^ 3 0)
1
> (^ 7 1)
7
>
```

## author's try:
```
(define t1
(lambda ( n m)
(cond
((zero ? m) 1)
(else ( x n (t n (sub1 m)))))))
```

# $length$

## My try:
```lisp
(define length
    (lambda (lat)
      (cond
        ((null? lat)
         0)
        (else
         (add1 (length (cdr lat)))))))
```

# $pick$

## My pick:

```lisp
(define pick
    (lambda (n lat)
      (cond
        ((= n 1)
         (car lat))
        (else
         (pick (sub1 n) (cdr lat))))))
```

# $rempick$

## My try
```lisp
(define rempick
    (lambda (n lat)
      (cond
        ((= n 1)
         (cdr lat))
        (else
         (cons (car lat) (rempick (sub1 n) (cdr lat)))))))
```
# $no-nums$

## My try
```lisp
(define no-nums
    (lambda (lat)
      (cond
        ((null? lat)
         '())
        ((number? (car lat))
         (no-nums (cdr lat)))
        (else
         (cons (car lat) (no-nums (cdr lat)))))))
```

# $all-nums$

## My try:
```lisp
(define all-nums
    (lambda (lat)
      (cond
        ((null? lat)
         '())
        ((number? (car lat))
         (cons (car lat) (all-nums (cdr lat))))
        (else
         (all-nums (cdr lat))))))
```

# $eqan?$

## My frist try
```lisp
> (define eqan?
    (lambda (a1 a2)
      (cond
        ((and (number? a1) (number? a2))
         (= a1 a2))
        ((and (atom? a1) (atom? a2))
         (eq? a1 a2))
        (else
         #f))))
 
> (eqan? 4 5)
#f
> (eqan? 'a 'b)
atom?: undefined;
cannot reference an identifier before its definition
```

## My second try
```lisp
(define eqan?
    (lambda (a1 a2)
      (cond
        ((and (number? a1) (number? a2))
         (= a1 a2))
        ((or (number? a1) (number? a2))
         #f)
        (else
         (eq? a1 a2)))))
```

# $occur$
## My first try
```lisp
(define occur
    (lambda (a lat)
      (cond
        ((null? lat)
         #f)
        ((eqan? a (car lat))
          #t)
        (else
         (occur a (cdr lat))))))
```
## My second try
```lisp
(define occur
    (lambda (a lat)
      (cond
        ((null? lat)
         0)
        ((eqan? a (car lat))
         (add1 (occur a (cdr lat))))
        (else
         (occur a (cdr lat))))))
```

# $one?$

## My try:
```lisp
(define one?
    (lambda (n)
      (cond
        ((number? n)
         (= n 1))
        (else
         #f))))
```

## author's try(simplified version)
```lisp
(define one ?
(lambda (n)
(= n 1)))
```

## errata?
I think the codes above are wrong, author didn't guarantee n should be number. So if n is atom, `contract violation`. 

# $rember*$

## My try(fail $o(╥﹏╥)o$)
```
(define rember*
    (lambda (a l)
      (cond
        ((null? l)
         '())
        ((eq? a (car l))
         (rember* a (cdr l)))
        (else
         (cond
           (not (atom? l)
            (cons (cons (car (car (rember* a (cdr (car l))))) (rember* a (cdr (car l)))) (rember* a (cdr l)))))))))
```

## author"s codes
```lisp
(define rember*
    (lambda (a l)
      (cond
        ((null? l)
         '())
        ((atom? (car l))
         (cond
           ((eq? a (car l))
            (rember* a (cdr l)))
           (else
            (cons (car l) (rember* a (cdr l))))))
        (else
         (cons (rember* a (car l)) (rember* a (cdr l)))))))
```

My mistake is that I did not think of the use of `eq? a (car l)`. I used `sq? a l`, so the argument must be dived into atom and list. However, `cdr` has guarantee l is a list, so `eq? a (car l)` is perfect. 

# $insert*$
##My try:
```lisp
(define insert*
    (lambda (new old l)
      (cond
        ((null? l)
         '())
        ((atom? (car l))
         (cond
           ((eq? old (car l))
            (cons (car l) (cons new (insert* new old (cdr l)))))
           (else
            (cons (car l) (insert* new old (cdr l))))))
        (else
         (cons (insert* new old (car l)(insert* new old (cdr l))))))))
```

# The First Commandment
> $final version$
> When recurring on a list of atoms, lat, ask  two questions about it: (null? lat) and else.
> When recurring on a number, n, ask two two questions about it: (zero? n) and else.
> When recurring on list of S-expressions, l, ask three question about it: $(null? l), (atoms? (car l))$, and else.

# The Fourth Commandment
> $final version$
> Always change at least one argument while recurring. 
> When recurring on a list of atoms, $lat$, use $cdr lat$. When recurring on a number, $n$ use (sub1 n). And when recurring on a list of S-expressions, $l$ use (car l) and (cdr l) if neither $(nul? l)$ nor $(atom? (car l))$ are true. 
> It must be changed to be closer to termination. The changing argument must be tested in the termination condition: 
> when using $cdr$, test termination with $null?$ and 
> when using $sub1$, test termination with $zero?$. 

# $occur*$

## My try:
```lisp
(define occur*
    (lambda (a l)
      (cond
        ((null? l)
         0)
        ((atom? (car l))
         (cond
           ((eq? a (car l))
            (add1 (occur* a (cdr l))))
           (else
            (occur* a (cdr l)))))
        (else
         (+ (occur* a (car l))(occur* a (cdr l)))))))
```

# $subst*$
## My try:
```lisp
(define subst*
    (lambda (new old l)
      (cond
        ((null? l)
         '())
        ((atom? (car l))
         (cond
           ((eq? old (car l))
            (cons new (subst* new old (cdr l))))
           (else
            (cons (car l) (subst* new old (cdr l))))))
        (else
         (cons (subst* new old (car l)) (subst* new old (cdr l)))))))
```

# $insertL*$
## My try:
```lisp
(define insertL*
    (lambda (new old l)
      (cond
        ((null? l)
         '())
         ((atom?(car l))
           (cond
             ((eq? old (car l))
               (cons new (cons (car l) (insertL* new old (cdr l)))))
             (else
               (cons (car l) (insertL* new old (cdr l))))))
         (else
          (cons (insertL* new old (car l))(insertL* new old (cdr l)))))))
```

# $member*$
## My try:
```lisp
(define member*
    (lambda (a l)
      (cond
        ((null? l)
         #f)
        ((atom? (car l))
         (cond
           ((eq? a (car l))
            #t)
           (else
            (member* a (cdr l)))))
        (else
         (or (member* a (car l))(member* a (cdr l)))))))
```

# $leftmost$

## My try:
```lisp
(define leftmost*
    (lambda (l)
      (cond
        ((atom? (car l))
         (car l))
        (else
         (leftmost (car l))))))
```

# $eqlist?$
## My try:
```lisp
(define eqlist?
    (lambda (l1 l2)
      (cond
        ((and (null? l1) (null? l2))
         #t)
        ((and (atom? (car l1)) (atom? (car l2)))
         (and (eq? (car l1) (car l2)) (eqlist? (cdr l1) (cdr l2))))
        (else
         (and (eqlist? (car l1) (car l2)) (eqlist? (cdr l1) (cdr l2)))))))
```

## author's codes(a little complex $*-*$)
```lisp
(define eqlist?
(lambda ( l1 l2)
(cond
((and ( null? l1 ) ( null? l2)) #t)
((and (null? l1 ) ( atom? (car l2)))
#f)
((null? l1) #f)
((and ( atom? ( car l1)) (null? l2))
#f)
((and ( atom? ( car l1 ))
( atom? ( car l2)))
(and ( eqan ? ( car l1 ) (car l2))
( eqlist? ( cdr l1 ) ( cdr l2))))
(( atom? ( car l1)) #f)
((null? l2) #f)
(( atom? ( car l2)) #f)
(else
(and ( eqlist? ( car l1 ) ( car l2))
( eqlist? ( cdr l1 ) ( cdr l2)))))))
```

## author's codes(simplify)
```
(define eqlist ?
(lambda (l1 12)
(cond
((and (null? 11 ) (null? 12)) #t )
((or ( null? 11 ) (null? 12)) #f)
((and ( atom? ( car l1))
( atom? (car 12)))
(and (eqan? (car 11 ) (car 12))
( eqlist? (cdr 11) (cdr 12))))
((or ( atom? ( car l1 ))
( atom? (car 12)))
#f)
(else
(and (eqlist? (car 11 ) (car 12))
(eqlist? (cdr 11 ) ( cdr l2)))))))
```

# The Sixth Commandment

> Simplify only after the functuion is correct. 

Then author simplify various functions......but I used to imply these 
optimization

# $numbered?$
To speak honestly, I'm not familiar with  $(quote +)$ and $(quote \times)$. In that case

## My try(fail $o(╥﹏╥)o$)
```
(define numbered?
    (lambda (aexp)
      (cond
        ((atom? (car aexp))
         (and (or (eq? (car aexp) '+) (eq? (car aexp) '*) (eq? (car aexp) '^) (number? (car aexp))) (numbered? (cdr aexp)))))))
```

## author's codes(jdon't know aexp is an arithmetic expression)
```lisp
(define numbered?
(lambda ( aexp)
(cond
((atom? aexp) (number? aexp))
((eq? ( car ( cdr aexp)) (quote +))
(and (numbered? ( car aexp))
(numbered?
( car ( cdr ( cdr aexp))))))
((eq? ( car ( cdr aexp)) (quote x ))
(and (numbered? ( car aexp))
(numbered?
( car ( cdr ( cdr aexp))))))
((eq? ( car ( cdr aexp)) (quote t))
(and (numbered? ( car aexp))
(numbered?
( car ( cdr ( cdr aexp)))))))))
```

## author's codes(already know aexp is arithmetic expression)

```lisp
(define numbered ?
(lambda ( aexp)
(cond
(( atom? aexp) (number? aexp))
(else
(and (numbered? ( car aexp))
(numbered ?
( car ( cdr ( cdr aexp)))))))))
```

# The Seventh Commandment
> Recur on the $subparts that are of the same nature$:
> * On the sublists of a list
> * On the subexpressions of an arithmetic expression. 

# $value$

## author's codes(in-order)

```lisp
(define value
(lambda ( nexp)
(cond
( ( atom? nexp) nexp)
((eq? ( car nexp) (quote +))
( -11- (value ( cdr nexp))
( value ( cdr (cdr nexp)))))
(( eq? ( car nexp) (quote x ))
( x (value ( cdr nexp))
( value ( cdr (cdr nexp)))))
(else
(t (value ( cdr nexp ))
( value ( cdr (cdr nexp))))))))
```

## author's codes(pre-order)
```lisp
(define value
(lambda ( nexp)
(cond
( ( atom? nexp) nexp)
( ( eq? ( operator nexp) (quote +))
( + (value (1 st-sub-exp nexp))
(value ( 2nd-sub-exp nexp))))
( ( eq? ( operator nexp) (quote x ))
( x (value (1 st-sub-exp nexp))
(value ( 2nd-sub-exp nexp))))
(else
( t (value ( 1st-sub-exp nexp))
(value ( 2nd-sub-exp nexp)))))))
```

# The Eighh Commandment

> Use help functions to abstract from representations.  

# $set?$
## My try:
```lisp
(define set?
    (lambda (lat)
      (cond
        ((null? lat)
         #t)
        ((member? ((car lat) (cdr lat)))
         #f)
        (else
         (set? (cdr lat))))))
```

# $makeset$
## My try:
```lisp
(define makeset
    (lambda (lat)
      (cond
        ((null? lat)
         '())
        ((member? (car lat) (cdr lat))
         (makeset (cdr lat)))
        (else
         (cons (car lat) (makeset (cdr lat)))))))
```

## author's version(multirember )
```lisp
(define makeset
(lambda ( lat)
(cond
((null? lat) (quote ()))
(else ( cons ( car lat)
(makeset
( multirember ( car lat)
( cdr lat))))))))
```

# $subset?$
## My try:
(define subset?
    (lambda (set1 set2)
      (cond
        ((null? set1)
         #t)
        ((member? (car set1) set2)
         (subset? (cdr set1) set2))
        (else
         #f))))
```
## author's codes($and$)
```scheme
(define subset?
(lambda (set1 set2)
(cond
( ( null? set1) #t )
(else
(and (member? ( car set1) set2)
(subset? ( cdr set1 ) set2))))))
```


# $subset?$
## My try:
```lisp
(define eqset?
    (lambda (set1 set2)
      (cond
        ((null? set1)
         (and (subset? set2 set1) (subset? set2 subset1))))))
```

# $intersect$
## My try:
```
(define intersect?
    (lambda (set1 set2)
      (cond
        ((null? set1)
         #f)
        (else
         (or (memset? (car set1) set2) (intersect? (cdr set1) set2))))))
```

# union
## My try:
```lisp
(define union
    (lambda (set1 set2)
      (cond
        ((null? set1)
         set2)
        ((member? (car set1) set2)
         (union (cdr set1) set2))
        (else
         (cons (car set1) (union (cdr set1) set2))))))
```

# $intersectall$

## My try:((fail $o(╥﹏╥)o$))
```lisp
(define intersectall
   (lambda (l-set)
     (cond
       ((null? (car (car l-set)))
        '())
       ((and (member? (car (car l-set)) (car car ((cdr l-set)))))
        (cons (car (car l-set)) (intersectall (cons (cdr (car l-set)) (cdr l-set))))
       (else
        (intersectall (cons (cdr (car l-set)) (cdr l-set))))))))
```

## author's codes 
```lisp
(define intersectall
(lambda ( l-set)
(cond
( (null? ( cdr l-set)) ( car l-set))
(else (intersect ( car l-set)
( intersectall ( cdr l-set)))))))
```

# $a-pair?$ 
## My try: (simplify)
```
> (define a-pair?
    (lambda (x)
      (and (not (null? (cdr x)))(null? (cdr (cdr x))))))
> (a-pair? '(f))
#f
> (a-pair? '(a s))
#t
> (a-pair? '((df d (w))x))
#t
> (a-pair? '((2)(pair)))
#t
```
## author's codes
```lisp
(define a-pair?
(lambda (x)
(cond
((atom? x) #f)
(( null? x) #f)
(( null? ( cdr x)) #f)
(( null? ( cdr ( cdr x))) #t )
(else #f ))) )
```

# $reveral$
# author's codes:
```lisp
(define revrel
(lambda ( rel)
(cond
((null? rel) (quote ()))
(else ( cons ( build
(second ( car rel))
(first ( car rel)))
( revrel ( cdr rel)))))))
```
