## Use case and why you should consider caching?

![Caching](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123172123.png)

- Reduced Latency
- Reduced Load on the database
- Reduced Network Costs.

# Types of Caches by Level

- Client Side Caching
- CDN Caching 
- Web Server Caching
- Database Caching
- Application Caching 

### **Summary of Cache Types by Level**

| **Cache Level**                    | **Description**                                           | **Typical Use Case**                            |
| ---------------------------------- | --------------------------------------------------------- | ----------------------------------------------- |
| **L1, L2, L3 CPU Cache**           | Embedded in the CPU, used for fast data access            | Storing frequently accessed instructions/data   |
| **DRAM Cache**                     | Main memory cache, often managed by OS                    | Storing data to speed up access from RAM        |
| **Page Cache (OS Level)**          | Caching of disk data in memory (OS-managed)               | Speeding up access to frequently accessed files |
| **Disk Cache (HDD/SSD)**           | Internal storage cache (e.g., in SSDs or HDDs)            | Reducing latency in data access from storage    |
| **Application Cache**              | Caching at the application level (e.g., Redis, Memcached) | Storing session data, query results, etc.       |
| **Content Delivery Network (CDN)** | Distributed caching at the edge, near the end user        | Serving static content quickly to users         |
| **Browser Cache**                  | Caching static assets in the browser                      | Speeding up page loads for returning users      |
| **Distributed Cache**              | Caching across multiple servers or nodes (e.g., Redis)    | Caching data for large-scale applications       |



# Types of Caches by design

1. Global Cache

![Global Cache](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123173637.png)

3. Distributed Cache

![Distributed Cache](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123173802.png)

![Distributed Cache 2](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123173920.png)

## Caching Strategies and Invalidation


1. Write-Around Cache (Cache-aside , Lazy-Loading) - Reactive Approach

![Write-Around Cache ](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123174837.png)


The **Cache-aside** pattern, also known as **Lazy-Loading**, is one of the most common caching strategies. In this pattern, the application is responsible for managing the cache. It checks the cache for data and, if the data is not present (a **cache miss**), it fetches the data from the underlying data store (e.g., a database) and populates the cache.
#### How it works:

- When an application needs data, it first checks the cache.
- If the data is **not found** (a cache miss), the application loads the data from the **main database** or other data source.
- The application then stores the result in the cache for future requests.
- Cache entries typically expire after a certain time period (TTL — Time-to-Live).

#### Pros:

- Simple to implement.
- The cache is updated lazily, only when data is requested.
- No need for additional infrastructure if the cache is small or infrequently accessed.
- Cache stays slim and contains data it really needs.

#### Cons:

- **Cache misses** can lead to performance degradation because the cache is only updated on demand.
- **Stale data** issues can occur if data in the database changes but the cache isn't refreshed.
- Cache gets filled only after a cache miss, meaning 3 trips.

#### Use cases:

- Caching results of expensive database queries.
- Caching external API responses.

#### Example:

- In an e-commerce application, the product details page might check the cache first for product information. If the cache doesn’t have the information, the app fetches it from the database, stores it in the cache, and returns the response.


2. Write-Through Cache - Proactive approach

![Write-Through Cache ](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123175805.png)

A **Write-through** cache strategy ensures that whenever data is written to the cache, it is also immediately written to the underlying data store (e.g., database). This ensures that the cache and the data store are always synchronized.

#### How it works:

- When an application writes data to the cache, it writes that data to the **database** simultaneously.
- This prevents cache and database inconsistencies because the data store is always up-to-date.

#### Pros:

- Strong consistency between cache and data store.
- No need for complex cache invalidation strategies because the cache always reflects the latest data.

#### Cons:

- Slower write operations because writing to both the cache and database is required.
- May lead to additional I/O load on the underlying data store, especially in high-volume systems.
- Infrequently-requested data is also written to the cache, resulting in a larger cache.

#### Use cases:

- Caching frequently updated data that needs to be kept in sync with the database.
- Scenarios where high consistency is a priority over performance.

#### Example:

- In a banking application, when a user makes a deposit, both the cache and the database are updated simultaneously to reflect the new balance.

3. Write-Back (Write-behind) Cache

![Write-Back Cache ](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241123181016.png)


In a **Write-behind** or **Write-back** cache strategy, data is written to the cache first, and then asynchronously written to the underlying data store after a delay. The cache can also store data for later persistence to the database.

#### How it works:

- The application writes data to the cache, and then the cache asynchronously writes data to the database (sometimes in batches).
- The data in the cache is eventually written back to the database after a specified interval or when a batch threshold is reached.

#### Pros:

- Reduces load on the database by writing data asynchronously.
- Write performance is faster because the database write is delayed.
- It is useful for reducing database contention and writes.

#### Cons:

- Risk of **data loss** in case of cache failure (since data may not be immediately persisted to the database).
- Potential for **inconsistency** between the cache and the database, especially if the system crashes before the write-back happens.

#### Use cases:

- Caching data that doesn't need immediate persistence.
- Use cases where high write throughput is needed and eventual consistency is acceptable.

#### Example:

- A caching layer for an e-commerce website where user actions (like adding products to the cart) are written to the cache first, and only periodically written to the database.

#### 4.  **Read-through Cache**

The **Read-through** cache pattern is similar to Cache-aside, but with one key difference: the cache itself is responsible for loading data from the underlying data store when a cache miss occurs. The application does not need to load data directly.

#### How it works:

- The application asks the cache for data.
- If the data is not present in the cache (cache miss), the cache itself fetches the data from the underlying data store.
- The cache stores the result for future requests.

#### Pros:

- Simple for the application, since it only interacts with the cache.
- No need for manual management of cache population; it’s done automatically.

#### Cons:

- Like Cache-aside, cache misses can lead to slower performance since the cache is responsible for fetching data.
- Requires cache management to handle when data should be evicted or refreshed.

#### Use cases:

- Systems where cache population should be completely handled by the caching layer.
- Frequently queried data that can be cached automatically.

#### Example:

- A news website where the cache stores the latest headlines. When a user requests the headlines, if they are not cached, the cache fetches them from the news database.

### Eviction Policies

Cache eviction is critical in managing limited memory. Eviction policies define how the cache handles old or unused data when it runs out of space. Several popular eviction strategies include:

#### **Least Recently Used (LRU)**

- **Description**: The LRU eviction policy removes the least recently used items from the cache when space is needed. This ensures that frequently accessed data stays in the cache.
- **Use case**: Best for caches where recency (last time) of access is an important factor in determining which data should be retained.

#### **Least Frequently Used (LFU)**

- **Description**: LFU evicts items that are accessed the least over a given period. This ensures that the cache keeps the most frequently accessed items.
- **Use case**: Best for scenarios where access frequency, not recency, determines the importance of cached data.

#### **First-In-First-Out (FIFO)**

- **Description**: FIFO evicts the oldest items in the cache first, regardless of how frequently or recently they have been accessed.
- **Use case**: Simple to implement, but not as efficient for high-performance systems compared to LRU or LFU.

#### **Time-to-Live (TTL) / Expiration-based Eviction**

- **Description**: Each cached item is given a time-to-live (TTL). When the TTL expires, the item is evicted from the cache. This ensures that stale data doesn’t remain in the cache.
- **Use case**: Best for caching data that changes over time and needs to be periodically refreshed.

---

### **6. Distributed Cache**

In a **Distributed Cache** design, the cache is not stored on a single machine, but is instead spread across multiple machines or nodes, making it horizontally scalable.

#### How it works:

- The cache is shared across multiple systems or servers in a distributed manner. This allows for scaling the cache as demand increases.
- The data is partitioned and stored across various nodes, and a **distributed caching system** (e.g., **Redis**, **Memcached**) ensures that all nodes are in sync.

#### Pros:

- High scalability, capable of handling massive data volumes.
- Provides fault tolerance and redundancy through replication and partitioning.

#### Cons:

- More complex to manage compared to single-node caches.
- Requires a network connection for accessing the cache, which can add some latency.

#### Use cases:

- Large-scale web applications, e-commerce sites, and services with high traffic, where a distributed cache is required to support many users.

---

### **7. Multi-Level Cache**

In a **Multi-Level Cache** design, data is cached at different levels, usually starting with a local cache (like in-memory cache) followed by a distributed cache and then an external cache (like a CDN). Each level of cache stores a portion of the data to improve access times.

#### How it works:

- **Level 1**: Local in-memory cache (e.g., Redis, Memcached, or in-process cache).
- **Level 2**: Distributed cache shared among multiple servers.
- **Level 3**: External cache, such as a CDN, storing static content.
- Each level handles different types of data and traffic, improving response times at various scales.

#### Pros:

- Highly scalable and efficient, since it caches data at multiple layers, each optimized for specific access patterns.
- Reduces load on the primary data store by having multiple cache layers.

#### Cons:

- Complex to implement and manage, especially with multiple cache levels.
- Cache invalidation and consistency across levels can be challenging.

#### Use cases:

- High-performance systems with diverse data types (e.g., web applications with both dynamic and static content).

---

### **Conclusion**

Each **cache design** serves different use cases, balancing trade-offs between complexity, consistency, and performance. The choice of cache design depends on the requirements of the application, such as **write patterns**, **read patterns**, **data consistency**, and **scalability**.

- **Cache-aside** and **Read-through** caches are common when the application needs control over cache population.
- **Write-through** and **Write-behind** caches ensure data consistency with the underlying store.
- **Eviction policies** ensure cache memory is managed


### How Redis removes expired keys?


> Redis keys are expired in two ways: a **passive way**, and an **active way**.
> 
> A key is passively expired simply when some client tries to access it, and the key is found to be timed out.
> 
> Of course this is not enough as there are expired keys that will never be accessed again. These keys should be expired anyway, so periodically Redis tests a few keys at random among keys with an expire set. All the keys that are already expired are deleted from the keyspace.
> 
> Specifically this is what Redis does 10 times per second:
> 
> Test 20 random keys from the set of keys with an associated expire. Delete all the keys found expired. If more than 25% of keys were expired, start again from step 1. This is a trivial probabilistic algorithm, basically the assumption is that our sample is representative of the whole key space, and we continue to expire until the percentage of keys that are likely to be expired is under 25%
> 
> This means that at any given moment the maximum amount of keys already expired that are using memory is at max equal to max amount of write operations per second divided by 4.

- [How Redis expires keys](https://redis.io/commands/expire#how-redis-expires-keys)
- [How expires are handled in the replication link and AOF file](https://redis.io/commands/expire#how-expires-are-handled-in-the-replication-link-and-aof-file)
