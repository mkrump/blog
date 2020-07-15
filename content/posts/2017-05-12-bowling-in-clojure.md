---
categories: 
  - apprenticeship
date: "2017-05-12T15:30:00Z"
title: Bowling in Clojure
---

I just finished my first two weeks working with Clojure. It was my first exposure to Clojure, and it was the first time that I’ve worked with a purely functional language, so there was definitely some adjustment required. At the same time not everything feels completely foreign, since many data analysis tools tend to use a functional approach. For example, below is one way to calculate average `Sepal.Length` by `Species` for the `iris` dataset in R. The second argument to the `tapply` function acts as a partitioning vector for the first argument, while the `mean` function is applied to each of the partitioned vectors.

```R
head(iris)
#  Sepal.Length Sepal.Width Petal.Length Petal.Width Species
#1          5.1         3.5          1.4         0.2  setosa
#2          4.9         3.0          1.4         0.2  setosa
#3          4.7         3.2          1.3         0.2  setosa
#4          4.6         3.1          1.5         0.2  setosa
#5          5.0         3.6          1.4         0.2  setosa
#6          5.4         3.9          1.7         0.4  setosa

tapply(iris$Sepal.Length, iris$Species, mean)
#    setosa versicolor  virginica
#     5.006      5.936      6.588
```

So using higher order functions and passing functions as parameters feels relatively familiar. However, some other things definitely felt very weird. Not being able to assign a variable for instance is something that I’m still figuring out how to cope with. However, it’s been a good experience and I’m enjoying learning something so completely different from what I’m used to. 

Each morning I’ve been doing the [Bowling Kata](http://butunclebob.com/files/downloads/Bowling%20Game%20Kata.ppt) in Clojure. The idea is to create a simple `scorer` that scores a series of rolls, adjusting for strikes and spares. It’s a problem that seems very natural to think about in an object-oriented language like C#, “Loop over an array of rolls, incrementing the score if it’s a strike I do this … etc”. However, I found it kind of hard to think about this problem functionally. I’m sure part of it is that I just need to deprogram myself after years of working with imperative / object-oriented languages. 

The first week I just created my own solution without looking at any Clojure solutions. I ended up with a somewhat new solution each day, but every version had a decidedly iterative flavor to it. In the version below you can see me passing around an array with all of the various items that I need to track, while iterating over all of the rolls. I didn’t think this was too terrible for a first attempt, but at the same time it didn’t really feel in the spirit of a functional language like Clojure.

```clojure
(defn rolls [] (vec (repeat 21 0)))

(defn roll [rolls roll pins] (assoc rolls roll pins))

(defn get-n-rolls [rolls roll n] (subvec rolls roll (+ roll n)))

(defn roll-aggregator [number-of-rolls]
  (partial (fn [rolls roll]
             (reduce + (get-n-rolls rolls roll number-of-rolls)))))
             
(defn spare? [rolls roll] (= 10 ((roll-aggregator 2) rolls roll)))

(defn spare-bonus [rolls roll] ((roll-aggregator 3) rolls roll))

(defn strike? [rolls roll] (= 10 ((roll-aggregator 1) rolls roll)))

(defn strike-bonus [rolls roll] ((roll-aggregator 3) rolls roll))

(defn no-bonus [rolls roll] ((roll-aggregator 2) rolls roll))

(defn score
  ([rolls] (score rolls 0 0 0))
  ([rolls frame roll current-score]
   (if (< frame 10)
     (cond (strike? rolls roll)
           (recur rolls (inc frame) (+ 1 roll) (+ current-score (strike-bonus rolls roll)))
           (spare? rolls roll)
           (recur rolls (inc frame) (+ 2 roll) (+ current-score (spare-bonus rolls roll)))
           :else
           (recur rolls (inc frame) (+ 2 roll) (+ current-score (no-bonus rolls roll))))
     current-score)))
```

The second week, I practiced the kata using [Jeremy Neander's solution](http://www.jneander.com/writing/bowling-game-kata-in-clojure/), which I’ve reposted below. A decent amount of the two solutions are similar, but the two score functions definitely look very different. The functional version is very declarative and almost reads like a sentence, “sum the score of 10 frames". The recursive call to `to-frames` combined with dropping the already processed rolls feels really elegant, very functional, and was something that I wouldn’t have thought to do. 

```clojure
(defn sum [rolls] (reduce + rolls))

(defn spare? [rolls]
  (= 10 (sum (take 2 rolls))))

(defn strike? [rolls]
  (= 10 (sum (take 1 rolls))))

(defn rolls-for-frame [rolls]
  (if (or (spare? rolls)
          (strike? rolls))
    (take 3 rolls)
    (take 2 rolls)))

(defn rest-rolls [rolls]
  (if (strike? rolls)
    (drop 1 rolls)
    (drop 2 rolls)))

(defn to-frames [rolls]
  (lazy-seq
    (cons (rolls-for-frame rolls)
      (to-frames (rest-rolls rolls)))))

(defn score [rolls]
  (sum (flatten (take 10 (to-frames rolls)))))

```