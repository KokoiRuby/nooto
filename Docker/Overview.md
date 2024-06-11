:confused: **Arch Evol?** 

Monolithic

:smile: Simple, easy to deploy, test and scale in horizontal.

:cry: Heavy, slow build, recursive issues due to changes, hard to CI/CD.

↓

Mircroservices

:smile: Decoupled, clear division, independent deploy, easy to CI/CD.

:cry: Complexity ↑ due to msg-based or RPC btw proc, partial-failure handle, partitioned DB.



:confused: **Mircroservice-able?**

- Code reivew which can be split from business logic.

- Dep by static code analysis tool.
- Concurrency & Memory requirement.



:confused: **What is Container?**

- **In-box proc by Linux Namespace + Cgroup + Union FS.**
- Security + Isolation + Portability + Quotable
- [vs. VM](https://www.docker.com/resources/what-container/)
  - Resource utiliztion ↑
  - Fast to start
  - Consistency Runtime
  - Easy to CI/CD, migrate, maintain, scale



:confused: **What is [Docker](https://www.docker.com/)?**

- A platform to streamline container building, shipping & running。



:confused: **[Docker Arch](https://docs.docker.com/get-started/overview/#docker-architecture)?**

- client → dockerd → registries



:confused: **Pros vs. Cons?**

- :smile: light, portable, fast start/stop, utility, commnuity
- :cry: shared-kernel security, limited orchestration



:confused: **[Install](https://docs.docker.com/engine/install/)?**



:confused: **Quickstart?**

```bash
# run centos container
$ docker run -itd --name <name> centos 

# list container
$ docker ps
$ docker ps -a

# show local images
$ docker images

# chk
$ docker inspect 

# exec cmd in
$ docker exec -it <id> ls

# into
PID=`docker inspect --format "{{.State.Pid}}" <container>`
$ nsenter --target $PID --mount --uts --ipc --net --pid

# copy into/from
$ docker cp file <id>:/file-to-path
$ docker cp <id>:/file-to-path file

# build & push to local reg
$ docker build -t cncamp/httpserver:${tag}
$ docker push cncamp/httpserver:v1.0

# run
$ docker run -d cncamp/httpserver:v1.0
```

