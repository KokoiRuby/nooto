:confused: **Recall 12-Factor?**

- Proc: stateless & share nothing.
- Backend: for persistent data.
- No session sticky: shell be kept in cache DB.
- Src → Exec at Build phase.



:confused: **What is Dockerfile?**

- A txt file that contains a series of **instructions** on how to build a Docker image.



:confused: **Build Context?**

- Dir where `docker build` is being executed.
- Ctxt will be passed to dockerd.
  - Keep what's necessary otherwise long transfer, too many to build → large image



:confused: **Build Cache?**

- Docker will check each instruction if had any cache, re-use if matched.
- Layer ID per instruction, to determine.
  - string   → ENV | cmd like apt-get
  - checksum → ADD/COPY
- :cry: If cache invalidation, all layers above will become invalid.
- :smile: not-subject-to-change put to lower.



:confused: **[Multi-stage](https://docs.docker.com/build/building/multi-stage/) build?**

- Reduce image size
- Copy dep from previous build stage.



:confused: **[tini](https://docs.docker.com/reference/cli/docker/container/run/#init)?**

- container init proc to capture SIGTERM for graceful shutdown.

- handle child proc exit to aovid zombie (no parent).

  



:bookmark_tabs: [Cheatsheet](https://docs.docker.com/build/building/packaging/)

```dockerfile
# base image, recommend alpine
FROM [--platform=<platform>] <image> [@<digest>] [AS <name>]

# to identify & categorize
# docker images -f label=k1="v1"
LABEL k1="v1" k2="v2" ...

# must &&，otherwise it will be cached → failed to install package
RUN apt-get update && apt-get install .

# arg only, leave main to ENTRYPOINT
CMD ["param1", "param2" ...]
CMD ["exeutable", "param1", "param2" ...]

# port
EXPOSE <port> [<port>/<protocol>]

# env
ENV k=v

# add src file to dst
# use curl | wget && untar instead of url
ADD [--chown=<user>:<group>] <src>... <dest>

# copy src file to dst
# easy, no untar, friendly to multi-stage
COPY [--chown=<user>:<group>] <src>... <dest>

# main
ENTRYPOINT ["exeutable", "param1", "param2" ...]
ENTRYPOINT ["exeutable"]

# dir as vol to mount
# /var/lib/docker/volumes/<containerid>/_data
VOLUME ["<path>"]

# user & group
USER <user>[:<group>]

# cd
WORKDIR /path/to/workdir
```



:confused: **Image Management?**

```bash
$ docker save/load
$ docker tag <id> <reg>/<repo>/<name>:<tag>
$ docker push <repo>/<name>
```



## [Best Practice](https://docs.docker.com/build/building/best-practices/)

- Use multi-stage builds
- Choose the right base image
  - [Docker Official Images](https://hub.docker.com/search?image_filter=official&_gl=1*3ztb66*_gcl_au*Mzk1MDk1MjAyLjE3MTI2NzEyMzc.*_ga*MTA4MzEwNjIyNC4xNzEyNjcxMjM3*_ga_XJWPQMJYHQ*MTcxODEwMjIxNC4zNS4xLjE3MTgxMDIyODcuNjAuMC4w) | [Verified Publisher](https://hub.docker.com/search?image_filter=store) | [Docker-Sponsored Open Source](https://hub.docker.com/search?image_filter=open_source)
- Exclude with .dockerignore
- No unncessary packages
- Sort multi-line args in alphabet
- Only RUN/COPY/ADD will create new layer, multi RUN use `&&`
- One file per COPY
- [tini](https://github.com/krallin/tini) as init for multi-proc container

