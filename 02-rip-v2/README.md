# RIPv2 Routing Lab (Cisco Packet Tracer)

A three-router lab that uses RIP version 2 to let three LANs reach each other dynamically, replacing the static routes from my previous lab.

## Topology

- 3 x Cisco 2911 routers connected in a chain by two point-to-point /30 links
- 3 LANs, each with a PC
- Router-to-router links: `1.1.1.0/30` and `2.2.2.0/30`
- LANs: `192.168.10.0/24`, `192.168.20.0/24`, `192.168.30.0/24`

## What I configured

- IP addressing on every interface (/30 on the router links, /24 on the LANs)
- RIP version 2 with auto-summary disabled, so subnet masks are carried in updates:

```
router rip
 version 2
 no auto-summary
 network 192.168.10.0
 network 1.1.1.0
```

Each router advertises only the networks directly connected to it. In RIP the `network` command takes the network address only, with no mask or wildcard.

## Verification

- `show ip route`: routes learned through RIP appear with the `R` code and administrative distance 120
- Ping between PCs in different LANs succeeds

## RIP authentication (note)

I attempted to secure RIP updates with MD5 authentication. Packet Tracer does not support the `ip rip authentication` commands on this router, so this lab runs without it.

On real devices, or on an IOS image that supports it, I would configure it on every router, on the interfaces facing other routers only:

```
key chain RIPKEY
 key 1
  key-string <shared-secret>
!
interface GigabitEthernet0/1
 ip rip authentication mode md5
 ip rip authentication key-chain RIPKEY
```

Why: without authentication, any device running RIP on the segment could inject fake routes. With MD5, routers accept updates only from neighbors that share the key.

## Key concepts

- RIP is a distance-vector protocol. Metric is hop count (maximum 15). Updates go out every 30 seconds.
- RIPv1 is classful (no subnet mask in updates). RIPv2 is classless, supports VLSM and authentication, and sends updates to multicast `224.0.0.9`.

## Files

- `RIP.pkt`: the Packet Tracer lab file
