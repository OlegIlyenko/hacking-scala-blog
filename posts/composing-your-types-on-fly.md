## Composing Your Types on Fly

Let me show you class from one of my projects:

    class FileTransferHandler extends TransferHandler with Publisher {
	    // ...
	}

`TransferHandler` is swing class that generally handles copy/paste and D&amp;D stuff. `Publisher` is trait that comes from scala-swing. So I generally composing here 2 independent things and making then work together.

As next step I define method, that requires it's argument to be both - `TransferHandler` and `Publisher`. In my codebase I have only one type that extends both of them, and it's `FileTransferHandler`. So I can use it to define my method, right?

    def register(handler: FileTransferHandler) {
        // ...
    }

The problem here is that the body of the `register` method does not care whether `handler` is `FileTransferHandler` or `ImageTransferHandler` or even `StringTransferHandler`. It only requires it to be `TransferHandler` and `Publisher`. And it's exactly what Scala allows you to express using `with` keyword! So now we can write improved version of `register` method:

    def register(handler: TransferHandler with Publisher) {
        // ...
    }
		
So you don't actually need any concrete class or trait to express such constraint.

<!-- more -->

### Combining with structural types

Scala has very nice feature called **structural types**. Let me describe it to you in example. Recently I was writing some Scala tests for Java classes. It appears, that a lot of classes had methods like `init()` and `destroy()`. Not all classes had both of them - some need only to be initialized, but don't require explicit destruction and vice versa. Also there are no interfaces for this - they all were normally instantiated using *spring* with something like 

    &lt;bean class="..." init-method="init" destroy-method="destroy" /&gt;
    

Structural typing helped me a lot to deal with such classes. I was able to define methods like this one:

	def cleanup(list: List[{def destroy()}]) = 
		list foreach (_.destroy())

Next I would like to define method, that will initialize and destroy objects automatically for me, because I can just forget about one of these steps. So here my first attempt:

    def safe[T &lt;: {def init(); def destroy()}, R](t: T)(body: T =&gt; R) =     
        try {
            t.init()
            body(t)
        } finally {
            t.destroy()
        }

    safe(new PaymentService) { service =&gt;
        service.makePayment()
    }

So it works! But can we implement it better? The problem here - is that we repeating ourselves. I already used signature of `destroy` method in the `cleanup` method, and I don't want to repeat myself again. It would be nice to define some traits/interfaces like `Init` or `Destroy`, but I don't want to change all existing classes in order to implement them. Actually I even don't need any concrete trait for this - I just need to define some kind of alias for `{def init()}` and `{def destroy()}`. 

**Type aliases** serve exactly this purpose. Let's define them:

	type Init = {def init()}
	type Destroy = {def destroy()}

Nice, now I have pretty names for the structural types. `safe` method needs both of them and, as you probably already guessed, I can combine them using `with`:

    def safe[T &lt;: Init with Destroy, R](t: T)(body: T =&gt; R) = // ...

If you wish, you can also define type alias for the combination:

    type InitDestroy = Init with Destroy

	def safe[T &lt;: InitDestroy, R](t: T)(body: T =&gt; R) = // ...

Hope you enjoyed.