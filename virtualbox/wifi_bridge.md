# Bridged vms on host wifi adapter

- IP6 doesn't seem to work right.
- Best to disable it in the vm

If you run docker in the vm, it will cause the vm to lose connectivity.

- In the vm, set a static IP4 address, outside the range of your DHCP
- Disable IP6 in vm.
- Set the vm's network adapter to promiscuous mode: Allow VMs
