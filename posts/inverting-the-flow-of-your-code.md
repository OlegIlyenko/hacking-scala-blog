## Inverting the Flow of Your Code

In [previous post](/post/49044250853/side-effecting-without-braces) I described, how you can make side-effects in very concise way. There is one complementary operation that I also use a lot. This time it helps you to visualize the flow of the code. Let me demonstrate you this code:

    case class Item(id: Int, name: String)

    def getItem(id: Int): Item = // ...
    def prepareForView(item: Item): Map[String, String] = // ...
    def renderPage(model: Map[String, String]): String = // ...

    def getItemPage(itemId: Int) = 
        renderPage(prepareForView(getItem(itemId)))

You have probably already saw this kind of code a lot. Such method chaining, that I showed in body of the `getItemPage` method, is pretty common. I personally find it awkward - the code flow is turned inside out. I call the method `renderPage` at first, even if this is something that would be used only at the end and produces the result value of the whole function. My brain just work in other direction: When I'm writing this code, at first I think, how I can get `Item` from `id` and then, how can I get the model for the view ... and finally how to produce page. We can do better!

<!-- more -->

What we can do is to define another small implicit conversion:

    object WorkflowHelper {
        implicit def anyWithWorkflowHelpers[T](target: T) = new {
            def |&gt;[R](fn: T =&gt; R) = fn(target)
        }
    }

It adds `|&gt;` method to any object and allows us to provide some function that transforms it's argument to something else (definition is even easier than `~` method defined in [previous post](/post/49044250853/side-effecting-without-braces)).

Now let's try to use it:

    import WorkflowHelper._

    def getItemPage(itemId: Int) = 
        getItem(itemId) |&gt; prepareForView |&gt; renderPage

Now it feels much more natural, isn't it?

Ok, in this example for every bit of it's logic I had some other function. It's also common, because we often split our logic in several smaller pieces and than have some main method that calls them all. But sometimes methods do not have suitable signature. For example `prepareForView` most probably will also require locale:

    def prepareForView(item: Item, locale: Locale): Map[String, String] = null

But in order to handle this we can use vary nice Scala feature called **currying**. It allows use to adapt `prepareForView` method to our needs:

    def getItemPage(itemId: Int) = 
        getItem(itemId) |&gt; (prepareForView(_, Locale.ENGLISH)) |&gt; renderPage

In this case *underscore* plays the role of placeholder. You generally saying, that provided `Item` instance should be inserted here.

But wait... often we have more than one object to deal with! How can we handle this? Let's say, we have also `User` in play which also should be prepared for view. This time tuple comes to rescue. Here is how we can implement this:

    def prepareForView(item: Item): Map[String, String] = // ...
    def prepareForView(item: User): Map[String, String] = // ...
    def renderPage(model: Map[String, AnyRef]): String = // ...
    def getCurrentUser: User = // ...

    def getItemPage(itemId: Int) = 
        (getItem(itemId) |&gt; prepareForView, getCurrentUser |&gt; prepareForView) |&gt;
            {case (item, user) =&gt; renderPage(Map("item" -&gt; item, "user" -&gt; user))}

In this case I prepare tuple and give it to the another function. Actually it's partial function, that's because I'm also able to make pattern matching directly there! Of course, you can make it a little bit differently and prepare complete model for the page first and then just render the page:

    def getItemPage(itemId: Int) = 
        Map(
            "item" -&gt; (getItem(itemId) |&gt; prepareForView), 
            "user" -&gt; (getCurrentUser |&gt; prepareForView)
        ) |&gt; renderPage

I'm not claiming, that `|&gt;` method is useful in any situations, but I definitely believe, that there are a lot of situations where it can make code more readable and easier to write.

By the way, I'm not the first one who came up with this idea. For example [scalaz](http://code.google.com/p/scalaz/)'s [Identity](http://scalaz.github.com/scalaz/scalaz-2.9.1-6.0.2/doc.sxr/scalaz/Identity.scala.html#48273) trait also has `|&gt;` method which has exactly the same signature and purpose.