# BGP AS Path Prepend Topology

## Overview

This topology is designed to practice and understand the **BGP AS Path Prepend** attribute, which is used to influence inbound traffic by making a path appear less preferable. In this scenario, we manipulate AS Path Prepend to make R3 prefer R2 over R1 when reaching R1’s network.

## Topology Details

### Routers and Interfaces

```
- R1
  - Interface to R2: 192.168.12.1/24
  - Interface to R3: 192.168.13.1/24
- R2
  - Interface to R1: 192.168.12.2/24
  - Interface to R3: 192.168.23.2/24
- R3
  - Interface to R1: 192.168.13.3/24
  - Interface to R2: 192.168.23.3/24
  - Loopback: 3.3.3.3/32
```

### BGP AS Numbers

```
- R1: AS 65001
- R2: AS 65002
- R3: AS 65003
```

## BGP Configuration

### BGP Peering

```
- R1 neighbors:
  - R2 (192.168.12.2) - AS 65002
  - R3 (192.168.13.3) - AS 65003
- R2 neighbors:
  - R1 (192.168.12.1) - AS 65001
  - R3 (192.168.23.3) - AS 65003
- R3 neighbors:
  - R1 (192.168.13.1) - AS 65001
  - R2 (192.168.23.2) - AS 65002
```

### Configuring AS Path Prepend on R1

To make **R3 prefer R2** over R1 when reaching `1.1.1.1/32`, we configure AS Path Prepend on R1:

#### **Route Map on R1**

```bash
route-map PREPEND permit 10
 set as-path prepend 65001 65001 65001
```

#### **Apply Route Map to BGP Neighbor (R3 → R1)**

```bash
router bgp 65001
 neighbor 192.168.13.3 route-map PREPEND out
```

## Expected Behavior

- **Before applying AS Path Prepend:**
  - R3 may prefer the direct path to R1 (`192.168.13.1`).
- **After applying AS Path Prepend on R1:**
  - R3 should now send traffic via **R2 (192.168.23.2)** first before reaching R1.

## Verification Commands

### Check BGP Routes

```bash
show ip bgp 1.1.1.1
```
- Ensure that on **R3**, the best path (`>`) is via `192.168.23.2` (R2), and the path via `192.168.13.1` has a longer AS path.

### Check Routing Table

```bash
show ip route 1.1.1.1
```
- The next hop should be `192.168.23.2` instead of `192.168.13.1`.

### Traceroute from R3 to 1.1.1.1

```bash
traceroute 1.1.1.1 source 3.3.3.3
```
- Expected output:

```
1  192.168.23.2  (R2)
2  192.168.12.1  (R1)
3  1.1.1.1       (R1 Loopback)
```

## Troubleshooting

If traffic is still going **directly to R1**, check:

1. **R1's BGP table**:
   ```bash
   show ip bgp 1.1.1.1
   ```
   - Ensure **AS Path is prepended (65001 65001 65001)**.
2. **R3’s BGP table**:
   ```bash
   show ip bgp 1.1.1.1
   ```
   - Ensure R3 selects `192.168.23.2` as the best path.
3. **R3’s Routing Table**:
   ```bash
   show ip route 1.1.1.1
   ```
   - If the next hop is still `192.168.13.1`, check IGP/static routes.
4. **Reset BGP sessions**:
   ```bash
   clear ip bgp * soft
   ```

## Conclusion

This topology demonstrates how **BGP AS Path Prepend** can control inbound traffic by making a specific path appear less preferable. By adding extra AS hops on **R1**, we successfully reroute traffic from **R3 → R1** to **R3 → R2 → R1**.

