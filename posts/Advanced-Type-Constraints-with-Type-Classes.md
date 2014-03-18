## Advanced Type Constraints with Type Classes

This weekend I was continuing my work on the next version of [Scaldi][1]. Once again I faced the problem that was bothering me quite some time. In [Scaldi][1] you can inject a dependency with something like this:

    val server = inject [HttpServer]

In this case `inject` method will get `HttpServer` class as a type argument and everything works as expected. But what would happen if you forget to specify the type or you just defined the type of the `server` but don't provide it explicitly to the `inject` method? Will it even compile or maybe type inference will be able to figure out the type?

    val server = inject
    val server1: HttpServer = inject

Unfortunately, in both cases it will compile, but the inferred type would be `Nothing`. It appears to be a [known problem][2] and it will not be fixed. Ideally I would like to see a compilation error in this case, so I started to look for a solution. After some time I actually was able to find the way to prevent `Nothing` inference and, by taking this idea even further, found a way to define type constraints in a very generically in the method signature. They will allow you, for example, to describe union types with `(T =:= String) Or (T =:= Int)` or even more complex rules like `(T <:< Color) And Not[T =:= Color] And Not[(T =:= Yellow.type) Or (T =:= Blue.type)]`. 

[[MORE]]

### Preventing "Nothing" Inference

Type `Nothing` has very interesting property by being a [bottom type][3], which means that it's a subtype of all other classes. And it turns out that you can use (or misuse) this property in order to implement a type class, that can detect that something is **not** `Nothing`, because all other classes simply don't have this property (except other bottom types). So lets define `NotNothing` type class:

    @implicitNotFound("Sorry, type inference was unable to figure out the type. You need to provide it explicitly.")
    trait NotNothing[T]
  
    object NotNothing {
      private val evidence: NotNothing[Any] = new Object with NotNothing[Any]
  
      implicit def notNothingEv[T](implicit n: T =:= T): NotNothing[T] =
        evidence.asInstanceOf[NotNothing[T]]
    }

Interestingly enough compiler would not be able to find an `NotNothing[Nothing]` evidence in this case. It still puzzles me why exactly this is the case, but looks like it has something to do with type inference algorithm. I made a small experiment that shows, that as soon as compiler tries to infer T, then `=:=` will fail for `Nothing`:

    scala> def foo[T](implicit a: T =:= T) = a
    foo: [T](implicit a: =:=[T,T])=:=[T,T]

    scala> foo[Nothing]
    res2: =:=[Nothing,Nothing] = <function1>

    scala> foo
    <console>:9: error: type mismatch;
     found   : =:=[Nothing,Nothing]
     required: =:=[T,T]
    Note: Nothing <: T, but class =:= is invariant in type From.
    You may wish to investigate a wildcard type such as `_ <: T`. (SLS 3.2.10)
    Note: Nothing <: T, but class =:= is invariant in type To.
    You may wish to investigate a wildcard type such as `_ <: T`. (SLS 3.2.10)
                  foo
                  ^

If you know exactly how and why it works like this, then I would appreciate it you share it in the comments.

Anyways, this seem to work, so let's define `inject` method in terms of `NotNothing`:

    def inject[T](implicit tt: TypeTag[T], nn: NotNothing[T]) =
      tt.toString

and then test it in REPL:

    scala> inject
    <console>:12: error: Sorry, type inference was unable to figure out the type. You need to provide it explicitly.
                  inject
                  ^

    scala> inject[Nothing]
    <console>:12: error: Sorry, type inference was unable to figure out the type. You need to provide it explicitly.
                  inject[Nothing]
                        ^

    scala> val foo: String = inject
    <console>:11: error: Sorry, type inference was unable to figure out the type. You need to provide it explicitly.
           val foo: String = inject
                             ^

    scala> inject[String]
    res6: String = TypeTag[String]

    scala> inject[Int]
    res7: String = TypeTag[Int]

As you can see, it works as expected - code does not compile as soon as `Nothing` is involved. And as a bonus, you can even specify a nice error message with `implicitNotFound` annotation.

### Implementing "Not" Type Class

It this point I decided to bring this idea even further. If I'm able to produce an evidence that some type `T` is not `Nothing`, then I also should be able to prove, that some type `T` is does **not** have some evidence. It actually was my dream for quite some time to be able to get an evidence that some other evidence does not exist rather than get an evidence when it exists (which is the standard ways to work with type classes). It turns out that it's not only possible, but also pretty straightforward to implement.

In order to do this, we need some type to represent an existence/non-existence:

    sealed trait Existence
    trait Exists extends Existence
    trait NotExists extends Existence

Now lets create a type class, that can prove, that some other evidence exists or does not exist:

    trait IsTypeClassExists[TypeClass, Answer]
  
    object IsTypeClassExists {
      private val evidence: IsTypeClassExists[Any, Any] =
        new Object with IsTypeClassExists[Any, Any]
  
      implicit def typeClassExistsEv[TypeClass, Answer](implicit a: TypeClass) =
        evidence.asInstanceOf[IsTypeClassExists[TypeClass, Exists]]
  
      implicit def typeClassNotExistsEv[TypeClass, Answer] =
        evidence.asInstanceOf[IsTypeClassExists[TypeClass, NotExists]]
    }

And finally lets implement `Not` type class, that can be easily expressed in terms of `IsTypeClassExists`:

    @implicitNotFound("Argument does not satisfy constraints: Not ${T}")
    trait Not[T]
  
    object Not {
      private val evidence: Not[Any] = new Object with Not[Any]
  
      implicit def notEv[T, Answer](implicit a: IsTypeClassExists[T, Answer], ne: Answer =:= NotExists) =
        evidence.asInstanceOf[Not[T]]
    }

This type class is pretty powerful and can give us a proof, that some other type class does not exist for some arbitrary type `T`. Lets first reimplement `inject` method in terms of `Not`:

    def inject[T](implicit tt: TypeTag[T], nn: Not[T =:= Nothing]) =
      tt.toString

and repeat our REPL session:

    scala> inject
    <console>:11: error: Argument does not satisfy constraints: Not =:=[T,Nothing]
                  inject
                  ^
    
    scala> inject[Nothing]
    <console>:11: error: Argument does not satisfy constraints: Not =:=[Nothing,Nothing]
                  inject[Nothing]
                        ^
    
    scala> val foo: String = inject
    <console>:10: error: Argument does not satisfy constraints: Not =:=[T,Nothing]
           val foo: String = inject
                             ^
    
    scala> inject[String]
    res3: String = TypeTag[String]
    
    scala> inject[Int]
    res4: String = TypeTag[Int]

As you can see, it behaves exactly like previous version, but now it's implemented in much more generic way. Error message is a little bit more cryptic but I find it pretty straightforward to understand it's meaning.  

### Describing Arbitrary Complex Type Constraints

In order to make this picture complete, we are still missing `Or` and `And` type classes. Implementing these should be even easier than `Not`, so let's start with `And`:

    @implicitNotFound("Argument does not satisfy constraints: ${A} And ${B}")
    trait And[A, B]
  
    object And {
      private val evidence: And[Any, Any] = new Object with And[Any, Any]
  
      implicit def bothExistEv[A, B](implicit a: A, b: B) =
        evidence.asInstanceOf[And[A, B]]
    }

and `Or`:

    @implicitNotFound("Argument does not satisfy constraints: ${A} Or ${B}")
    trait Or[A, B]
  
    object Or {
      private val evidence: Or[Any, Any] = new Object with Or[Any, Any]
  
      implicit def aExistsEv[A, B](implicit a: A) =
        evidence.asInstanceOf[Or[A, B]]
  
      implicit def bExistsEv[A, B](implicit b: B) =
        evidence.asInstanceOf[Or[A, B]]
    }

Now we should be able to describe any kind of logical expressions, that constrain the type, in the method signature. Let's go though some examples.

**Union types? No problem whatsoever**

    def union[T](t: T)(implicit c: (T =:= String) Or (T =:= Int)) = t match {
      case s: String => println(s"Some nice string: $s")
      case i: Int => println(s"Some int: $i")
    }

or if like, you can define nice type lambda:

    type V[A, B] = {type l[T] = (T <:< A) Or (T <:< B)}

    def union[T: (String V Int)#l](t: T) = t match {
      case s: String => println(s"Some nice string: $s")
      case i: Int => println(s"Some int: $i")
    }

and make sure that it works as expected:
  
    scala> union("Test")
    Some nice string: Test
    
    scala> union(265)
    Some int: 265
    
    scala> union(2.0)
    <console>:11: error: Argument does not satisfy constraints: =:=[Double,String] Or =:=[Double,Int]
                  union(2.0)
                       ^

**More complex examples**

Here I want to sort the list of any element type except lists with strings and numbers in it

    def sort[T: Ordering](list: List[T])(implicit c: Not[(T =:= String) Or Numeric[T]]) =
      list.sorted

Or in this example I want method to accept only specific colors:

    sealed trait Color
  
    case object Red extends Color
    case object Green extends Color
    case object Blue extends Color
    case object Yellow extends Color
    case object Magenta extends Color

    def printColor[T](t: T)(implicit c: (T =:= Red.type) Or (T =:= Green.type) Or (T =:= Magenta.type)) =
      println(t) 

or instead of defining a whitelist, I can define a blacklist:

    def printColor[T](t: T)(implicit c: (T <:< Color) And Not[T =:= Color] And Not[(T =:= Yellow.type) Or (T =:= Blue.type)]) =
      println(t)

and in REPL:

    scala> printColor(Yellow)
    <console>:11: error: Argument does not satisfy constraints: And[<:<[Yellow.type,Color],Not[=:=[Yellow.type,Color]]] And Not[Or[=:=[Yellow.type,Yellow.type],=:=[Yellow.type,Blue.type]]]
                  printColor(Yellow)
                            ^
    
    scala> val c: Color = Magenta
    c: Color = Magenta
    
    scala> printColor(c)
    <console>:12: error: Argument does not satisfy constraints: And[<:<[Color,Color],Not[=:=[Color,Color]]] And Not[Or[=:=[Color,Yellow.type],=:=[Color,Blue.type]]]
                  printColor(c)
                            ^
    
    scala> printColor(Magenta)
    Magenta

This is the power of type classes. I hope you enjoyed! I also created [a gist for you to play with][4].

[1]: http://scaldi.org
[2]: https://issues.scala-lang.org/browse/SI-2609
[3]: http://en.wikipedia.org/wiki/Bottom_type
[4]: https://gist.github.com/OlegIlyenko/8508980