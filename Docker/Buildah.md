:confused: What is [Buildah](https://buildah.io/)?

- A tool to facilitate building OCI container images.



:smile: **vs. docker build?**

- Lighter, no dockerd needed
- Smaller size
- Improved security
- **Support single layer creation** → layers commit → image



:confused: [Install](https://github.com/containers/buildah/blob/main/install.md)?



:confused: [Tutorial](https://github.com/containers/buildah/tree/main/docs/tutorials)?

Intro

```bash
# into rootless mode
$ buildah unshare bash
```

```bash
# chk image & working container (wc)
$ buildah images
$ buildah containers

# build a wc from a image
$ container=$(buildah from fedora) && echo $container
$ buildah run $container bash

# install inside wc
$ buildah run $container -- dnf -y install java
$ buildah run $container java
```

```bash
# from scratch
$ newcontainer=$(buildah from scratch) && echo $newcontainer

# overlay mnt point, considered root fs of container
$ export newcontainer
$ buildah unshare
$ scratchmnt=$(buildah mount $newcontainer) && echo $scratchmnt

# install
$ dnf install --installroot $scratchmnt bash coreutils --setopt install_weak_deps=false -y

# into
$ buildah run $newcontainer sh

# craete a file
$ cat > runecho.sh << EOF
#!/usr/bin/env bash
for i in `seq 0 9`;
do
	echo "This is a new container from ipbabble [" $i "]"
done
EOF

# perm
$ chmod +x runecho.sh

# run
$ buildah run $newcontainer /usr/bin/runecho.sh

# test with podman
$ sudo apt-get install podman
$ buildah copy $newcontainer ./runecho.sh /usr/bin
$ buildah config --cmd /usr/bin/runecho.sh $newcontainer
$ buildah commit $newcontainer newimage

# run
$ podman run --rm newimage
```

```bash
# clean
$ buildah delete fedora-working-container working-container
$ buildah rmi <imaged>
```

```bash
# build from Dockerfile
$ buildah build -t name .
```

Reg & Repo (TODO)

On-build (TODO)