# Packet Tracer Lab: EtherChannel Configuration and Routing

This lab focuses on configuring different types of EtherChannel using various protocols, routing between switches, and load-balancing methods. The goal is to ensure that all PCs can reach the server and the switches are correctly configured for efficient data traffic management.

### **Pre-configured Items:**
- **End host IPs and Switch Virtual Interface (SVI) IP addresses** have already been set up.
    - **PC1:** 172.16.1.1/24
    - **PC2:** 172.16.1.2/24
    - **SRV1:** 172.16.2.1/24
    - **SVI IP (on VLAN 1):** 172.16.1.254 (ASW1), 172.16.2.254 (ASW2)
    - **DSW1/DSW2:** IPs in the 10.0.0.0/30 subnet for inter-switch routing.

<img src= "https://github.com/ro-drick/EtherChannel-Configuration/blob/main/etherchannel.PNG">

### **Objectives:**
1. Configure Layer 2 EtherChannel between **ASW1** and **DSW1** using **LACP** as a trunk.
2. Configure Layer 2 EtherChannel between **ASW2** and **DSW2** using **PAgP** as a trunk.
3. Configure Layer 3 EtherChannel between **DSW1** and **DSW2** using **Static EtherChannel**.
4. Set up routing to allow PCs to communicate with **SRV1**.
5. Identify and document the default EtherChannel load-balancing method.
6. Modify the EtherChannel configuration to load-balance based on source and destination IP addresses.

---

## **Step-by-Step Instructions**

### 1. Configure Layer 2 EtherChannel between ASW1 and DSW1 using LACP

#### On ASW1:
- Enter configuration mode for interfaces `Gig0/1` and `Gig0/2`.
- Bundle the interfaces using LACP (mode active).
```bash
ASW1# conf t
ASW1(config)# interface range Gig0/1 - 2
ASW1(config-if-range)# channel-group 1 mode active
ASW1(config-if-range)# switchport mode trunk
ASW1(config-if-range)# exit
```

#### On DSW1:
- Enter configuration mode for interfaces `Gig1/0/3` and `Gig1/0/4`.
- Bundle the interfaces using LACP (mode active).
```bash
DSW1# conf t
DSW1(config)# interface range Gig1/0/3 - 4
DSW1(config-if-range)# channel-group 1 mode active
DSW1(config-if-range)# switchport mode trunk
DSW1(config-if-range)# exit
```

---

### 2. Configure Layer 2 EtherChannel between ASW2 and DSW2 using PAgP

#### On ASW2:
- Enter configuration mode for interfaces `Gig0/1` and `Gig0/2`.
- Bundle the interfaces using PAgP (mode desirable).
```bash
ASW2# conf t
ASW2(config)# interface range Gig0/1 - 2
ASW2(config-if-range)# channel-group 2 mode desirable
ASW2(config-if-range)# switchport mode trunk
ASW2(config-if-range)# exit
```

#### On DSW2:
- Enter configuration mode for interfaces `Gig1/0/3` and `Gig1/0/4`.
- Bundle the interfaces using PAgP (mode desirable).
```bash
DSW2# conf t
DSW2(config)# interface range Gig1/0/3 - 4
DSW2(config-if-range)# channel-group 2 mode desirable
DSW2(config-if-range)# switchport mode trunk
DSW2(config-if-range)# exit
```

---

### 3. Configure Layer 3 EtherChannel between DSW1 and DSW2 using Static EtherChannel

#### On DSW1:
- Configure interfaces `Gig1/0/1` and `Gig1/0/2` as Layer 3 interfaces and bundle them in a static EtherChannel.
```bash
DSW1# conf t
DSW1(config)# interface range Gig1/0/1 - 2
DSW1(config-if-range)# no switchport
DSW1(config-if-range)# channel-group 3 mode on
DSW1(config-if-range)# exit
DSW1(config)# interface port-channel 3
DSW1(config-if)# ip address 10.0.0.1 255.255.255.252
DSW1(config-if)# exit
```

#### On DSW2:
- Configure interfaces `Gig1/0/1` and `Gig1/0/2` as Layer 3 interfaces and bundle them in a static EtherChannel.
```bash
DSW2# conf t
DSW2(config)# interface range Gig1/0/1 - 2
DSW2(config-if-range)# no switchport
DSW2(config-if-range)# channel-group 3 mode on
DSW2(config-if-range)# exit
DSW2(config)# interface port-channel 3
DSW2(config-if)# ip address 10.0.0.2 255.255.255.252
DSW2(config-if)# exit
```

---

### 4. Configure Routes to Allow PC1/PC2 to Reach SRV1

#### On DSW1:
- Add a static route for network `172.16.2.0/24` to reach SRV1 through DSW2.
```bash
DSW1# conf t
DSW1(config)# ip route 172.16.2.0 255.255.255.0 10.0.0.2
DSW1(config)# exit
```

#### On DSW2:
- Add a static route for network `172.16.1.0/24` to allow return traffic from SRV1 to PCs through DSW1.
```bash
DSW2# conf t
DSW2(config)# ip route 172.16.1.0 255.255.255.0 10.0.0.1
DSW2(config)# exit
```

---

### 5. Identify the Default EtherChannel Load-Balancing Method

To check the default load-balancing method on each switch:

```bash
Switch# show etherchannel load-balance
```
- By default, most switches use **source MAC address** as the load-balancing method for Layer 2 EtherChannels.

---

### 6. Configure Load-Balancing Based on Source and Destination IP Addresses

To change the load-balancing method to **source and destination IP address**, use the following command on each switch:

```bash
Switch# conf t
Switch(config)# port-channel load-balance src-dst-ip
Switch(config)# exit
```

---

## **Verification Steps:**
1. **EtherChannel status:** Verify that EtherChannel is functioning correctly by checking the status of the Port-Channel interfaces.
   ```bash
   Switch# show etherchannel summary
   ```
2. **Routing:** Verify that the PCs can ping the SRV1 and vice versa.
   ```bash
   PC1> ping 172.16.2.1
   ```

---

## **Troubleshooting Tips:**
- Ensure the EtherChannel interfaces are up and bundled correctly. Misconfigured modes (LACP vs. PAgP) between switches may cause EtherChannel to fail.
- Use `show spanning-tree` to ensure that STP isn’t blocking any Port-Channel interfaces.
- Ensure that the static routes on DSW1 and DSW2 are correct and match the network addressing in the topology.

## **Conclusion**

In this lab, we successfully configured various types of EtherChannel—Layer 2 EtherChannel using both LACP and PAgP, as well as a Layer 3 EtherChannel using static bundling. These EtherChannels improved redundancy and load-balancing in the network, ensuring efficient use of available bandwidth. We also configured routing between switches to enable communication between different VLANs and ensure that the PCs could reach the server. Finally, we modified the load-balancing method to better distribute traffic based on source and destination IP addresses, optimizing the network's performance. By completing this lab, you have gained hands-on experience with critical concepts in network redundancy, load balancing, and inter-VLAN routing, which are key to building scalable and robust networks.

## Acknowledgements


Special thanks to **Jeremy's IT Lab** for providing valuable resources and tutorials that greatly contributed to the completion of this exercise. His in-depth explanations and practical demonstrations have been instrumental in enhancing my understanding of Cisco networking concepts and the effective use of Packet Tracer.

For more information and additional resources, visit [Jeremy's IT Lab](https://jeremysitlab.com/) and check out his YouTube for the full course, [Jeremy's IT Lab Free CCNA 200-301 | Complete Course](https://www.youtube.com/playlist?list=PLxbwE86jKRgMpuZuLBivzlM8s2Dk5lXBQ)
