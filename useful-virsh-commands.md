# Useful Virtsh Commands

The list of commands below are useful when dealing with several VMs, clones, and snapshots. One important fact about snapshots is that it will keep the old date/time, so you may want to reboot the machine to fix the time.


## Base commands


### Clone a VM

```
virt-clone --original <base-vm-name> --name <new-vm-name> --auto-clone
```

### Start/ Stop/ Reset a VM

```
virsh start <vm-name>
virsh stop <vm-name>
virsh reset <vm-name>

```

### List snapshots of a VM

```
virsh snapshot-list <vm-name> --name
```

### Create a snapshot of a VM

```
virsh snapshot-create-as --domain <vm-name> --name <snapshot-name>
```

### Revert a snapshot of a VM


```
virsh snapshot-revert <vm-name> <snapshot-name>
```

## Complex examples (combining commands/ scripts)

### Revert to the first snapshot of a VM

```
VM='test-postgresql'
SNAPSHOT=$(virsh snapshot-list $VM --name | head -1)
virsh snapshot-revert $VM $SNAPSHOT

```

### Revert snapshots and reset group of VMs

```
VMS=(
  "test-k8s-master"
  "test-k8s-node1"
  "test-k8s-node2"
  "test-k8s-node3"
  "test-k8s-node4"
)

for vm in "${VMS[@]}"; do
  snapshot=$(virsh snapshot-list $vm --name)
  virsh snapshot-revert $vm $snapshot
  virsh reset $vm
done
```

### Another version of restoring snapshot and restart VMs

```
#/bin/bash

SNAPSHOT="before-core"

VMS=(
  "test-rhel7" 
  "test-rhel8" 
  "test-centos7" 
  "test-centos8"
)


for vm in "${VMS[@]}"; do
  echo "[-] Reverting snapshot \"$SNAPSHOT\" of VM \"$vm\"..."
  virsh snapshot-revert $vm $SNAPSHOT > /dev/null 2>&1 &
done

for vm in "${VMS[@]}"; do
  echo "[+] Restarting VM \"$vm\"..."
  virsh reset $vm > /dev/null 2>&1
done

# Restore all VMs in parallel.
# parallel --jobs ${#VMS[@]} virsh snapshot-revert {} $SNAPSHOT ::: ${VMS[@]}

# Restart all VMs in parallel.
# parallel --jobs ${#VMS[@]} virsh reset {} ::: ${VMS[@]}

exit 0
```
