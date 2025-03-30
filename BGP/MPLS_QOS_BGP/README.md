Here is the updated **README** without the actual configurations but still detailing the network and its setup:

---

# **MPLS QoS and BGP Configuration with VRF Setup**

## **Overview**

This document describes the configuration of a network consisting of **4 routers** (R1, R2, R3, and R4). The network utilizes **MPLS**, **QoS**, and **BGP** with **VRF** for traffic separation and optimal routing. The configuration assumes that **R4** is a **customer router** connected to **R1** and communicates with **R3** through **VRF1**.

## **Network Topology**

```
       [R1] 
      /    \
   e0/0    e0/1
 10.0.12.1 10.0.13.1
  /           \
[R2]---------[R3]
  \           /
  e0/1      e0/0
10.0.23.1  10.0.23.2
  |
  |
[R4]  <- Customer Router
  |
  |  e0/0   192.168.4.1/24  ->  R1 e0/2
```

- **R1**: Core Router, connects to **R2**, **R3**, and **R4**.
- **R2** and **R3**: Intermediary routers connecting **R1** with **R4**.
- **R4**: Customer router connected to **R1** via **VRF1**.

## **Steps for Configuration**

### **1. R1 Configuration**
R1 is the core router that connects to **R2**, **R3**, and **R4**. It has **MPLS**, **iBGP**, and **QoS** configured. Here's a summary of the configuration:

- **MPLS** enabled on the interfaces.
- **BGP** iBGP sessions with **R2**, **R3**, and **R4**.
- **QoS Policy** with **Class-1** (high priority) and **Class-2** (lower priority) applied on **Ethernet0/0** and **Ethernet0/1**.

### **2. R2 Configuration**
R2 is the intermediary router that connects **R1** to **R3** and plays a role in **iBGP** and **MPLS** forwarding. Here's the configuration:

- **BGP** iBGP session between **R1** and **R3**.
- **MPLS** configured to forward traffic.

**Key Points:**
- **Ethernet0/0** on **R2** connects to **R1** (IP 10.0.12.2).
- **Ethernet0/1** connects to **R3** (IP 10.0.23.2).
- **MPLS** enabled.

### **3. R3 Configuration**
R3 is another intermediary router, responsible for connecting **R2** and **R4** (via **VRF1**) and implementing **MPLS**.

**Key Points:**
- **Ethernet0/1** on **R3** connects to **R2** (IP 10.0.23.1).
- **Ethernet0/0** connects to **R4** (IP 192.168.4.2).
- **BGP** iBGP session between **R1** and **R3**.

### **4. R4 Configuration (Customer Router)**
R4 is the customer router connected to **R1** through **Ethernet0/0**. The **VRF1** configuration allows **R4** to be separated into its own routing context. The **BGP** session with **R1** ensures that **R4** can communicate with the **MPLS** network.

**Key Points:**
- **Ethernet0/0** on **R4** is configured for **VRF1** with the IP address **192.168.4.1/24**.
- **BGP** session with **R1** at **192.168.4.2**.
- **QoS** policies applied on **Ethernet0/0** for traffic prioritization.

---

## **Configuration Files**

The configurations for **R1**, **R2**, **R3**, and **R4** are as follows:

- **R1**: Core router with MPLS, BGP, and QoS configurations.
- **R2**: Intermediary router between **R1** and **R3**.
- **R3**: Another intermediary router, connecting **R2** to **R4** via **VRF1**.
- **R4**: Customer router with **VRF** configuration and **QoS** policies applied.

---

## **Testing Connectivity**

1. **Ping Test**:
   - From **R4** to **R3**: 
     ```bash
     ping vrf VRF1 192.168.4.2
     ```
   - Ensure that the routing tables and BGP sessions are correctly established for proper forwarding.

2. **Verify BGP and MPLS**:
   - Use `show ip bgp summary` and `show mpls forwarding-table` on the routers to check BGP sessions and MPLS labels.

3. **Verify QoS**:
   - Use `show policy-map interface Ethernet0/0` on **R4** to check the **QoS policy** applied to the interface.

---

## **Troubleshooting**

- **BGP Not Establishing?**
  - Verify that **R4** and **R1** have the correct neighbor configuration.
  - Ensure that the IP addresses on interfaces are correct and that **MPLS** is enabled on the interfaces.
  - Check if there are any **ACLs** or **firewall settings** blocking BGP.

- **QoS Not Working?**
  - Check if the correct **preference values** are being set for **Class-1** and **Class-2**.
  - Ensure the **QoS policy** is applied to the correct interfaces.

---

## **Conclusion**

This README outlines the steps to configure **MPLS**, **iBGP**, **QoS**, and **VRF** on the routers in this network. It ensures proper isolation of customer traffic, optimal routing with BGP, and traffic engineering using **MPLS**. It also implements **QoS** for prioritizing traffic flows.

Let me know if you need any further details or if there are any issues with the setup!
