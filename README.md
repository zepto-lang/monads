# monads

A minimal monad library in and for zepto.

## Usage

The following monads are implemented (with basically the same functions as in their Haskell equivalent):

### Identity

The Identity monad. It won't do anything to the data.

````clojure
(load "monads/identity")
(monads:print (monads:identity "1")) ; => (:identity 1)
```

It's really not that interesting.

### Maybe

The Maybe monad. It has two constructors, `monads:just` and `monads:nothing`:

```clojure
(load "monads/maybe")
(monads:print (monads:just 1)) ; => (:just 1)
(monads:print (monads:nothing)) ; => (nothing nil)
(monads:print ((monads:bind (monads:just 1)) (lambda (val) (monads:just (add1 val))))) ; => (:just 2)
; this gets increasingly tedious the more transformations are applied.
; A doM macro exists to mitigate this, it works with any monad.
; It does not have an implicit return (yet)
(monads:print
  (monads:doM 
    (first-value (monads:just 1))
    (second-value (monads:just 2))
    (monads:just (+ first-value second-value)))) ; => (:just 3)
; of course, nothing will compose, but hijack the computation
(monads:print
  (monads:doM
    (first-value (monads:just 1))
    (boom (monads:nothing))
    (second-value (monads:just 2))
    (monads:just (+ first-value second-value)))) ; => (:nothing nil)
```

### Either

The Either monad. Like Maybe it has two constructors, `monads:left` and `monads:right`,
the only difference is that the error case (Left) contains a error message/object:

```clojure
(load "monads/either")
(monads:print (monads:right :foobar)) ; => (:right :foobar)
(monads:print (monads:left "the bomb exploded :(")) ; (:left the bomb exploded :()
; To do something with the Either monad, use the monads:either macro like so:
(monads:either (monads:left "that sucks, mate") ; first the monad
  (val => (monads:right (add1 val))) ; then what should happen in the right case
  (err => (error err "in monad"))) ; lastly what should happen in the left case
```

That's all, folks!

### Serial

The Serial monad. It assigns a serial number to all monad instances that are
consecutively created in calls to `monads:bind` and, by extension, in a `monads:doM`
construct. The default value (if none is supplied in the constructor) is 1.

```clojure
(load "monads/serial")
(monads:print
  (monads:doM
    (x (monads:serial "first value"))
    (y (monads:serial "second value"))
    (z (monads:serial "third value"))
    (monads:serial "last"))) ; this serial monad will have the number 4
```

I am not sure the implementation is sensible that way. It is somewhat convenient, though.

### List

The List monad. It takes in a list as value. Bind functions are performed
element-wise. Usage could look like this:

```clojure
(load "monads/list")
(write
  (monads:doM
    (el (monads:list [1 2 3 4]))
    (* el 2))) ; => (2 4 6 8)
```

## Hacking your own monads

There is a special monads datatype defined in "monads/monads", it is constructed like so:

````clojure
(load "monads/monads")
(define bind (lambda (f) (error "nope, not today")))
(define return (lambda (val) (error "i don't exist either")))
(monads:generic "the value it should take" :the-name-of-the-monad return bind)
; A full example:
(define (monads:identity' value)
 (monads:generic value :another-identity (lambda (val) (monads:identity val)) (lambda (f) (f value))))
```

There is a bit of redundancy there, I admit. But it works.

<br/>

Have fun!
