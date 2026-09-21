# PPRC Network Infrastructure Project

A Cisco networking project that designs and configures a small institutional network composed of three building LANs, a DMZ for public services, and an external ISP connection.

The network is designed for redundancy, scalability, dynamic routing, Internet access, public services, and secure device administration.

## Project Overview

The infrastructure includes:

- 3 separate LANs, one for each building
- Wired and wireless client access
- Redundant switching inside each building
- Dynamic routing with OSPF
- DHCP-based client addressing
- NAT/PAT for Internet access
- A DMZ hosting public services
- DNS, HTTP, FTP, and MAIL servers
- Secure remote administration with SSH
- Port Security on access ports
- Administrative session timeout with `exec-timeout`

## Network Architecture

The project uses three main network areas:

| Area | Network | Purpose |
|---|---|---|
| Intranet | `172.16.0.0/16` | Internal user networks |
| DMZ | `210.1.1.64/27` | Public-facing services |
| ISP link | `210.1.1.32/27` | External Internet connectivity |

Each building has its own `/24` subnet, allowing up to 254 usable host addresses.

### Building LANs

| Building | VLAN | Subnet | Default Gateway |
|---|---:|---|---|
| Building 1 | 2 | `172.16.2.0/24` | `172.16.2.1` |
| Building 2 | 3 | `172.16.3.0/24` | `172.16.3.1` |
| Building 3 | 4 | `172.16.4.0/24` | `172.16.4.1` |

### ISP Connection

| Device | Address |
|---|---|
| Institution edge router | `210.1.1.34/27` |
| ISP router | `210.1.1.33/27` |

## DHCP

DHCP is configured directly on the router acting as the default gateway for each LAN.

The first 9 addresses in every LAN are excluded from dynamic allocation so they can be reserved for gateways and network equipment.

Example configuration for LAN 3:

```text
ip dhcp excluded-address 172.16.3.1 172.16.3.9

ip dhcp pool Ioana3
 network 172.16.3.0 255.255.255.0
 default-router 172.16.3.1
 dns-server 210.1.1.68
```

Because DHCP runs on the local gateway router, no `ip helper-address` is required.

## Dynamic Routing — OSPF

OSPF is enabled on the routers to provide dynamic routing between all internal networks.

Key characteristics:

- All routers participate in **OSPF Area 0**
- Routes are learned dynamically
- Neighbor adjacencies are verified in `FULL` state
- Alternative paths are available if a router link fails
- Redundant router links form a triangle topology
- STP handles Layer 2 redundancy between switches

Useful verification commands:

```text
show ip ospf neighbor
show ip route ospf
```

## NAT / PAT

Internal hosts use private IPv4 addresses from `172.16.0.0/16`.

PAT is configured on the edge router so multiple internal hosts can share public addresses when accessing the external network.

Example:

```text
access-list 8 permit 172.16.0.0 0.0.255.255
ip nat inside source list 8 pool ioana overload
```

Interfaces toward the intranet are configured as `ip nat inside`, while the interface toward the ISP is configured as `ip nat outside`.

Verification:

```text
show ip nat translations
```

## DMZ Services

The DMZ uses public addresses from `210.1.1.64/27`.

| Server | IP Address | Service | Hostname |
|---|---|---|---|
| Server0 | `210.1.1.66` | HTTP | `www.ioana.ro` |
| Server1 | `210.1.1.67` | FTP | `ftp.ioana.ro` |
| Server2 | `210.1.1.68` | DNS | `dns.ioana.ro` |
| Server3 | `210.1.1.69` | MAIL | `mail.ioana.ro` |

### DNS

The DNS server resolves the service hostnames to their corresponding IPv4 addresses.

Example:

```text
www.ioana.ro  -> 210.1.1.66
ftp.ioana.ro  -> 210.1.1.67
dns.ioana.ro  -> 210.1.1.68
mail.ioana.ro -> 210.1.1.69
```

### HTTP

The HTTP server hosts the institution's web page and can be accessed through:

```text
http://www.ioana.ro
```

### FTP

The FTP server requires authentication and supports permissions such as:

- Read
- Write
- Delete
- Rename
- List

Example client access:

```text
ftp ftp.ioana.ro
```

### MAIL

The mail server hosts mailboxes under the `ioana.ro` domain.

Example address:

```text
user@ioana.ro
```

The DNS configuration includes the required mail-related record so clients can send and receive messages through the mail server.

## Security

### SSH Administration

Remote administration is performed using SSH instead of Telnet.

The configuration includes:

- Local user authentication
- RSA keys
- SSH-only access on VTY lines
- Restricted administrative privilege level

Example local user:

```text
username ioana77 privilege 7 password ioana77
```

### Port Security

Port Security is enabled only on access ports connected to end devices.

Uplink ports between switches and routers are excluded.

Configured behavior includes:

```text
switchport mode access
switchport port-security
switchport port-security maximum 2
switchport port-security violation restrict
switchport port-security mac-address sticky
```

Verification:

```text
show port-security interface <interface>
```

### Administrative Session Timeout

Console and VTY sessions are automatically closed after 5 minutes of inactivity.

```text
line console 0
 exec-timeout 5 0
 logging synchronous

line vty 0 4
 exec-timeout 5 0
 logging synchronous
```

## Testing and Verification

The project was validated through practical tests covering:

- DHCP address assignment
- Connectivity between LANs
- OSPF neighbor formation and route learning
- Failover over redundant paths
- NAT/PAT translations
- DNS resolution
- HTTP access
- FTP authentication and file transfer
- MAIL delivery between users
- SSH remote administration
- Port Security status
- Automatic session timeout

## Main Technologies

- IPv4 subnetting
- VLAN-based LAN separation
- DHCP
- OSPF
- NAT / PAT
- STP
- DNS
- HTTP
- FTP
- Email services
- SSH
- Port Security
- Cisco IOS CLI

## Project Goal

The goal of the project is to demonstrate how routing, addressing, public services, redundancy, Internet connectivity, and device security can be integrated into a reliable and scalable institutional network.
