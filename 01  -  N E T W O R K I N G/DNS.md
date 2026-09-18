[[DNS Audit]]

**Port:** 53
# Introduction



![[DN.png]]
#### TLD (Top-Level Domain) :

- **gTDL** (Generic TLD) : tell the domain name's purpose (e.g. *.com* for commercial purpose, *.gov* for government).
- **ccTDL** (Country Code TLD) : for a location (e.g. *.fr* for France).
#### Second-Level Domain :

A string limited to 63 characters (a-z, 0-9 and hyphens) placed before the TLD (e.g. *google.com* where *google* is the second-level domain).
#### Subdomain :

A string similar to the Second-Level Domain limited to 63 characters (a-z, 0-9 and hyphens) placed on the left-hand side of the Second-Level Domain. It's possible to use multiple subdomains split with periods but the length is limited to 253 characters. 

## DNS Record Types (Commons)

#### A Record

Resolve to IPv4 addresses (e.g. 104.26.10.229)
#### AAAA Record

Resolve to IPv6 addresses (e.g. 2606:4700:20::681a:be5)
#### CNAME Record

These records resolve to another domain name, for example, TryHackMe's online shop has the subdomain name [store.tryhackme.com](http://store.tryhackme.com) which returns a CNAME record [shops.shopify.com (opens in new tab)](http://shops.shopify.com). Another request would then be made to [shops.shopify.com (opens in new tab)](http://shops.shopify.com) to work out the IP address.
#### MX Record

These records resolve to the address of the servers that handle the email for the domain you are querying, for example an MX record response for [tryhackme.com](http://tryhackme.com) would look something like [alt1.aspmx.l.google.com (opens in new tab)](http://alt1.aspmx.l.google.com). These records also come with a priority flag. This tells the client in which order to try the servers, this is perfect for if the main server goes down and email needs to be sent to a backup server.
#### TXT Record

TXT records are free text fields where any text-based data can be stored. TXT records have multiple uses, but some common ones can be to list servers that have the authority to send an email on behalf of the domain (this can help in the battle against spam and spoofed email). They can also be used to verify ownership of the domain name when signing up for third party services. Here are a few examples:

- ```_acme-challenge.example.com TXT "token_value_here"```
- ````@ TXT "v=spf1 ip4:192.0.2.0/24 include:_spf.google.com include:amazonses.com ~all"````
- ```dmarc.example.com TXT "v=DMARo Live) value. This value is a number represented in seconds that the response should be saved for locally until you have to look it up again. Caching saves on having to make a request every time you communicate with a server.C1; p=reject; rua=mailto:dmarc-reports@example.com; adkim=s; aspf=s; pct=100"```
- ```@ TXT "MS=ms12345678"```

![[DNS_request.png]]
## How a DNS request works :

1. The computer checks its local cache to see if the user previously looked up the address

2. A recursive DNS server is provided by the ISP. It has a local cache and it checks in it. 

3. The root servers redirect the user to the correct TLD server, depending of the request. For example, if the user requests google.com, the root server will recognize the TLD of *.com* and refer the correct TLD server that deals with *.com* addresses.

4. The TLD server knows where to find the authoritative server to answer the DNS request and the authoritative server is also known as the nameserver for the domain.

5. Depending on the record type, the DNS record is then set back to the Recursive DNS Server, where a local copy will be cached for future requests. DNS records all come with a **TTL** (Time To Live), a number in seconds that the response should be save for locally. 

![[DNS-Resolution-process.png]]

## Queries

Every DNS queries is assigned a unique 16-bits identification number, generated randomly that must be returned upon resolving the query to ensure the authenticity of the response.

# Manipulation

Installation of the python code used for DNS Exfiltration and Infiltration

```bash
git clone https://github.com/kleosdc/dns-exfil-infil
```

Then install the required modules

```bash
sudo pip3 install -r requirements.txt
```

Install the program used for DNS Tunneling

```bash
sudo dnf install iodine
```

Finally, download [[Wireshark]]

```bash
sudo dnf install -y tshark
```

To set up an environment that needs a **Public Domain Name** and a **Public Server**, follow this [tutorial](https://www.youtube.com/watch?v=p8wbebEgtDk).

## Record commands

## **nslookup** Commands

Basic A Record Lookup :

```bash
nslookup website.com
```

Use **-type=** to look up specific DNS record types :

```bash
nslookup -type=MX website.com
```
```bash
nslookup -type=CNAME www.website.com
```
 
 Add a DNS server address after teh domain to query that server directly. Useful for checking if a specific resolver has updated records :
 
 ```bash
 nslookup website.com 1.1.1.1
 ```
 ```bash
 nslookup -type=CNAME website.com 8.8.8.8
 ```
### **dig** Commands

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

## DNS Exfiltration

**DNS Exfiltration** is a cyberattack on servers via the DNS. It's mostly used by adding strings containing the desired 'loot' to DNS UDP requests. The string containing the loot would then be sent to a rogue DNS server that is logging these requests. 

## DNS Infiltration

**DNS Infiltration** is another method used to exploit the various vulnerabilities within an organization's DNS. 

## DNS Tunneling

![[DNS_exf.png]]

# DNS Pentesting

### Basic information

The **Domain Name System (DNS)** serves as the internet's directory, allowing users to access websites through **easy-to-remember domain names** like google.com, instead of the numeric Internet Protocol (IP) addresses. By translating domain names into IP addresses, the DNS ensures web browsers can quickly load internet resources, simplifying how we navigate the online world. 

Default port: ```53```

```
PORT      STATE  SERVICE   REASON
53/tcp    open   domain    Microsoft DNS
5353/udp  open   zeroconf  udp-response
53/udp    open   domain    Microsoft DNS
```

## Automation

```bash
for sub in $(cat <WORDLIST>);do dig $sub.<DOMAIN> @<DNS_IP> | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done

dnsenum --dnsserver <DNS_IP> --enum -p 0 -s 0 -o subdomains.txt -f <WORDLIST> <DOMAIN>
```

## DNS Recursion DDoS

If **DNS recursion** is enabled, it's possible to spoof the origin on the UPD packet in order to make the DNS send the response to the victim server. 

To check if a DNS supports **recursion**, query a domain name and check if the **flag "ra"** is in the response :

*Example : *
```bash
dig website.com A
```

**Available** :
![[dig_ra.png]]

## Exfiltration

Try to connect to the DNS server via SSH :

```bash
ssh -l <USER> <IP>
```

Then enter the password if you have it after the following line

```bash
<USER>@<IP>'s password: 
```
## Infiltration

The type of DNS Record usually used to infiltrate data into a network is **TXT**.

Using **nslookup** to save the value into the *filename* :

```bash
nslookup -type=txt <SUBDOMAIN>.<DOMAIN> | sed -n 's/.*text = "\(.*\)"/\1/p' > <FILENAME>
```

## DNS Tunneling 

Using [Iodine](https://code.kryo.se/iodine/) :

For **Ubuntu** and others Debian based distributions :

```bash
sudo apt install iodine
```

For **Fedora** based distributions :
```bash
sudo dnf insatll iodine
```

Start the iodined on the AWS server using the following arguments :
```bash
sudo iodined -b <PORT> -f <DNS_IP> <SUBDOMAIN>
```

For example :

![[dns_tunneling_iodined.PNG]]

Then run iodine on the Client machine using these arguments : 

```bash
sudo iodine <IP> <SUBDOMAIN>
```

If everything is correctly setup, you can try to ping the DNS server IP :

```bash
ping <DNS_IP>
```

Now, generate a SSH key and upload the content of **id_rsa.pub** to the DNS Tunnel Server : 

```bash
ssh-keygen -t rsa
```

You should see the **id_rsa.pub** content in the **authorized_keys** by using :

```bash
cat authorized_keys
```

Then SSH the DNS Tunnel Server with the following command :

```bash
ssh <USER>@<DNS_IP> -D 8080
```


### DNS PTR (Reverse DNS)

Reverse DNS is the opposite of a A/AAAA dig request. Instead of searching the IP address of the DNS server using the domain name or the hostname, we search the domain name/hostname using the IP address. DNS PTR records are similarly stored that other DNS records type :
```<name> <ttl> <class> <type> <rdata>```

###### In IPv4 :
An, A record must exist for every PTR record stored as the IP address is broken into segments and then reversed. 
For example, the PTR record of IPv4 address 8.8.4.4 for domain dns,google will be stored under 4.4.8.8.in.addr.arpa.

###### How to check the PTR record or perform a reverse IP lookup ?

```bash
dig -x <IP> +short
```

###### Common issues PTR records :
- mismatched reverse DNS and forward DNS records :
	One of the most frequent issues is when the forward DNS record (A record) and the reverse DNS (PTR record) do not match. For instance, the PTR record for an IP address might point to a domain but the A record for this domain may resolve to a different IP address than expected. This can cause mail servers and other service to flag the system as potentially suspicious.
- missing PTR records :
	A PTR record that is completely missing can prevent reverse DNS lookups from being successful. This is often the case with dynamically assigned IP addresses (such as those used by residential ISPs) or new servers that haven't been fully configured. Without a PTR record, some services (e.g., email servers) may reject connections or mark them as spam.
- Invalid or Incomplete PTR Records
	Sometimes, PTR records may exist but be incorrectly configured. For example, a PTR record might point to a non-existent or incorrect domain name. This can result in the inability to resolve the IP address to a valid hostname, leading to security concerns or email deliverability issues.
- Reverse DNS Lookup Failures
	Reverse DNS lookup failures can occur for a variety of reasons, such as incorrect DNS server configurations, network connectivity problems, or DNS server outages. These failures prevent systems from resolving IP addresses to hostnames, which can break functionality for certain services or applications.
- Dynamic IP Addressing and Reverse DNS
	When using dynamic IP addresses (e.g., in cloud environments or with ISPs that provide dynamic IP ranges), it’s possible that reverse DNS records will change frequently, or may not be configured properly at all. This can make it difficult to maintain accurate and consistent PTR records.

### DRDoS amplification attacks :
Include ICMP (Internet Control Message Protocol) Flooding, TCP Flooding, DNS Amplification Flooding, NTP (Network Time Protocol) Amplification Flooding, SNMP (Simple Network Management Protocol) Amplification Flooding, SSDP (Simple Service Discovery Protocol) Amplification Flooding and Memcached Amplification Flooding. 

![[DRDoS.jpg]]

Attacker sends request packets to multiple Recursive DNS servers. The DNS servers send the response packets to the victim host, which ca be larger than the request packets, causing a denial of service to the victim host. 

Check this [YouTube video](https://www.youtube.com/watch?v=LMgY-gTdDoY).

## Resources :

[THM room 1](https://tryhackme.com/room/dnsindetail)
[TMH room 2](https://tryhackme.com/room/dnsmanipulation)