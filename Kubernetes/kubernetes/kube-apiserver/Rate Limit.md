:confused: **Why?**

- Avoid overload & protect resource.
- Prevent abuse
- QoS



:confused: **Alg?**

- **Fixed Window Counter**: threshold in fixed windows, drop if exceeded, refresh in next window.



![Fixed Window Counter](https://www.codereliant.io/content/images/2023/07/1-1.png)



- **Sliding Window Counter**: split to multiple, if req > window.max, right shift.



![Sliding Window Counter](https://www.codereliant.io/content/images/2023/07/2-1.png)



- **Leaky Bucket**: const. rate out, drop if full.



![Leaky Bucket](https://www.codereliant.io/content/images/2023/07/4-1.png)



- **Token Bucket**: token const. in, drop if token full. req given token to proc.



![Token Bucket](https://www.codereliant.io/content/images/2023/07/3-1.png)



:cry: **Limitation?**

- Course-grained, not friendly to multi-user & different scenarios.
- Sinlge queue, shared bucket.
- Unfair, starve req at tail.
- No prio control.



:smile: **API Priority & Fairness**

- Fined-grained to categorize & isolate in-coming.

- FlowSchema → (Distinguisher) → (Flow with) PriorityLevelConfiguration → QueueSet

  - <u>Shuffle Sharding</u> alg to pick one Q to cache.
  - <u>Fair Queuing alg</u> to pick one Q to pop oldest to proc.

  ```bash
  # ls all prio
  $ kubectl get --raw /debug/api_priority_and_fairness/dump_priority_levels
  
  # ls all q
  $ kubectl get --raw /debug/api_priority_and_fairness/dump_queues
  
  # ls all req in q
  $ kubectl get --raw /debug/api_priority_and_fairness/dump_requests
  
  # metrics
  $ kubectl get --raw /metrics 
  ```

| PriorityLevel (Default) | Req                                                          |
| ----------------------- | ------------------------------------------------------------ |
| system                  | system:nodes → kubelet                                       |
| leader-election         | system:[kube-controller-manager\|kube-scheduler] → ep/cm/lease |
| workload-high           | built-in controller                                          |
| workload-low            | sa                                                           |
| global-default          | others like interactive kubectl                              |
| exempt                  | system:masters                                               |
| catch-all               | 5 concurrency only, rejected with HTTP 429                   |



:confused: **Flags?**

| Node#                            | Default | 1000~3000 | >3000 |
| -------------------------------- | ------- | --------- | ----- |
| `--max-requests-inflight`        | 400     | 1500      | 3000  |
| `--max-mutating-requests-flight` | 200     | 500       | 1000  |



