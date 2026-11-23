# Static Routing Lab | Three Routers, Three LANs

My first static-routing lab in Cisco Packet Tracer. Three routers connect three separate LANs using manually configured IPv4 routes.

## What this lab covers

- IPv4 addressing with /24 LANs and /30 point-to-point links.
- Next-hop static routes between remote networks.
- Host default gateways and the return path for traffic.
- Troubleshooting a local ping that works while inter-LAN pings fail.

## Lab file

Download [the Packet Tracer lab](static_routing.pkt) and open it in Cisco Packet Tracer 9.0.1.0858 or a compatible version.
## Topology and addressing

```text
Router0 Gi0/1 -------- Gi0/1 Router1 Gi0/2 -------- Gi0/1 Router2
     |                         |                          |
   Gi0/0                     Gi0/0                      Gi0/0
     |                         |                          |
    PC0                       PC1                        PC2
192.168.10.0/24          192.168.20.0/24           192.168.30.0/24
```

| Device | Interface | IPv4 address | Subnet mask | Default gateway |
| --- | --- | --- | --- | --- |
| Router0 | GigabitEthernet0/0 | 192.168.10.1 | 255.255.255.0 | N/A |
| Router0 | GigabitEthernet0/1 | 1.1.1.1 | 255.255.255.252 | N/A |
| Router1 | GigabitEthernet0/0 | 192.168.20.1 | 255.255.255.0 | N/A |
| Router1 | GigabitEthernet0/1 | 1.1.1.2 | 255.255.255.252 | N/A |
| Router1 | GigabitEthernet0/2 | 2.2.2.1 | 255.255.255.252 | N/A |
| Router2 | GigabitEthernet0/0 | 192.168.30.1 | 255.255.255.0 | N/A |
| Router2 | GigabitEthernet0/1 | 2.2.2.2 | 255.255.255.252 | N/A |
| PC0 | FastEthernet0 | 192.168.10.100 | 255.255.255.0 | 192.168.10.1 |
| PC1 | FastEthernet0 | 192.168.20.100 | 255.255.255.0 | 192.168.20.1 |
| PC2 | FastEthernet0 | 192.168.30.100 | 255.255.255.0 | 192.168.30.1 |

The router-to-router networks are `1.1.1.0/30` and `2.2.2.0/30`. These are the original lab addresses, not private RFC 1918 ranges. Keep this topology isolated; for a new real-world design, use an appropriate addressing plan rather than copying these transit addresses.

## Static routes

These commands are taken from the running configurations stored in the supplied Packet Tracer file, not reconstructed from screenshots.

### Router0

```ios
ip route 192.168.20.0 255.255.255.0 1.1.1.2
ip route 2.2.2.0 255.255.255.252 1.1.1.2
ip route 192.168.30.0 255.255.255.0 1.1.1.2
```

### Router1

```ios
ip route 192.168.10.0 255.255.255.0 1.1.1.1
ip route 192.168.30.0 255.255.255.0 2.2.2.2
```

### Router2

```ios
ip route 1.1.1.0 255.255.255.252 2.2.2.1
ip route 192.168.10.0 255.255.255.0 2.2.2.1
ip route 192.168.20.0 255.255.255.0 2.2.2.1
```

A route selects the next router on the way to a destination network. For example, Router0 sends traffic for `192.168.30.0/24` to Router1 at `1.1.1.2`; Router1 then forwards it to Router2 at `2.2.2.2`.

The edge routers also have routes to the far transit network. No default route is configured. A default route is not needed to reach the networks explicitly covered by these connected and static routes. Internet connectivity is outside this lab's scope.

## Troubleshooting: missing PC default gateways

**Symptom:** inter-LAN pings were failing even though the routers had static routes to the remote LANs.

**Diagnostic:** a ping from PC0 to its local router, `192.168.10.1`, succeeded. This confirmed local reachability, but did not prove that PC0 had a default gateway configured. A device can reach its local subnet without using a gateway.

**Cause found:** no default gateway had been entered on the PCs.

**Fix:** use each PC's local router interface as its default gateway:

| PC | Default gateway |
| --- | --- |
| PC0 | 192.168.10.1 |
| PC1 | 192.168.20.1 |
| PC2 | 192.168.30.1 |

After the gateway correction, connectivity was reported working. The supplied `.pkt` confirms that all three PCs now contain the gateways listed above.

**Lesson:** router routes are only part of end-to-end connectivity. Hosts must know where to send off-subnet traffic, and the reply must also have a valid path back.

## Verification and limits

- The router tables, topology screenshot and saved configuration inspected for this write-up agree on the addressing and static routes.
- The saved network confirms the PC addresses, masks and gateways in the table above.
- Successful connectivity after the fix was reported during the lab session. No final end-to-end ping screenshot is included, and the saved network has not been re-run as part of preparing this repository.
- All three routers have running configurations in the `.pkt`, but their stored startup-configuration sections are empty. Before testing a reload, use `copy running-config startup-config` on each router to save its current configuration to startup storage, then save the Packet Tracer file again.
- The displayed device names are Router0, Router1 and Router2. Their IOS hostname remains `Router`, so the CLI prompt alone does not distinguish them.

### Commands to repeat the checks

These are suggested tests, not a record of captured successful output.

On a router, `show ip interface brief` checks interface addressing and status; `show ip route` checks the routing table. `C` denotes a connected route, `L` a local interface address and `S` a static route.

On PC0, check the configuration with `ipconfig`, then test in this order:

```text
ping 192.168.10.1
ping 192.168.20.100
ping 192.168.30.100
```

The first test checks local router reachability. The other two test communication with the remote PCs. Repeat tests from the other PCs to check traffic in both directions.

## Repository layout

This is the first lab in the `CCNA_LABS` collection. Its documentation is stored at `01-static-routing/README.md`. Other labs can be added in separate numbered folders.

The Packet Tracer file, configuration exports and screenshots are not yet attached to this folder. The addressing plan and static route commands above are included directly so this README is usable on its own.

## Reference

[Cisco: Troubleshooting TCP/IP](https://www.cisco.com/en/US/docs/internetworking/troubleshooting/guide/tr1907.pdf), checking host configuration, local connectivity and the route in both directions.
