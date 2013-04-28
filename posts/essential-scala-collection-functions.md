## Essential Scala Collection Functions

Let me ask you a question. How many times you wrote similar code?

<pre><code>case class User(id : Int, userName: String)

val users: List[User] = // ....
val resultUsers: List[User] = // ....

for (i &lt;- 0 until users.size) {
    if (users(i).userName != "test") {
        resultUsers += users(i)
    }
}
</code></pre>

May imperative languages like Java, C, C++ have very similar approach for iteration. Some of them have some syntactic sugar (like Java for each loop). There are many problems with this approach. I think the most important annoyance is that it's hard to tell, what this code actually does. Of course, if you have 10+ years Java experience, you can tell it within a second, but what if the body of the `for` is more involving and has some state manipulatin? Let me show you another example:

<pre><code>def formatUsers(users: List[User]): String = {
	val result = new StringBuilder 

	for (i &lt;- 0 until users.size) {
		val userName = users(i).userName

		if (userName != "test") {
			result append userName

			if (i &lt; users.size - 1) {
				result append ", "
			}
		}
	}

	return result.toString
}
</pre></code>

Was it easy to spot the purpose of this code? I think more important question: how do you read and try to understand this code? In order to understand it you need to iterate in your head - you create simple trace table in order to find out how values of `i` and `result` are changing with each iteration. And now the most important question: is this code correct? If you think, that it is correct, then you are wrong! It has an error, and I leave it to you to find it. It's pretty large amount of code for such a simple task, isn't it? I even managed to make an error it.

I'm going to show you how this code can be improved, so that intent is more clear and a room for the errors is much smaller. I'm not going to show you complete Scala collection API. Instead I will show you 6 most essential collection functions. These 6 function can bring you pretty far in your day-to-day job and can handle most of your iteration needs. I going to cover following functions: filter, map, flatMap, foldLeft, foreach and mkString.

Also note, that these functions will work on any collection type like `List`, `Set` or even `Map` (in this case they will work on tuple `(key, value)`) 

<!-- more -->

### filter

`filter` is easy one. Let me show you how it can be used in order to simplify my first example:

    val resultUsers = users filter (_.userName != "test")

As you can see, what left is most essential part of my algorithm. `filter` takes a predicate function as it's argument (function that returns `Boolean`: <code>User =&gt; Boolean</code> in my example) and uses it to filter the collection.

### map

`map` converts one collection into another with the help of your function. Here is an example:

	val userNames = users map (_.userName)

In this code I just converting my list of users in list of user names. As you can guess `userNames` has type `List[String]`.

What if I have two kinds of users: `AdminUser` and `NormalUser`. For each of them I want to use different logic - I don't want to leak user names of my admins! In place of function I can use something called partial function - it's special function that can have only one argument and can be defined only for subset of it's values. The best thing about it, is that it has special syntax, that allows to pattern match on it's argument. Here is an example that describes how it looks like:

<pre><code>case class AdminUser(userName: String)
case class NormalUser(userName: String)

val users = List(AdminUser("bob"), NormalUser("john"))
val userNames = users map {
    case NormalUser(userName) =&gt; userName
    case _ =&gt; "secret"
}

assert(userNames == List("secret", "john"))
</pre></code>


This feature can be also useful, if you are working with `Map` collection. During iteration `Map` gives you tuple with key and value in it. First attempt to use `map` function can look like this:

<pre><code>val users = Map(1 -&gt; User(1, "bob"), 2 -&gt; User(2, "john"))
val userNames = users map (_._2.userName)
</pre></code>

`_2` is field on tuple that returns it's second value. As you can see it's a little bit cryptic. But you can improve it by using partial function:

<pre><code>val userNames = users map {case (id, user) =&gt; user.userName}
</pre></code>

### flatMap

`flatMap` is surprisingly very similar to `map`. It also converts one collection into another. The difference is that it will also flatten all elements of the resulting collection. So if your function returns some kind of container like `List` or `Set`, then `flatMap` will also put all it's elements in the result. Let me show you it in action:

<pre><code>val suffixes = List("ain", "o")
val words = suffixes flatMap (suffix =&gt; List("P" + suffix, "G" + suffix))

assert(words == List("Pain", "Gain", "Po", "Go"))
</pre></code>

As you can see, everything is merged in one resulting list. If you will use `map`, the result would be: `List(List(Pain, Gain), List(Po, Go))`. Here is another, more practical, example:

	def downloadAllImages(url: String): List[File] = //....

    val urls = List("http://www.google.com", "http://www.example.com")
	val images: List[File] = urls flatMap downloadAllImages

Now that you already know `map` and `filter` functions, you probably already have some ideas, how `formatUsers` can be simplified. We can try to partly rewrite it using `map` and `filter`:

	val safeUserNames: List[String] =
		users map (_.userName) filter (_ != "test")

But what if I have `AdminUser` and `NormalUser` like in previous examples, but this time I want to show only normal users. In order to archive this in one `flatMap` we can use partial function mentioned before. I also need to tell `flatMap` somehow, that I don't want some of the items from the original collection - namely admin users. We can achieve this with very useful class - `Option`. It's also container which can hold either a single value (in this case it has type `Some[T]`) or nothing at all (in this case it has type `None`). As you can guess `flatMap` will just ignore all `None` values because they do not contain anything in them:

<pre><code>val users = List(AdminUser("bob"), NormalUser("test"), NormalUser("john"))
val userNames = users flatMap {
    case NormalUser(userName) if userName != "test" =&gt; Some(userName)
    case _ =&gt; None
}

assert(userNames == List("john"))
</pre></code>

### foldLeft

`foldLeft` is just combining all values of the collection into one resulting value. For example you can sum all numbers in the collection with initial value 0:

    val nums = List(1, 2, 3, 4, 5)
    val sum = nums.foldLeft(0)(_ + _)

`foldLeft` has 2 argument lists - first is the initial value and the second is your function that tells `foldLeft` how you want to combine two items together. Following imperative code demonstrates how `foldLeft` actually works, so it will produce the same results:

<pre><code>var sum = 0

for (i &lt;- 0 until nums.size) {
    sum = sum + nums(i)
}
</pre></code>

`foldLeft` can be very useful, especially if you want to pass some value through the whole iteration process. Let's continue our implementation of `formatUsers` and combine all user names in nice comma separated string:

	userNames.tail.foldLeft(userNames.head)(_ + ", " + _)

As you can see used 2 new small functions: `head` that returns the first element of the collection and `tail` - it returns collection of the same type with all element except the first one. But this kind of operations can be simplified, so I encourage you to read further.

### foreach

`foreach` just iterates through the whole collection and execute your function on each of the elements. The return type of your function and `foreach` is `Unit`. This means, that it does not returns anything of interest and even if you return something - it would be just ignored. This function is useful for making side effects. For example I can print all elements of the collection like this:

	users foreach println

It's behavior is very similar to Java's own for each loop. Here is equivalent in Java: 

<pre><code>// Java code
for (User user: users) {
	println(user)
}
</pre></code>

### mkString

And the last, but not least - `mkString`. It's small and useful function that just joins all elements of the collection in one string. You can also provide separator that would be used between elements. Let's continue with our new `formatUsers` implementation and format user names:

	userNames mkString ", "

`mkString` is very simple function and sometimes people wonder whether it can be useful for some more complicated tasks. Even in our simple case - we originally don't have list of user names, but we have the list of the users. `mkString` is not enough for this. But we can use already familiar functions to achieve our goal - print nice list of user names by having only list of users:

	users map (_.userName) mkString ", "

This function composition is very nice way to work with the collections.

### Improving `formatUsers`

Now that you already know all of the essential tools that collection framework provides us, it should be straightforward to write a better implementation of the `formatUsers` method. Here is the whole implementation:

    def formatUsers(users: List[User]): String = 
        users map (_.userName) filter (_ != "test") mkString ", "

The first and most important thing to notice - it's correct. I had not enough room to make the same error. I also was able to extract the most essential parts of the algorithm and put it in front of you. Boilerplate and language ceremony is also reduced dramatically.

Another important aspect of this implementation is how you reading and writing it. Now you don't think in terms of iterations and variable mutation. You are concentrating yourself on the set of collection transformations. Each transformation/step produces some correct and meaningful result. In my example I have three distinct steps: map, filter and make string. Each of them produces something that is valid and self contained. 

In imperative version I had only 2 meaningful states: at the beginning and at the end of the method. Everything in between is broken/incomplete state, that can't be used for anything else. When I'm trying to understand it, I need to keep the whole algorithm in head in order to understand how it works and what results it can produce. In contrast, in my function version each step is self contained and can be analyzed separately.

In my new implementation I can, without any code modifications, extract `users map (_.userName)`, `userNames filter (_ != "test")` or `safeUserNames mkString ", "` to some methods and they can potentially be reused by other code. So my ability to reuse this code is also increased greatly.

### for comprehension

`map`, `flatMap`, `filter` and `foreach` are so popular, that Scala even has special syntax for them and you already saw it. `for` is not a `for` loop that you know from languages like Java and C. It's much more powerful and allows following three things: iterate collections, filter them and assign variables. Here is an example:

<pre><code>val safeUserNames = for (user &lt;- users; userName = user.userName; if userName != "test") yield userName
</pre></code>

The same can also be written without semicolons with following syntax:

<pre><code>val safeUserNames = 
	for {
		user &lt;- users 
		userName = user.userName
		if userName != "test"
	} yield userName
</pre></code>

What's interesting about `for` is that under the hood it would be compiled to the series of `map`, `flatMap`, `filter` and `foreach` method calls. It can provide very nice syntax when you have some nested loops like this:

<pre><code>val allUserPrivileges = 
	for {
		user &lt;- users
		role &lt;- user.roles
		privilege &lt;- user.privileges
	} yield privilege
</pre></code>

### Performance considerations

Some of you may be concerned about performance, and it's valid argument. It's clear, that now in each step I'm iterating through the whole collection. In my concrete example it's not a problem because I need this anyway even in imperative version. But imperative algorithms often use `break` and `continue` in order to stop iteration when needed value is found or required amount of the result elements is reached. If you will use methods like `map` or `filter`, then collection would be always completely processed.

There are two nice ways to deal with this. The first one is *laziness*. If you will call **view** on a collection, then you will receive lazy version of it. Here is an example:

	val safeUserNames = users.view map (_.userName) filter (_ != "test")

At this point neither `map` nor `filter` are evaluated. `safeUserNames` will evaluate only when it would be explicitly asked to make it. You can use `force` method to make it. In this case you will apply `filter` and `map` on every element in the collection. `mkString` and `foldLeft` work the same way. But you don't have to make it. For example you can `take` first 5 safe user names like this:

    safeUserNames take 5

If you don't know how many element you need, but you know when to stop, then use `takeWhile`:

<pre><code>safeUserNames takeWhile (un =&gt; un(0) == 'a')</pre></code>

assuming that list is sorted, this code will return only user names that start with 'a'.

The second way to improve performance is to use parallel collections. All you need to do is to call **par** method on the collection - it will return parallel version of it. So this `map` operation will process it's elements in parallel:

    users.par map (_.userName)

### Conclusion

I encourage you to use more functional style when you are working with collections. It has a lot of advantages over imperative style. But I think the most important is that it makes you think in terms of collection transformations which makes it much easier to spot the intent of the code. This makes it easier for you to write code and for other people to read it.

I hope you will find this post interesting and inspiring. If you have any questions, suggestions or something else to tell me - just drop a comment, I would be very happy :) You feedback is highly appreciated!