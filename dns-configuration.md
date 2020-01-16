# DNS Configuration

By doing this you will be able to connect to your virtual machines by using the internal hostname. 
For example, if you just created a VM called `test-posgresql`, then you can connect to it by using its name 
(no matther the automatic IP it got from the internal DHCP).

Example of a SSH connection:

```
ssh test-posgresql
```

## Step 1

Configure a different network for KVM (using virt-manager, for example) with DHCP enabled and the name you want for 
your local domain. For example, if you want to use the internal network `10.0.0.0/24` and local domain `mynet`, 
you will have something like this:

![Virt Manager Network Configuration](/images/virt-manager-network-config.png)

Once you created your network, ensure your virtual machines uses it.

![VM Network Configuration](/images/vm-network-configuration.png)

## Step 2

Edit the systemd-resolved configuration file to work with the new private network you created in KVM.

```
sudo vim /etc/systemd/resolved.conf
```

Ensure you set the parameters `DNS` to the gateway of your private network (in the example above, 10.0.0.1) and `Domains` to the name of that network (i.e, mynet). You file should look like (you may leave the other options commented):

```
[Resolve]
DNS=10.0.0.1
Domains=mynet
```

## Step 3

Edit your KVM network to keep the domain as local only.

```
virsh net-edit mynet
```

Then add the option `localOnly='yes'` in the domain tag. It should look like:

```
    <domain name='mynet' localOnly='yes'/>
```

## Step 4

Restart the `systemd-resolved`, `libvirtd`, and `NetworkManager` services.

```
sudo systemctl restart systemd-resolved

sudo systemctl restart libvirtd

sudo systemctl restart NetworkManager

```

Note: you may restart your machine if you want.

## Step 5

Restart any virtual machine that is already running. After that, you will be able to connect to it using only its hostname.

**Important note**: what is used as hostname is the internal hostname of the virtual machine (the one you can see when you run the command `hostname`). If you just cloned a virtual machine, you need to change its hostname and eventually reboot the VM. For example:

```
$ sudo hostnamectl set-hostname test-barman
```

## Optional Step

If you want to change the network hostname of a fresh cloned VM without turning it on for the first time, you can do this by using `libguestfs-tools`.

```
$ sudo apt install libguestfs-tools
```

The example below shows the clone operation and the change of the hostname using `libguestfs-tools`.


### Create a new VM by cloning a base VM.

```
virt-clone --original bbd-base-vm --name test-postgresql --auto-clone
```

### Change the hostname of the new VM.

```
sudo virt-customize -a /var/lib/libvirt/images/test-postgresql.qcow2 --hostname test-postgresql
```

### Start the new VM.

```
virsh start test-postgresql
```