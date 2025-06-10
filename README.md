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
   - **Applicability for Notification**: This could be used if your notification system is triggered by external calls to specific endpoints on the monolith that you can intercept and redirect. However, if the notification logic is spread throughout the monolith's internal code and not accessed via a clear external boundary, Branch by Abstraction might be more suitable.
  
3. **Decorating Collaborator Pattern**: This pattern involves allowing the call to the monolith to proceed as normal, and then, based on the result of that call, triggering behavior in an external microservice. This could be implemented with a proxy that intercepts responses.
   - **Applicability for Notification**: This might apply if notifications are consistently triggered after a specific action is completed by the monolith (e.g., after an order is successfully placed). The proxy would detect the successful outcome and trigger the notification microservice.
  
4. **UI Composition**: If the notification system also involves user interface components (e.g., settings, history), you could use UI composition patterns to have the new microservice serve those parts of the UI incrementally.


Given that notification logic can often be triggered from various internal points within an application, the Branch by Abstraction pattern seems like a strong candidate based on the explicit example provided in the sources.

Key Considerations When Extracting the Notification Service:

- **Define the Goal**: Why are you splitting out notifications? Is it to improve the ability to scale notification delivery independently (e.g., handling email vs. SMS vs. push notifications)? Is it for fault tolerance, so a failure in one notification method doesn't bring down other parts of the system? Is it to allow a separate team to own notification development? Your goal will influence how you define the service boundary and what functionality to include.
- **Data**: Does the notification system have its own data (e.g., recipient preferences, notification history, templates)? If so, this data should ideally be owned by the new microservice. This involves decomposing the database, which is a significant challenge. You may need to migrate data or use patterns like Monolith as Data Access Layer or Multischema Storage during the transition.
- **Communication Style**: How will the monolith (and potentially other future services) communicate with the new notification microservice? Options include synchronous (like REST API calls) or asynchronous (like sending messages or events). Event-driven communication (where the monolith emits an "OrderCreated" event, and the notification service subscribes) can allow for more loose coupling and easier future additions of other event consumers. A mix of styles is common.
- **Verification and Testing**: As you extract functionality, it's vital to verify that the new microservice works correctly and that the integration points don't break existing functionality. Techniques like Parallel Run (running old and new side-by-side and comparing results) can be very helpful. Implementing automated tests and potentially using consumer-driven contracts can help ensure compatibility. Testing in production using techniques like feature toggles or canary releases is also a recommended practice.
- **Technology Choice**: While microservices are technology agnostic, you can initially build the new microservice using .NET 4.8 or a newer .NET version if your team is comfortable. You can choose the right technology for this specific service later if needed.
- **Enabling Capabilities**: Ensure you have foundational capabilities in place, such as log aggregation (considered a prerequisite), monitoring, and potentially distributed tracing to understand how calls flow across the system. Automation is key as the number of services grows.

In summary, to extract the notification system from your legacy .NET 4.8 monolith, begin by defining your clear goal for the migration. Consider using the Branch by Abstraction pattern to incrementally pull the notification logic out by creating an abstraction in the monolith and gradually replacing the internal implementation with calls to your new microservice. Plan for migrating any notification-specific data and choose an appropriate communication style. Crucially, invest in rigorous testing and verification techniques like Parallel Run and automated tests, and ensure you have foundational operational capabilities like log aggregation in place.

---
### References:
- **Building Microservices: Designing Fine-Grained Systems** by Sam Newman
- **Monolith to Microservices** by Sam Newman
- **Mastering API Architecture: Design, Operate, and Evolve API-Based Systems** by James Gough, Daniel Bryant, Matthew Auburn- 
- **Practical Microservices Architectural Patterns** by Binildas Christudas


