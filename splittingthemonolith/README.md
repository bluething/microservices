### We must have a goal

The goal is not the microservices. Think if current architecture can still bring us to end goal. Consider easier ways to achieve that end goal before considering microservices.  
We must focus on the benefits of architectural changes.

### Incremental migration

Martin Fowler said "If you do a big-bang rewrite, the only thing you’re guaranteed of is a big bang."

If we decided to migrate to microservices, do it incrementally.  
An incremental approach will help you learn about microservices as you go, and will also limit the impact of getting something wrong. Choose one or two areas of functionality, implement them as microservices, get them deployed into production, and reflect on whether it worked. Think about part of our codebase that give most benefit if we separate it.

Real system architecture is a constantly evolving thing, adapting as needs and knowledge change.  
By making your migration to microservices an incremental journey, you are able to chip away at the existing monolithic architecture, delivering improvements along the way, and importantly, knowing when to stop.

### Is monolith bad?

Remember software spectrum from [reactivemicroservices](https://github.com/bluething/rectivearchitecture/tree/master/reactivemicroservices).  
Don't focus on "not having the monolith", focus instead on the benefits you expect your change in architecture to bring. It is common for the existing monolithic architecture to remain after a shift towards microservices.  
The author of the book said "the dead of monolith because dying on dead technology, is tied to infrastructure that needs to be retired, or is perhaps an expensive 3rd party system that you want to ditch."

#### The Dangers Of Premature Decomposition

There is a danger in creating microservices when you have an unclear understanding of the domain. Define a right boundaries is a must. We can use DDD technique.

### Find the right boundary

Good service is a service that have loose coupling and high cohesion. To achieve this we need a code that can be treated in isolation and worked on without impacting the rest of the codebase.  
Bounded contexts is a good candidate, because by definition they represent cohesive and yet loosely coupled boundaries in an organization.

#### What should we do then?

1. Identify the high-level bounded contexts that we think exist in our organization.
2. Create packages representing these contexts, and then move the existing code into them. Don't forget to have proper testing. Maybe we will see code which is not included in any package. Time to think existing or new boundary.
3. Analyze the dependencies between packages. Search for inter package dependency. Remember, the package is a bounded context representation and bounded context represent our organization. Communication that occurs in the organization must be the same as communication in our system.

### What To Split First?

Want to scale the application? Functionality which currently constrain the system’s ability to handle load are going to be high on the list.  
Want to improve time to market? Look at the system’s volatility to identify those pieces of functionality that change most frequently, and see if they would work as microservices.

The decision about which functionality to split into a microservice will end up being a balance between these two forces  
1. How easy the extraction is?  
2. The benefit of extracting the microservice in the first place.

The author of the book suggest for the first couple of microservices would be to pick things which lean a bit more towards the "easy" end of the spectrum.  
If we failed, maybe microservices does not suit us and the organization.

### Decomposition By Layer

Usually we deal with 3 tiers application, extract in terms of its user interface, backend application code, and data.  
The mapping from a microservice to a user interface is often not 1:1.

#### Code First

Move the logic to new service, connect to existing database.  
Make sure we can move the data. Think whether extraction is viable, and how you will go about it.

#### Data First

The data move to new database, the logic still in monolith.

### Useful Decompositional Patterns

#### Strangler Fig Pattern

It's a wrapper. Wrapping an old system with the new system over time, allowing the new system to take over more and more features of the old system incrementally.  
![strangler](https://drive.google.com/uc?export=view&id=1TRRLx5TBfOHTKonS5CzMHa-uOUtOXIDf)

#### Parallel Run

Running both your monolithic implementation of the functionality and the new microservice implementation side-by-side, serving the same requests, and comparing the results.

#### Feature Toggle

A feature toggle is a mechanism that allows for a feature to be switched off or on, or to switch between two different implementations of some functionality.

### Data Decomposition Concerns

#### Performance

Happen when a service need to look up data in the monolith (or other service). SELECT query + API call.  
The solution, we look up in bulk or cache the necessary data.

#### Data Integrity

When we spit the service, we can no longer rely on the database to enforce the integrity.  
What happen if we delete reference data in the monolith database? Maybe we can use soft delete.  
Or we can duplicate only necessary data (some columns), but consistency still an issue.

#### Transactions

How about ACID?  
Do we need to distribute the transaction? I think no, it's too complex. If we want, we can use saga pattern.

#### Tooling

There are limited tools to help us move the database.   
[Flywaydb](https://flywaydb.org/) and [Liquibase](https://www.liquibase.org/) is two of them.  
Database has a state, this is the difficulty factor.

#### Reporting Database

We abstract the data behind the service. What if we need to access the data directly instead of via an API, maybe for reporting purpose?

###### Read replica

We can create a dedicated database which is designed for external access, and make it the responsibility of the microservice to push bare minimum of data from internal storage to the externally accessible reporting database. The database owner same as with the microservices.

The key point is:
1. Never expose the data! Put only necessary data into the reporting database. The schema also doesn't need to be the same. Reporting database is a subset of service database.
2. Treat reporting database like an end point. It is the job of the microservice maintainer to ensure that compatibility of this endpoint is maintained even if the microservice changes its internal implementation detail.

The challenges we will face:  
1. Reporting service must aware of schema changes in reporting database (coupling).  
2. Limited options in database optimization to support the microservices itself and reporting. The optimized schema maybe different for both cases.   
3. The microservices and reporting database maybe not same type or vendor. Need to think which type or vendor suit for both need.

###### Data retrieval via API

Get data via API calls. Is ok for small report.

What if the report need to proceed large amount of data? Of course the process will be slow and make the service busy. Don't try to keep local copy of this data. The data maybe stale when we use it.

How about pull data periodically? Usually we need large amount of API calls to get the data.  
Can we utilize cache to decrease service load? We may still experience cache miss.

The recommended way is creating batch API. The API will return HTTP response code 201, and the data will be put somewhere that accessible by reporting service.  
We can also use 202 response code to indicate the process is running then reporting service need to poll the resource waiting until it retrieve a 201.

If you prefer to return the data as http response, you can expose the API that receive list of ID and use pagination in your response.

###### Data pumps

One of the downsides of retrieving the data by standard HTTP calls is the overhead of HTTP when we’re making many calls, together with the overhead of having to create APIs that may exist only for reporting purposes.

We can create standalone service (same owner with the database and microservices) that pull the data from database then push into reporting database. The pump service is like a mapper.  
With this approach we will have schema coupling. It's ok because makes reporting easier.

###### Event data pumps

![event data pump](https://github.com/bluething/microservices/blob/main/images/eventdatapump.png?raw=true)

Instead of pull and push the data from database how about the service publish an event to subscriber? Using an event we don't need to send all data, sent deltas is enough.  
The main downsides to this approach are that all the required information must be broadcast as events, and it may not scale as well as a data pump for larger volumes of data that has the benefit of operating directly at the database level.

###### Backup data pumps

Similar with data pumps, but using backup database. Pay attention about existing backup mechanism.

### Additional reading

[StranglerFigApplication](https://martinfowler.com/bliki/StranglerFigApplication.html)  
[Strangler Fig pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/strangler-fig)  
[Feature Toggles (aka Feature Flags)](https://www.martinfowler.com/articles/feature-toggles.html)  
[Coding with Feature Flags: How-to Guide and Best Practices](https://thysniu.medium.com/coding-with-feature-flags-how-to-guide-and-best-practices-3f9637f51265)  
[Pattern: Saga](https://microservices.io/patterns/data/saga.html)  
[Saga Pattern in Microservices](https://www.baeldung.com/cs/saga-pattern-microservices)  
[ACID Compliance: What It Means and Why You Should Care](https://mariadb.com/resources/blog/acid-compliance-what-it-means-and-why-you-should-care/)  
[ACID Properties in DBMS](https://www.geeksforgeeks.org/acid-properties-in-dbms/)  
[Best Practices for Moving from a Monolith to Microservices](https://rollbar.com/blog/best-practices-for-moving-from-a-monolith-to-microservices/)