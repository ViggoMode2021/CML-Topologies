# BGP Local Preference Topology

## Overview
This topology is designed to practice and understand the **BGP Local Preference** attribute, which is used to influence outbound traffic within an Autonomous System (AS). In this scenario, we manipulate Local Preference to make R1 prefer R2 as the best path to reach R3.

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

### Setting Local Preference on R2
To make **R1 prefer R2** over R3 for reaching `3.3.3.3/32`, we configure Local Preference on R2:

#### **Route Map on R2**
```bash
route-map LOCALPREF permit 10
 set local-preference 200
```

#### **Apply Route Map to BGP Neighbor (R3 → R2)**
```bash
router bgp 65002
 neighbor 192.168.23.3 route-map LOCALPREF in
```

## Expected Behavior

- **Before applying Local Preference:**
  - R1 may prefer the direct path to R3 (`192.168.13.3`).
- **After applying Local Preference on R2:**
  - R1 should now send traffic via **R2 (192.168.12.2)** first before reaching R3.

## Verification Commands

### Check BGP Routes
```bash
show ip bgp 3.3.3.3
```
- Ensure that on **R1**, the best path (`>`) is via `192.168.12.2` (R2).

### Check Routing Table
```bash
show ip route 3.3.3.3
```
- The next hop should be `192.168.12.2` instead of `192.168.13.3`.

### Traceroute from R1 to 3.3.3.3
```bash
traceroute 3.3.3.3 source 1.1.1.1
```
- Expected output:
```
1  192.168.12.2  (R2)
2  192.168.23.3  (R3)
3  3.3.3.3       (R3 Loopback)
```

## Troubleshooting

If traffic is still going **directly to R3**, check:
1. **R2's BGP table**:
   ```bash
   show ip bgp 3.3.3.3
   ```
   - Ensure **Local Preference = 200**.
2. **R1’s BGP table**:
   ```bash
   show ip bgp 3.3.3.3
   ```
   - Ensure R1 selects `192.168.12.2` as the best path.
3. **R1’s Routing Table**:
   ```bash
   show ip route 3.3.3.3
   ```
   - If the next hop is still `192.168.13.3`, check IGP/static routes.
4. **Reset BGP sessions**:
   ```bash
   clear ip bgp * soft
   ```

## Conclusion

This topology demonstrates how **Local Preference** can control outbound traffic by influencing BGP path selection within an AS. By increasing the Local Preference on **R2**, we successfully reroute traffic from **R1 → R3** to **R1 → R2 → R3**.

