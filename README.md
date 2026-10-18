# Multi-Site Enterprise Network (Cisco Packet Tracer)

A two-site enterprise network designed and built in Cisco Packet Tracer, with a headquarters (HQ) and a branch office connected through a router-to-router link running OSPF.

## Project Overview

The goal of this project was to design a small enterprise network, configure each device from scratch, and verify that all departments and both sites can communicate.

**HQ site**
- 1 Cisco 2911 router
- 1 multilayer switch connected to the router
- 3 access switches, each connected to the multilayer switch
- End devices split into three VLANs (Management, Sales, Services)
- A central DHCP and DNS server in the Services VLAN

**Branch site**
- 1 Cisco 2911 router
- 1 Cisco 2960 switch
- 3 PCs

The HQ and branch routers are connected directly (Gig0/1 to Gig0/1).

## Topology

![Network Topology](topology.png)

## Addressing Plan

| Network | VLAN | Subnet | Gateway |
|---|---|---|---|
| Management | 10 | 10.10.10.0/24 | 10.10.10.1 |
| Sales | 20 | 10.10.20.0/24 | 10.10.20.1 |
| Services | 30 | 10.10.30.0/24 | 10.10.30.1 |

The DHCP/DNS server uses **10.10.30.10**.

## DHCP and DNS

One central server provides DHCP and DNS for all the HQ VLANs, with a separate pool for each VLAN:

| Pool | Network | Default Gateway | DNS Server | Start IP |
|---|---|---|---|---|
| management-pool | 10.10.10.0/24 | 10.10.10.1 | 10.10.30.10 | 10.10.10.2 |
| sales-pool | 10.10.20.0/24 | 10.10.20.1 | 10.10.30.10 | 10.10.20.2 |
| service-Pool | 10.10.30.0/24 | 10.10.30.1 | 10.10.30.10 | 10.10.30.20 |

The Services pool starts at 10.10.30.20, which leaves the lower addresses free for devices with static IPs. Because the server sits in a different VLAN from most clients, DHCP relay (`ip helper-address`) is configured on the router so every VLAN can reach it.

## What I Implemented

- VLAN segmentation for the HQ departments (Management, Sales, Services)
- Trunk links between the multilayer switch and the access switches
- Inter-VLAN routing on the HQ router so departments can communicate
- A central DHCP and DNS server with one pool per VLAN, using DHCP relay
- OSPF dynamic routing between the HQ and Branch routers

## Security

- Access control lists (ACLs) to control traffic between network segments
- Port security on access ports
- SSH for secure remote management

## Configuration Files

Device configurations are in the [`configs`](./configs) folder, exported from each device with `show running-config`. Passwords in the configs are lab-only.

## Verification

I tested the network with these checks:

1. `show vlan brief` to confirm VLANs and port assignments
2. `show ip interface brief` to confirm interfaces are up
3. `show ip ospf neighbor` to confirm the OSPF neighbor relationship between the routers
4. `show ip route` to confirm routes to the remote site
5. DHCP: confirmed that PCs receive an address from the correct pool
6. `ping` between PCs in different HQ VLANs
7. `ping` from an HQ PC to a Branch PC

## Troubleshooting Log

**Problem:** PCs could not get an IP address from the DHCP server in the Services VLAN.

**Cause:** A link that needed to carry traffic from several VLANs was configured as an access port. An access port carries only one VLAN, so DHCP requests from the other VLANs could not reach the server.

**Fix:** I changed that link to a trunk port, which carries multiple VLANs using 802.1Q tagging. After that, every PC received an address from the correct pool.

**Lesson:** Access ports carry one VLAN; any link that must carry several VLANs needs to be a trunk. When DHCP fails, I now check VLAN assignments and trunk links first.

## What I Learned

- Planning the IP addressing and VLAN design before configuring made troubleshooting much easier
- How inter-VLAN routing works on a router
- How DHCP relay lets a central server serve several VLANs
- How OSPF shares routes between two sites
- The difference between access ports and trunk ports, and when each is needed
- How to use `ping` and `show` commands to find where connectivity breaks

## Tools

- Cisco Packet Tracer
- Cisco IOS CLI

## About Me

Dalia Ahmed, Network Engineer with a CCNA certification, currently pursuing CCNP.
[LinkedIn](https://www.linkedin.com/in/dalia-ahmed-2a370122b)
