# NordVPN_3.7.2_KillSwitch_DNS_Leak_Linux

### tl;dr
NordVPN client software in Linux isn't trustworthly.  
The killswitch feature suffers **DNS leaking**.  
***I highly recommend to run NordVPN software in windows and use a linux VM with a NAT network interface.***

### The report

> This vulnerability was tested on `Linux kali 5.6.0-kali1-amd64` and `Linux Debian 10`.  
> My client software version is `NordVPN Version 3.7.2`  
  
I've found that NordVPN KillSwitch suffer DNS leaking on Linux machines.  
These are my settings :  
```shell
Technology: OpenVPN
Protocol: UDP
Kill Switch: enabled
CyberSec: disabled
Obfuscate: disabled
Notify: enabled
Auto-connect: disabled
DNS: disabled
```
  
First thing I 've noticed was that on Kali the killswitch won't working after every restart.  
I have to connect one time then everything will work as expected.  
  
Now to the dns leak...  
This was a dig output when connected to the VPN :  
```shell
; <<>> DiG 9.16.2-Debian <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54792
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		11	IN	A	172.217.19.206

;; AUTHORITY SECTION:
google.com.		2624	IN	NS	ns1.google.com.
google.com.		2624	IN	NS	ns3.google.com.
google.com.		2624	IN	NS	ns4.google.com.
google.com.		2624	IN	NS	ns2.google.com.

;; Query time: 80 msec
;; SERVER: 103.86.96.100#53(103.86.96.100)
;; WHEN: Wed May 27 08:08:47 EDT 2020
;; MSG SIZE  rcvd: 127
```
  
Nothing strange here. 
The software is querying correctly the nordvpn DNS server : `103.86.96.100`.  
But if I disconnect from the VPN and I try again a query for `google.com` :
```shell
; <<>> DiG 9.16.2-Debian <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3491
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		185	IN	A	216.58.209.46

;; Query time: 52 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Wed May 27 08:09:04 EDT 2020
;; MSG SIZE  rcvd: 55
```
The domain is resolved anyway!! Even though the killswitch is activated...  
The DNS query is relayed to my ISP DNS resolver (`192.168.1.1`).  
  
I've tested even on Windows and seem to not suffer this leak.  
  
If you want to bypass the killswitch problem and still use Linux I recommend to use a VM hosted on windows and with the net interface set to `NAT` (**NOT** in `bridge` mode).  
Obv the NordVPN client software should run on Windows.

Regards.
