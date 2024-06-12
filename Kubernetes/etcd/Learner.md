:confused: **Challenges?**

1. New Cluster member **overloads** Leader by "member add"



<img src="https://etcd.io/docs/v3.5/learning/img/server-learner-figure-01.png" alt="server-learner-figure-01" style="zoom:67%;" />



2. **Network Partition** (3-node cluster)



<img src="https://etcd.io/docs/v3.5/learning/img/server-learner-figure-02.png" alt="server-learner-figure-02" style="zoom:67%;" />



​	a. Leader isolation



<img src="https://etcd.io/docs/v3.5/learning/img/server-learner-figure-03.png" alt="server-learner-figure-03" style="zoom: 67%;" />



​	b. Cluster Split 3+1



![server-learner-figure-04](https://etcd.io/docs/v3.5/learning/img/server-learner-figure-04.png)



​	c. Cluster Split 2+2



![server-learner-figure-05](https://etcd.io/docs/v3.5/learning/img/server-learner-figure-05.png)



​	d. Quorum Lost



<img src="https://etcd.io/docs/v3.5/learning/img/server-learner-figure-06.png" alt="server-learner-figure-06" style="zoom:80%;" />



3. Cluster Misconfigurations



![server-learner-figure-08](https://etcd.io/docs/v3.5/learning/img/server-learner-figure-08.png)



![server-learner-figure-09](https://etcd.io/docs/v3.5/learning/img/server-learner-figure-09.png)



:smile: **++ Learner**

- No joining vote & qurom won't increment, only catch up Leader.
- Promote to Follower when it's finished.
- Allow status check only but R/W.



![server-learner-figure-10](https://etcd.io/docs/v3.5/learning/img/server-learner-figure-10.png)



![server-learner-figure-11](https://etcd.io/docs/v3.5/learning/img/server-learner-figure-11.png)



![server-learner-figure-13](https://etcd.io/docs/v3.5/learning/img/server-learner-figure-13.png)