# Firewall Overview

Firewalls are one of the core components of a network security.

Common types of firewalls and how they function:

| Method | Description | Advantages | Disadvantages |
|---|---|---|---|
| **NAT Firewall** | Translates private IP addresses of multiple machines into one or a small pool of public IP addresses when accessing the Internet. | - Protects many machines behind a small number of public IP addresses.<br>- Simplifies network administration.<br>- Can control access by opening or closing ports. | - Cannot prevent malicious activity after users connect to an external service. |
| **Packet Filter** | Inspects individual packets based on header information such as IP addresses, ports, and protocols, then applies rules to allow or block traffic. | - No client-side configuration is required.<br>- Fast because connections are made directly without a proxy.<br>- Highly customizable through firewall rules such as `iptables`. | - Cannot filter application-level content.<br>- Complex network architectures with subnets, DMZs, or NAT can make rules difficult to configure. |
| **Proxy Firewall** | Acts as an intermediary between clients and the Internet. Clients send requests to the proxy, which makes the requests on their behalf. | - Provides control over which applications and protocols can access the Internet.<br>- Can cache frequently accessed data to reduce bandwidth usage.<br>- Provides detailed logging and monitoring. | - Often supports only specific applications or protocols.<br>- Some application services cannot run directly behind a proxy.<br>- Can become a network bottleneck because all traffic passes through the proxy. |


## 1. Netfilter and IPTables

The Linux kernel features a powerful networking subsystem called Netfilter. It provides stateful and stateless packet filtering as well as NAT and IP masquerading. Netfilter also has the ability to mangle IP header information for advanced routing and connection management.

Iptables is the command-line administration tool used to configure and control Netfilter. 

## 2. Basic Firewall Configuration

By default, firewall rules are saved in the `/etc/sysconfig/iptables` or `/etc/sysconfig/ip6tables` files. If the firewall rules fail, the fallback file `/etc/sysconfig/iptables.fallback` is applied if exist.

The firewall rules are only active if the `iptables` service is running. The follwing command is used to manually start the service:

```bash
service iptables restart
```

To ensure that iptables starts when the system is booted, use the following command:

```bash
chkconfig --level 345 iptables on
```


