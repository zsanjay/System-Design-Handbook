### Disk Storage, Data Indexing, And A Use Case For B-Trees

Database management systems (such as PostgreSQL) need to be able to access, read, and write massive amounts of data quickly. Unlike accessing data stored in memory (RAM), data on disk can be significantly more expensive to interact with (more on that below). In this article, we explore how data is stored on disk, why we use indexing to keep track of that data, and finally, what B-Trees are and how they enable us to speed up many common database queries.

# **How Data Is Stored On Disk**

First things first! Before diving into indexing, we need to understand how data is stored on disk. You can think of the hard drive in your computer as looking something like in _Figure 1_ below.

![](https://miro.medium.com/v2/resize:fit:371/1*XSr_pawtZqy9HP7wfzYPJA.png)

Figure 1: Elements of a disk— Wikipedia

Disks are made up of **tracks** (a circular ring, like the outer crust of a pie), **disk sectors** (a section of the disk, like a single slice of pie), a **track sector** or **block** (the intersection of a track and a sector, like the outer crust of a single slice of pie), and an **actuator arm.** A block is the smallest storage unit on a disk and is typically 512 bytes in size. The actuator arm has a magnetic head to read and write data stored underneath it. When you need to access a specific piece of data on disk, the disk may spin and the actuator arm may rotate so that the block where the piece of data is stored is positioned underneath the magnetic head.

Let’s say that we have a database table with 1,000 records in it that are stored on a disk, such as the one below.

![](https://miro.medium.com/v2/resize:fit:545/1*Y9yHmJq3H4YCKR062EWXPQ.png)

Figure 2: sample entries of our database table — Microsoft

Each block on disk may only be able to store a few of the 1,000 records in our table; let’s assume our records are stored in 250 blocks. If we needed to retrieve the record with ID = 938, and our machine didn’t know which block housed that particular record, it would have to search each of the 250 blocks until it found the record with ID = 938. Moving from one block to another would require the arm would rotate and the disk to spin, which could prove burdensome on our simple query.

Enter _indexing_.

# **Single And Multi-Level Indexing**

What if we had some metadata that the machine could look at first, like a road sign, that pointed to the block containing the record in question? Such an idea exists, and it is called an **index**. An index is a table whose rows point to the location of other blocks on disk. Using our example, the index table could have 250 rows pointing to the blocks which contain our 1,000 records. The index would still live on disk across multiple blocks, but much fewer blocks than the underlying data. Instead of searching through 250 blocks of data to find our record, the machine can search through the index to know where to go next.

Things get more interesting when our database gets large enough that a single index becomes too large. We now have many blocks on disk storing our single index alone and it can take the machine quite a long time to search every index block to find the location of one particular piece of data. What if we added yet another index whose rows pointed to the blocks of our first index? Now to find a piece of data the machine could scan through this second-level index to get the block of our first-level index which would then point to the block which contains our underlying data. Depending on the size of the database, we can add even more levels of indexes, as represented by _Figure 3_ below.

![](https://miro.medium.com/v2/resize:fit:480/1*6Vcu1I5riyTxqcdBuJOz0A.png)

Figure 3: multi-level indexing — TutorialsPoint

In this image, we can see that the Outer Index points to a block that contains multiple Inner Indexes, which points to yet another block that contains pieces of data. _Doesn’t this kind of look like a sideways tree?_

Enter _B-Trees_.

![](https://miro.medium.com/v2/resize:fit:551/1*nS6S8fXgTGeRPdJK7Q971Q.png)

Figure 4: a B-Tree?

Nope. Not that.

# **What Is A B-Tree?**

Let’s build up our mental model of a B-Tree by starting with a simple binary search tree, like this one:

![](https://miro.medium.com/v2/resize:fit:301/1*Ixy3u2uvWx07Ihfq7jspNw.png)

Figure 5: sample binary search tree — Tyler Holland

Each node has a value and is a parent to at most 2 child nodes. A child node with no children itself is a _leaf node_ (e.g. nodes with values 1, 5, and 14 above). All children to the left of a node have values less than that node and all children to the right of a node have values greater than that node.

On average, search, insert and delete operations in a binary search tree have a time complexity of O(log _n_), since, starting from the root node, each operation only requires traversing the height of the tree, and not every single node. However, if the tree is skewed (see _Figure 6_ below), our operations can have a worst-case complexity of O(_n_), because the height of the tree **is** _n._

![](https://miro.medium.com/v2/resize:fit:285/1*F7NPrvr8I_ed3g73IF56og.png)

Figure 6: right-skewed binary search tree

Let’s soup up our binary search tree by making it **self-balancing**. Every time we insert or delete a node, we shuffle the tree around so that its height is minimized (i.e. our tree always looks more like _Figure 5_ than _Figure 6_). This way we can guarantee that search, insert and delete operations are performed in O(log _n_) time. This comes at the expense of some added work every time we insert or delete a new node, but if we are performing a lot of search operations on the tree, this trade-off makes sense.

Let’s further soup up our binary search tree by removing the restriction that each node has at most 2 children, instead of allowing each node to have _M_ children, where _M_ is the order of the tree. Additionally, we’ll let each node have _multiple_ values in it, instead of just 1 as is the case with a binary search tree. Specifically, each node can have at most _M-1 keys_ in it. There are a couple of other properties we need to layer on: every non-leaf node has at least _M / 2_ children and the root node has at least two children (assuming it is not a leaf node).

Our binary search tree has evolved so much that we should probably stop calling it one. What we have constructed is a **B-Tree** and it looks something like this:

![](https://miro.medium.com/v2/resize:fit:365/1*10lSD76xy3c-lsTIUdKQjg.png)

Figure 7: a B-Tree of order 3 — Daisy Tang

_Figure 7_ represents an order-3 B-Tree, which means each node has at most 2 keys and at most 3 children. Notice how the root node **(6, 17)** has three subtrees: the one to the left of **6**, which contains values less than 6, the one in between **6** and **17**, which contains values between 6 and 17, and the one to the right of the **17**, which contains values above 17.

# **Database Indexing Using B-Trees**

B-Trees are particularly useful when data is stored on disk. Our multi-level index from _Figure 3_ can be implemented using a B-Tree, where each leaf node is a block on disk with multiple keys pointing to other blocks containing the underlying data. The parent nodes one level up from the leaf nodes are blocks that point to the first-level index. The keys within each node are kept in sorted order for sequential traversing before moving to the right child node to continue the search.

PostgreSQL allows for indexing using several different algorithms but the default index type is B-Tree. It easily handles queries including equality comparisons (using operators <, ≤, =, ≥, and >) and keywords such as _BETWEEN_, _IN_, _IS NULL_ / _IS NOT NULL_, and some uses of _LIKE_.

# Conclusion

We learned about how data is stored on disk and why searching lots of different disk blocks can result in an expensive query. We then covered how indexing can make the search process faster and, depending on the size of the database, might require a multi-level index. Finally, we saw how a B-Tree can be used to implement a multi-level index and that it is the default index type used by many database management systems, including PostgreSQL.

#### References

https://www.youtube.com/watch?v=aZjYr87r1b8

https://drive.google.com/file/d/1-tTAmQ_eGGrYpCIj7K3HVbP-Pogfoe5A/view?usp=drive_link

https://drive.google.com/file/d/1LPvAmK55nw1s0zwl0EgKpdsRd6SI9o2h/view?usp=drive_link
