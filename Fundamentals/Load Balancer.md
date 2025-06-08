### In this lesson, we explain how to cover load balancing in a system design interview.

A _load balancer_ is a type of server that distributes incoming web traffic across multiple backend servers. Load balancers are an important component of scalable Internet applications: they allow your application(s) to scale up or down with demand, achieve higher availability, and efficiently utilize server capacity.

![load balancer](https://exponent-content.s3.amazonaws.com/load_balancer_891b3528f0.png)

## Why we need load balancers[](https://www.tryexponent.com/courses/system-design-interviews/load-balancers#why-we-need-load-balancers)

In order to understand why and how load balancers are used, it's important to remember a few concepts about distributed computing.

First, web applications are deployed and hosted on **servers**, which ultimately live on hardware machines with finite resources such as memory (RAM), processor (CPU), and network connections. As the traffic to an application increases, these resources can become limiting factors and prevent the machine from serving requests; this limit is known as the system's **capacity**. At first, some of these scaling problems can be solved by simply increasing the memory or CPU of the server or by using the available resources more efficiently, such as multithreading.

At a certain point, though, increased traffic will cause any application to exceed the capacity that a single server can provide. The only solution to this problem is to add more servers to the system, also known as **horizontal scaling.** When more than one server can be used to serve a request, it becomes necessary to decide _which_ server to send the request to. That's where load balancers come into the picture.

## **How load balancers work**[](https://www.tryexponent.com/courses/system-design-interviews/load-balancers#how-load-balancers-work)

A good load balancer will efficiently distribute incoming traffic to maximize the system's capacity utilization and minimize the queueing time. Load balancers can distribute traffic using several different strategies:

- Round robin: Servers are assigned in a repeating sequence, so that the next server assigned is guaranteed to be the least recently used.
- Least connections: Assigns the server currently handling the fewest number of requests.
- Consistent hashing: Similar to database sharding, the server can be assigned consistently based on IP address or URL.

Since load balancers must handle the traffic for the entire server pool, they need to be efficient and highly available. Depending on the chosen strategy and performance requirements, load balancers can operate at higher or lower network layers (HTTP vs. TCP/IP) or even be implemented in hardware. Engineering teams typically don't implement their own load balancers and instead use an industry-standard **reverse proxy** (like HAProxy or Nginx) to perform load balancing and other functions such as SSL termination and health checks. Most cloud providers also offer out-of-the-box load balancers, such as Amazon's Elastic Load Balancer (ELB).

## When to use a load balancer[](https://www.tryexponent.com/courses/system-design-interviews/load-balancers#when-to-use-a-load-balancer)

You should use a load balancer whenever you think the system you're designing would benefit from increased capacity or redundancy. Often load balancers sit right between external traffic and the application servers. In a microservice architecture, it's common to use load balancers in front of each internal service so that every part of the system can be scaled independently.

Be aware, load balancing _cannot_ solve many scaling problems in system design. For example, an application can also succumb to database performance, algorithmic complexity, and other types of resource contention. Adding more web servers won't compensate for inefficient calculations, slow database queries, or unreliable third-party APIs. In these cases, it may be necessary to design a system that can process tasks asynchronously, such as a job queue (see the Web Crawler question for an example).

Load balancing is notably distinct from **rate limiting**, which is when traffic is intentionally throttled or dropped in order to prevent abuse by a particular user or organization.

## Advantages **of load balancers**[](https://www.tryexponent.com/courses/system-design-interviews/load-balancers#advantages-of-load-balancers)

- **Scalability.** Load balancers make it easier to scale up and down with demand by adding or removing backend servers.
- **Reliability**. Load balancers provide redundancy and can minimize downtime by automatically detecting and replacing unhealthy servers.
- **Performance**. By distributing the workload evenly across servers, load balancers can improve the average response time.

## Considerations[](https://www.tryexponent.com/courses/system-design-interviews/load-balancers#considerations)

- **Bottlenecks**. As scale increases, load balancers can themselves become a bottleneck or single point of failure, so multiple load balancers must be used to guarantee availability. [DNS round robin](https://en.wikipedia.org/wiki/Round-robin_DNS) can be used to balance traffic across different load balancers.
- **User** **sessions.** The same user's requests can be served from different backends unless the load balancer is configured otherwise. This could be problematic for applications that rely on session data that isn't shared across servers.
- **Longer deploys.** Deploying new server versions can take longer and require more machines since the load balancer needs to roll over traffic to the new servers and drain requests from the old machines.


## Top 6 Load Balancing Algorithms

![Load Balancing Algorithms](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241124110329.png)

### There are two main categories of Load Balancing Algorithms.

1. Static 
2. Dynamic 

Static Algorithms - Distribute the requests without taking into account server real time condition and performance metrics.

1. Round Robin
2. Sticky Round Robin
3. Weighted Round Robin
4. Hash

Dynamic Algorithms - Distribute the requests taking into account server real time condition and performance metrics.

1. Least Connections
2. Least Time

### 1. **Round Robin**

- **Description**: This is one of the simplest load balancing algorithms. It distributes incoming requests to the servers in a circular order, without considering the current load or the health of the servers.
- **Use Case**: Best for scenarios where all servers have similar capacity and no dynamic load tracking is needed.
- **Example**:
    - Request 1 → Server 1
    - Request 2 → Server 2
    - Request 3 → Server 3
    - Request 4 → Server 1, and so on.

### 2. **Weighted Round Robin**

- **Description**: An enhancement of the basic round robin, where each server is assigned a weight based on its processing power. Servers with higher weights receive more requests.
- **Use Case**: Suitable when servers have different capabilities (e.g., some are more powerful than others).
- **Example**:
    - Server 1 (weight 3) → receives 3 requests for every 1 request sent to Server 2 (weight 1).

### 3. **IP Hash**

- **Description**: This algorithm uses a hash of the client’s IP address to determine which server will handle the request. This ensures that a client consistently connects to the same server, which can be useful for session persistence (sticky sessions).
- **Use Case**: Useful for applications where session persistence is important and where clients should consistently be directed to the same server.
- **Example**: The IP address is hashed, and based on the result, a server is selected. For instance, hash(IP address) % number of servers = server index.

## 4. Sticky Round Robin

**Sticky Round Robin** is a variant of the traditional **Round Robin** load balancing algorithm that also incorporates **session persistence**, often referred to as "sticky sessions". In a typical round robin, each incoming request is directed to a different server in a circular pattern, without considering session affinity (i.e., whether or not a user should be consistently routed to the same server).

**Sticky Round Robin** modifies this approach by ensuring that once a client is directed to a particular server, subsequent requests from that same client are sent to the same server, while still following a round-robin pattern for new clients.

### How Sticky Round Robin Works:

1. **Initial Request**: The first request from a client is directed to a server according to the round-robin algorithm.
    
2. **Session Persistence**: A "sticky session" is created, usually through a mechanism like a **cookie** or **session ID**. This session identifies the client.
    
3. **Subsequent Requests**: For all subsequent requests from the same client (based on their session ID or cookie), the load balancer uses the stored session information to send the client’s requests to the **same server** where the first request was routed.
    
4. **Round Robin Continuation**: After the session is "stickied" to a server, new clients (or those without an existing session) continue to follow the round-robin distribution to servers.
    

### Benefits of Sticky Round Robin:

- **Session Persistence**: Clients that need session data (like logged-in users) can interact with the same server consistently, preventing the issues that arise when a session's state is lost if requests are distributed randomly or cyclically to different servers.
- **Simplified Load Distribution**: The algorithm still follows a round-robin pattern for new clients or clients without an active session, which helps balance the overall load across servers.

### Use Case:

Sticky Round Robin is particularly useful in scenarios where:

- **Stateful applications** are being used, where maintaining session state on a particular server is important (e.g., shopping carts, logged-in user data).
- The application can tolerate **minor load imbalances** since once a session is assigned to a server, subsequent requests from that client will always go to that same server.
### Important Considerations:

- **Session Expiry**: Sticky sessions often rely on time-based expiration or session invalidation (for example, if the session cookie expires or the user logs out).
- **Server Failover**: If the sticky server goes down, the load balancer needs a way to redirect the client to a different server, which may involve clearing or resetting the session, or using another mechanism to track the session.
- **Imbalance in Load**: Since sticky sessions ensure a client consistently hits the same server, this can lead to uneven load distribution if some servers handle more sessions than others. This can be mitigated with more advanced mechanisms like **weighted round robin** or **least connections** in conjunction with sticky sessions.

### 5. **Least Connections**

- **Description**: Requests are sent to the server with the fewest active connections (i.e., the server currently under the least load).
- **Use Case**: Ideal for environments where the number of concurrent connections is a more relevant metric than time or processing power.
- **Example**: If Server A has 2 active connections and Server B has 5, the next request will go to Server A.
### 6. **Least Response Time**

- **Description**: This algorithm directs traffic to the server with the lowest response time (i.e., the server that can process requests the fastest).
- **Use Case**: Ideal for situations where minimizing latency is a priority, and servers may have varying response times.
- **Example**: If Server A responds in 100ms and Server B responds in 300ms, the next request will go to Server A.


