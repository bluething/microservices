## Implementing Microservice Communication

Technology selection should be based on the desired communication style!

### Looking for the Ideal Technology

#### What should we think?

##### Make backwards compatibility easy

Don't break the client!  
Ideally we have ability to validate that the changes we have made are backwards compatible—and have a way to get that feedback before we deploy our microservice into production.

##### Make interface explicit

This means that it is clear to a consumer of a microservice as to what functionality that microservice exposes. For an engineer it means avoid a situation where a change to a microservice causes an accidental breakage in compatibility.

##### Keep APIs technology-agnostic

This means avoiding integration technology that dictates what technology stacks we can use to implement our microservices.  
Innovation in the world of technology is very fast.

##### Make service simple for consumers

Our services must be easy to use by our consumers.

##### Hide internal implementation detail

Good system is a system that have loose coupling between their components. High coupling means high maintenance cost. Don't bound our consumer to internal implementation.  
Any technology that pushes us to expose internal representation detail should be avoided.

### Technology Choices

#### Remote Procedure Calls (RPC)

Frameworks that allow for local method calls to be invoked on a remote process. Common options include SOAP and gRPC.  
There are a number of different RPC implementations in use. Most of the technology in this space requires an explicit schema.  
In the context of RPC we have Interface Definition Language, or IDL. In SOAP, we have Web Service Definition Language (WSDL). Those schema makes it easier to generate client and server stubs for different technology stacks.

The ease of generation of client-side code is one of the main selling points of RPC!

Typically, using an RPC technology means you are buying into a serialization protocol. For example protobuf in gRPC.

Some implementations are tied to a specific networking protocol (like SOAP, which makes nominal use of HTTP), whereas others might allow you to use different types of networking protocols, which themselves can provide additional features.  
For example, TCP offers guarantees about delivery, whereas UDP doesn't but has a much lower overhead. This can allow us to use different networking technology for different use cases.

##### The challenges

###### Technology coupling

Some RPC mechanisms, like Java RMI, are heavily tied to a specific platform, which can limit which technology can be used in the client and server.

This technology coupling can be a form of exposing internal technical implementation details. For example, the use of RMI ties not only the client to the JVM, but the server too.

There are a number of RPC implementations that don’t have this restriction—gRPC, SOAP, and Thrift

###### Local calls are not like remote calls

The core idea of RPC is to hide the complexity of a remote call. With in-process calls we don't worry about the performance. With RPC, though, the cost of marshalling and un-marshalling payloads can be significant, not to mention the time taken to send things over the network.

Also remember networks aren't reliable! A failure could be caused by the remote server returning an error, or by you making a bad call.  
Can you tell the difference, and if so, can you do anything about it? And what do you do when the remote server just starts responding slowly?

###### Brittleness

What if there are changes in our schema? The client must adapt. Although they don't need those changes.

In practice, objects used as part of binary serialization across the wire can be thought of as "expand-only" types. This brittleness results in the types being exposed over the wire and becoming a mass of fields, some of which are no longer used but can't be safely removed.

##### Where to use it

Just be aware of some potential pitfalls associated with RPC if we're going to pick this model.  
Don't abstract our remote calls to the point where the network is completely hidden, and ensure that we can evolve the server interface without having to insist on lock-step upgrades for clients.  
Finding the right balance for our client code is important, for example. Make sure our clients aren't oblivious to the fact that a network call is going to be made.

Client libraries are often used in the context of RPC, and if not structured right they can be problematic.

#### REST

#### GraphQL

#### Message Brokers