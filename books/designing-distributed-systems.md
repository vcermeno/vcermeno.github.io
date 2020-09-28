## Designing Distributed Systems - Notes

### The Sidecar Pattern
 - The sidecar container adds additional functionality to the application container.
 - Uses: 
    - Adapt legacy applications
    - Debugging Application (readouts)
    - Simple Paas
  - Design for modularity and reusability:
    - Parametrize your containers
      - Each parameter represtens an input that can customize a generic container to a specific situation
    - Create the API surface of your container
      - All aspects of how your containers interact with the world are part of your API
      - A Clean API defines the sidecar's services(it's essentia; and expected functions) and provides a clean separation from the application
    - Document the operation of your container
      - Good practice to include ```EXPOSE``` in Dockerfiles
      - If you use env vars to parametrize your container, use ```ENV``` directive in Dockerfile to help document usage
      - Always use ```LABEL``` directive to add metadata(email, webpage, version/latest) for your Docker image
      
### Ambassadors
  - Ambassador container brokers interactions between the application container and the rest of the world.
  - Uses:
    - Sharding- Two Approaches
      1. Build all sharding logic into the sharded service, and include a stateless load balancer - directs traffic to appropriate shard. 
         - Upside? No need to change client-side(main app) logic as it will still interact with one service- the ambassador
         - Downside? More complicated deployment for the sharded service
      2. Integrate a client-side ambassador that is a *sharding ambassador proxy* - recieves requests from application, contains logic to route requests to shards, returns result to application.
         - Upside? Simplifies sharding service deployment. Separation of concerns.
         - Downside? Complicates client-side deployment
         - Example: Sharded Redis
           1. Create Redis Kubernetes Cluster(3 replicas) using Stateful Set(will give us unique DNS names for each shard)
           2. Use Kubernetes ```Service``` to get DNS names for each replica
           3. Use twemproxy(lightweight, performant proxy by Twitter) to point to created replicas
              - Serve the redis protocol on the same port to give the app container access to the ambassador
           4. Deploy proxy into Kubernetes ```ConfigMap```
           5. Define your pod - will include 2 containers: application and ambassador container
     - Service Brokering
       - A service broker ambassador can be used to prrovide service discovery across multiple environments(i.e. public cloud, physical datacenter, private cloud)
     - Request Splitting
       - Request splitting: where some fraction of all requests are nto served by the main production service but directed to a different implentation of the service
         - Often used to **tee**  or split traffic so that all traffic goes to both the production ystem and a newer, undeployed version
         - The responses from the production service are returned to the user, while the responses from the tee-d service are ignored.
         
### Adapters
  - In the adapter pattern, the adapter container, is used to modify the interface of the application container so that it conforms to the predefined interface that is expected of all applications.
  - When each application provides metrics using a different format and interface, it is very difficult to collect all of those metrics in a single place for visaulaiztion and alerting. This is the perfect situstion for the adapter pattern.
  - Monitoring
    - When monitoring your software, you want a single solution that can automaticallly discover and monitor any appliation that is deployed in your environment
    - Applying the adapter pattern to monitoring, the adapter container contains the tools for transforming the moitoring interface exposed by the application container into the interface expected by the general-purpose monitoring system.
    - Pros:
      - Decoupling the system in this fashion makes for a more comprehensible, maintainable system.
      - The monitoring container can be reused with multiple different application containers.
      - Deploying the monitoring adapter as a seperate container ensures that each container gets its own dedicated resources(CPU, memory)
 - Logging
    - Different application containers can log informtation in different formats, but the adapter container can transform that data into a single structured representation that can be c onsumed by your log aggregator.
    - While the application container may log to a file, the adapter container can redirect that file to stdout.
    - Reason to use adapter pattern vs. modifying application container:
      - Often, we are reusing a container produced by a third-party. In these cases, deriving a slightly modified image we have to maintain is significantly more expensive than developing an adapter container we can run alongside the other party's image.
 - Adding a Health Monitor
   - Allows for richer health checks (i.e. run queries against the database)
   - The adapter container container is a simple container that only contains the shell script for determining the health of the application(i.e. database). The script can be set up as the health check for the application container and can perform whatever rich health checks our application requires.
     - If the checks fail, the application can be restarted
   - While you can implement a health check within the application container itself, this ignores the strong benefot derived from modularity- the work becomes inherently decoupled and more easily shared.
   
### Serving Patterns
  - Reliability, scalability, and separation of concerns dictate that real-world systems are built out of many different components, spread across multiple machines.
  - In contrast to single-node patterns, the multi-node distributed patterns are more loosely coupled.
  - Microservices = multi-node distributed software architectures
  - Microservices stand in contrast to monolithic systems, which tend to place all functionality of a service within a single, tightly coordinated application.
    - Benefits:
      - Microservices break down an application into small pieces, each focused on providing a single service; Reduced team size reduces overhead
      - The introduction of formal APIs in ebtween different microservices reduces the need for tight synchronization among teams. 
        - The team providing the API unnderstand surface area it needs to keep stable
        - The team consuming the API can rely on a stable service wihtout worrying about the details
      - Decoupling of services = better scaling
        - Some services are stateless and can simply scale horizontally, whereas other systems maintain state and require sharding or other approaches to scale. By seperating each service out, each service can use the approach to scaling that suits it best.
    - Cons:
      - Debugging the system when failures occur is significantly more difficult
      - Difficult to design and architect
        - Microservices based system uses multiple methods of communicating between services, different patterns(e.g. synchronous, asynchronous, message-passing, etc.); and multiple different patterns of coordination and control among the services.
        - If a microservices architecture is made up of well-known patterns, then it is easier to design because many of the design practices are specified by the patterns.
      
