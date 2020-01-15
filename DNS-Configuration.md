# DNS Configuration

By doing this you will be able to connect to your virtual machines by using the internal hostname. 
For example, if you just created a VM called `test-posgresql`, then you can connect to it by using its name 
(no matther the automatic IP it got from the internal DHCP).

Example of a SSH connection:

```
$ ssh test-posgresql
```

## Step 1

Configure a different network for KVM (using virt-manager, for example) with DHCP enabled and the name you want for 
your local domain. For example, if you want to use the internal network `10.0.0.0/24` and local domain `mynet`, 
you will have something like this:

