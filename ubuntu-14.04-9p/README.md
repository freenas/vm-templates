## Login instructions:

```
Username: root
Password: <none>
```

## Notes:

This VM contains Ubuntu 14.04 Core root filesystem and Ubuntu 14.04 kernel.
Root filesystem is not a block device, but a ZFS dataset with root filesystem
mounted in the VM using 9p sharing. Thanks to that, the VM is not limited by its
fixed image size and is free to expand to the size of ZFS pool containing
root filesystem dataset on the host. 
