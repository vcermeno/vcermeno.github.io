# Some Notes on Books I'm reading...
## Designing Distributed Systems

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
  - Ambassaor container brokers interactions between the application container and the rest of the world.
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
     - Request Splitting

## Designing Data-Intesive Applications

### The Trouble With Distributed Systems
