## Generic OPTIONS HTTP Method Handling With akka-http

> The OPTIONS method represents a request for information about the communication options available on the request/response chain identified by the Request-URI.

<div style="text-align: right">&mdash;&nbsp;<a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.2">HTTP/1.1 Spec</a></div>

Sounds like pretty helpful HTTP method to use and implement, especially for the REST API. Unfortunately, not many people decide implement it for
their own API. I think partly we should thank for this our web frameworks and libraries. In past I tried to implement generic `OPTIONS` method
handler in many different frameworks (like [Play][1], [Unfiltered][2]
or some java frameworks), but unfortunately my experience was very frustrating to say the least (I really don't want to write this logic manually for every single resource in my API). In most cases you ether required to use reflection
or speculatively try all methods against routing black box in order to find out which HTTP methods are handled and which are not.

[akka-http][3] is the first HTTP library in my experience that
provides right set of abstractions for this problem. I was able to implement `Options` handler with it in very natural way and in just few lines
of code, so without further ado let me show you how it works.

<!-- more -->

### Rejections

I really like the concept of rejections in [akka-http][3]. Just imagine following route definition:

    val route: Route =
      pathPrefix("orders") {
        pathEnd {
          get {
            complete("Order 1, Order 2")
          } ~
          post {
            complete("Order saved")
          }
        } ~
        path(LongNumber) { id =>
          put {
            complete(s"Order $id")
          }
        }
      }

When, for instance, `PUT /orders` request arrives, library will try to match it and go through `get` and `post` directives of `"orders"` path.
Both of them will reject the request, because user tried to `put`. The interesting thing is that along the way akka-http will collect all these
rejections and give them to the `RejectionHandler` when it will realise, that no handlers are defined for this request.

By providing implicit instance of our own `RejectionHandler` we can define generic responce for `OPTIONS` method on any path and also define behaviour
in case requested HTTP method has no route defined:

    implicit def rejectionHandler =
      RejectionHandler.newBuilder().handleAll[MethodRejection] { rejections =>
        val methods = rejections map (_.supported)
        lazy val names = methods map (_.name) mkString ", "

        respondWithHeader(Allow(methods)) {
          options {
            complete(s"Supported methods : $names.")
          } ~
          complete(MethodNotAllowed, s"HTTP method not allowed, supported methods: $names!")
        }
      }
      .result()

According to the spec, you need to at least return `Allow` header with the list of supported HTTP methods. Additionally we also list supported HTTP methods in the
response body.

Now let's try it out with `curl`:

    $ curl -X OPTIONS -v http://localhost:8080/orders
    < HTTP/1.1 200 OK
    < Allow: GET, POST
    < Server: akka-http/2.3.11
    < Date: Sun, 21 Jun 2015 14:59:49 GMT
    < Content-Type: text/plain; charset=UTF-8
    < Content-Length: 30
    <
    Supported methods : GET, POST.

we also get similar result for `/orders/{id}`:

    $ curl -X OPTIONS -v http://localhost:8080/orders/1234
    < HTTP/1.1 200 OK
    < Allow: PUT
    < Server: akka-http/2.3.11
    < Date: Sun, 21 Jun 2015 15:01:25 GMT
    < Content-Type: text/plain; charset=UTF-8
    < Content-Length: 24
    <
    Supported methods : PUT.

If you ask for non-supported method like `PATCH`, you will get `405 Method Not Allowed`, as expected:

    $ curl -X PATCH -v http://localhost:8080/orders
    < HTTP/1.1 405 Method Not Allowed
    < Allow: GET, POST
    < Server: akka-http/2.3.11
    < Date: Sun, 21 Jun 2015 15:03:08 GMT
    < Content-Type: text/plain; charset=UTF-8
    < Content-Length: 54
    <
    HTTP method not allowed, supported methods: GET, POST!

I hope you enjoyed. I encourage you implement `OPTIONS` HTTP method support in your own API in order to provide better REST API
for people to consume (now there is no excuse not to).

For your convenience I also created this gist with the all of the source code:

<script src="https://gist.github.com/OlegIlyenko/c4c7199e6eba3d1dff37.js"></script>

[1]: https://www.playframework.com/
[2]: http://unfiltered.databinder.net/Unfiltered.html
[3]: http://doc.akka.io/docs/akka-stream-and-http-experimental/current/scala.html