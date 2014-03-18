## Dependency Injection in Akka with Scaldi

As a follow-up to the post about [Scaldi Play integration](http://hacking-scala.org/post/51407241538/easy-dependency-injection-in-play-framework-with-scaldi) I would like to introduce you an integration with the Akka in this post. 


Akka integration is pretty recent addition to scaldi and interestingly enough has not much to add to the core library in order to smoothly integration with the Akka. In order to use it you need to add `scaldi-akka` to the SBT build, so let's do this first:

	libraryDependencies ++= Seq(
	  "org.scaldi" %% "scaldi-akka" % "0.3.1"
	)

[[MORE]]

You may notice, that the groupId of scaldi has changed to `org.scaldi` since version 0.3. Version 0.3 is also a recent release, which has introduced some nice features like binding lifecycle (you can find the complete list of new feature in the [changelog](https://github.com/scaldi/scaldi/blob/master/CHANGELOG.md#v03-02032014).

Now you should be ready to go. The only new thing that `scaldi-akka` adds is `AkkaInjectable`, which provides 2 additional inject methods:

* `injectActorRef` - creates a new actor with the help of `ActorRef` factory which should be implicitly available in the scope.
* `injectActorProps` - injects `Props` for the `Actor`, so that you can create new `Actor`s yourself with the help of the `ActorRef` factory.

If you still wondering what `Injectable` actually is, I can recommend you to read [one of my previous posts](http://hacking-scala.org/post/51407241538/easy-dependency-injection-in-play-framework-with-scaldi). It explains the basic concepts of the library and shows an example application that uses Scaldi for the dependency injection. 

I mentioned `ActorRef` factory above. It can be one of two things:

* `ActorContext` - it always implicitly available within an `Actor` and can be used to create a new actors in the context of current actor
* `ActorSystem`

Here is a small example of how you can use `AkkaInjectable` to inject (which actually means create in case of actors) another actor:

	class Receptionist (implicit inj: Injector) extends Actor with AkkaInjectable {
	  val userService = inject [UserService]

	  val orderProcessorProps = injectActorProps [OrderProcessor]
	  val priceCalculator = injectActorRef [PriceCalculator]

	  def receive = {
	    case PlaceOrder(userName, itemId, netAmount) =>
          val processor = context.actorOf(orderProcessorProps)
          // ...
	  }
	}

Or alternatively, if you want to create an actor somewhere else (not inside an actor), you need to provide an implicit `ActorSystem` in the scope:

	import scaldi.akka.AkkaInjectable._

	implicit val appModule: Injector = // ...

	implicit val system = inject [ActorSystem]

	val receptionist = injectActorRef [Receptionist]

So far so good. We have created some actors that are able to use `inject`. The only thing that remains now is to create a module that binds them together with other dependencies and the `ActorSysyem` itself:

	class OrderModule extends Module {
	  bind [UserService] to new SimpleUserService
	  
	  bind [ActorSystem] to ActorSystem("ScaldiExample") destroyWith (_.shutdown())

	  binding toProvider new Receptionist
	  binding toProvider new OrderProcessor
	  binding toProvider new PriceCalculator
	}

As you can see, now you also can define how binding can be destroyed with the `destroyWith` function. It is part of the binding lifecycle feature that was introduced in Scaldi version 0.3.

At this point I would like to point out how `Actor` are bound. It is important, that you bind then with `toProvider` function. It will make sure, that Scaldi always creates new instances of the `Actor` classes when you injecting them with `injectActorRef` or `injectActorProps`. These two methods actually use Akka mechanisms to configure an actor instance under-the-hood, but the actor instance creation itself is always delegated to Scaldi. During this process, Akka requires the delegate to always create new instances of an actor, so by binding `Actor`s with `toProvider` you are fulfilling the protocol, that Akka implies.

I created a [small example application](https://github.com/scaldi/scaldi-akka-example) that demonstrates an integration between Akka and Scaldi and all the things described in this post. You can also use it as a [Typesafe Activator template](https://typesafe.com/activator/template/scaldi-akka-example).

Hope you enjoyed this post. As always, any of kind of feedback is very welcome!