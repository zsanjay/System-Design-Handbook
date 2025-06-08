References :

	https://konghq.com/blog/engineering/api-gateway-vs-load-balancer

https://www.youtube.com/watch?v=UX11tVIkYUg&list=PL5Lsd0YA4OMFvX88T5xH93NqBALI7TENz&index=5

![API Gateway vs Load Balancer](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123185513.png)

## Reverse Proxy

![Reverse Proxy](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123191252.png)

## API Gateway

![API Gateway](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123191601.png)

## Ingress Controller (Kubernetes)

![Ingress Controller](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123191833.png)

## Kubernetes Cluster

![Kubernetes Cluster](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123192458.png)

## MultiCluster Configuration Under High Load.

![Kubernetes MultiCluster](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123192731.png)

# API Gateway vs Load Balancer: Which is Right for Your Application?

API gateways and load balancers are useful tools for building modern applications. While they have some functionality overlaps, they're distinct tools with different purposes and use cases. In this article, we'll discuss the differences between [API gateways](https://konghq.com/blog/learning-center/what-is-an-api-gateway) and load balancers, give examples of their implementations, and how to choose the right tool for your web application.

## What is a load balancer?

A load balancer is a component that distributes API request traffic across multiple servers. This improves system responsiveness and reduces failures by preventing overloading of individual resources.

When a request is made to a service (e.g., a website or video) the load balancer receives the request and decides which server will process and respond to it. This is demonstrated in the following image, where a load balancer sits between various clients and the backend server farm that will serve the requests.

![](https://kongwp.imgix.net/wp-content/uploads/2023/04/Screen-Shot-2023-04-25-at-10.37.20-AM.png?auto=compress%2Cformat)

### What are the benefits of a load balancer?

Load balancers can complete a variety of useful functions, including:

- As a layer of indirection between the client and server, a load balancer can shield the client from a direct connection to the server. This means that a server may change — or be removed from the farm — and the client won't know or be impacted.
- Load balancers can detect when a server is malfunctioning and remove them from rotation. They can also know which servers are currently fastest and which are saturated. Based on this info, the load balancer assigns more or fewer connections. This is a useful health and performance tracking function with practical benefits.
- Some load balancers can handle more complex tasks, such as HTTPS offloading. When doing so, the clients have a secure connection to the load balancer over HTTPS, while the load balancer opens an HTTP connection to the backend server.
- Some load balancers do more work still, such as caching and content-aware routing.

### Load balancing use cases

Now that we know what a load balancer does, we can think of situations where it might be useful. 

It turns out that any time we're creating a network service that will be supported by multiple servers, a load balancer is in order. Why? It helps with optimizing the utilization of our servers and gives us a measure of reliability if one of our servers fails or is taken out of rotation.

## What is an API gateway?

Check out our full article on [API gateways](https://konghq.com/blog/learning-center/what-is-an-api-gateway) for more details. But the TL;DR version is: an API is an interface that allows two computing resources to communicate with each other, and an API gateway is an intermediary between API consumers and APIs. 

### Why use an API gateway?

So why do we need an intermediary between API consumers and the APIs? The gateway applies useful higher-level functionality that makes our APIs more secure, observable, and reliable when they're deployed in a distributed manner (as they would be in a network). Examples of these functions include:

#### Traffic control

Caching and rate limiting are easily the two most common traffic control mechanisms a gateway can apply. 

- When **caching**, the gateway gives the API consumer a faster response, while shielding the backend servers from excessive, repetitive load. 
- With **rate limiting**, the gateway can prevent potential overuse or abuse from consumers by reducing the total number of requests a consumer can issue in a certain period of time. 

Other traffic control capabilities exist, such as request validation, and transformation, for example.

#### Authentication and authorization

The API gateway can additionally [authenticate and authorize the API consumer](https://konghq.com/blog/learning-center/api-gateway-authentication). 

Let's assume that an API is to be protected, and the consumer must offer credentials as a form of identification before consuming the API. The gateway can play this role and eliminate this burden that otherwise the various APIs will need to implement. This is important as we can't be sure each API is identifying and permitting consumers to access them consistently. 

With an API gateway, we formalize this process and make it uniformly applied on all APIs. This gives us a more consistent security posture with less risk.

Learn more about [Common API Authentication Methods](https://konghq.com/blog/engineering/common-api-authentication-methods)

#### Metrics and logging

As APIs are discrete units of functionality, they may be offered as [products](https://konghq.com/blog/enterprise/productizing-apis). API owners therefore would like to know how much their products are used, and by which users. An API gateway can collect metrics about the APIs, API latency, error rate, and even gather specific information about each request.

#### Load balancing

Yes, APIs can do _some_ of the work associated with load balancers, but they aren't a direct substitute for them. API gateways also direct traffic for various backend servers and apply active and passive health checks to determine when to take a server out of rotation. In the next section, we'll cover this in more depth and show how an API gateway can be used in combination with a load balancer.

### Common use cases for API gateway

The most common use case for an API gateway is to offer a layer of indirection between API consumers and a variety of APIs. 

This diagram illustrates this use case for a single API. In practice, an API gateway can direct traffic to tens or hundreds of APIs.

![](https://kongwp.imgix.net/wp-content/uploads/2023/04/Screen-Shot-2023-04-25-at-10.41.49-AM.png?auto=compress%2Cformat)

## API gateway and load balancer together

Now that we've covered the basics for a gateway and load balancer, let's see how we can use both of them together. The following diagrams show an evolution of our architecture as our requirements grow.

Let's assume we have an API, that API has consumers, and we're not using an API gateway or load balancer.

![](https://kongwp.imgix.net/wp-content/uploads/2023/04/Screen-Shot-2023-04-25-at-10.43.11-AM.png?auto=compress%2Cformat)

Next, we realize that our API consumers are growing, and our API service is getting overloaded. We need to spin up a couple of more instances of this API server, and we'll use a load balancer to help spread the load over them.

![](https://kongwp.imgix.net/wp-content/uploads/2023/04/Screen-Shot-2023-04-25-at-10.44.34-AM.png?auto=compress%2Cformat)

We introduced a load balancer, which in addition to spreading the load over our API endpoints, also does HTTPS offloading for us. But our clients are still growing. While we can add more API servers, we'd like to have caching that's API-aware and varies its cache accuracy by a few factors. We also want to limit how much each single client can make calls in a time period. Finally, we want to collect some more metrics on the API usage. We'll be introducing an API gateway. Just like our APIs, we'll make it highly available.

![](https://kongwp.imgix.net/wp-content/uploads/2023/04/Screen-Shot-2023-04-25-at-10.46.54-AM.png?auto=compress%2Cformat)

We introduced a pair of gateways that apply all the policies we need. Since those gateways are effectively services, we're putting a load balancer in front of them. We can add more gateways if we need — or apply maintenance on one at a time without disrupting the activity of our API clients. We also introduced a second set of API endpoints that the gateway will balance the load against and apply a different set of policies. This architecture should serve us well for the time being, and it takes advantage of both a load balancer and API gateway.

Load balancers and API gateway can improve the quality, security, performance and reliability of services. They work well together and are optimized for complementary purposes.

Note - when we have system under not much pressure (like 500 requests/hour) and we don't have complicated microarchitecture with a lot of different APIs reverse proxy + load balancer is enough. Otherwise we can use API Gateway to take benefit from e.g service discovery.
