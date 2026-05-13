# Day 4 - Networking Fundamentals

## Overview
Configured static IP addressing and practiced essential network 
troubleshooting tools on RHEL 10.

## Environment
- Host: CachyOS Linux
- Hypervisor: VMware Workstation Pro 25H2
- VM: Red Hat Enterprise Linux 10.1
- Kernel: 6.12.0-124.52.1.el10_1.x86_64

## Tasks Completed

### Static IP Configuration
- Identified dynamic DHCP assignment as unsuitable for server use
- Configured static IP using nmcli and NetworkManager
- Set static IP: 172.16.200.128/24
- Set gateway: 172.16.200.2
- Set primary DNS: 8.8.8.8, secondary: 172.16.200.2
- Changed ipv4.method from auto to manual
- Applied changes with nmcli connection down/up
- Verified dynamic flag removed from interface

### Network Troubleshooting Tools
- ip addr show — view interface configuration and IP addresses
- ip route show — view routing table and default gateway
- ss -tlnp — view listening ports and owning processes from inside
- nmap — scan open ports from network perspective
- Understood relationship between ss and nmap for firewall diagnosis

### DNS Tools
- Installed bind-utils for dig and nslookup
- Used dig to query A records and read TTL, query server, response time
- Used nslookup for quick DNS lookups
- Used dig -x for reverse DNS lookups and understood PTR records
- Confirmed all DNS queries routing through 8.8.8.8

### traceroute
- Installed and ran traceroute to google.com
- Read hop by hop path from VM through VMware NAT, home router,
  Lumen/Qwest Colorado Springs gateway, to Google's network
- Understood how to identify where connectivity breaks down

## Key Commands
- nmcli connection show
- nmcli connection modify
- nmcli connection down/up
- ip addr show
- ip route show
- ss -tlnp
- nmap
- dig
- dig -x
- nslookup
- traceroute
