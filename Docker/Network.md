:confused: **Run a container?**

- client docker → (REST) → dockerd → (gRPC) → containerd → (shim) runc



:confused: **[Network](https://docs.docker.com/network/)?**

| Driver      | Description                                                  |
| :---------- | :----------------------------------------------------------- |
| `bridge`    | The default network driver.                                  |
| `host`      | Remove network isolation between the container and the Docker host. |
| `none`      | Completely isolate a container from the host and other containers. |
| `overlay`   | Overlay networks connect multiple Docker daemons together.   |
| `ipvlan`    | IPvlan networks provide full control over both IPv4 and IPv6 addressing. |
| `macvlan`   | Assign a MAC address to a container.                         |
| `container` | Attach a container to another container's networking stack.  |

- bridge

  - veth pair: one end to container eth0, the other end to docker0

  ```bash
  # iptables-save
  
  # src in brdige network
  # not going to docker0 nic
  # SNAT
  -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
  ```

  ```bash
  # start container
  $ docker run -d --name nginx -p 8080:80 nginx
  
  # not from docker0
  # dst port to 8080
  # DNAT to 172.17.0.2:80
  -A DOCKER ! -i docker0 -p tcp -m tcp --dport 8080 -j DNAT --to-destination 172.17.0.2:80
  ```

- underlay

  - container CIDR in node CIDR → each container

- overlay

  - L2 in L3 VXLAN
  - Outer MAC/IP + Outer UDP + VXLAN Header + Inner MAC/IP