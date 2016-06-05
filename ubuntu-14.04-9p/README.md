## Login instructions:

```
Username: root
Password: <none>
```

## Notes:

This VM contains an Ubuntu 14.04 Core root filesystem with the Ubuntu 14.04 kernel.
The Root filesystem is not a block device, but a ZFS dataset with the root filesystem
mounted in the VM using 9pfs sharing. This means that they VM is not limited to a
fixed image size and is free to expand to the size of ZFS pool containing the
root filesystem dataset on the host. 
