### [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

- Scratch space.
- LC sticks to pod.
- `emptyDir.medium` → "Memory" → tmpfs → counted in container mem usage ← Cgroup.
- UC: cache

```bash
# loc
/var/lib/kubelet/pods/<pod-uid>/volumes/kubernetes.io~empty-dir/<volume-name>
```

```yaml
apiVersion: v1
kind: Pod
# ...
spec:
# ...
  volumes:
  - name: cache-vol1
    emptyDir:
      sizeLimit: 500Mi
    # medium: Memory
  - name: cache-vol2
    emptyDir: {}
```

### [hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)

- Mnt a file/dir from node FS to pod.
- UC: needs to fetch all pod log on node; access /proc for stats.
- [type](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath-volume-types)

```bash
# pod log stdout
/var/log/pods/<namespace>_<pod-name>_<pod-uid>
```

```yaml
apiVersion: v1
kind: Pod
# ...
spec:
# ...
  volumes:
  - name: host-vol
    hostPath:
      path: /data/foo
      type: Directory
```

:warning:

- Content would change if pod is scheduled to another node.
- Scheduling won't take hostPath resource into consideration.
- Leftover on node if pod is deleted.

