
Distributed consensus is a fundamental primitive for constructing fault-tolerant, strongly-consistent distributed systems. Though many distributed consensus algorithms have been proposed, just two dominate production systems: Paxos, the traditional, famously subtle, algorithm; and Raft, a more recent algorithm positioned as a more understandable alternative to Paxos.
In this paper, we consider the question of which algorithm, Paxos or Raft, is the better solution to distributed consensus? We analyse both to determine exactly how they differ by describing a simplified Paxos algorithm using Raft's terminology and pragmatic abstractions. 
We find that both Paxos and Raft take a very similar approach to distributed consensus, differing only in their approach to leader election. Most notably, Raft only allows servers with up-to-date logs to become leaders, whereas Paxos allows any server to be leader provided it then updates its log to ensure it is up-to-date. 
Raft's approach is surprisingly efficient given its simplicity as, unlike Paxos, it does not require log entries to be exchanged during leader election. We surmise that much of the understandability of Raft comes from the paper's clear presentation rather than being fundamental to the underlying algorithm being presented.


### What makes Raft so successful?

1.  Pragmatic presentation
2. Simplification
3. Underlying algorithm

![Distributed Consensus](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241124155527.png)

### Configuration

There are 2 clients and 3 servers in the above image and among these 3 server one is leader and 2 are followers. 


### Normal operation

1. The leader appends each operation to its log and asks the other servers to do the same.
2. Once a majority of servers have acknowledged, the leader can mark the operation as committed.

![Normal operation](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241124160129.png)


### Handling leader failure.

- If a follower believes that the leader has failed then it becomes a candidate and asks the other servers to vote for it.
- If a candidate receives votes from a majority of servers then becomes a leader.

![Handling leader failure](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241124160439.png)


### The key difference

- Both algorithms guarantee that a new leader's log will contain all previously committed enteries.
- Paxos followers will vote for any candidate, but the votes must include any log entries that the candidate is missing.
- Raft followers will only vote for a candidate if the candidate's log is at least as up-to-date.

![The key difference](https://github.com/zsanjay/Obsidian-Notes/blob/main/assets%2Fimages%2F20241124160953.png)

### Which is the best algorithm?

- **Both algorithms take the same approach to distributed consensus.**
- No significant difference in understandability between the two algorithms.
- Raft's leader election is surprisingly efficient for its simplicity.


#### References

https://www.youtube.com/watch?v=JQss0uQUc6o



