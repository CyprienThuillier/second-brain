# KVM
Connection with the .xml file

Create a virtual "Switch" into the PC. The VMs will be connected on this switch.
```bash
<bridge name='virbr0'>
```

Define the IP address of this switch. Its the path for the VMs.
```bash
<ip address='192.168.122.1'>
```

Allow the virtual DHCP server. This server give an IP address in the range set in the file (e.g. 192.168.122.2 and 192.168.122.254)
```bash
<dhcp>
```

NAT for *Network Address Translation*. It allows the VMs to use the internet connection of the host PC to navigate on the web.
```bash
<forward mode='nat'>
```

