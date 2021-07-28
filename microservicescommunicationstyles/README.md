Getting communication between microservices right is problematic, make sure to consider the different types of communication we might want.  
Think about _synchronous blocking_ and _asynchronous non-blocking_ communication mechanisms, as well as comparing _request-response_ collaboration with _event-driven_ collaboration.

Synchronous or asynchronous is to describe the relation between services.  
Blocking or non-blocking is to describe the situation in the service.

### From In-Process To Inter-Process

In-process means calls within a single process.  
Inter-process means calls between different processes _across a network_.

##### Performance

In in-process communication, compiler and runtime make an impact to performance. Including inlining the invocation, so it's as though there was never a call in the first place.  
Different with inter-process communication, network will be an important factor.  
1000 calls across an API boundary vs 1000 network calls between two microservices. Reducing the network calls amount seems like a good idea.

The data sent between the two microservices is the next factor.  
In in-process, data transfer is only passing a memory address.  
In inter-process, the data actually has to be _serialized_ into some form that can be _transmitted_ over a network. The data then needs to be sent, and _deserialized_ at the other end. Pay attention to _the amount of data_ being sent or received and _serialization mechanisms_. Another approach is offloaded data to a file system and pass around a reference to that file location instead.

##### Changing Interfaces

It's easy to do refactoring in in-process communication, the IDE will help us.  
How about inter-process? A backward compatibility is a must! We either need to do a lock-step deployment with consumers, making sure they are updated to use the new interface, or else find some way to phase the rollout of the new microservice contract.

##### Error handling

Within a process, the errors either expected and easy to handle. Remember fail fast!  
In inter-process communication the source of the error comes from several places. For example networks get disconnected, containers get killed due to consuming too much memory, and in extreme situations, bits of your data centre can catch fire. We can classify into five categories:  
1. Crash Failure. Server crash, need reboot.  
2. Omission Failure. We sent something, but we didn't get a response, or we except downstream to sent us the message, but they didn't.  
3. Timing Failure. Something happened too late (we didn't get it in time), or something happened too early.  
4. Response Failure. We got a response, but it just seems wrong.  
5. Arbitrary Failure. Known as Byzantine failure, this is when something has gone wrong, but all participants are unable to agree if the failure has occurred (or why).

It's important to have a _richer set of semantics_ for returning errors in a way that can allow for clients to _take appropriate action_. For example HTTP, they have 400 and 500 series codes being reserved for errors.

### Technology for Inter-process communication

##### How to choose?

Think about the style of communication you want then look for the right technology to implement those styles.  
Also consider latency, security and scalability.

##### The styles

![microservicescommunicationstyles](https://github.com/bluething/microservices/blob/main/images/microservicescommunicationstyles.png?raw=true)

A microservice architecture as a whole may have a mix of styles of collaboration, some interactions just make sense as request-response, some make sense as event-driven.

#### Synchronous Blocking

###### Advantages

It's simple

###### Disadvantages

Synchronous calls leads to temporal coupling between upstream and downstream. The use of synchronous calls can therefore make a system more vulnerable to cascading issues caused by downstream outages more readily than asynchronous calls

###### Where To Use It

Use it for simple microservice architectures. Start to see other style if you have a long chain calls.  
What the problem if we have long chained calls? Resource contention, especially for uppermost upstream.

#### Asynchronous Non-blocking

Asynchronous call doesn't always mean non-blocking!  
Asynchronous call help us to make our services loose couple, but there are additional complexity if we choose asynchronous style.

##### Communication Through Common Data

The upstream microservice changes some common data, which one or more microservices later make use of. The flow of information is in a single direction. Two common examples of this pattern are the data lake, and the data warehouse.

###### Advantages

It's simple. Technologies agnostic.

###### Disadvantages

There are the gap between data being published and data being processed, need low latency network.  
The common data store becomes a potential source of coupling. It's a tight coupling.  
The robustness of the communication will also come down to the robustness of the underlying data store.

###### Where To Use It?

Use it when our services have restrictions in what technology they can use. Another case is when sharing large volumes of data.

##### Request-Response Communication

The implementation can be a synchronous blocking or an asynchronous non-blocking. Pay attention to timeout handling.  
In a synchronous call the connection is kept open, waiting for the downstream microservice to respond.  
With an asynchronous call we utilize a message broker. Upstream put message to queue then consume by downstream. As a response, downstream put message to other queue (they need to know about this) to consume by upstream. We need to relate the response to the original request.

###### Where To Use It?

Use it when the result of a request is needed before further processing can take place. Also, if a microservice wants to know if a call didn’t work, so that it can carry out some sort of compensating action.

##### Event-Driven Communication

A microservice emits events which may or may not be received by other microservices. An event is a statement about something that _has occurred_, nearly always something that has _happened_ inside the world of the microservice that is emitting the event.  
It emits the event when required, and that is the end of its responsibilities. The responsibility push to other service.

The event emitter is leaving it up to the recipients to decide what to do. With request-response, the microservice sending the request knows what should be done, and is telling the other microservice what it thinks needs to happen next.

We can use message broker to implement it. Remember to keep your middleware dumb, and keep the smarts in the endpoints.  
Another approach is to try to use HTTP as a way of propagating events, for example ATOM.

###### Events vs messages

An event is a fact - a statement that something happened, along with some information about exactly what happened. A message is a thing we send over an asynchronous communication mechanism, like a message broker.  
The message is the medium, the event is the payload.

With a request, we are asking a microservice to do something, and providing the required information for the requested operation to be carried out.  
With an event we are broadcasting a fact that other parties might be interested in, but as the microservice emitting an event can't and shouldn't know who receives the event.

How do we know what information other parties might need from the event? Using identifier and look up to upstream.  
Another way is put everything into an event. The issue is how big the message?  
How about less knowledge principle? Maybe we can send two different types of events. Another problem raises. The complexity to manage multiple events and ensuring those events actually get fired.

Event is a contract between our service and the other. The more data we put into an event, the more assumptions external parties will have about an event.

###### Where To Use It?

Use it when we can to broadcast the information, and in situations where you are happy to invert intent.

#### Synchronous / asynchronous and blocking / non-blocking visualization

Got this from [SO post](https://stackoverflow.com/a/31298006)

Imagine this situation at bookstore  
Module X: "I".  
Module Y: "bookstore".  
X asks Y: do you have a book named "Building Microservices, 2nd Edition"?

1. `blocking`: before Y answers X, X keeps waiting there for the answer. Now X (one module) is blocking. X and Y are two threads or two processes or one thread or one process? we DON'T know.  
2. `non-blocking`: before Y answers X, X just leaves there and do other things. X may come back every two minutes to check if Y has finished its job? Or X won't come back until Y calls him? We don't know. We only know that X can do other things before Y finishes its job. Here X (one module) is non-blocking. X and Y are two threads or two processes or one process? we DON'T know. BUT we are sure that X and Y couldn't be one thread.  
3. `synchronous`: before Y answers X, X keeps waiting there for the answer. It means that X can't continue until Y finishes its job. Now we say: X and Y (two modules) are synchronous. X and Y are two threads or two processes or one thread or one process? we DON'T know.  
4. `asynchronous`: before Y answers X, X leaves there and X can do other jobs. X won't come back until Y calls him. Now we say: X and Y (two modules) are asynchronous. X and Y are two threads or two processes or one process? we DON'T know. BUT we are sure that X and Y couldn't be one thread.

Pay attention in `non-blocking` scenario. What happens with X after give Y a question?  
X may come back every two minutes to check if Y has finished its job? _Or_ X won't come back until Y calls him? We don't know.

In `asynchronous` scenario, X won't come back until Y calls him. Only on case will happen with X.

```text
// thread X
while (true)
{
    msg = recv(Y, NON_BLOCKING_FLAG);
    if (msg is not empty)
    {
        break;
    }
    else
    {
        sleep(2000); // 2 sec
    }
}

// thread Y
// prepare the book for X
send(X, book);
```  
non-blocking & synchronous scenario. Thread X didn't enter blocked state. Most time this loop does something nonsense but in CPU's eyes, X is running, which means that X is non-blocking.  
X can't continue to do any other things (X can't jump out of the loop) until it gets the book from Y.

### Additional reading

[Communication in a microservice architecture](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/communication-in-microservice-architecture)  
[How does non-blocking IO work under the hood?](https://medium.com/ing-blog/how-does-non-blocking-io-work-under-the-hood-6299d2953c74)  
[Explain non-blocking I/O like I’m five](https://blog.codecentric.de/en/2019/04/explain-non-blocking-i-o-like-im-five/)  
[Message Brokers](https://www.ibm.com/cloud/learn/message-brokers)  
[An introduction to Message Brokers](https://medium.com/@xaviergeerinck/an-introduction-to-message-brokers-9bd203b4ebbd)