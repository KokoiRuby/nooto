:confused: **What is local vol?**

- A mounted local storage dev.
- Can only be used as statically created PV.
- More durable & portable than hostPath since awarenss of volume node.
- Subject to node availablity.
- Not be able to be shared by multi-pods.

```yaml
apiVersion: v1
kind: Pod
# ....
spec:
# ...
  volumes:
  - name: my-volume
    local:
      path: /mnt/disks/data
```



:confused: **Flow?**

- 



:warning: **Challenge?**

- LV from same dev → IO Interference.
- If backend is a dedicted physical, won't be a problem.