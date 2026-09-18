IPv4 can support a maximum of 4 billion devices. One solution to address depletion is Network Address Translation (NAT) :

![[nat.svg]]

Instead of assigning a rare public IPv4 address to every individual device, the router acts as a middleman. When a local device sends a request, the router swaps the device's private IP with its own public IP and assigns a unique **port number** to track the request. When the external server replies to that port, the router checks its translation table and forwards the data back to the correct original device.