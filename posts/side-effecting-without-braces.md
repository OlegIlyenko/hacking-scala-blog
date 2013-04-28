## Side-effecting without braces

Sometimes I find myself in following situation: I have nice one-expression method like this:

    def formatUsers(users: List[User]): String = 
        users.map(_.userName).mkString(", ")


Now I want to print the results to the `System.out` in order to quickly check the result that this function produces. 

Common way of solving this problem is to write something like this:

    def formatUsers(users: List[User]): String = {
        val result = users map (_.userName) mkString ", " 
        println(result)
        result
    }

Too much code for such trivial task. In Scala we can do better than this! 

<!-- more -->

Here is the way you can improve it. At first we need to define small implicit conversion:

<pre><code>
object SideEffects {
    implicit def anyWithSideEffects[T](any: T) = new {
        def ~(fn: T =&gt; Unit) = {
            fn(any)
            any
        }
    }
}
</pre></code>

With this code I can make `~` method available on any object. The only thing it does - is executes function you provided and returns original object. So the result of this function is completely ignored (that's because it's return type is `Unit`), but it's Ok for our purpose - it's exactly what we want to archive here.

Ok, now we can use it. Here is my first attempt:

    import SideEffects._

    def formatUsers(users: List[User]): String = 
        users.map(_.userName).mkString(", ") ~ (s =&gt; println(s))

And, as you can guess - it works. But we can do even better! `println` is also functions that accepts one argument, so we don't need to wrap it in another function:

    def formatUsers(users: List[User]): String = 
        users.map(_.userName).mkString(", ") ~ println

Now it looks much better with much less code, isn't it?

You also can use it everywhere in expressions. Here is another example:

    def formatUsers(users: List[User]): String = 
        users.map(_.userName ~ println).mkString(", ") 

It just prints all processed user names for me. 

Generally side-effects is not very good thing, but sometimes than can be helpful. Debug output, that I demonstrated, is one example. Another good example is integration with Java, which consists mostly of side-effects. Let's look in this code:

	val frame = new JFrame
	frame.setVisible(true)
	useFrame(frame)

You probably already guess how we can improve this situation. Of course with our brand new, shiny `~`!

    useFrame(new JFrame ~ (_ setVisible true))

Hope you will find it helpful and enjoyed this post.

### Small update about theoretical grounds

Seems that this approach even has some theoretical grounds. So if you are interested, it's called **K combinator** and you can find some information about it (with code examples in Ruby) in [this post](http://www.markhneedham.com/blog/2010/05/03/coding-the-kestrel/).

Actually that post reminds me on another use case for the `~` method, that I'm using sometimes. It looks like this:

<pre><code>
def createFrame = new JFrame ~ { frame =&gt;
	import frame._
	
	setVisible(true)
	setTitle("Test Frame")
	setSize(200, 100)
	// ...
}
</pre></code>

I like this pattern because it makes intent of the code much cleaner and visually splits method in 2 parts: the first one defines object you want to return and the second configures it.