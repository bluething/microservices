## How to Model Microservices

### What we need to know?

1. #### Information hiding.
2. #### Coupling.
3. #### Cohesion.
4. #### Boundaries around our microservices.

### Information hiding
Information hiding describes a desire to hide as many details as possible behind a module (or in our case microservice) boundary.  
What are the advantages of information hiding:  
1. Improved Development Time. We can allow for more work to be done in parallel, and reduce the impact of adding more developers to a project  
2. Comprehensibility. Each module can be looked at in isolation, and understood in isolation.  
3. Flexibility. Allowing for changes to be made to the functionality of the system without requiring other modules to change. Modules can be combined in different ways to deliver new functionality.

Microservices can be viewed as a modular application. The connections between modules are the assumptions which the modules make about each other. By reducing the number of assumptions that one module (or microservice) makes about another, we directly impact the connections between them, easier to make changes.

Keep in mind, only share what you absolutely have to, and only send the absolute minimum amount of data that you need.

### Cohesion

In microservices world, we want the functionality grouped in such a way that we can make changes in as few places as possible.  
To do that we want to find boundaries within our problem domain that help ensure that related behavior is in one place, and that communicate with other boundaries as loosely as possible.    
Cohesion applies to the relationship between things inside a boundary (a microservice in our context).

### Coupling

A good service is a service that have loosely coupled, a change to one service should not require a change to another.  
What make a service tight coupling? A classic mistake is to pick an integration style that tightly binds one service to another, causing changes inside the service to require a change to consumers. Another cause is number of different types of calls from one service to another, also have an impact to performance.  
Coupling describes the relationship between things across a boundary.

#### Relationship between Couples and Cohesion

If related functionality is spread across our system, changes to this functionality will ripple across those boundaries, implying tighter coupling.  
Constantine Law by Larry Constantine said "A structure is stable if cohesion is strong and coupling is low."  
Find the right balance between them.

#### Types Of Coupling

Some coupling is inevitable, what we can do is reduce how much coupling we have.

##### Domain Coupling

A situation where one microservice needs to interact with another microservice, because it needs to make use of the functionality that the other microservice provides.  
![domain coupling](https://drive.google.com/uc?export=view&id=1Kp0WD3S6BG0Chx8AvgBXF0CdLCyJ2NN3)

This kind of coupling is inevitable, keep this to a minimum though.  A microservice which needs to talk to lots of downstream microservices might point to a situation where too much logic has been centralized.

##### Temporal coupling

![temporal coupling](https://drive.google.com/uc?export=view&id=14MOZuk1CHzyr7qPFZRq8fToF35SxkuZ-)  
Both microservices need to be up and available and communicate with each other at the same time in order for the operation to complete. Temporal coupling isn’t always bad, it’s just something to be aware of.

##### Pass Though Coupling

A situation where one microservice passes data to another microservice purely because it is needed by some other further downstream microservice.  
![pass through coupling](https://drive.google.com/uc?export=view&id=1jHZQa-mZgbiXnvhdqUsskqjEs8WZYMby)  
A change to the required data downstream can cause a more significant upstream change, changes in both Order Processor and Warehouse services.

The solution?

###### Direct call to the service (not using intermediate service)

![solution1](https://drive.google.com/uc?export=view&id=1Iql3t8u-lwW9ieqANeQgweyAF-robLa5)

The new problem is increasing domain coupling at Order Processor. Also, more complexity because we need to call Warehouse service twice and Order Processor need to aware about this.

###### Make intermediate service handle the downstream

![solution2](https://drive.google.com/uc?export=view&id=13iqrh73yVUSFs3k9iH6jgRagulSPzRRM)

The new problem come when downstream (Shipping) need mandatory data that must be provided by upstream (Order Processor).

###### Middle service will bypass data

Middle service totally unaware of the structure of data needed by downstream, only pass the field.

##### Common Coupling

![common coupling](https://drive.google.com/uc?export=view&id=1-2EKHDSIwg3j1-Dn4TpTwnKLgAhtuBqR)  
Multiple microservices making use of the same shared datasource.

The solution? Create a finite state machine.A state machine can be used to manage the transition of some entity from one state to another, ensuring invalid state transitions are prohibited.  
The problem with finite state machine is who took the responsibility to manage the state?

![finite state machine responsibility](https://drive.google.com/uc?export=view&id=1MVTRbWUF349aliDu6_e5BwCtVCb7d6Zz)  
This is a thin wrapper around database CRUD operations. It is a sign that a weak cohesion and tighter coupling, as logic that should be in that service to manage the data is instead spread elsewhere in your system.  
Sources of common coupling are also potential sources of resource contention.

##### Content Coupling

Avoid this!  
We must have a clear separation between what can be changed freely, and what cannot.

![content coupling](https://drive.google.com/uc?export=view&id=1jq4QI2R0CSoCzKQRJW7cwjNaeaiZV0tw)  
A situation where an upstream service reaches into the internals of a downstream service and changes its internal state.  
In content coupling the ownership become less clear, and it becomes more difficult for developers to change a system.

### Boundaries

Is all about DDD!

### Alternatives to Business Domain Boundaries

#### Volatility

Volatility based decomposition has you identify the parts of your system going through more frequent change, and extract that functionality into their own services where they can be more effectively worked on.

#### Data

The nature of the data you hold and manage can drive you towards different forms of decomposition. Segregation of data is often driven by a variety of privacy and security concerns.

#### Technology

#### Organizational

We must take account of the existing organizational structure when considering where and when to define boundaries, and in some situations perhaps even consider changing the organizational structure to support the architecture we want.  
What happens if our organizational structure changes? Maybe we need examine an existing microservice.