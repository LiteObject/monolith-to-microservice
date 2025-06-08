# Monolith to Microservice

## What is a Microservice?
First, let's quickly recap what a microservice is according to the sources. Microservices are an approach to distributed systems that promote the use of finely grained services that can be changed, deployed, released, and scaled independently. They are modeled around a business domain. This independent deployability is considered a key concept, meaning you can deploy one service without needing to deploy others. They communicate via networks. Microservices offer benefits like options for scaling, improved robustness, and technology heterogeneity. However, they also introduce a host of complexity, especially in the operational space.

## Approach to Building (or Extracting) a Microservice
Adopting microservices should be a conscious decision based on rational decision-making, not a goal in itself. You should have a clear understanding of what you are trying to achieve. It is strongly recommended to try simpler approaches first.

If you decide microservices are the right path for your goal, the recommended approach is an incremental migration, especially when starting from an existing monolithic system. You should chip away at the monolith, extracting functionality bit by bit. Start somewhere small, implement it as a microservice, deploy it to production, and then reflect on whether it helped you get closer to your end goal. This incremental approach helps you learn and limits the impact of mistakes. The monolith itself is not inherently bad and can remain, potentially in a diminished capacity.

### Extracting the Notification System - Potential Patterns
1. **Branch by Abstraction Pattern**: This pattern is explicitly mentioned and used as an example for extracting Notification functionality. It is suitable when the functionality you want to extract is deeper inside the existing system and cannot be intercepted easily at the perimeter. It involves making changes to the monolith's codebase to allow the old and new implementations to coexist safely.
  - How to apply:
    - **Create an abstraction** (e.g., an interface or seam) for the notification functionality within the monolith.
    - **Change existing clients** within the monolith to use this new abstraction. This should be done incrementally and shouldn't change functional behavior initially.
    - **Create a new implementation** of the abstraction that calls your new notification microservice. The bulk of the functionality moves to the microservice.
    - **Switch over the abstraction** to use the new implementation. This can often be done using feature toggles. Feature toggles allow you to switch between the old and new implementations in production.
    - **Clean up** by removing the old implementation from the monolith once you are confident the new service is working correctly.
2. **Strangler Fig Pattern**: This pattern involves wrapping the old system with the new one and redirecting calls to the new service over time. It relies on intercepting calls to the existing system and routing them to the new microservice if the functionality has been migrated. It's very useful when the monolith acts as a black-box system or functionality is accessed from the perimeter.
  - **Applicability for Notification**: This could be used if your notification system is triggered by external calls to specific endpoints on the monolith that you can intercept and redirect. However, if the notification logic is spread throughout the monolith's internal code and not accessed via a clear external boundary, Branch by Abstraction might be more suitable
