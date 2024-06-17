:confused: **K8s networking model?**

- inter-pod access withuot NAT.
- inter-node access without NAT.
- Same viewpoint of Container IP either ext. or int.
  - in-pod containers share same network stack (same net ns) via localhost:port by [pause](https://www.devopsschool.com/blog/what-is-pause-container-in-kubernetes/)



:confused: **CNI [Plugin](https://github.com/containernetworking/plugins)?** CNI Impl.

- IPAM - IP Alloocation
- Main
  - `bridge`: Creates a bridge, adds the host and the container to it.
  - `ipvlan`: Adds an [ipvlan](https://www.kernel.org/doc/Documentation/networking/ipvlan.txt) interface in the container.
  - `loopback`: Set the state of loopback interface to up.
- Meta
  - `portmap`: An iptables-based portmapping plugin. Maps ports from the host's address space to the container.
  - `bandwidth`: Allows bandwidth-limiting through use of traffic control tbf (ingress/egress).
  - `firewall`: A firewall plugin which uses iptables or firewalld to add rules to allow traffic to/from the container.



:confused: **How does it work?**

- CR reads conf from CNI dir & loads plugin **exec** to setup container network via CNI (ADD/DEL).
- CR flags: `--cni-conf-dir` (`/etc/cni/net.d`) → `--cni-bin-dir` (`/opt/cni/bin`)



:construction_worker: **Plugin Guideline**

- CR needs to create a new net ns before calling any plugins.

- CR needs to decide containers belong to which net ns, which plugins need to be run.

- CR needs to load CNI conf & make sure which plugins must be run.

- CNI conf in JSON.

- CR needs to exec plugins in order.

  

- CR must revert exec after containers in end-of-life.

- CR allow concurrency network setup for different containers.

- ADD always followed with DEL

- ContainerID to identify

- Only one ADD to same network, same container, same nic.