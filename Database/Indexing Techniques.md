
1) Hash Index :
	They are basic hashmaps in memory. (NOT disk - as we need random hashmap access instantly and not seek it) 
	- Thus limited by memory space, expensive
	- Fast reads and writes 
	- Bad for range queries 
	- To add durability, we add WAL - write to disk as log (easy as pointer is always at the end) and then write to the hashmap.

2) B-Trees :
	Self balancing tree, entirely kept on disk 
	- Hence NO limit on size 
	- Advantage having lesser concern over durability 
	- Slower reads than “Hash index” technique as 
	- disk vs memory speed.
	- Reads are fast - O(LogN) to retrieve, but slow writes 
	- Good for range queries (as stored in tree structure) 
	- Durability can be ensured by NOT updating tree in place, rather updating the pages/block separately file and then updating the reference later on. 

3) LSM Tree + SSTable 
- In-memory tree table (AVL, red-black), self balanced 
- Little faster writes compared to B-Trees, but slower than hash index 
- Slower reads speeds than B-Trees, as it has to check all SStables (Also slower range queries) - Durability handled by WAL - which can be replayed later
	- Space concern of LSM trees as in-memory, However we have SStables which are outputted to disk 
	- Hence No of keys limit not a concern (to store by space) 
	- SSTable - sorted, immutable 
	- Finding key - can go on taking time - it is first checked in tree (memtable) and only then across SSTable backwards from creation time.
	- To delete, its implemented by tombstone value - indicating deleted. It is added to the latest SSTable entry.
	
LSM Optimizations 
Use Sparse index 
- Also known as index for SSTable

- Take “certain” keys of SSTable and create a sparse index with corresponding stored memory location. 
	- Can use binary search to find in-between keys

Bloom Filters 
- Can vaguely tell, if certain “key” present in the SSTable 

Compaction 
- Merges SSTables in background - O(N) operation 
- Cons is extra CPU processing in background could be bad