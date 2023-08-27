# Docker-IPv6-HowTo  
a short straight-forward approach  
  
## HowTo  
  
---
  
### Theory: Define a Local IPv6 subnet we want to use  
  
https://dnschecker.org/ipv6-address-generator.php  
  
This example input:  
```
Global ID: 0000000000
Subnet ID: 0000
```
  
will result in:  
Name | Value
--- | ---
Prefix / L | fd
Global ID | 0000000000
Subnet ID | 0000
Network | fd00::
CIDR | fd00::/64
IPv6 Address Format | fd00:0000:0000:0000:XXXX:XXXX:XXXX:XXXX
Start Range | fd00:0000:0000:0000:0000:0000:0000:0000
End Range | fd00:0000:0000:0000:ffff:ffff:ffff:ffff
Block Size | 18446744073709551616

That is 2^64 addresses - so we gonna split this next  
  
---
  
### Theory: Define Docker subnets / Subnetting IPv6
  
https://subnettingpractice.com/ipv6-subnet-calculator.html  
  
Goal is to split the /64 subnet to smaller subnets  
  
My personal split logic:  
Prefix | Total IP Addresses | Split
--- | --- | ---
64 | 2^64 (18,446,744,073,709,551,616) | 64k x /80 subnets
80 | 2^48 (281,474,976,710,656) | 64k x /96 subnets
96 | 2^32 (4,294,967,296) | 64k x /112 subnets 
112 | 2^12 (4,096) | -

#### Subnetting fd00::/64 into /80s
```
Subnetting fd00::/64 into /80s gives 65536 subnets ...
fd00::/80
fd00::1:0:0:0/80
fd00::2:0:0:0/80
fd00::3:0:0:0/80
...
```
  
#### Subnetting fd00::/80 into /96s
```
Subnetting fd00::/80 into /96s gives 65536 subnets ...
fd00::/96
fd00::1:0:0/96
fd00::2:0:0/96
fd00::3:0:0/96
...
fd00::a:0:0/96
...
```
 
---
  
### Setup  
  
https://docs.docker.com/config/daemon/ipv6/  
  
**Check: Local subnets should all be unique and not overlapping!**  
  
**Important: At the moment this document is released IPv6 is still experimental**  
  
**Hint "mixed environments":**  
I personally encountered problems in mixed environments.  
Example: Traefik Network IPv6 with other networks still running in IPv4.  
Above resulted in containers (sometimes/random!) not being able to reach each other.  
Declaring all networks to v6 seems to do the trick!  
  
#### Example Setup  
  
`fd00::1:0:0/96`  
for the default bridge (fixed-cidr-v6)  
  
`fd00::a:0:0/96`  
for the docker daemon to derive other networks from (default-address-pools)  
  
---
  
`/etc/docker/daemon.json`  
I also adjusted the size used for base: 192.168.0.0/16 from 20 to 24  
```
{
    "ipv6": true,
    "fixed-cidr-v6": "fd00::1:0:0/96",
    "experimental": true,
    "ip6tables": true,
    "default-address-pools": [
        {
            "base": "172.17.0.0/16",
            "size": 16
        },
        {
            "base": "172.18.0.0/16",
            "size": 16
        },
        {
            "base": "172.19.0.0/16",
            "size": 16
        },
        {
            "base": "172.20.0.0/14",
            "size": 16
        },
        {
            "base": "172.24.0.0/14",
            "size": 16
        },
        {
            "base": "172.28.0.0/14",
            "size": 16
        },
        {
            "base": "192.168.0.0/16",
            "size": 24
        },
        {
            "base": "fd00::a:0:0/96",
            "size": 112
        }        
    ]
}
```
  
---
  
`docker-compose.yml`:
```
networks:
  name:
    enable_ipv6: true
```
  
---
  
## Links  
  
### DNSChecker.org Local IPv6 Address Generator  
https://dnschecker.org/ipv6-address-generator.php  
  
### Netgate Docs IPv6 Subnetting  
https://docs.netgate.com/pfsense/en/latest/network/ipv6/subnets.html  
  
### SubnettingPractice.com IPv6 Subnet Calculator  
https://subnettingpractice.com/ipv6-subnet-calculator.html  
  
### Docker Official Documentation  
https://docs.docker.com/config/daemon/ipv6/  
