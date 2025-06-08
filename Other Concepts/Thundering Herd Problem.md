
Imagine a bull running towards you when you are relaxing on a farm.

![](https://miro.medium.com/v2/resize:fit:640/1*gJU3bmk09A_6o5co44_zFg.jpeg)

Source: Google

What would you do? Easy “I would just get out of the way” you would say. Well how about now?

![](https://miro.medium.com/v2/resize:fit:333/1*WJreBQ0JTjrbuvT20_VZag.jpeg)

Source: google

If it’s not clear it’s a herd of raging bulls. Unless you have extreme luck you would likely be trampled. That’s the exact situation of the servers when a huge amount of requests greater than it’s compute power comes in. This is commonly known as “**thundering herd”** problem. In this blog, I will try to discuss the various scenarios and possible solutions from my understanding through various references.

The thundering herd problem can occur when there is a cascading failure — say you have 3 servers running and a load balancer

![](https://miro.medium.com/v2/resize:fit:481/1*81TahBbPSvKZzfsF9Ce9pw.png)

Let us say each server can handle a certain number of requests(say 500) per second and the load balancer distributes the requests to handle the load. But what if the server 1 goes down while handling 500 requests ( after completing say 200 requests). So the rest of the 300 requests would be sent to server 2 . Let’s say server 2 comparatively has lesser computing power or even if it has the same computing power it is handling more than it’s designated capacity (QPS). So the server 2 crashes after a while ( after handling 200 requests). Now the first 200 + requests directed at server 2 goes to server 3. Now remember server 3 may additionally also have to handle new requests. So there we have the thundering herd problem . By now let’s say if you are using JVM based languages there is a high probability of out of memory error and server 3 crashes now. So there you have the scenario of “**cascading failure”** . Now, you may ask why not scale up (using terraform) and add another server? But in this situation of cascading failure there is a high probability that server would crash too despite higher computing power and memory allocations.

## What is the Thundering Herd Problem?

Imagine a scenario, where an event causes one or more of your services to experience an enormous surge in traffic, overwhelming their capacity. This can lead to one or more dependencies, such as a database, becoming overloaded and unresponsive, ultimately resulting in service failure (_cascading failures_). Such events could include multiple service instances failing and redirecting all traffic to a single instance, a viral image or video receiving huge viewership, or an online sale during a festival causing a database overload. This situation, where cascading failures lead to service unavailability due to a sudden spike in incoming traffic, is termed the _Thundering Herd Problem_.


![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7a4d49ca-120c-43d8-a3d4-fc70cd8a3332_834x702.png)


Fig. Thundering Herd Problem

## How to Handle the Situation?

#### Exponential Jitter and Retry

When a service fails to respond, the instinctive solution is to retry the request, assuming a transient failure. However, this approach can lead to the Thundering Herd scenario or exacerbate an existing one, as all clients retry simultaneously, overwhelming system resources. Instead, if clients retry at random intervals, the overloaded resource gets time to recover and respond. This randomness in retry timing, known as _Jitter_, helps distribute the load more evenly and prevents further strain on the system.


![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff928067b-5ad2-454a-ba59-c50827a126cd_927x792.png)


Fig. Exponential Jitter and Retry

#### Queueing Requests

Consider a scenario where a request to fetch an image from the cache results in a miss, leading to the request being fulfilled from the origin datastore. If a large number of simultaneous requests experience cache misses and are forwarded to the data store, this can create the Thundering Herd problem. Since all the requests are for the same image, only a single request should be forwarded to the datastore for fulfillment. The remaining requests can be queued and served from the cache once it is updated after the initial request returns from the data store.


![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe064f974-cff3-4065-a697-eb0baeaa6bcf_880x871.png)


Fig. Queueing Requests

#### Load Balancing

Every large-scale application at some point requires service replication on the backend to handle increasing traffic. However, if this traffic is not evenly distributed across all service replicas, it can overwhelm specific instances. Using load balancers to distribute the load uniformly helps prevent the Thundering Herd problem.


![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8e3b85f2-0983-46e4-b869-088e098f7533_791x365.png)

Fig. Load Balancing

#### Rate Limiting

If a service exposes APIs, providing unlimited access to its clients can be disastrous if one or more clients abuse it. Scenarios like DDOS attacks or scheduled batch jobs can trigger the Thundering Herd problem. Implementing rate limiting to control how frequently a client can call the API can help manage high-throughput clients and prevent such issues.


![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbf3e383a-c026-4ecf-9084-c7e3354cc84a_700x776.png)

Fig. Rate Limiting

#### Circuit Breaker

A service dependency, such as a database, can fail due to the Thundering Herd problem. Similar to how an MCB (Miniature Circuit Breaker) protects a circuit by breaking it when there's a sudden spike in electric voltage, a service can implement a circuit breaker approach. This approach halts sending further outgoing requests to the dependency until it recovers and is ready to handle traffic again.


![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2c9a3560-7999-43d7-bef0-89e95370b6a7_786x708.png)

Fig. Circuit Breaker


### References 

https://newsletter.scalablethread.com/p/how-to-handle-sudden-bursts-of-traffic

https://www.youtube.com/watch?v=XuG7wGGTWoc