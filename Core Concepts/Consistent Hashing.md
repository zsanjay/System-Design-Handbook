
Consistent hashing is a technique used in distributed systems to distribute data across a cluster of servers, minimizing redistribution when adding or removing servers. It involves arranging data and servers in a circular space, or “hash ring,” and finding the nearest server clockwise from the hashed data point. This approach, often used in systems like Redis and DynamoDB, ensures efficient data distribution and load balancing.


Let's build up our intuition via a motivating example.

Imagine you're designing a ticketing system like TicketMaster. Initially, your system is simple:

- One database storing all event data
- Clients making requests to fetch event information
- Everything works smoothly at first

![[20250410131132.png]]

But success brings challenges. As your platform grows popular and hosts more events, a single database can no longer handle the load. You need to distribute your data across multiple databases – a process called sharding.

![[20250410131222.png]]

The question we need to answer is: **How do we know which events to store on which database instance?**

### First Attempt: Simple Modulo Hashing

The most straightforward approach to distribute data across multiple databases is modulo hashing.

1. First, we take the event ID and run it through a hash function, which converts it into a number
2. Then, we take that number and perform the modulo operation (%) with the number of databases
3. The result tells us which database should store that event


![[20250410130943.png]]

In code, it looks like this:

```python
database_id = hash(event_id) % number_of_databases
```

For a concrete example with 3 databases:

```
Event #1234 → hash(1234) % 3 = 1 → Database 1
Event #5678 → hash(5678) % 3 = 0 → Database 0
Event #9012 → hash(9012) % 3 = 2 → Database 2
```

Great! This works well, until you run into a few problems.

The first problem comes when you want to add a fourth database instance. To do so, you naively think that all you need to do is run the modulo operation with 4 instead of 3.

```python
database_id = hash(event_id) % 4
```

You change the code, and push to production but then your database activity goes through the roof! Not just for the fourth database instance either, but for all of them.

What happened was the change in the hash function did not only impact data that should be stored on the new instance, but it changed which database instance almost _every_ event was stored on.

![[20250410131737.png]]

For example, event #1234 used to map to database 1, but now, hash(1234) % 4 = 0 so that data instead needs to be moved to database 0.

This means the data needs to be moved from database 1 to database 0. This isn't an isolated case - most of your data needs to be redistributed across all database instances, causing massive unnecessary data movement. This causes huge spikes in database load and means users are either unable to access data or they experience slow response times.

Let's look at another problem with simple modulo hashing.

Imagine a database went down. This could be due to anything from a hardware failure to a software bug. In any case, we need to remove it from the pool of databases and redistribute the data across the remaining instances until we can pull a new one online.

Our hash function now changes from database_id = hash(event_id) % 3 to database_id = hash(event_id) % 2 and we run into the exact same redistribution problem we had before.

![[20250410132001.png]]

Clearly simple modulo hashing isn't cutting it. Enter, **Consistent Hashing**.

### Consistent Hashing

Consistent hashing is a technique that solves the problem of data redistribution when adding or removing a instance in a distributed system. The key insight is to arrange both our data and our databases in a circular space, often called a "hash ring."

Here's how it works:

1. We first create a hash ring with a fixed number of points. To keep it simple, let's say 100.
2. We then evenly distribute our data across the hash ring. In the case where we have 4 databases, then we could put them at points 0, 25, 50, and 75.
3. In order to know which database an event should be stored on, we first hash the event ID like we did before, but instead of using modulo, we just find the hash value on the ring and then move clockwise until we find a database instance.

![[20250410132213.png]]


```
In reality, a hash ring usually has a hash space of 0 to 2^32 - 1 not 0-100, but the concept is the same.
```

How did this solve our problem? Let's look at what happens when we add or remove a database:

**Adding a Database (Database 5)** Let's say we add a fifth database at position 90 on our ring. Now:

- Only events that hash to positions between 75 and 90 need to move.
- These events were previously going to DB1 (at position 0)
- All other events stay exactly where they are!

![[20250410132507.png]]

Whereas before almost all data needed to be redistributed, with consistent hashing, we're only moving about 30% of the events that were on DB1, or roughly 15% of all our events.

**Removing a Database** Similarly, if Database 2 (at position 25) fails:

- Only events that were mapped to Database 2 need to move
- These events will now map to Database 3 (at position 50)
- Everything else stays put

![[20250410132713.png]]

#### Virtual Nodes

We've made good progress, but there is still just one problem. In our example above where we removed database 2, we had to move all events that were stored on database 2 to database 3. Now database 3 has 2x the load of database 1 and database 4. We'd much prefer if we could spread the load more evenly so database 3 wasn't overloaded.

The solution is to use what are called "virtual nodes". Instead of putting each database at just one point on the ring, we put it at multiple points by hashing different variations of the database name.

![[20250410133035.png]]


For example, instead of just hashing "DB1" to get position 0, we hash "DB1-vn1", "DB1-vn2", "DB1-vn3", etc., which might give us positions 15, 25, 40 and so on. We do this for each database, which results in the virtual nodes being naturally intermixed around the ring.

Now when Database 2 fails:

- The events that were mapped to "DB2-vn1" will be redistributed to Database 1
- The events that were mapped to "DB2-vn2" will go to Database 3
- The events that were mapped to "DB2-vn3" will go to Database 4
- And so on...

This means the load from the failed database gets distributed much more evenly across all remaining databases instead of overwhelming just one neighbor. The more virtual nodes you use per database, the more evenly distributed the load becomes.

## Consistent Hashing in the Real World

While our example focused on scaling a database, note that consistent hashing applies to any scenarios where you need to distribute data across a cluster of servers. This cluster could be databases, sure, but they could also be caches, message brokers, or even just a set of application servers.

We see consistent hashing used in many heavily relied on, scaled, systems. For example:

1. [Redis](https://www.hellointerview.com/learn/system-design/deep-dives/redis) Cluster: Uses consistent hashing to distribute keys across nodes
2. [Apache Cassandra](https://www.hellointerview.com/learn/system-design/deep-dives/cassandra): Uses consistent hashing to distribute data across the ring
3. [Amazon's DynamoDB](https://www.hellointerview.com/learn/system-design/deep-dives/dynamodb): Uses consistent hashing under the hood
4. [Content Delivery Networks (CDNs)](https://en.wikipedia.org/wiki/Content_delivery_network): Use consistent hashing to determine which edge server should cache specific content

## When to use Consistent Hashing in an Interview

Most modern distributed systems handle sharding and data distribution for you. When designing a system using Redis, DynamoDB, or Cassandra, you typically just need to mention that these systems use consistent hashing under the hood to handle scaling.

However, consistent hashing becomes a crucial topic in infrastructure-focused interviews where you're asked to design distributed systems from scratch. Here are the common scenarios:

1. Design a distributed database
2. Design a distributed cache
3. Design a distributed message broker

In these deep infrastructure interviews, you should be prepared to explain several key concepts. You'll need to articulate why consistent hashing provides advantages over simple modulo-based sharding approaches. You should also be ready to discuss how virtual nodes help achieve better load distribution across the system. Additionally, be prepared to explain strategies for handling node failures and additions to the cluster. Finally, you'll want to demonstrate your understanding of how to address hot spots and implement effective data rebalancing strategies.

The key is recognizing when to go deep versus when to simply acknowledge that existing solutions handle this complexity for you. Most system design interviews fall into the latter category!

## Conclusion

Consistent hashing is one of those algorithms that revolutionized distributed systems by solving a seemingly simple problem: how to distribute data across servers while minimizing redistribution when the number of servers changes.

While the implementation details can get complex, the core concept is beautifully simple - arrange everything in a circle and walk clockwise. This elegant solution is now built into many of the distributed systems we use daily, from Redis to DynamoDB.

In your next system design interview, remember: you usually don't need to implement consistent hashing yourself. Just know when it's being used under the hood, and save the deep dive for those infrastructure-heavy questions where it really matters.








