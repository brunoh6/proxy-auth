
# Linux Proxy Lab with Authentication (Squid + ACL)

## Description

This project is a practical implementation of network segmentation, firewall rules, and an authenticated proxy using Squid on Ubuntu Server 22.04. It is part of the coursework for "Information Protection in Computer Networks" at UrFU. The environment is simulated using Cisco equipment and Linux-based virtual machines.

## Topology Overview

The network is divided into three zones:
- DMZ (Demilitarized Zone)
- Internal Servers
- Internal Clients

## Node Table

| Zone             | Node  | Interface | DMZ Interface | Internal-Servers Interface | Internal-Clients Interface | IP Address   |
|------------------|-------|-----------|---------------|-----------------------------|-----------------------------|--------------|
| -                | GW-FW | fa1/0     | Gi0/3         | -                           | -                           | 172.16.1.1   |
| -                | GW-FW | fa2/0     | -             | Gi0/3                       | -                           | 172.16.2.1   |
| -                | GW-FW | fa3/0     | -             | -                           | Gi0/3                       | 172.16.2.3   |
| DMZ              | DMZ1  | e0        | Gi0/0         | -                           | -                           | 172.16.1.2   |
| DMZ              | DMZ2  | e0        | Gi0/1         | -                           | -                           | 172.16.1.3   |
| DMZ              | DMZ3  | e0        | Gi0/2         | -                           | -                           | 172.16.1.4   |
| Internal-servers | SRV1  | e0        | -             | Gi0/0                       | -                           | 172.16.2.2   |
| Internal-servers | SRV2  | e0        | -             | Gi0/1                       | -                           | 172.16.2.3   |
| Internal-servers | SRV3  | e0        | -             | Gi0/2                       | -                           | 172.16.2.4   |
| Internal-clients | PC1   | e0        | -             | -                           | Gi0/0                       | 172.16.3.2   |
| Internal-clients | PC2   | e0        | -             | -                           | Gi0/1                       | 172.16.3.3   |
| Internal-clients | PC3   | e0        | -             | -                           | Gi0/2                       | 172.16.3.4   |

## Main Configuration

### Gateway-Firewall (Cisco)

#### NAT and Routing

```shell
ip routing
access-list 1 permit 172.16.0.0 0.0.255.255
ip nat inside source list 1 interface fa0/0 overload
interface fa1/0
 ip nat inside
interface fa2/0
 ip nat inside
interface fa3/0
 ip nat inside
interface fa0/0
 ip nat outside
ip route 0.0.0.0 0.0.0.0 192.168.36.2
```

#### DHCP Pools

```shell
ip dhcp pool INTERNAL_CLIENTS
 network 172.16.3.0 255.255.255.0
 default-router 172.16.3.1

ip dhcp pool INTERNAL_SERVERS
 network 172.16.2.0 255.255.255.0
 default-router 172.16.2.1

ip dhcp pool DMZ
 network 172.16.1.0 255.255.255.0
 default-router 172.16.1.1
```

#### ACL Rules

```shell
access-list 100 deny tcp 172.16.3.0 0.0.0.255 any eq www
access-list 100 deny tcp 172.16.3.0 0.0.0.255 any eq 443
access-list 100 permit ip any any
```

### Proxy Server (Squid on Ubuntu SRV1)

```bash
sudo apt update
sudo apt install squid apache2-utils
sudo htpasswd -c /etc/squid/passwords bruno
```

#### squid.conf

```bash
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwords
auth_param basic realm Proxy Authentication
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive off

acl authenticated proxy_auth REQUIRED
http_access allow authenticated
http_access deny all
```

#### Client Environment Variables

```bash
export http_proxy="http://172.16.2.2:3128"
export https_proxy="http://172.16.2.2:3128"
```

## Validation

- Proxy prompts for credentials when browsing
- curl returns "407 Proxy Authentication Required" without login
- Squid logs confirm requests are intercepted
- ACLs prevent direct external access if not using proxy

## Author

Bruno Paolo Huamán Vela  
UrFU – Information Security of Telecommunication Systems  
Group RI-411050
