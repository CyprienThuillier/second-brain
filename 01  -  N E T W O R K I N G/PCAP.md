**pcap** is an API for capturing network traffic.

## Reading

You can read a ```.pcap``` file using **tcpdump** :

```bash
tcpdump -r <filename>.pcap
```

The ```-r``` option reads saved **pcap** files instead of listening on an interface. 

You can also use tshark ([[Wireshark]]) to read these files and apply filters :
For example, this command will filter DNS queries by dns.qry.name :

```bash
tshark -r <filename>.pcap -T fields -e dns.qry.name
```
