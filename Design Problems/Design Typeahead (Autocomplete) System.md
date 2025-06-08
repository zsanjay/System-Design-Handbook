
A typeahead system, often called autocomplete, is a feature in software programs that predicts the rest of a word or phrase that a user is typing. This functionality helps users to find and select from a pre-populated list of values as they type, reducing the amount of typing needed and making the input process faster and more efficient.

An everyday example of a typeahead system is Google's search box. As you begin typing your search query, Google provides a dropdown list of suggested search terms that match what you have typed so far. These suggestions are based on popular searches and the content indexed by Google's search engine. This allows you to see and select your intended search without having to type out the entire term, making the search process faster and more efficient.

When a user begins to type in the search box, the typeahead feature could suggest book titles, authors, or genres that match the typed characters. For instance, if the user types in "Har", the system could suggest "Harry Potter", "Harper Lee", etc. This kind of typeahead system can greatly enhance the user experience by providing instant feedback and helping users to formulate their search queries.

The search ranking is based on frequency of user inputs, updated daily. For example, if "Harry Potter" is searched 10 times and "Harper Lee" is searched 5 times, "Harp Lessons" is searched 2 times in the last 24 hours, then they should be ranked in that order.

In this problem, we will design a typeahead system for a large e-commerce site. The system will provide search suggestions to users as they type in the search box. The suggestions will be based on the most popular searches and the content indexed by the e-commerce site's search engine.

## Functional Requirements

- The system should return search suggestions as the user types in the search box.
- The system should return top 10 results for each query.
- The system should prioritize more relevant suggestions (e.g., based on popularity or other relevance metrics).
- The system should handle updates to the data, e.g., new entries added.
- Use a Trie to store the data.

## Non functional Requirements

Scale requirements:

- 100 million daily active users
- Each user makes 20 queries a day
- Search data should be updated daily

Other requirements:

- Availability: The system should be highly available and aim for near-zero downtime. The system should have error handling mechanisms to prevent crashes and ensure it can handle large amounts of traffic.
- Scalability: The system should support 100 million daily users.
- Latency: The system should provide search suggestions in real-time (within a few hundred milliseconds).
- Efficiency: The system should efficiently use memory and computational resources.
- Consistency: Since the system will be read-only for the clients, consistency is not a primary concern.

## Resource Estimation

Assumptions:

- DAU (Daily Active Users): 100 Million
- Average search queries/user/day: 20

It's worth noting that a new request is made every time a user types in a new character. We can see this by checking out Google's requests in Chrome Devtool's network tab:

![Google Search Autocomplete Requests](https://systemdesignschool.io/problems/3-typeahead/google-search-autocomplete-requests.png)![](https://systemdesignschool.io/search.svg)

Assuming the average number of characters of a query is 10, then the number of requests per query is 10 * average searches daily per user of 20 = 200 queries/user/day.

Estimations:

Read:

- QPS (Queries per second): 100M users * 200 queries / user / (24 * 3600) seconds = ~230k queries/sec.
- Assume 2x peak traffic, peak QPS is 230k *2 = 460k queries/sec.

Write:

Assume 10% of the queries are new, each query is 10 character and each character is 2 bytes, then each query adds 20 bytes. This 100MM * 200 queries * 10% * 20 bytes = 40GB. For a year, it's 14,600 GB = 14.6TB, a large but managable amount of storage.

Use the [resource estimator](https://systemdesignschool.io/resource-estimator?dau=100000000&read_write_ratio=10:1&write_operations=20&data_retention=12&data_per_write_request=20&precision_mode=true) to calculate.

### Performance Bottleneck

Given these numbers, the bottlenecks are likely to be in handling QPS and retrieval speed for providing suggestions in real-time.

## API Design

Our system can expose the following REST APIs:

```
GET /suggestions?q={search-term}
```

Response should include a list of suggested terms, ordered by relevance:

```
{
    "suggestions": ["suggestion1", "suggestion2", ..., "suggestionn"]
}
```

## High-Level Design

![Typeahead System Design Diagram](https://systemdesignschool.io/solutions/3-typeahead/typeahead-design-diagram.png)![](https://systemdesignschool.io/search.svg)

#### Read Path

1. The client sends an HTTP request to the `GET /suggestions` interface to start a query;
2. The load balancer distributes the request to a specific application server;
3. The application server queries the index stored in the cache;
4. The application server queries the database if data satisfying the query is not found in the cache;

#### Write Path

5. Access log from the application servers are aggregated.
6. 7. 8. Scheduled jobs kick off [batch processing pipelines](https://systemdesignschool.io/concept/batch-processing) that read the logs and use the aggregated data to update the index in the database and the cache.

## Detailed Design

### How to Implement the Index?

#### Approach 1: Using a Trie

A Trie (or Prefix Tree) is a tree-like data structure that is used to store a dynamic set or associative array where the keys are usually strings. All the descendants of a node have a common prefix of the string associated with that node, and the root is associated with the empty string. Feel free to learn more about [trie from a data structure perspective](https://algo.monster/problems/trie_intro).

In the context of the typeahead system, each node in the Trie could represent a character of a word. So, a path from the root to a node gives us a word in the dictionary. The end of a word is marked by an end of word flag, letting us know that a path from the root to this node corresponds to a complete, valid word. The time complexity to get to a node is `O(log(length of prefix))`.

The frequency of the words are stored in the node themselves. To find the top results, we can find all the nodes in the subtree and sort them by the frequency.

![Trie Frequency](https://systemdesignschool.io/solutions/3-typeahead/trie-frequency.png)![](https://systemdesignschool.io/search.svg)

![Trie Frequency](https://systemdesignschool.io/solutions/3-typeahead/trie-frequency-2.png)![](https://systemdesignschool.io/search.svg)

#### Approach 2: Using Inverted Indexes

An inverted index is a data structure used to create a [full-text search](https://systemdesignschool.io/concept/full-text-search-database). In an inverted index, there is a record for each term or word, which contains a list of documents that this term appears in. This approach is used by most full-text search frameworks such as Elasticsearch.

In the context of the typeahead system, we could store all the prefixes of a word, along with the word itself, in the inverted index. The search operation would then retrieve the list of words corresponding to a given prefix.

Here is how the Inverted Index would look:

```
{
    "c": ["car", "cat"],
    "ca": ["car", "cat"],
    "car": ["car"],
    "cat": ["cat"],
    "d": ["dog"],
    "do": ["dog"],
    "dog": ["dog"]
}
```

Each key in the index is a prefix, and the value is a list of words that have that prefix.

When searching for the prefix "ca", the system would look up "ca" in the index and retrieve the associated list, which is ["car", "cat"].

As you can see, the Trie and the Inverted Index provide the same results, but they store the data in different ways and the search operations work differently.

#### Approach 3: Using Predictive Machine Learning Models

Predictive models use machine learning algorithms to predict future outcomes based on historical data. In the context of the typeahead system, we could use a predictive model to suggest the most likely completions of a given prefix, based on the popularity of different completions in the past. This predictive nature is fundamentally how popular tools like ChatGPT work.

This approach could give us more relevant suggestions, but it would be more complex to implement and maintain. Moreover, the suggestions would only be as good as the quality and quantity of the historical data.

In this article, we will use the simple Trie approach in our design. As we will see in the detailed design, the optimized data storage is similar to inverted index.

### How to Use a Trie to Implement Typeahead

The basic implementation we explored earlier would work for a small amount of data. But as the data set gets large, it gets inefficient to find all children of a node and sort them. One common way to make algorithms faster is to trade space for time. We can pre-compute the results and store them in each node.

#### Pre-compute to make search faster

However, if the subtree is large, this can get inefficient quickly. A better way is to pre-process the data and store the top results in the node directly. We don't have to navigate down the subtree to find results, significantly improving speed. This comes at the cost of an increase in storage since the nodes are now storing more data.

![Trie Frequency](https://systemdesignschool.io/solutions/3-typeahead/trie-frequency-3.png)![](https://systemdesignschool.io/search.svg)

Here's the python implementation of this idea:

```
class TrieNode:
    def __init__(self):
        self.children = {}
        self.top_words = []

    def insert(self, word, frequency):
        # If word is already in top_words, update its frequency
        for i, (freq, wrd) in enumerate(self.top_words):
            if wrd == word:
                self.top_words[i] = (frequency, word)
                self.top_words.sort(key=lambda x: (-x[0], x[1]))
                if len(self.top_words) > 5:
                    self.top_words = self.top_words[:5]
                return

        # If word is not in top_words, add it
        self.top_words.append((frequency, word))
        self.top_words.sort(key=lambda x: (-x[0], x[1]))
        if len(self.top_words) > 5:
            self.top_words = self.top_words[:5]


class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word, frequency):
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
            node.insert(word, frequency)

    def typeahead(self, prefix):
        node = self.root
        for char in prefix:
            if char not in node.children:
                return []
            node = node.children[char]
        return [word for _, word in node.top_words]

trie = Trie()
trie.insert('apple', 5)
trie.insert('ape', 3)
trie.insert('apricot', 2)
trie.insert('banana', 6)
trie.insert('bandana', 1)
trie.insert('ban', 4)
trie.insert('app', 7)

print(trie.typeahead('ap'))  # ['app', 'apple', 'ape', 'apricot']
print(trie.typeahead('ban'))  # ['banana', 'ban', 'bandana']

```

The time complexity now is really `O(L)` where `L` is the length of the prefix.

#### Flatten the trie for storage

For a regular trie, we can [serialize it as a tree](https://algo.monster/problems/serializing_tree) by traversing it and writing each node with null nodes represented as a special character like '#'.

For our read-optimized prefix version of the trie though, the simplest way is to store it as a distributed hash table where every single prefix is the key and the value is the top k results.

```
{
    'c': [('cat', 50), ('car', 20)],
    'ca': [('cat', 50), ('car', 20)],
    'car': [('car', 20)],
    'cat': [('cat', 50)],
    'd': [('do', 100), ('dog', 50)],
    'do': [('do', 100), ('dog', 50)],
    'dog': [('dog', 50)],
}
```

Some people like to call this a ["Prefix Hash Tree"](https://people.eecs.berkeley.edu/~sylvia/papers/pht.pdf) and even reference the paper but it's not quite the same as the one described in the paper. The idea is quite simple, create an entry for each prefix. Storing it becomes much simpler due to the key-value nature of the format.

## Data Store

### Database Type

Because a "prefix hash table" is essentially a series of key-value pairs, we can simply store it in a [key-value database](https://systemdesignschool.io/concept/key-value-database) like DynamoDB, MongoDB or Redis.

### Data Schema

In the case of a type-ahead or autocomplete system built on Elasticsearch using an inverted index, the data schema would be fairly straightforward. These are usually keyword-based. For this reason, you primarily care about the terms themselves, and less about their relationships or other attributes. However, for the sake of being comprehensive, let's discuss a possible schema that includes some typically useful fields that can be used to rank keywords:

```
{
  "id": "unique_document_id",
  "term": "search_term",
  "popularity": "term_usage_frequency",
  "timestamp": "latest_usage_time"
}
```

Explanation of each field:

- id: Unique identifier for each document.
- term: This is the actual term that the user will search for. It is the field on which the inverted index would be created.
- popularity: This column refers to a count of how often the term is used. This could be useful in ranking autocomplete suggestions - more popular terms may be ranked higher in the type-ahead system.
- timestamp: This is the timestamp of the most recent usage of the term. This can be used to boost the relevance of terms that are seeing more recent usage.

### Database Partitioning

For the pre-computed prefix hash table, we can use [hash partition](https://systemdesignschool.io/concept/partitioning). The basic approach is to use a hash function to transform the key into a hash value, and then use that hash value to determine which partition (or shard) the key-value pair should be stored in.

### Database Replication

Key-value data stores like DynamoDB and MongoDB have replication built-in. We can configure the number of replicas to ensure high availability and data durability.

### Data Retention and Cleanup

We can set a retention policy for the Elasticsearch index, to automatically delete data that is older than a certain period. This would help us manage the size of the index, and ensure that the suggestions are based on recent data.

### How to Build the Index?

We can build the index offline, using a [batch processing](https://systemdesignschool.io/concept/batch-processing) system like Hadoop or Spark. The system would process the logs of the e-commerce site, extract the search queries, and build the index. The index would be stored in a distributed file system like HDFS or S3, and loaded into the Elasticsearch cluster.

For the simple requirement of this problem, we will use only frequency data, aggregated from the web servers' access logs.

In real-world applications, there are often multiple data sources. For example, [Pinterest uses users' pin data](https://evernote.com/blog/an-inside-look-at-type-ahead-search/) and [Facebook uses friends relationships](https://www.facebook.com/notes/10158791367817200/) .

## Caching

[Caching](https://systemdesignschool.io/fundamentals/caching) is an essential part of the design for a typeahead system due to its read-heavy nature. By storing frequently accessed data in a cache, we can reduce the latency and the load on the backend servers.

#### Client-side Caching

Client-side caching involves storing search results directly in the user's browser. This approach significantly reduces latency for repeated searches, as the results can be fetched from the cache instead of requiring a network call to the server. However, the cache size is limited by the storage capacity of the browser, and the cache is not shared between different browsers or devices.

#### Server-side Caching

Server-side caching involves using a distributed cache like Redis or Memcached to store search results on the servers. This approach can reduce the load on the database, as the results can be fetched from the cache instead of requiring a database query.

### What If We Want Real-time Typeahead Suggestions?

If we want real-time results, we need to update frequencies in real-time. Due to the heavy write operations, writing to disk would be way too slow. We will want to do this in memory. The creator of Redis has a blog post on [how to implement autocomplete in Redis](http://oldblog.antirez.com/post/autocomplete-with-redis.html). The idea is to use [sorted set](https://systemdesignschool.io/concept/redis-tutorial) to store the frequencies and use `ZINCRBY` command to update frequencies.

### How to Handle Typos

We can use Fuzzy Matching to handle typos. Fuzzy matching algorithms can be employed to find approximate matches for a given query, even if it contains typographical errors. Popular fuzzy matching algorithms include Levenshtein distance, [Damerau-Levenshtein](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance) distance, and [Jaro-Winkler distance](https://en.wikipedia.org/wiki/Jaro%E2%80%93Winkler_distance). These algorithms measure the similarity between strings based on the number of insertions, deletions, substitutions, or transpositions required to transform one string into another. By considering suggestions that are close to the original query, we can provide relevant results despite minor errors.

Let's consider a user query with a typo, such as "aple." To handle the typo, we can use a fuzzy matching algorithm like Levenshtein distance to find approximate matches. Compute the Levenshtein distance between "aple" and each indexed term. Find the terms with the smallest distance (e.g., within a certain threshold). Retrieve the corresponding entries from the index for those terms. For example, the system might find that "aple" is close to "apple" with a Levenshtein distance of 1. The suggested term is "apple."

Alternatively, you could incorporate a spell-checking algorithm such as the [SymSpell](https://github.com/wolfgarbe/SymSpell) or [Peter Norvig's spell correction algorithm](https://norvig.com/spell-correct.html). These algorithms usually involve creating a dictionary of known words, then for each input word, generating a set of candidate words that are a small edit distance away and selecting the most likely candidate based on some probability model.


### References

- https://systemdesignschool.io/problems/typeahead/solution
- https://www.youtube.com/watch?v=SG9CPplNGgo

  ![[document.pdf]]