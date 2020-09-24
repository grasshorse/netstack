[documents index](../)
# LAN Overview

![netstackEdgeNode](./netstackEdgeNode.svg)

0. SOHO LAN subnet typically 192.168.1.0/24
1. [network.ns.lan https://192.168.128.1](https://192.168.128.1) ng.ns.lan (ng - node 1 network gateway)
2. [docarch.ns.lan https://192.168.128.2](https://192.168.128.2) dg.ns.lan (dg - documents archive gateway)
3. [backups.ns.lan https://192.168.128.3](https://192.168.128.3) bg.ns.lan (bg - backups gateway)
4. [storage.ns.lan https://192.168.128.4](https://192.168.128.4) sg.ns.lan (sg - storage gateway)
5. [compute.ns.lan https://192.168.128.5](https://192.168.128.5) cg.ns.lan (cg - compute gateway)

Current netstack proceedure is to use [https://domains.google/](https://domains.google/) and [github pages](https://github.com).

- [Network IPAM and Subnets - pfsense](./network/pfsense/)
- [Storage Setup - freenas](./storage/freenas/)
- [Compute Setup - proxmox](./compute/proxmox/)
