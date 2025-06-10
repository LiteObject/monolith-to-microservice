# DDD (Domain Driven Design) & Microservices

Domain Driven Design (DDD) plays a fundamental role in building microservices. Microservices are intended to be modeled around a business domain, and DDD provides the primary mechanism for achieving this.

Here's how DDD influences microservice architecture according to the sources:
1. **Defining Service Boundaries:** DDD is crucial for finding sensible boundaries for microservices.
   - The concept of a **Bounded Context (BC)** from DDD is central to this. Microservices derive from BCs. Each microservice implements a specific end-to-end domain or business capability within a certain context boundary.
   - BCs represent units of cohesion with well-defined interfaces to the wider system. They are considered excellent starting points for defining microservice boundaries.
   - A BC is an autonomous subsystem that must own its domain model, encompassing both data and logic/behavior.
   - While a BC is a **logical boundary**, a microservice is like a BC but is specified as a distributed service. Though often, a BC correlates to one business microservice, a BC or business microservice can sometimes be composed of several physical services.
   - Identifying BCs is often the **first challenge** when defining microservice boundaries. This involves identifying decoupled areas of data and different contexts within the application, where each context might have a different business language.
  
2. Promoting Independent Deployability, Low Coupling, and High Cohesion:
   - By aligning microservices to the boundaries defined by the business domain (BCs), changes are more likely to be **isolated to a single microservice**. This reduces the need to coordinate changes across multiple services and potentially multiple teams, which is essential for enabling **independent deployability**.
   - DDD, combined with the principle of **information hiding**, helps create services with stable boundaries and promotes **low coupling** and **strong cohesion**. Domain coupling, where one service needs to interact with another's functionality, is unavoidable but should be minimized.
   - Modeling services around the business domain prioritizes **high cohesion of business functionality**, ensuring related behavior is in one place.
  
3. Encapsulating Domain Logic and State:
   - Microservices encapsulate the state and the code that manages state transitions. DDD's **Aggregate** pattern is useful here. Aggregates are collections of objects managed as a single entity and act as self-contained state machines for domain concepts.
   - A microservice instance can manage one or more aggregates, but an aggregate should not be split across multiple microservices.
   - The microservice should **hide its internal implementation details**, including its data storage. This is a key aspect of DDD's information hiding and is vital for stable boundaries and independent deployability. Sharing databases directly between microservices is problematic and undermines independent deployability.
  
4. Aligning Architecture with Organization:
