[documents index](../)
# LAN Overview

![netstackEdgeNode](./netstackEdgeNode.svg)

0. SOHO LAN subnet
1. [network.ns.node](https://192.168.100.1) ng.netstack.org ng (network gateway)
2. [docarch.ns.node](https://192.168.100.2) dg.netstack.org ng (documents gateway)
3. [backups.ns.node](https://192.168.100.3) bg.netstack.org ng (backups gateway)
4. [storage.ns.node](https://192.168.100.4) sg.netstack.org ng (storage gateway)
5. [compute.ns.node](https://192.168.100.5) cg.netstack.org ng (compute gateway)

Current netstack proceedure is to use [https://domains.google/](https://domains.google/) and [github pages](https://github.com).

- [Network IPAM and Subnets - pfsense](./lan/network/pfsense/)
- [Storage Setup - freenas](./lan/storage/freenas/)
- [Compute Setup - proxmox](./lan/compute/proxmox/)
