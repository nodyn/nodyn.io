# Nodyn Roadmap

Since we switched strategies and have begun focusing almost exclusively on
the `process.binding` integration that the Node.js Javascript depends on, we
have made great strides and major headway towards full API compatibility. This
approach was [discussed briefly](/posts/welcome-to-nodyn) by Bob McWhirter in
our inaugural post. The details of this change are worthy of more explanation,
but I will save that for another day, and just examine what this change means
for the overall plans through the rest of this year.

<!-- more -->

## Project Vision

While we are aware of other efforts such as Avatar.js, our goal is to become
the defacto Node.js runtime on the JVM. As part of this effort we intend to
accomplish the following.

* Provide full Node.js API compatibility
* Integrate with existing Java technologies and publishing these technologies
  as NPM modules
* Work with NPM maintainers to determine the best way to include Java-based
  NPM modules in NPM repositories
* Integrate with existing Java servers such as [Vert.x](http://vertx.io) and 
  [Wildfly](http://wildfly.org/)
* Ensure that the user workflow can be similar if not identical to native
  Node.js development


## Project Timeline

At the moment, we have not nailed down any specific dates other than a goal
of having an initial release by the end of 2014. The current known GitHub issues
are now organized into two [milestones on GitHub](https://github.com/nodyn/nodyn/milestones).

Milestone one, or M1, is focused on fleshing out the Node.js API and ensuring
that we provide comprehensive Java and/or Javascript binding coverage. Our
overall coverage will be measured by three high-level goals.

* Generate http://nodyn.io using harp npm module via nodyn
* Create example apps using the connect.js framework
* Create example apps using the express.js framework

Accomplishing these three simple goals will ensure that we have broad coverage
of the Node.js API for both networking and file system operations. While we know
this will not ensure complete coverage, it will take us a long way towards that
goal.

For our second milestone, M2, we anticipate full API coverage. Here we will
expand and build on the functionality delivered in M1. This milestone will also
include a reorganization of the code base so that we take better advantage of
the existing joyent/node.git repository structure - eliminating the need to
copy and paste the Node.js Javascript files from Joyent as they change.
Some specific goals are:

  * In addition to running basic express.js applications, nodyn should be
    capable of hosting the [Ghost blogging platform](https://github.com/tryghost/Ghost).  
  * Expand the universe of framework examples:
    * https://github.com/flatiron/flatiron
    * https://github.com/koajs/koa
    * https://github.com/totaljs/framework/
    * https://github.com/mcavage/node-restify
    * https://github.com/socketstream/socketstream
    * https://github.com/balderdashy/sails/
  * Build nodyn in the context of the existing joyent/node.git repository structure
  * Publish NPM modules for Vert.x event bus usage
  * Publish Vert.x modules so that nodyn can run inside of Vert.x

## User Feedback

Is there anything you would like to see on our roadmap that is not here? Are there
other NPM modules that we should ensure work on Nodyn?  If you are a Java
developer, what Java technologies would you like to integrate with? We are here
to help. Leave your comments here, or join us in 
[#nodyn on freenode](http://webchat.freenode.net/?channels=nodyn).

