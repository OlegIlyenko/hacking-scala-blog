## Introduction to Type Dynamic

In Scala 2.10 class `Dynamic` would be enabled by default, so I think that it's the right time to look at it in more detail. This class can be used for many different purposes. I think, that an integration with dynamic languages is the most obvious one, but I would like to show you something different. I will demonstrate you how to create parameterizable extractors, that can be parametrized directly within `case` expressions.

Class `Dynamic` works very similar to Ruby's `method_missing` or Groovy's `methodMissing`/`propertyMissing`. So you can call non-existing methods on classes that extend `Dynamic`. Such calls would be rewritten by compiler to following method invocations: `applyDynamic`, `applyDynamicNamed`, `selectDynamic`, `updateDynamic`. I will describe each of these methods in the next sections. 

<!-- more -->

### `applyDynamic`

It's, probably, the most popular method. You can define it like this:

	class DynamicTest extends Dynamic {
	    def applyDynamic(methodName: String)(args: Any*) {
	      println(s"You called '$methodName' method with " +
	          s"following arguments: ${args mkString ", "}")
	    }
	}

And as you can see, I can call non-exiting methods on instances of the `DynamicTest` class:

	scala&gt;  val test = new DynamicTest
	test: DynamicTest = DynamicTest@fb7ac7

	scala&gt;  test.method1("a", "b", 123)
	You called 'method1' method with following arguments: a, b, 123

	scala&gt;  test method2 "testing"
	You called 'method2' method with following arguments: testing

### `applyDynamicNamed`

I find it very nice, that named arguments are also supported. Each argument that you pass to the function is converted to the `Tuple` where the first element of the tuple is argument name and the second is value. Here is its signature:

	class DynamicTest extends Dynamic {
	    def applyDynamicNamed(name: String)(args: (String, Any)*) {
	        println(s"You called '$name' method with " +
	            s"following argiuments: ${args map (a =&gt; a._1 + "=" + a._2) mkString ", "}")
	    }
	}

And it's usage (as you can see, argument name would be empty string if you don't use named argument):

	scala&gt;  test.createUser("abc", name = "John", age = 20)
	You called 'createUser' method with following argiuments: =abc, name=John, age=20

By the way, in all examples above I have used `Unit` as return type, but you actually not limited to it. You can return anything you want from these methods, you can even have as many type parameters as you want.

### `selectDynamic`

Now comes setter/getter method (properties) support. If you want to handle method invocation, that does not involve any arguments or parenthesis, than you need `selectDynamic` - it's probably the simplest of them (still it can be very useful as you will see later in my bigger example). You can define it like this:

	class DynamicTest extends Dynamic {
	    def selectDynamic(name: String) = s"value of $name"
	}

And it's usage:

	scala&gt;  test.firstName
	res0: String = value of firstName

	scala&gt;  test.`some other property ?`
	res1: String = value of some other property ?


### `updateDynamic`

This is setter part. You can use it to handle assignments. It's signature:

	class DynamicTest extends Dynamic {
	  def updateDynamic(name: String)(value: Any) {
	    println(s"You have just updated property '$name' with value: $value")
	  }
	}

I actually had issues with this one. If you will try to use it in REPL (and some other contexts), you will get something like this:

	scala&gt;  test.firstName = "John"
	:12: error: value selectDynamic is not a member of DynamicTest
	val $ires0 = test.firstName
	             ^
Seems that compiler treats it as getter, and tries to find `selectDynamic`. I think, that the reason for this is a snapshot version of Scala 2.10 compiler that I currently using. So let's hope that it would be fixed soon. Meanwhile, you can test `updateDynamic` like this:

	object MyApp extends App {
	  val test = new DynamicTest

	  test.firstName = "John"
	} 

This works as expected and prints:

	You have just updated property 'firstName' with value: John

### Parameterizable Extractors

This was my dream for a long time! Wouldn't it be nice, if you can specify regexp directly in pattern matching expression instead of defining it elsewhere and then use your variable in pattern match or to extract specific elements of the `Map`? With type `Dynamic` you can actually archive this, and here is example it's usage:

	Map("firstName" -&gt; "John", "lastName" -&gt; "Doe") match {
		case p.firstName.lastName.Map(
		      Some(p.Jo.StartsWith(fn)),
		      Some(p.`.*(\\w)$`.Regexp(lastChar))) =&gt;
		  println(s"Match! $fn ...$lastChar")
		case _ =&gt; println("nope")
	} 

As you see, I have provided parameters to the extractor and performed pattern match in the same `case` expression. One `selectDynamic` is enough to implement it:

	class ExtractorParams(params: List[String]) extends Dynamic {
	  val Map = new MapExtractor(params)
	  val StartsWith = new StartsWithExtractor(params)
	  val Regexp = new RegexpExtractor(params)

	  def selectDynamic(name: String) =
	    new ExtractorParams(params :+ name)
	}

	object p extends ExtractorParams(Nil)

As you can see, each call to the `selectDynamic` adds on parameter to the parameter `List`, and when I'm finished with parameters, I can call concrete extractor like `StartsWith` or `Map`. These extractor implementations are pretty straightforward:

	class RegexpExtractor(params: List[String]) {
	  def unapplySeq(str: String) =
	    params.headOption flatMap (_.r unapplySeq str)
	}

	class StartsWithExtractor(params: List[String]) {
	  def unapply(str: String) =
	    params.headOption filter (str startsWith _) map (_ =&gt; str)
	}

	class MapExtractor(keys: List[String]) {
	  def unapplySeq[T](map: Map[String, T]) =
	    Some(keys.map(map get _))
	}

### Conclusion

I hope you enjoyed this small introduction to new type `Dynamic`. I also hope, that I was able to demonstrate you, that it can be used for pretty unexpected use-cases and not only for integration with dynamic languages. You can find more information about `Dynamic` type in the [correspondent SIP-17](http://docs.scala-lang.org/sips/pending/type-dynamic.html). 