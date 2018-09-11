---
categories: 
  - apprenticeship
date: "2017-05-25T08:30:00Z"
mathjax: true
title: Mutual Recursion and Trampoline
---

Something that you learn pretty quickly working with Clojure is to prefer `recur` over normal recursive function calls for deeply nested recursive function calls. In the example below, the pure recursive version of `factorial` results in a `StackOverflowError`, while the version using `recur` works fine.
 
Many other functional languages guarantee that recursive tail calls (such as `factorial-tc`) will consume only constant stack space. Clojure doesn’t provide this guarantee. Users have to explicitly specify when this optimization should be made by using `recur` [[1](https://clojure.org/about/functional_programming)]. The benefit is that users know for certain that recursive calls using `recur` will only consume constant stack space, additionally the compiler alerts users if they're trying to make a non-tail call to `recur`. Whereas, for languages that automatically optimize tail calls, users wouldn’t necessarily know if they made a mistake by supplying a non-tail call, and thereby were no longer guaranteed constant stack size for the recursive calls.
 
```clojure
(defn factorial [n]
  (loop [cnt n]
    (if (< cnt 2)
      1
      (* n (factorial (dec n))))))
 
(factorial 100003N)
;StackOverflowError
 
(defn factorial-tc [n]
  (loop [cnt n acc 1]
    (if (< cnt 2)
      acc
      (recur (dec cnt) (* acc cnt)))))
 
(factorial-tc 100003N)
;A very big number
```
 
However, it's not always possible to use `recur`. One such case where `recur` is not possible is mutual recursion. In a mutual recursion each function is defined in terms of the other. Wikipedia uses the following example of determining if a number is even or odd to demonstrate mutual recursion [[2](https://en.wikipedia.org/wiki/Mutual_recursion)]. Since neither `mutual-even?` nor `mutual-odd?` is calling itself it’s not possible to use `recur` in this situation.
 
```clojure
(defn is-even? [n]
  (letfn
    [(mutual-odd? [n]
       (if (= n 0)
         false
         (mutual-even? (- n 1))))
     (mutual-even? [n]
      (if (= n 0)
        true
        (mutual-odd? (- n 1))))]
    (mutual-even? n)))
    
(is-even? 10)
;true
(is-even? 11)
;false
(is-even? 10000)
;StackOverflowError  
```
 
However, for functions that are defined mutually recursively Clojure provides the `trampoline` function as a way to maintain the mutually recursive structure, while still guaranteeing constant stack size. It’s pretty easy to convert the non-trampolined function to the trampolined version. The only change required is that each recursive function call must be converted to an anonymous function and then the initial function call invoked via `trampoline`. Notice that the trampolined version of `is-even?` looks almost exactly the same as the non-trampolined version except for these very slight changes. The benefit is that there is no longer a `StackOverflowError` for large numbers.
 
```clojure
(defn trampoline-is-even? [n]
  (letfn
    [(mutual-odd? [n]
       (if (= n 0)
         false
         #(mutual-even? (- n 1))))
     (mutual-even? [n]
       (if (= n 0)
         true
         #(mutual-odd? (- n 1))))]
    (trampoline mutual-even? n)))
 
(trampoline-is-even? 10)
;true
(trampoline-is-even? 11)
;false
(trampoline-is-even? 10000)
;true
```
 
The `trampoline` function is pretty simple (see below). Each call invokes its argument `f` and binds the return value to `ret`. If `ret` is a function it recurses with `ret` as its new arguement and if not it returns the value of `ret`. However, remember that the various function calls have been converted to now return anonymous functions, so it’s no longer necessary to maintain a reference to the original calling functions (as they’re now actually returning something). The `trampoline` then handles the invocation of these function references bouncing between its scope and the various callable functions scopes' (like a trampoline!). 
 
```clojure
(defn trampoline
  "trampoline can be used to convert algorithms requiring mutual
  recursion without stack consumption. Calls f with supplied args, if
  any. If f returns a fn, calls that fn with no arguments, and
  continues to repeat, until the return value is not a fn, then
  returns that non-fn value. Note that if you want to return a fn as a
  final value, you must wrap it in some data structure and unpack it
  after trampoline returns."
  {:added "1.0"
   :static true}
  ([f]
   (let [ret (f)]
     (if (fn? ret)
       (recur ret)
       ret)))
  ([f & args]
   (trampoline #(apply f args))))
```
 
The even / odd function is a pretty contrived example, since there are obviously better ways to check if a number is even or odd. Also, if we wanted to maintain the recursive structure for `is-even?` and `is-odd?`, mutually recursive calls can generally be converted to a singly recursive call by inlining one of the functions like below.   
 
```clojure
(defn is-even? [n]
  (if (= n 0)
    true
    (if (= (- n 1) 0)
      false
      (recur (- (- n 1) 1)))))
```
 
However, sometimes it's much easier and more natural to define functions using mutual recursion. Finite state machines are one such case. Finite state machines are used to represent an abstract machine that can be in only one state at any given time. They’re a very powerful, but compact way to represent complicated rules and interactions with inputs, so they are a broadly useful tool that _"are significant in many different areas, including electrical engineering, linguistics, computer science, philosophy, biology, mathematics, and logic."_[[3](https://en.wikipedia.org/wiki/Finite-state_machine)] 
 
A FSM is described by an initial state, a list of potential states, along with conditional statements describing the transitions from state to state. Below is an example of a finite state machine from the [_The Joy of Clojure_](https://www.manning.com/books/the-joy-of-clojure-second-edition). 
 
Each state contains a list of valid transitions, additionally some states contain the keyword `:done`. The FSM can terminate either when it receives the keyword `:done` if valid or ends up in an `invalid` state. Otherwise, it transitions to the next state and processes the next instruction. You can see how each state and its associated transitions are defined mutually recursively. This results in a very clear description of valid states along with how the transitions can occur.
 
```clojure
;trampoline + elevator
(defn elevator-trampoline [commands]
  (letfn
    [(ff-open [[_ & r]]
       "When the elevator is open on the 1st floor
       it can either close or be done."
       #(case _
          :close (ff-closed r)
          :done true
          false))
     (ff-closed [[_ & r]]
       "When the elevator is closed on the 1st floor
        it can either open or go up."
       #(case _
          :open (ff-open r)
          :up (sf-closed r)
          false))
     (sf-closed [[_ & r]]
       "When the elevator is closed on the 2nd floor
       it can either go down or open."
       #(case _
          :down (ff-closed r)
          :open (sf-open r)
          false))
     (sf-open [[_ & r]]
       "When the elevator is open on the 2nd floor
       it can either close or be done"
       #(case _
          :close (sf-closed r)
          :done true
          false))]
    (trampoline ff-open commands)))
    
(def boring-elevator-ride
  (concat (take 100000 (cycle [:close :open])) '(:done)))
 
(elevator-trampoline [:close :up :open :close :down :open :done])
;true
(elevator-trampoline [:close :open :close :up :open :open :done])
;false
;(elevator-trampoline boring-elevator-ride)
;true
 
;No trampoline + elevator
(defn elevator-no-trampoline [commands]
  (letfn
    [(ff-open [[_ & r]]
       "When the elevator is open on the 1st floor
       it can either close or be done."
       (case _
         :close (ff-closed r)
         :done true
         false))
     (ff-closed [[_ & r]]
       "When the elevator is closed on the 1st floor
        it can either open or go up."
       (case _
         :open (ff-open r)
         :up (sf-closed r)
         false))
     (sf-closed [[_ & r]]
       "When the elevator is closed on the 2nd floor
       it can either go down or open."
       (case _
         :down (ff-closed r)
         :open (sf-open r)
         false))
     (sf-open [[_ & r]]
       "When the elevator is open on the 2nd floor
       it can either close or be done"
       (case _
         :close (sf-closed r)
         :done true
         false))]
    (ff-open commands)))
 
(elevator-no-trampoline [:close :up :open :close :down :open :done])
;true
(elevator-no-trampoline [:close :open :close :up :open :open :done])
;false
(elevator-no-trampoline boring-elevator-ride)
;StackOverflowError   clojure.lang.LazySeq.seq (LazySeq.java:49)
```
The two versions of elevator are exactly the same except the first uses `trampoline` to deal with the mutually recursive calls while the 2nd version does not. Again, the non-trampolined version is fine when a limited number of transitions are supplied as inputs, however when a large number of transitions are required like in `boring-elevator-ride` a `StackOverflowError` occurs for the non-trampoline version. 
 
It's interesting to work through the call to `trampoline` + `elevator`. First, `trampoline` creates a new anonymous function `#(apply ff-open commands)`. `ff-open` is then invoked and its results bound to `ret`. Also, notice that `ff-open` destructures its arguments into `_` and `r`, it then evaluates `_` in the `case` statement. If a valid “non `:done`” transition is supplied `ff-open` encloses the remaining arguments `r` inside of another anonymous function and returns this uninvoked function. `trampoline` then invokes this new anonymous function with the remaining arguments. This process continues until a non-function is returned from `elevator-trampoline`. I thought this was a pretty neat example and it seems very Clojuresque to use recursion + destructuring to jointly process the function calls and the initial argument list.
 
Mutual recursion isn't something that is extremely common, but there are certain situations when it's definitely the most reasonable and potentially the only way to define certain functions. In these cases Clojure via `trampoline` makes it very easy for users to make use of mutual recursion while guarding against stack overflow issues.
