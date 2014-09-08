
# Nodyn and Vert.x?

[Yesterday], we pointed out that Nodyn uses [Vert.x][vertx] under the covers to 
power our event-loop (replacing the [libuv][libuv] event-loop in Node.js).  Additionally,
Nodyn has an NPM module that can make bits of Vert.x directly usable
by your Node.js applications.

<!-- more -->

One of the key components of Vert.x, which does not exist in the Node.js
APIs, is the [eventbus][eventbus].  The eventbus allows you to register handlers
for specific addresses, and have them receive messages and perform work.

Optionally, these handlers can *reply* back to the original sender.

All of this occurs asynchronously, of course, and allows you to create
complex topologies of components that communicate purely through
message-passing.

# The `vertx2-core` NPM module

Currently in [GitHub][vertx2-core], the `vertx2-core` NPM module takes the internal
Vert.x, and makes it available to your application.

    var vertx = require('vertx2-core')

From there, you can access the `eventbus` and interact with it
as both a producer and a consumer.

    var registration = vertx.eventbus.register( 'some.address', function(msg) {
      // handle incoming messages here
    } );

    vertx.eventbus.send( 'some.address', ... );

While the native Vert.x API is a "fluent" API, where `register()` returns the
eventbus back to you, for Node.js semantics, the `register()` method returns
a `HandlerRegistration` object.  This object provides access to the typical
`ref()` and `unref()` methods, to control if registration keeps the event-loop
alive or not.

Additionally, the `HandlerRegistration` object provides a handy `unregister()`
method to easily unregister (and therefor `unref()`) the handler.  If at a later
point in time you wish to re-register, you can use the same object and simply
call `register()`.

The reason we need the `ref()` and `unref()` functionality is due to the fact
that it's possible to write a script that does nothing but sets up a an
eventbus listener. Without the ref-counting, the script would complete after
registering the listener, the event-loop would think it has nothing left to do,
and the process would exit.  That's not good. With the `ref()` and `unref()`
methods, you can explicitly control if the eventbus handler counts towards
the active task-list, keeping the event-loop alive. This becomes more important
when you have a cluster of Vert.x processes running cooperatively and possibly
have scripts that do nothing other than wait for the event-bus to deliver
some messages.

# Interoperability

Vert.x itself is both polyglot and a standalone platform which can cluster many
nodes of itself, allowing the event-bus to span many processes.  With a complex
mesh of eventbus listeners distributed across possibly many physical machines,
the load can be distributed and your application can scale as needed.

The `vertx2-core` NPM module, along with the eventbus binding, provides interoperability
with other Vert.x nodes and handlers.  These other nodes could be running inside of
Nodyn, or simpy as standalone Vert.x instances.  If running as standalone Vert.x instances,
the handlers could be written in any language Vert.x supports, including Clojure, Groovy
and Java.

# Everyone Loves Tacos

And now, some example code, demonstrating a Taco Server, which allows a person to
place an order for some amount of tacos through the web, and receive a response
as to how long he or she will have to wait to have their glorious tacos delivered.

The basic architecture is this:

  * The taco _kitchen_ sets up a Vert.x eventbus listener on the address of `kitchen`.
    This listener waits for messages that contain an `amount` field, and calculates
    how long it will take to produce that number of tacos, and then replies with
    its answer. It also prints to the console when it receives a message.
  * A regular Node HTTP server is created on port 9000.
  * The Node HTTP request-handler sends a message on the Vert.x eventbus to the
    `kitchen` address, and provides another handler to accept the asynchronous
    reply from the kitchen.
  * When the reply is received from the kitchen, the request-handler uses that
    information to provide a response to the originating web client, on port 9000.


    var http = require('http');
    var vertx = require('vertx2-core');

    var registration = vertx.eventbus.register( 'kitchen', function(message) {
      console.log( 'KITCHEN: Someone ordered ' + message.body.amount + ' tacos' );
      message.reply( { wait_time: message.body.amount * 0.2 } );
    });

    var server = http.createServer( function(request, response) {
      var url = request.url;
      var parts = url.split( '/' );
      var amount = parts[1];
    
      vertx.eventbus.send( 'kitchen', { amount: amount } , function(message) {
        response.write( amount + ' tacos will be ready in ' + message.body.wait_time + ' minutes' );
        response.end();
      } );
    } );

    server.listen( 9000, function() {
      console.log( "Server listening on port 9000" );
    } );

By launching this app via `nodyn taco_kitchen.js`, you will ultimately see on the console:

    Server listening on port 9000

Using our favorite web-browser, we surf to URLs such as this, to order 102 tacos (we _love_ tacos):

    http://localhost:9000/102

In our browser, we see:

    102 tacos will be ready in 20.400000000000002 minutes

Additionally in our server-side console, we see:

    KITCHEN: Someone ordered 102 tacos

# Wrapping Up

Vert.x is one of the reasons running Node.js on the JVM using Nodyn makes sense. The Vert.x
eventbus allows for the creation of components scattered across a network of machines, and 
have them all serve a common application.  Since the eventbus is asynchronous, it perfectly
matches the Node.js APIs and semantics for responding to things such as web requests.

In Nodyn, we still have some practical work to do in order to faciliate clustering of the
Vert.x inside of a Nodyn process with an existing out-of-process Vert.x grid, but that's 
coming down the pipeline.

[Yesterday]: http://nodyn.io/posts/welcome-to-nodyn
[libuv]: https://github.com/joyent/libuv
[vertx]: http://vertx.io/
[eventbus]: http://vertx.io/core_manual_js.html#the-event-bus
[vertx2-core]: https://github.com/nodyn/vertx2-core/

