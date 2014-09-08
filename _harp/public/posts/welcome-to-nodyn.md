
# Who, What, How?

[Nodyn][nodyn] is [Red Hat][redhat]'s implementation of [Node.js][nodejs] on top of the Java Virtual Machine.
Currently Nodyn is tracking the 0.12 release of Node.js. The project is lead by
[Lance Ball][lance] with [Bob McWhirter][bob] pitching in.

Lance and Bob are part of the team responsible for [TorqueBox][torquebox], and are familiar
with taking languages and frameworks that don't run on the Java Virtual Machine
already, and making them run on the JVM.

<!-- more -->

The foundations of Nodyn include [DynJS][dynjs], [Netty][netty], and [Vertx][vertx].  Initially Nodyn was
conceived as a clean-room implemention of the public-facing APIs exposed by
Node.js, but this route turned out to be problematic for a few reasons:

  * The public-facing APIs were not always very well-documented.
  * Various NPM modules would use un-documented APIs.

Re-evaluating the Node.js codebase, and spending some weekends reading the
Javascript and C++ code with a cold glass of beer, the team figured out how
to re-use the existing actual Javascript code from Node.js, and simply replace
the C++ bits.  In a fortunate twist, the C++ bits are replaced with some Java
and some Javascript.  Nodyn, arguably, is implemented more in Javascript than
Node.js is.  This is possible due to the Java integration facilities of DynJS.

Given this new strategy, the public-facing APIs provided by Node.js, as
implemented in the various .js files in their `lib/` directory, are also
exactly the same in Nodyn.  Also, it turns out that the API surface defined 
by the various `process.binding(...)` objects is significantly smaller than
the public-facing APIs, reducing the amount of work required by the Nodyn team.

For instance, in a clean-room implementation, all of `http.js` would need to be
implemented, from scratch.  By building upon actual Node.js sources, the
`http.js` module requires only an `HTTPParser` implementation, and builds itself
on top of the `net.js` module.

Some of the Node.js APIs, particular `child_process` and much of `fs` rely upon
facilities not present in the JVM. Nodyn leverages the jnr-posix library (the
same library used by JRuby) to provide capabilities for working with integer-based
file-descriptors, and sending these file-descriptors to forked child processes.

# But WHY?

While we've answered the *Who*, the *What* and the *How*, we should still address
the *Why* of Nodyn.  Node.js is, on the outside, a single-threaded framework. Java
can handle threads without a problem.  But Java can still perform better with reactive,
asynchronous frameworks.  Just because Java *can* use threads like magic, each thread 
does still have a certain amount of overhead (stack space, context-switching, etc),
so if we can avoid using them, that's great!  

Additionally, just as Node.js likes to say "Node.js is multi-threaded everywhere,
except *your* code", Nodyn follows the same route. There are lots of threads inside
of Nodyn, but only a single thread ever executes your code.

That *is* great, but it doesn't doesn't answer the question, does it?

The JVM has a *lot* going for it. There's a ton of libraries existing for it. There's
tons of enterprises that are already familiar with running JVM-based stacks in their
data-centers.  And these guys would like to board the hype train of Node.js.  Nodyn allows
that.

Additionally, while it's convenient to run multiple Node.js processes to use many processors,
Nodyn will soon provide the ability to run multiple Node.js "processes" within a single
JVM process.  Nodyn has been constructed, from the ground up, without any shared global 
state or variables. It is super-simple to embed many completely separate Node.js runtimes
inside the JVM using Nodyn.

# What's left?

Lots!

We still need to figure out how Nodyn lives within the large Node.js community and
ecosystem. Specifically, it'd be great to be able to have JVM-flavored NPM modules,
possible as alternative to C++-based modules.

The Ruby world saw this transition, with the support of `-java` gems being added
to the RubyGems packaging system.  A single named gem may have both a C-based version
or a Java-based version.

Additionally, there are still parts of the Node.js core APIs that we haven't necessarily
implemented.  [Feel free to pitch in][github]!


[nodyn]: http://nodyn.io/
[redhat]: http://redhat.com/
[dynjs]: http://dynjs.org/
[netty]: http://netty.io/
[vertx]: http://vertx.io/
[nodejs]: http://nodejs.org/
[torquebox]: http://torquebox.org/
[lance]: http://twitter.com/lanceball
[bob]: http://twitter.com/bobmcwhirter
[github]: http://github.com/nodyn/nodyn

