title: 英语角
date: 2014-05-17 22:04:41
---

* The callback mechanism we have shown is *sufficient* to chain future results with *subsequent computations*.

* We therefore create another future purchase which makes a decision to buy only if it’s *profitable* to do so, and then sends a request. 

* We would have to repeat this pattern within the onSuccess callback, making the code *overly indented, bulky and hard to reason about.*

* For these two reasons, futures provide combinators which allow a more *straightforward composition.*

* One of the basic combinators is map, which, given a future and a mapping function for the value of the future,
produces a new future that is completed with the mapped value once the original future is successfully completed. 

* You can *reason about* mapping futures in the same way you reason about mapping collections.

* This exception *propagating semantics* is present in the rest of the combinators, as well.


<!-- [翻译](/2014/05/15/scala文档翻译-future&promise#functional-composition-and-for-comprehensions) -->

* I *procrastinate*… then *scramble* to get things done at the last second.
