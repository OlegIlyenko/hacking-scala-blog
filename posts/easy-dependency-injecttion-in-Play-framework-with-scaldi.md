## Easy Dependency Injection in Play Framework with Scaldi

In this post I would like to make small introduction to Scaldi and show you how easy it is to use it together with Play. [Scaldi](http://scaldi.github.io/scaldi/Scaldi.html) is dependency injection library for Scala. It's very lightweight (without any dependencies) and provides nice Scala DSL for binding dependencies and injecting them. It's more dynamic approach to do dependency injection in comparison to the cake pattern, because not everything is checked at the compile time. This can be seen as an disadvantage, but I personally believe, that there is place for this approach. It can be vary useful in many circumstances and many people can find it more natural (and easy), especially if they are coming from Java background.

There are only 3 most important traits that you need to know, in order to make dependency injection with Scaldi:

* **Injector** - it's a container for the bindings, that you have defined in the module.
* **Module** - gives you nice syntax to create bindings with `bind` and `binding`. `Module` also extends `Injector` trait and implicit `Injector` instance always available when you are defining your bindings
* **Injectable** - the only responsibility of this trait is to provide you with `inject` function (so it just provides nice syntax for injecting dependencies). It's important to understand, that it's the only the purpose of it. So it completely stateless and knows nothing about actual bindings you have defined in the module. In order to actually find and inject dependencies, `inject` function always takes an implicit parameter of type `Injector`

<!-- more -->

### Defining Managed Classes

Let's take a small example. Suppose you have have `MessageService` trait like this:

	trait MessageService {
	  def getGreetMessage(name: String): String
	}

Now let's define one implementation of it, where we will inject greeting string:

	class OfficialMessageService(implicit inj: Injector) extends MessageService with Injectable {
	  val officialGreeting = inject [String] (identified by "greeting.official")
	
	  def getGreetMessage(name: String) = s"$officialGreeting, $name!"
	}

You have probably noticed 2 things, that are required for injection to work:

* We extended `Injectable` in order to make `inject` function available 
* We declared implicit parameter of type `Injector` in order to provide bindings to the inject function.

You are not required to always extend `Injectable` in order to use `inject` - you can just import it. This will  work as good as in previous example:

	import scaldi.Injectable._

	class SomeService(implicit inj: Injector) extends MessageService {
	  val dep = inject [SomeOtherService]
	}

### Creating a Module

Now that we've created a managed class, we need to add it to the `Module`:

	class UserModule extends Module {
	  bind [MessageService] to new OfficialMessageService
	
	  binding identifiedBy "greeting.official" to "Welcome"
	}

In the module context you are able to instantiate `OfficialMessageService` because it always have implicit `Injector` available in a scope, as I mentioned earlier. I also defined another binding with identifier  `greeting.official` because `OfficialMessageService` needs it and will try to inject it at some point.

### Integration with Play App

Now we will try to integrate our new module in the Play app. The first thing you need to do is to add dependency on **scaldi-play** in the project file (`build.sbt`). Something like this:

	name := "scaldi-play-example"

	version := "1.0-SNAPSHOT"

	libraryDependencies ++= Seq(
	  "com.github.scaldi" %% "scaldi-play" % "0.2.2"
	)

	play.Project.playScalaSettings

Now you are able to use Scaldi in the project. Play application normally has it's initialization logic in the `Global` object, so we need to add `ScaldiSupport` in it:

	object Global extends GlobalSettings with ScaldiSupport {
	  def applicationModule = new UserModule
	}

Nice! Your Play application is now uses Scaldi for the dependency injection, but unfortunately it doesn't do anything, so let's fix it. Let's create a simple index page (`app/views/index.scala.html`) that will show greeting message to the user:

	@(message: String)
	
	@main("Test Page") {
	    &lt;h1&gt;@message&lt;/h1&gt;
	}

Now we need to create a controller for it. But instead of creating a singleton object, let's make a managed class (simmilat to the `OfficialMessageService` class):

	class Application(implicit inj: Injector) extends Controller with Injectable {
	  val messageService = inject [MessageService]
	
	  def index = Action {
	    Ok(views.html.index(messageService.getGreetMessage("Test User")))
	  }
	}

As you can see, `MessageService` is injected and its used to produce nice greet message. 

As you probably already aware, Play only works with singleton controllers, which means, that it requires `Application` controller to be an `object` instead of `class`. Thankfully Play 2.1 introduced new feature, which makes integration with dependency injection framework like Scaldi possible. In order to use this feature, you need to prefix managed controllers with `@` in the `conf/routes` file. So let's make it:

	GET  /                 @controllers.Application.index

The one last thing that remains is to add controller to the new module:

	class WebModule extends Module {
	  binding to new Application
	}

and to compose `WebModule` with `UserModule` in the `Global` object:

	object Global extends GlobalSettings with ScaldiSupport {
	  def applicationModule = new WebModule :: new UserModule
	}

At this point you should be able to run Play app and view the index page, that will show you: `Welcome, Test User!`.

### Injecting Play Configuration

**scaldi-play** provides integration with Play configuration (`conf/application.conf`) out of the box. So you can, for example, define `greeting.official` property there:

	greeting.official = Welcome

and then just remove one extra binding for it from the `UserModule`:

	class UserModule extends Module {
	  bind [MessageService] to new OfficialMessageService
	}

It will continue to work as before. You can also inject other primitive types like `Int` or `Boolean` and not only `String`. If you would like to use `configuration` object directly, then you need inject it like this:

	val config = inject [play.api.Configuration]

### Distinguishing Between Modes

Suppose we have another implementation of `MessageService` that we want to use when Play app is in the `dev` or `test` mode:

	class SimpleMessageService extends MessageService {
	  def getGreetMessage(name: String) = s"Hi, $name"
	}

With Scaldi it's pretty easy to make. You just need to define `UserModule` like this:

	class UserModule extends Module {
	  bind [MessageService] when (inDevMode or inTestMode) to new SimpleMessageService
	  bind [MessageService] when inProdMode to new OfficialMessageService
	}

`inDevMode`, `inTestMode` and `inProdMode` are just functions that produce `Condition` objects. `Condition`s are used by Scaldi to decide, whether binding is available for injection.

I hope you liked this small introduction to [scaldi](http://scaldi.github.io/scaldi/Scaldi.html) and [scaldi-play](https://github.com/scaldi/scaldi-play). You can find example application, that I described here, in this github repo:

[https://github.com/scaldi/scaldi-play-example](https://github.com/scaldi/scaldi-play-example)

Please feel free to fork and hack it :)