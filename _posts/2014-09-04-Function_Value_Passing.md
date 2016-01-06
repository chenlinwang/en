---
layout: post
title: Function Value Passing in Scheme
categories: [Scheme]
---

# Background

After I have learned the environment mode about scheme on [SICP](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-20.html#%_sec_3.1). I went on trying to do some oop with scheme. And after finishing a little project, I am confused with the variable referring mechanisms inside scheme.

# Code

```Scheme
;; to reduce the redundant function to return class variable value
(define (class-variable v)
  (lambda (self) v))             ;; anonymous function 1

;; example class to show my doubts
(define (make-ob p)
  (lambda (method)               ;; anonymous function 2
    (case method
      ((p) (class-variable p))
      ((set-p) (lambda (self new-p) (set! p new-p)))
      (else 'non-method))))

(define ob (make-ob 1))          ;; call 1
(apply (ob 'p) (list ob))        ;; call 2
(apply (ob 'set-p) (list ob 2))  ;; call 3
(apply (ob 'p) (list ob))        ;; call 4
```

And the result gives `1` and `2`.

# Question

How come the `p` value, in the `anonymous function 1`, whose frame is parallel to `anonymous function 2`'s, could be correct after I changed the `p` value in the latter's frame.

# Explanation

That is correct naturally. You see that the function `class-name` is defined under the global environment and so naturally when `class-name` is evaluated at `call 2` the `anonymous function 1` could  only see the value passing in. But the true is that every time we make a call like `call 2` and `call 4`, a new `anonymous function 1` is created with the correct value `p` in `anonymous function 2`'s frame, thus the value is correct.

And for the referring rules passing through the functions, it is simply the value, not the binding. We could see it here:

```Scheme
;; rewrite the class-name function
;; to reduce the redundant function to return class variable value
(define (class-variable v)
  (lambda (self)                 ;; anonymous function 1
    (set! v 3)
    v))

;; example class to show my doubts
(define (make-ob p)
  (lambda (method)               ;; anonymous function 2
    (case method
      ((p) (class-variable p))
      ;; add the get function to get the p in this frame
      ((dp) (lambda (self) p))
      ((set-p) (lambda (self new-p) (set! p new-p)))
      (else 'non-method))))

(define ob (make-ob 1))         ;; call 1
(apply (ob 'set-p) (list ob 2)) ;; call 2
(apply (ob 'p) (list ob))       ;; call 3
(apply (ob 'dp) (list ob))      ;; call 4
```

And the result gives `3` and `2`. Bingo, the `v` inside `anonymous function 1` could not infer the `p` inside `anonymous function 2`. And that is it!
