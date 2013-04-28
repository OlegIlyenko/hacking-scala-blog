## Progress Monitoring For Streams 

In this blog post I would like to show, how you can implement simple monitoring capabilities for standard Java's `InutStream` and `OutputStream`. By "monitoring capabilities" I mean ability to find out information like Speed, Estimated Time, Transfered Size, etc. Recently I implemented this for our internal deployment tool (simple Swing-based application written in Scala). After about 30 minutes of googling without much success, I actually tried to think about this and found out how simple it actually is. So I decided to share this with you. Additionally I also want to give you small demonstration of scala-swing which I'm enjoying a lot.

After reading this article you will find out how you can write application like this one:

![Downloader](http://github.com/OlegIlyenko/my-blog-images/raw/master/posterous/downloader.png)

<!-- more -->

### Creating Stream Filter

In order to get all metrics shown above, we need to know the full size of the file/stream and currently transfered amount of bytes. The full size is easy to find out from outside of the `Stream`, especially if you are working with files, but we need to monitor `Stream` in order to find out how much bytes are transfered already. Here is an example of such `InputStream`, that reports this with specified threshold:

<script src="https://gist.github.com/OlegIlyenko/2387873.js"></script>

You have probably noticed `ProgressListener`. It's function `Long =&gt; Unit` which we will discuss next.

### ProgressListener class

In order to show transferred number of bytes, we need to remember it somehow. `ProgressListener` function makes just this + calculates another metrics:

<script src="https://gist.github.com/OlegIlyenko/2387880.js"></script>

What's interesting about this, is that I do cont define function with function literal like: `(bytes: Long) =&gt; Unit`. Instead I'm creating a new class that extends `Function1[Long, Unit]` - it's allowed in Scala and sometimes can be very useful. Actually many Scala's collection classes also making this, so you can get i-th element of the list like this: `myList(i)`. In my case, it allowed me to have some internal state (`transfered` and `startTime` fields) and to parametrize function with some arguments (`availableBytes` and `tracker`) during it's creation. It's also possible to use function literal for this and still archive the same results - you just need to use currying and closures in this case. But I find, that it's more clear and readable to define `ProgressListener` as class in this particular case.

You can tweak this algorithm and make it more accurate. For example you can calculate `bps` depending on the bytes, that were transferred during last several seconds. Still I think, that this algorithm does it's job for many use cases.

### Progress class

Now let's take a look at `Progress` case class. It's purpose is to hold different metrics and format it when needed (formatting can be probably done somewhere else, but I decided to put it here just to keep it simple).

<script src="https://gist.github.com/OlegIlyenko/2387889.js"></script>

Here you can also find nice utility functions to format size and time. 

### User Interface With Scala-Swing

Now let's create simple application that downloads files and shows the progress:

<script src="https://gist.github.com/OlegIlyenko/2387901.js"></script>

This application throws all downloaded bytes away, so it's pretty useless, but I hope that it will demonstrate you scala-swing basics.

### Conclusion

Hopefully this will also give you some impressions on how easy it is to integrate with Java standard library from Scala. For your convenience I also [created gist with all these classes in it](https://gist.github.com/2388377), so you can just copy/paste it in your own projects and start playing with it.