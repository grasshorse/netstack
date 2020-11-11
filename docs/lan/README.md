[documents index](../)
# LAN Overview

![netstackEdgeNode](./netstackEdgeNode.svg)

| IP | lan | purpose |
|----|-----|---------|
| [https://192.168.128.1](https://192.168.128.1) | [https://ng.ns.lan](https://ng.ns.lan) | ng - network gateway | 
| [https://192.168.128.2](https://192.168.128.2) | [https://sg.ns.lan](https://sg.ns.lan) | sg - storage gateway | 
| [https://192.168.128.3](https://192.168.128.3) | [https://cg.ns.lan](https://cg.ns.lan) | cg - compute gateway | 
| [https://192.168.128.4](https://192.168.128.4) | [https://bg2.ns.lan](https://bg2.ns.lan) | bg - backups gateway secondary|
| [https://192.168.128.5](https://192.168.128.5) | [https://ng2.ns.lan](https://ng2.ns.lan) | ng - network gateway secondary| 
| [https://192.168.128.6](https://192.168.128.6) | [https://sg2.ns.lan](https://sg2.ns.lan) | sg - storage gateway secondary| 
| [https://192.168.128.7](https://192.168.128.7) | [https://cg2.ns.lan](https://cg2.ns.lan) | cg - compute gateway secondary| 
| [https://192.168.128.8](https://192.168.128.8) | [https://bg.ns.lan](https://bg.ns.lan) | bg - backups gateway |
| [https://192.168.128.9](https://192.168.128.9) | [https://dg.ns.lan](https://dg.ns.lan) | dg - documents gateway | 

Current netstack proceedure is to use [https://domains.google/](https://domains.google/) and [github pages](https://github.com) for community documents.  [Netstack https://netstack.org/docs](https://netstack.org/docs/)

## Netstack basics
Netstacks three technology pilars

- [Network - pfsense](./network/pfsense/)
- [Storage - freenas](./storage/freenas/)
- [Compute - proxmox](./compute/proxmox/)

## Netstack service
