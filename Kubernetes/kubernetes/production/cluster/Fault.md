:confused: **What K8s not aware of?**

- sys-level daemon, hardware, kernel, CR
- [node-problem-detector](https://github.com/kubernetes/node-problem-detector)
  - NodeCondition to set status → permanent fault
  - Event to nofity → tmp fault
  - Customized under: `/etc/kubernetes/addons/node-problem-detector`
  - Controller needed to listens obj & prevent pod scheduling to problematic node.



:confused: **Troubleshooting on node?**

- Privilege pod for ssh as jumpstart shares hostnework, pid and mnt to host FS.

- Logs

  ```bash
  $ journalctl -afu kubelet -S "yyyy-mm-dd HH:MM:SS"
  
  $ kubectl logs -f <pod> -c <container>
  $ kubectl logs -f <pod> -c <container> --previous
  $ kubectl logs -f <pod> --all-containers
  
  $ kubectl exec -it <pod> -- tail -f /path/to/log
  ```

  
