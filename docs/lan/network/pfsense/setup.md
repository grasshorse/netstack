[documents](../../../) [lan](../../) [network](../)

# pfsense setup

1. Default install via "pfsense" USB install key
  - Using Dell (Intel SR1560SF)
  - Defalut install user: admin pw: pfsense
  - IF VM XCP-ng: Network Interfaces - Check - Disable hardware checksum offload
  - Note interface assignments and lable ports and cables
  - Walk through wizard
    - Turn off Block RFC1918 Private Addresses and bogon networks (so we can use LAN address ranges)
    - Decide what the LAN subnet should be (default is 192.168.1.1/24) 191.168.252.0/23
    - Set admin password
  - Should have 2 interfaces WAN and LAN
  - Add Traffic Graphs to Dashboard 
2. DHCP Setup -> Services DHCP Server / LAN
  - Enbable DHCP Server
  - Range: 192.168.253.101 - 199 (Move new servers to MAC assignment)
  - View DHCP Static Mappings (at bottom)
  - Status -> DCHP Leaases View leases and move 101-199 to a static according to IP mappings
3. Add VLAN's (skip this for now... will have to deal with router)
  - Interfaces -> Assignments -> VLANs
    - VLANs add
    - Parent Interface: lan  VLAN Tag: 9 Description: ADM
    - Save
  - Interfaces -> Interfaces Assignments
    - Add (Select above VLAN) Save
    - Click on new interface
    - Check Enable Interface
    - Add Description: ADM
    - IPv4 Config Type: Static IPv4
    - Go down to Static IPv4 Config and the IP (192.168.9.1/24)
    - Uncheck reserve net blocking
    - Click SAVE
    - Click APPLY CHANGES
  - Services -> DHCP Server
    - Select ADM network
    - Enable DHCP Server
    - Range: 192.168.9.200-250
    - Save
4. Add Firewall Rules
  - Firewall -> Rules -> ADM
  - Add: 
    - Action - Pass
    - Interface - ADM
    - Addresss Family - IPv4
    - Protocal - any
    - Source - any
    - Destination - any
    - Description: dmzall
    - Save
  - Add-TOP:
    - Action - Pass
    - Interface - ADM
    - Addresss Family - IPv4
    - Protocal - IPv4 ICMP - echoreq
    - Source - any
    - Destination - any
    - Description: ghWANallowPing
    - Save
  - Add-TOP: (not yet)
    - Action - Block
    - Interface - ADM
    - Addresss Family - IPv4
    - Protocal - any
    - Source - any
    - Destination - LAN
    - Description: BLOCK to LAN
    - Save
  - Add-TOP... block all other networks you want blocked
5. Add VLAN tags to Switches / vlans 
  
### Equipment Docs
- [HPE C7000 Ciscos WS-CBS-3020GSG2](https://drive.google.com/drive/folders/0B1myz1MGUaPqSjB3MDJyRktYaDA)
- [Dell Intel SR1560SF](https://drive.google.com/drive/folders/0B1myz1MGUaPqSjB3MDJyRktYaDA)

### Notes
- [Tutorial - pfsense install: Lawrence](https://youtu.be/9kSZ1oM-4ZM)
  - [interface assignments](https://youtu.be/9kSZ1oM-4ZM?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h&t=588)
  - [re-assign interface](https://youtu.be/9kSZ1oM-4ZM?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h&t=651)
  - [Uncheck Block Networks](https://youtu.be/9kSZ1oM-4ZM?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h&t=881)
  - [run perf test](https://youtu.be/9kSZ1oM-4ZM?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h&t=1349)
  - [open 443 to remote admin](https://youtu.be/9kSZ1oM-4ZM?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h&t=1639)
  - [create firewall rules for a crapnetwork subnet](https://youtu.be/9kSZ1oM-4ZM?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h&t=1706)
  - [create UPnP for crapnetwork subnet](https://youtu.be/9kSZ1oM-4ZM?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h&t=1943)
  - [traffic shape crapnetwork subnet](https://youtu.be/9kSZ1oM-4ZM?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h&t=2069)
- [Tutorial - pfsense on XCP-ng: Lawrence tutorial](https://youtu.be/hy6RwgDm1p0?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h)
  - Check network performace [tutorial](https://youtu.be/hy6RwgDm1p0?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h&t=172)
    ```bash
    iperf3 -c 192.168.9.1
    iperf3 -c 192.168.9.1 -P 100 -t 20
    ```
  - Add XCP-ng tools [tutorial](https://youtu.be/hy6RwgDm1p0?list=PLjGQNuuUzvms3MhpsQ4zbe_Rlbo_0x01h&t=220)
- [Tutorial - Virtualization Lab Network Setup / Demo using XCP-NG, UniFi, pfsense and Xen Orchestra](https://www.youtube.com/watch?v=o1nwUfHsDHs)
- [Tutorial - pfSense VLAN and Guest Network Setup](https://www.youtube.com/watch?v=hhPGN4UJHAM)
- [Tutorial - tbd]()
- [Tutorial - tbd]()
- [Tutorial - tbd]()
