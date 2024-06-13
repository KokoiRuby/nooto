:confused: **vs.?**

- Consistency: All nodes/members in distributed system see the same data at the same time.
- Consensus: The process of reach Consistency.
- Election: The impl of Consensus.



:confused: **Election Alg?**

- Bully (MangoDB) | Raft (etcd) | ZAB (Zookeeper)



:confused: **[Raft](https://thesecretlivesofdata.com/raft/)?** Membership: Leader| Follower | Candidate

1. All members start as Follower with a random timer.
2. Member sends votes to rest when timer timeout.
3. Member becomes Leader if received N/2+1 acks & send **heartbeats** to rest
4. Members receive **heartbeat** become Follower & reset timer.
5. If no heartbeat received within timeout = Leader Lost, it becomes Candidate and Term+1.

**Note: One vote per term only.**



![image.png | left | 352x137](https://gw.alipayobjects.com/mdn/rms_da499f/afts/img/A*eTJ3SZlSpsIAAAAAAAAAAABjARQnAQ)



:confused: **Raft Features?**

| Feature              | Desc                                                         |
| -------------------- | ------------------------------------------------------------ |
| Election safety      | At most one leader can be elected in a given term.           |
| Leader append-only   | A leader never overwrites or deletes entries in its log. It only appends new entries. |
| Log matching         | If two logs contain an entry with the same index and term, then the logs are identical in all entries up through the given index. |
| Leader completeness  | If a log entry is committed in a given term, that entry will be presented in the logs of the leaders for all higher-numbered terms. |
| State machine safety | If a server has applied a log entry at a given index to its state machine, no other server will ever apply a different log entry for the same index. |



:confused: **When re-election will be triggered?**

- Leader downtime
- Follower election timeout
- Network Parition
- etcdctl move-leader
- etcdctl member [add|delete]



:confused: **Quorum tolerance?** Q = (N/2) + 1

| Node# | Quorum | Tolerance |
| ----- | ------ | --------- |
| 3     | 2      | 1         |
| 5     | 3      | 2         |
| 7     | 4      | 3         |



:confused: **Why odd member?**

- Make sure uneven votes, always more than half.
- Avoid even votes → re-election → extra cost



:confused: **State Machines?**

- Log-based

  - Each server stores a log.
- Each log entry contains a command.
    - HB → log nil.
- The state machine executes commands in order.
  



:confused: **WAL Entry?**

| Field | Desc                                                     |
| ----- | -------------------------------------------------------- |
| type  | 0 as Normal, 1 as ConfChange (e.g. member [add\|delete]) |
| term  | Leader term.                                             |
| index | Entry pos.                                               |
| data  | Raft Request.                                            |



:confused: **Log replication？**

- Flow
  1. Client W → Leader Consensus Module
  2. Leader Consensus Module writes (entry) to local log.
  3. Leader sends **AppendEntries RPC** to Followers (write to local log as well).
     - **Infinite retries AppendEntries RPC if Followers fail to replicate.**
  4. Leader commits to State Machine if receive majority of ACK
  5. Follower commit to State Machine
  6. Leader → Client



![image.png | left | 321x179](https://gw.alipayobjects.com/mdn/rms_da499f/afts/img/A*OiwGTZnO2uMAAAAAAAAAAABjARQnAQ)



:confused: **Log Matching？**

- **Index**: to identify entry pos in log.
- **Last Applied Index**: The index of the last entry that has been applied to the state machine. 
- Once Followers receive **AppendEntries RPC**, it will compare the index/term/content btwlocal & the ones sent from Leader.
  - If Leader index/term is larger/newer, Follower will start to replicate log (behind).
  - If Leader index/term macthed, will compare content & sync-up.



![image.png | left | 318x233](https://gw.alipayobjects.com/mdn/rms_da499f/afts/img/A*Bn5lR6TAWEwAAAAAAAAAAABjARQnAQ)



:confused: **Log Inconsistency?**

- Missing, Uncommited, Both, Different term. To solve:
  1. Find last agreed entry.
  2. Delete all entries ahead in Followers.
  3. Replicate all entries onward in Leader to Follower.



![image-20221117215917742](http://sm.nsddd.top/smimage-20221117215917742.png)



:confused: **Log Compaction?**

- Redundant or obsolete log entries → Snaphost



![image.png | left | 396x242](https://gw.alipayobjects.com/mdn/rms_da499f/afts/img/A*77gySo2CTewAAAAAAAAAAABjARQnAQ)
