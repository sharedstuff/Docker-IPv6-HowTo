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
  
IPv6 | Total IP Addresses | Split
--- | --- | ---
/64 | 2^64 (18,446,744,073,709,551,616) | 64k subnets of size /80
/80 | 2^48 (281,474,976,710,656) | 64k subnets of size /96
/96 | 2^32 (4,294,967,296) | -

IPv6 | IPv4 | Total IP Addresses | Split
--- | --- | --- | ---
/96 | * | 2^32 (4,294,967,296) | 256 subnets of size /104 (IPv4:/8)
/104 | /8 (x.*) | 2^24 (16,777,216) | 256 subnets of size /112 (IPv4:/16)
/112 | /16 (x.x.*) | 2^16 (65,536) | 256 subnets of size /120 (IPv4:/24)
/120 | /24 (x.x.x.*) | 2^8 (256) | -
  
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
  
##### Default bridge  
65,636 possible addresses in total (as 1 subnet)  
```
172.16.0.0/16
fd00::1:0/112
```
  
##### Default address pools
65,636 possible addresses in total (split to 256 subnets)  
```
172.17.0.0/16 # split to /24
fd00::2:0/112 # split to /120
```
  
---
  
`/etc/docker/daemon.json`  
```
{
  "ipv6": true,
  "fixed-cidr": "172.16.0.0/16",
  "fixed-cidr-v6": "fd00::1:0/112",
  "experimental": true,
  "ip6tables": true,
  "default-address-pools": [
    {
      "base": "172.17.0.0/16",
      "size": 24
    },
    {
      "base": "fd00::2:0/112",
      "size": 120
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
  
### Reserved IP addresses  
https://en.wikipedia.org/wiki/Reserved_IP_addresses  
  
### Docker IPv6 Official Documentation  
https://docs.docker.com/config/daemon/ipv6/  
  
### Local IPv6 Address Generator  
https://dnschecker.org/ipv6-address-generator.php  
  
### IPv6 Subnetting  
https://docs.netgate.com/pfsense/en/latest/network/ipv6/subnets.html  
  
### IPv6 Subnet Calculator  
https://subnettingpractice.com/ipv6-subnet-calculator.html  
  