[documents index](../)
# LAN Overview

![netstackEdgeNode](./netstackEdgeNode.svg)

| IP | lan | purpose |
|----|-----|---------|
| [https://192.168.128.1](https://192.168.128.1) | [https://ng.ns.lan](https://ng.ns.lan) | ng - network gateway | 
| [https://192.168.128.2](https://192.168.128.2) | [https://dg.ns.lan](https://dg.ns.lan) | dg - documents gateway | 
| [https://192.168.128.3](https://192.168.128.3) | [https://bg.ns.lan](https://bg.ns.lan) | bg - backups gateway | 
| [https://192.168.128.4](https://192.168.128.4) | [https://sg.ns.lan](https://sg.ns.lan) | sg - storage gateway | 
| [https://192.168.128.5](https://192.168.128.5) | [https://cg.ns.lan](https://cg.ns.lan) | cg - compute gateway | 

Current netstack proceedure is to use [https://domains.google/](https://domains.google/) and [github pages](https://github.com) for community documents.  [Netstack https://netstack.org/docs](https://netstack.org/docs/)

## Netstack basics
Netstacks three technology pilars

- [Network - pfsense](./network/pfsense/)
- [Storage - freenas](./storage/freenas/)
- [Compute - proxmox](./compute/proxmox/)

## Netstack service
