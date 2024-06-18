:confused: **FS perf?**

- Besides, mnt storage, FS will also affect container's perf.
- IOPS: OverlayFS > DeviceMapper.



:confused: **What is CSI?**

- A standard interface for container orchestrator to interact with various storage systems.



:confused: **Plugins?**

- **in-tree**: built into the k8s core codebase, released alongside k8s itself.
- **out-of-tree FlexVolume**: allow storage vendors to develop their volume plugins outside of.
- **out-of-tree** (CSI): preferred method since v1.15 :smile: CSI pluigin → (RPC) Driver as Impl → Storage Systems.



:confused: **Driver Deployment?**

| Module               | Desc                                |
| -------------------- | ----------------------------------- |
| external-attacher    | Attach vol to node.                 |
| external-provisioner | Create vol in storage system.       |
| external-resizer     | Resize vol.                         |
| external-snapshotter | Take snapshot for vol.              |
| node-driver-register | Reg CSI Driver on node.             |
| CSI driver           | Impl CSI & talks to storage system. |



![container-storage-interface_diagram1](https://525875.fs1.hubspotusercontent-na1.net/hub/525875/hubfs/container-storage-interface_diagram1.png?quality=high&width=960&name=container-storage-interface_diagram1.png)



:confused: **Take a look?** Openstack Cinder

- Controller Plugin
  - csi-attacher
  - csi-provisioner
  - csi-snapshotter
  - cinder-csi-plugin - attach/provision/snapshot/resize
  - livenessprobe
- Node Plugin
  - node-driver-registrar
  - cinder-csi-plugin - mnt/umnt
  - livenessprobe

```bash
$ kubectl get csidrivers.storage.k8s.io cinder.csi.openstack.org -o yaml
```

```yaml
spec:
  # require attach vol to node before mount.
  attachRequired: true
  # policy for how k8s handles fsGroup when mount vol.
  fsGroupPolicy: ReadWriteOnceWithFSType
  # k8s pass additional pod info to CSI driver during mnt.
  podInfoOnMount: true
  # CSI driver does not require re-publishing volumes after node or driver restarts.
  requiresRepublish: false
  # driver does not report storage capacity information.
  # k8s does not use storage capacity information for scheduling decisions
  storageCapacity: false
  # supports persistent volumes that outlive individual pods and can be reused.
  # supports ephemeral volumes that are tied to the lifecycle of individual pods.
  volumeLifecycleModes:
  - Persistent
  - Ephemeral
```

```bash
# per node
$ kubectl get csinodes.storage.k8s.io n200-vpod1-8986-controlplane-hqh9l -o yaml
```

```yaml
spec:
  drivers:
  # the number of volumes that the CSI driver can manage on this node.
  # if exceeded, the cluster will not be able to schedule new pods that require volumes managed by that CSI driver on that particular node.
  - allocatable:
      count: 25
    name: cinder.csi.openstack.org
    nodeID: c00b3746-e582-4956-ac2e-6243f477079b
    topologyKeys:
    - topology.cinder.csi.openstack.org/zone
```

```bash
$ kubectl get csistoragecapacities.storage.k8s.io
```

```bash
$ kubectl get deploy -n kube-system -o yaml openstack-cinder-csi-controllerplugin | grep image
$ kubectl get ds -n kube-system -o yaml openstack-cinder-csi-nodeplugin | grep image

# hostpath
/var/lib/kubelet/plugins/cinder.csi.openstack.org/csi.sock
/var/lib/kubelet/plugins_registry/cinder.csi.openstack.org-reg.sock
```

```bash
# pv on node
/var/lib/kubelet/plugins/kubernetes.io/csi/cinder.csi.openstack.org

# dev on node
$ lsblk
```



:confused: **Interaction?**

- kube-controller-manager
  - PV Controller
    - Claim Worker listening to PVC
    - Volume Worker listening to PV
    - **Invoke CSI Plugin.**
  - AD Controller
    - Listening to VA
    - Attach/dettach vol to/from node，update `node.Status.VolumesAttached`
- kubelet
  - Volume Manager
    - mnt node vol to pod/container.
      - /var/lib/kubelet
    - **Invoke CSI Plugin.**