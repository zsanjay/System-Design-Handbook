
[Sharding](https://planetscale.com/docs/sharding/sharding-quickstart) and partitioning are techniques to divide and scale large databases. Sharding distributes data across multiple servers, while partitioning splits tables within one server.

## [Database sharding and partitioning](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#database-sharding-and-partitioning)

Partitioning and sharding are two common ways to improve performance, manageability, and availability of larger databases. You need to understand the differences between the two solutions in order to determine the most appropriate approach for your database architecture.

### [What is sharding?](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#what-is-sharding)

Sharding, also known as horizontal partitioning, is a database partition approach that divides the database schema and distributes them across multiple instances or servers into smaller parts that are faster and easier to manage. When a database is sharded, a replica of the schema is created. This is then used to divide data to be stored in a shard based on a shard key. To make this possible, a special logic or identifier called a "shard key" is used to determine which specific instance or server holds the data to query.

### [How does sharding work?](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#how-does-sharding-work)

To gain a deeper understanding of how sharding operates, and how you can use it, let's consider a scenario to illustrate its effectiveness. Imagine a social media platform with millions of users worldwide. In this case, you can implement sharding based on geographical regions. For example, users from North America would have their data stored in instance 1, while users from Europe would be allocated to instance 2, and so forth.

These code snippets demonstrate a basic implementation of database sharding based on geographical regions in MySQL. The `users` table holds user data, while the `user_regions` table maps each region to a specific database instance. This allows for distributing user data across multiple database instances based on their respective regions.

```
-- Create the shard mapping table for users

-- Create the shard mapping table for user regions

CREATE TABLE user_regions (
  region VARCHAR(255) NOT NULL,
  instance_id INT NOT NULL,
  PRIMARY KEY (region, instance_id)
);

INSERT INTO user_regions (region, instance_id) VALUES ('North America', 1), ('Europe', 2), ('Asia', 3);
```

Here is the query that retrieves the user record for the `username` “johndoe” from the database instance responsible for storing users in the North America region. This query utilizes database sharding, which splits the user database tables into multiple instances.

```
-- Create a function to get the instance ID based on the username
DELIMITER $$

CREATE FUNCTION get_user_instance_id(username VARCHAR(255)) RETURNS INT
BEGIN
  DECLARE region VARCHAR(255);

  SELECT region INTO region FROM users WHERE username = @username;

  RETURN (SELECT instance_id FROM user_regions WHERE region = @region);
END $$

DELIMITER;

SELECT * FROM users WHERE username = 'johndoe';
```

By geographically dividing the data, sharding allows for localized access and efficient management of user information. This approach proves particularly beneficial when it comes to optimizing performance for user interactions. Imagine a user in Europe trying to retrieve their profile information. Instead of traversing through the entire database, the system can use a shard key to quickly pinpoint the specific shard (instance) where the data is located, leading to faster response times and a better user experience.

However, it's important to consider certain factors to ensure fair distribution of data across instances. The varying user populations across different regions should be taken into account. For instance, North America may have a significantly larger user base compared to other regions. To address this, a more intelligent sharding strategy can be used, such as a combination of geographical and demographic factors. This way, the distribution of data can be more balanced and reflective of the user distribution across the regions.

### [What are the advantages of using sharding?](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#what-are-the-advantages-of-using-sharding)

Some benefits of sharding include:

- Improved response time
- Maintenance tasks, like backups, take less time to complete
- Schema migrations complete faster
- Increased read/write throughput
- Increased storage capacity
- Improved availability
- Outages are more isolated and less impactful

To learn more about the benefits of sharding, check out our [Benefits of sharding blog post](https://planetscale.com/blog/three-surprising-benefits-of-sharding-a-mysql-database).

### [What are the disadvantages of using sharding?](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#what-are-the-disadvantages-of-using-sharding)

Sharding brings the complexity of managing tables that are distributed across multiple servers. It can be difficult to manage database queries. As the data grows, merging shards can become more complicated to handle. Using the wrong sharding architecture can slow down performance. Be sure to choose a sharding technique that allows a balanced data distribution across all shards.

## [What is partitioning?](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#what-is-partitioning)

Partitioning is just a general term referring to the process of dividing tables in a database instance into smaller sub-tables or partitions. These partitions can be accessed and managed separately to enhance performance, maintainability, and availability of the database.

The remainder of this article will cover partitioning.

### [When to use partitioning on a database](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#when-to-use-partitioning-on-a-database)

Querying a record in a database that has millions of records can be costly. To optimize the queries, database partitioning can help reduce the query response and resources. Here is an example of how you can use database partitioning to make the query faster.

```
-- Create the 'users' table with partitioning
CREATE TABLE users (
  id INT NOT NULL AUTO_INCREMENT, -- Unique ID for each user
  username VARCHAR(255) NOT NULL, -- User's username
  email VARCHAR(255) NOT NULL, -- User's email address
  password VARCHAR(255) NOT NULL, -- User's password
  PRIMARY KEY (id)
)
```

The provided code shows how to partition a database table based on the `id`column. The `PARTITION BY RANGE` clause is used to divide a table into multiple partitions. In this case, there are three partitions defined. There is partition `p_0`, `p_1`, and `p_2`.

```
PARTITION BY RANGE (id) (
  PARTITION p_0 VALUES LESS THAN (150000),
  PARTITION p_1 VALUES LESS THAN (250000),
  PARTITION p_2 VALUES LESS THAN (MAXVALUE)
);
```

Insert user data into the users table:

```
INSERT INTO users (username, email, password) VALUES
  ('johndoe', 'john.doe@example.com', 'password123'),
  ('janedoe', 'jane.doe@example.com', 'password456');
```

Now that the partitions are created, you can retrieve user data from the appropriate partition based on the ID range.

```
-- Example query to retrieve users with IDs less than 150,000
SELECT * FROM users PARTITION (p_0);

-- Example query to retrieve users with IDs between 150,000 and 250,000
SELECT * FROM users PARTITION (p_1);

-- Example query to retrieve users with IDs greater than 250,000
SELECT * FROM users PARTITION (p_2);
```

### [What are the advantages of database partitioning?](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#what-are-the-advantages-of-database-partitioning)

Partitioning a database allows you to distribute data across multiple physical or logical storage units called partitions. By dividing the data, you can improve query performance by reducing the amount of data that needs to be scanned or accessed. Database partitioning adds efficiency to querying the database. It also makes maintenance operations easier. When a database is partitioned, you can target a specific partition to query a record, rather than traversing through the entire dataset. For example, if you partition a database by date, queries that only need to access data from the last month can be executed much faster than if they had to access all the data in the database.

Accessing a database in parts can give security control. If you are storing some confidential information in the database, you can allow a certain group of users to access only partitions that don't have confidential information.

### [What are the disadvantages of database partitioning?](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#what-are-the-disadvantages-of-database-partitioning)

One of the disadvantages of database partitioning is the complexity it brings. Partitioning can streamline certain maintenance tasks, but it can also complicate other aspects. Complexity can increase the chances for errors. For instance, the management of backups and recovery procedures can become more difficult to handle when you have multiple partitions. It can also lead to a false sense of security: If you're not careful, having multiple partitions could lead to a data loss disaster. Juggling partitions can lead to wasted space, and it may be unnecessary for the average user.

## [What are the differences between sharding and partitioning?](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#what-are-the-differences-between-sharding-and-partitioning)

While sharding and partitioning share the common goal of dividing a large database into smaller ones, they have different approaches to achieve this. When sharding a database, the data is distributed across multiple servers, resulting in new tables spread across these servers. On the other hand, partitioning involves splitting tables within the same database instance. Sharding is referred to as horizontal scaling, and it makes it easier to scale as you can increase the number of machines to handle user traffic as it increases. Partitioning splits based on the column value(s). All columns should be retained when partitioned – just different rows will be in different tables. It is also easier to manage data with partitioning, as all partitions are in one database instance.

## [Conclusion](https://planetscale.com/blog/sharding-vs-partitioning-whats-the-difference#conclusion)

In conclusion, both sharding and partitioning are powerful techniques that enable scaling and efficient data management in large databases. By understanding their differences and considering factors such as data distribution, performance optimization, and manageability, you can choose the most appropriate approach for your specific database architecture. Implementing sharding or partitioning can significantly enhance the performance and scalability of your database, allowing it to handle increasing user traffic and serve millions of requests effectively.
