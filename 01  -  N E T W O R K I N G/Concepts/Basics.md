## Ping

```bash
ping <TARGET>
```

Useful flags :

```bash
-4
	Use IPv4 only
-6
	Use IPv6 only
-i
	Set the interval time of set ping requests
-v
	Verbose output
```

## Traceroute

```bash
traceroute <TARGET>
```

Useful flags :

```bash
-4, -6
	Force IPv4 / IPv6
-i
	Specify the interface
-T
	Use TCP SYN for probes
```

## Whois

```bash
whois <DOMAIN>
```

Allows you to query who a domain name is registered to. A web version also exists : [who.is](https://who.is)
## Dig

**dig** is the most powerful DNS query tool.
Basic DNS query :

```bash
dig website.com
```
Specify the record type after the domain name :

```bash
dig website.com MX
```
```bash
dig website.com AAAA
```

The **+short** flag strips all extra all information and returns only the answer :

```bash
dig website.com +short
```
```bash
dig website.com MX +short
```

The **+trace** flag trace the entire DNS resolution chain from root server to the authoritative nameserver. Essential for diagnosing propagation and delegation issues :

```bash
dig website.com +trace
```
```bash
dig website +short +trace
```

Use the **@server** syntax to direct the query to a specific resolver :

```bash
dig @8.8.8.8 website.com
```
```bash
dig @1.1.1.1 website.com A
```

## IP Addresses and Subnets

Look at your network using :

```bash
ifconfig
```

```bash
ip address show
ip a s
```

There is two types of IP addresses :
- Public IP addresses
- Private IP addresses

## Rooting 

![[basics_rooting.svg]]

## Telnet

The TELNET (Teletype Network) protocol is a network protocol for remote terminal connection. A TELNET client allows you to connect to and communicate with a remote system and issue text commands. Although initially it was used for remote administration, we can use ```telnet``` to connect to any server listening on a TCP port number.

```bash
telnet <IP> <PORT>
```

To leave the telnet environment, use ```Ctrl```+```]```, then use the command ```quit``` :
```bash
telnet > quit
```

You can enter requests inside the telnet environment. For example. enter :

```bash
GET / HTTP/1.1
Host: <HOST>
```

Then press ```Enter``` and you'll receive the response. 

![[telnet_request.png]]

## Audit using Telnet

###### Enumeration

You can check the [[Network Services]] page to see more about **enumeration**.

###### Port scanning

Check the [nmap](https://nmap.org/) page to learn more.

Useful flags :

```bash
-p-
	Scan all ports
-sV
	Try to get the protocol version for each port to check the connection on this port
```

