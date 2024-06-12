:confused: **vs.?**

- Consistency: All nodes/members in distributed system see the same data at the same time.
- Consensus: The process of reach Consistency.
- Election: The impl of Consensus.



:confused: **Election Alg?**

- Bully (MangoDB) | Raft (etcd) | ZAB (Zookeeper)



:confused: **[Raft](https://thesecretlivesofdata.com/raft/)?** Membership: Leader | Follower | Candidate

1. All members start as Follower with a random timer.
2. Member sends votes to rest when timer timeout.
3. Member becomes Leader if received N/2+1 acks & send heartbeats to rest
4. Members receive heartbeat become Follower & reset timer.
5. If no heartbeat received within timeout = Leader Lost, it becomes Candidate and Term+1.

**Note: One vote per term only.**



![image.png | left | 352x137](https://gw.alipayobjects.com/mdn/rms_da499f/afts/img/A*eTJ3SZlSpsIAAAAAAAAAAABjARQnAQ)



:confused: **Why odd member?**

- Make sure uneven votes, more than half + less than half
- Avoid same votes → re-election.



:confused: **State Machines?**

- Log-based

  - Each server stores a log.

  - Each log entry contains a command.

  - The state machine executes commands in order.

    

:confused: **Log replication？**

- Flow
  1. Client W → Leader Consensus Module
  2. Leader Consensus Module writes (entry) to local log.
  3. Leader sends **AppendEntries RPC** to Followers (write to local log as well).
  4. Leader commits to State Machine if receive majority of ACK
  5. Leader → Client



![image.png | left | 321x179](https://gw.alipayobjects.com/mdn/rms_da499f/afts/img/A*OiwGTZnO2uMAAAAAAAAAAABjARQnAQ)



:confused: **Log Matching？**

- Index: to identify entry pos in log.
- Last Applied Index: The index of the last entry that has been applied to the state machine. 
- Once Followers receive **AppendEntries RPC**, it will compare the index/term/content btwlocal & the ones sent from Leader.
  - If Leader index/term is larger/newer, Follower will start to replicate log (behind).
  - If Leader index/term macthed, will compare content & sync-up.



![image.png | left | 318x233](https://gw.alipayobjects.com/mdn/rms_da499f/afts/img/A*Bn5lR6TAWEwAAAAAAAAAAABjARQnAQ)



:confused: **Log Compaction?**

- Redundant or obsolete log entries → Snaphost



![image.png | left | 396x242](https://gw.alipayobjects.com/mdn/rms_da499f/afts/img/A*77gySo2CTewAAAAAAAAAAABjARQnAQ)



:confused: **Learner?**

- 