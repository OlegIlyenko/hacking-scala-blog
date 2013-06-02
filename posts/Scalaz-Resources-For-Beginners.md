## Scalaz - Resources For Beginners

[Scalaz](http://code.google.com/p/scalaz/) is very interesting Scala library. It can be pretty scary when you first look at it and at examples of it's usage. I also find, that at the beginning, advantages of this library are not very obvious. So at some point I asked myself: why are people so enthusiastic about it? So I started to learn it, but found that it's hard to find any resources that are targeting beginners - people who are new to Haskell, Category Theory, or advanced Scala features like *Type Classes* or *Higher Kinds*.

In this post I want to summarize all resources I found about scalaz or related to scalaz that are actually approachable by beginners. I started with [question at StackOverflow](http://stackoverflow.com/questions/4863671/good-scalaz-introduction) where I received many good answers. I also noticed, that this question was pretty popular, so I decided to write this post where I can summarize the answers and maybe add something more.

I personally believe that even if you don't completely understand every aspect of scalaz yet (I'm definitely not), it still can help you to write a better code. As far as I can can see, it helps to write more composable, reusable and side-effect free code. But in order to use it, you need to understand some key concepts that it heavily relies on. So let's start with ...

<!-- more -->

### Type Classes

Type Classes playing important role in Scala and they are really useful concept. They generally allow you to define some additional behavior (methods) to the class independently from this class itself. I believe, that this concept was taken from Haskell and implemented in Scala with the help of implicit parameters. Type classes can also help (at least partly) to solve [Expression problem](http://en.wikipedia.org/wiki/Expression_problem). You can find these presentations helpful on this topic:

* [Scalaz Presentation](http://vimeo.com/10482466) (by [Nick Partridge](https://twitter.com/#!/nkpart))
* [The ease of Scalaz](http://days2011.scala-lang.org/node/138/275) (by [Heiko Seeberger](https://twitter.com/#!/hseeberger))

I can highly recommend Nick's presentation - it's mostly not about Scalaz, but about the concepts it uses. Step by step, he implements these concepts in a simple example. For me personally it was eye opening. Heiko shows some of the features of Scalaz, including interesting of type-safe equals implemented with type class. 

I can also recommend you to watch Daniel's presentation, described in the next section, which touches this topic.

### Higher Kinds

> *Values* are to *types* as *types* are to *kinds* - *Daniel Spiewak*

It's very nice  feature of Scala language, which allows you to work with containers like `List` or `Option` in very generic manner. I highly recommend you this watch this presentation on this topic:

* [High Wizardry in the Land of Scala](http://vimeo.com/28793245) (by [Daniel Spiewak](https://twitter.com/#!/djspiewak))

Daniel describes 3 concepts: *Higher Kinds*, *Type Classes* and *Continuations*. He also shows, how you can implement heterogeneous list with this knowledge, as an example.

### Scalaz Itself

Here are some inspiring presentation about scalaz, which I would highly recommend:

* [Scalaz](http://skillsmatter.com/podcast/scala/scalaz) (by [Chris Marshall](https://twitter.com/#!/oxbow_lakes))
* [Practical Scalaz: making your life easier the hard way](http://skillsmatter.com/podcast/scala/practical-scalaz-2518) (by [Chris Marshall](https://twitter.com/#!/oxbow_lakes))
* [Scalaz: Functional Programming in Scala](http://www.infoq.com/presentations/Scalaz-Functional-Programming-in-Scala) (by [R&uacute;nar &Oacute;li Bjarnason](https://twitter.com/#!/runarorama))

in these videos Chris gives introduction to some useful features of Scalaz, describing them from the more practical perspective. 

If you want nice and quick introduction to scalaz, than I can recommend you this very nice [Scalaz "For the Rest of Us" Cheat Sheet](https://github.com/arosien/scalaz-base-talk-201208/raw/master/scalaz-cheatsheet.pdf) with correspondent [presentation slides](http://arosien.github.com/scalaz-base-talk-201208) by [Adam Rosien](https://twitter.com/arosien).

Here are some other resources for beginners:

* [The Scalaz Guide](http://dnene.bitbucket.org/docs/scalaz-guide/initial.html)
* [Learn you a scalaz - project on GitHub](https://github.com/jrwest/learn_you_a_scalaz)
* [Jason Zaugg's Intro to Scalaz](http://vimeo.com/15264203) (by [Jason Zaugg](https://twitter.com/#!/retronym))
* [learning Scalaz](http://eed3si9n.com/learning-scalaz/) (by [Eugene Yokota](https://twitter.com/eed3si9n))
* [Debasish Ghosh - posts about scalaz](http://debasishg.blogspot.com/search/label/scalaz) - as far as I saw, he describes more advanced topics
* [Functional IO in Scala with Scalaz](http://www.stackmob.com/2011/12/scalaz-post-part-2/) (from [StackMob](https://twitter.com/#!/stackmob))

### Lenses 

Lenses are generally something similar to getters and setters in function world.  Here are some useful resources about them:

* [Lenses: A Functional Imperative](http://www.youtube.com/playlist?p=PLEDE5BE0C69AF6CCE) (by [Edward Kmett](https://twitter.com/#!/kmett))
* [An Introduction to Lenses in Scalaz](http://www.stackmob.com/2012/02/an-introduction-to-lenses-in-scalaz/) (from [StackMob](https://twitter.com/#!/stackmob))

### Category Theory

In the hart of Scalaz (as well as Haskell) lies category theory. It's interesting topic, but can be somehow involving. [Heiko Seeberger](https://twitter.com/#!/hseeberger) wrote a great articles on this topic, which I find very beginner-friendly:

* [Introduction to Category Theory in Scala](http://hseeberger.wordpress.com/2010/11/25/introduction-to-category-theory-in-scala/)
* [Applicatives are generalized functors](http://hseeberger.wordpress.com/2011/01/31/applicatives-are-generalized-functors/)

If you still wonder what monad is, then following video will definitively give you some hints. I also find it very friendly to Java developers, that want to find out what actually monads are and their advantages. Video includes explanation of Option, Validation and List monads:

* [Scala Monads: Declutter Your Code With Monadic Design](http://marakana.com/s/scala_monads_declutter_your_code_with_monadic_design,1034/index.html) (by Rob Nikzad)

A lot of ideas, implemented in Scalaz, are borrowed from Haskell, so it can be helpful to learn some Haskell too. You can find a lot of information on this topic in this chapter of [Learn You a Haskell for Great Good!](http://learnyouahaskell.com/) book:

* [Functors, Applicative Functors and Monoids](http://learnyouahaskell.com/functors-applicative-functors-and-monoids)

### Conclusion

I found, that absorption of completely new concepts, is pretty hard and slow process where getting motivated is very important. I hope, that this material will motivate you and would be helpful in you thirst for knowledge.

If you know some other resources about Scalaz, then please share them with me - I would be very grateful and they will help me to enhance this post with even more material.