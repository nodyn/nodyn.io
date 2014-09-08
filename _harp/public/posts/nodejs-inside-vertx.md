
# Strike that. Reverse it. Node.js (Nodyn) inside Vert.x

[Yesterday] we talked about how to gain access to [Vert.x][vertx] functionality
within your Node.js scripts run by Nodyn.  Today we're going to invert
that idea, and show you how you can take existing Node.js workloads,
run them inside of Vert.x (including on a cluster) and integrate
them with existing *pure* Vert.x components you might already have.

<!-- more -->

# The `mod-nodyn` Vert.x module

Re-usable Vert.x components are packaged as *modules*, not unlike
NPM modules. The `mod-nodyn` [1] module has the ability to crank up a
Nodyn inside of Vert.x, using the existing Vert.x event-loop so that
everything plays friendly together.


# Tacos as a Service

Let's expand on our previous example, and deploy it as a cluster of
two Vert.x processes. Yesterday, we put all of it in the same
file, and did not really take advantage of the capabilities the
Vert.x *[eventbus]* really has to offer.  Today, we will.

The moving parts are:

  * `mod-nodyn` to run our Node.js workload.
  * `tacos-web.js` our Node.js-based webserver.
  * `tacos-kitchen.js` a Vert.x javascript component which determines how long it takes to make tacos.
  * `vertx` to run it all.


## The Node.js application

Our `tacos-web.js` Node.js-based web-service looks like the one yesterday,
except that it no longer includes the bit that calculates the amount of
time required to produce the requested number of tacos.  It still uses the
`vertx2-core` NPM module to be able to communicate over the Vert.x 
eventbus (see [yesterday's][yesterday] post).

    var http = require('http');
    var vertx = require('vertx2-core');
    
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

## The Vert.x Verticle

The `tacos-kitchen.js` file is a Vert.x *verticle*. It uses the normal 
Vert.x Javascript APIs, and is *not* a Node.js component.

This component simply registers an eventbus handler on the `kitchen` address and
waits for messages.  Notice the `vertx2-core` API for the eventbus differs from the
"native" `vertx/event_bus` API.  The `vertx2-core` APIs are still very fluid and
feedback from the community would be appreciated.

    var eventBus = require('vertx/event_bus');

    eventBus.registerHandler( "kitchen", function(message, replier) {
      java.lang.System.err.println( 'KITCHEN: Someone ordered ' + message.amount + ' tacos' );
      replier( { wait_time: message.amount * 0.2 } );
    } );
    
    java.lang.System.err.println( "loaded the kitchen!" );

# Running it all

## Clustering

One of the main benefits of Vert.x is the creation of clusters.  A Vert.x cluster 
is distinct from a Node.js cluster. In a Vert.x cluster,
every component still runs independently, but the eventbus spans all members
of the cluster, and you can communicate between them without being aware of
the actual physical topology.  To enable clustering, we just add the `-cluster`
flag when launching Vert.x processes.

In the easy case, Vert.x uses *multicast* to discover peers and wire up 
the distributed mesh of the eventbus.  Ultimately this is performed using
[Hazelcast][hazelcast], which can be configured for any environment you need to run in.

## Run `tacos-web.js` as a Node.js workload inside of Vert.x

To run `tacos-web.js` as a Node.js workload, we actually ask Vert.x to
run the `mod-nodyn` Vert.x module, and provide configuration so it knows
where to find the Node.js-based application.  Since we are creating
Tacos-as-a-Service, let's call this configuration file `taas.conf`. 
It simply contains JSON, with a single key, being `main`.  This is passed
to Nodyn's Node.js implementation just as a filename on the commandline
would be passed.

    {
      "main": "./tacos-web.js"
    }

Now, to launch the Node.js-based web portion of our app by asking Vert.x to run
our `mod-nodyn` module while passing in our `taas.conf` configuration and the
`-cluster` flag.

    $ vertx runmod io.nodyn.vertx~mod-nodyn~1.0.0-SNAPSHOT -conf taas.conf -cluster

I've enabled more verbose logging of the clustering subsystem, so we can watch what
happens.  The output of this command will ultimately appear similar to:

    Starting clustering...
    No cluster-host specified so using address 172.16.49.1
    null [dev] [3.2.3] Interfaces is enabled, trying to pick one address matching to one of: [10.0.1.*]
    null [dev] [3.2.3] Prefer IPv4 stack is true.
    null [dev] [3.2.3] Picked Address[10.0.1.25]:5701, using socket ServerSocket[addr=/0:0:0:0:0:0:0:0,localport=5701], bind any local is true
    [10.0.1.25]:5701 [dev] [3.2.3] Hazelcast Community Edition 3.2.3 (20140617) starting at Address[10.0.1.25]:5701
    [10.0.1.25]:5701 [dev] [3.2.3] Copyright (C) 2008-2014 Hazelcast.com
    [10.0.1.25]:5701 [dev] [3.2.3] Creating MulticastJoiner
    [10.0.1.25]:5701 [dev] [3.2.3] Address[10.0.1.25]:5701 is STARTING
    [10.0.1.25]:5701 [dev] [3.2.3]
    
    
    Members [1] {
	    Member [10.0.1.25]:5701 this
    }
    
    [10.0.1.25]:5701 [dev] [3.2.3] Address[10.0.1.25]:5701 is STARTED
    Server listening on port 9000
    Succeeded in deploying module


We notice that initially, we have one process running, forming a cluster of one node.  That's not super exciting.

## Run `tacos-kitchen.js` as a plain Vert.x verticle

Running the `tacos-kitchen.js` component is about the same, except we don't 
have to pass any configuration in, and we use the `run` command instead of the
`runmod` command, to run the file as specified on the commandline.

    $ vertx run tacos-kitchen.js -cluster

Once again, the output should look akin to this:

    Starting clustering...
    No cluster-host specified so using address 10.10.57.51
    null [dev] [3.2.3] Interfaces is enabled, trying to pick one address matching to one of: [10.0.1.*]
    null [dev] [3.2.3] Prefer IPv4 stack is true.
    null [dev] [3.2.3] Picked Address[10.0.1.25]:5702, using socket ServerSocket[addr=/0:0:0:0:0:0:0:0,localport=5702], bind any local is true
    [10.0.1.25]:5702 [dev] [3.2.3] Hazelcast Community Edition 3.2.3 (20140617) starting at Address[10.0.1.25]:5702
    [10.0.1.25]:5702 [dev] [3.2.3] Copyright (C) 2008-2014 Hazelcast.com
    [10.0.1.25]:5702 [dev] [3.2.3] Creating MulticastJoiner
    [10.0.1.25]:5702 [dev] [3.2.3] Address[10.0.1.25]:5702 is STARTING
    [10.0.1.25]:5702 [dev] [3.2.3] Connecting to /10.0.1.25:5701, timeout: 0, bind-any: true
    [10.0.1.25]:5702 [dev] [3.2.3] 60247 accepted socket connection from /10.0.1.25:5701
    [10.0.1.25]:5702 [dev] [3.2.3]
    
    Members [2] {
	    Member [10.0.1.25]:5701
	    Member [10.0.1.25]:5702 this
    }
    
    [10.0.1.25]:5702 [dev] [3.2.3] Address[10.0.1.25]:5702 is STARTED
    loaded the kitchen!
    Succeeded in deploying verticle

Here, we see that this second Vert.x process has successfully found and clustered with
the first one we launched (`tacos-web.js`).

# Order some tacos!

Now, as before, we point our browser to

    http://localhost:9000/57

and order 57 tacos.

The browser displays

    57 tacos will be ready in 11.4 minutes

While the `tacos-kitchen.js` console displays

    KITCHEN: Someone ordered 57 tacos

# Summary

Here, we've deployed a Node.js application, which uses Vert.x eventbus functionality in one
instance of Vert.x.  We've deployed a plain ol' Vert.x component (a verticle) in another,
distinct Vert.x process.  The pair of Vert.x processes have found each other, embraced each
other, and formed a cluster.  We've communicated between them, asynchronously.

Isn't that awesome?

# Notes

  * `mod-nodyn` lives in [GitHub][mod-nodyn]. If you'd like to follow along at home,
  check it out, and build it with `mvn install`.

[yesterday]: http://nodyn.io/posts/vertx-with-nodyn
[eventbus]: http://vertx.io/core_manual_java.html#the-event-bus
[vertx]: http://vertx.io/
[mod-nodyn]: https://github.com/nodyn/mod-nodyn
[hazelcast]: http://hazelcast.org/
[strikethat]: https://www.youtube.com/watch?v=ZWJo2EZW8yU
