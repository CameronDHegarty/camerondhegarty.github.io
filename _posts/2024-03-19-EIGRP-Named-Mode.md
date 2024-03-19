---
title: EIGRP Named Mode
date: 2024-03-19 15:30:00 -0500
categories: [Layer 3 Technologies,EIGRP]
tags: [eigrp,enarsi]     # TAG names should always be lowercase
---

Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!


## EIGRP Named Mode

* EIGRP Named mode centralizes all EIGRP configuration under the router configuration mode.
* Wide metrics are used by default
* Can be backwards compatible with classic EIGRP
* Address families are now configured under the same router configuration mode.
* VRF configured per address family in the eigrp named mode


## Configuration

When IPV6 address family is enabled, eigrp is automatically enabled for all IPv6 interfaces.


**Three main configuration modes:**

* address-family configuration mode - (config-router-af)#
* address-family topology configuration mode - (config-router-af-topology)#
* address-family interface configuration mode - (config-router-af-interface)#
        
All interface related configuration has now been moved under the routing configuration mode. Router eigrp A -> address-family ipv4 unicast autonomous-system 1 -> af-interface gig 0/0.

```
router eigrp A
 !
 address-family ipv6 unicast autonomous-system 100
  !
  topology base
  exit-af-topology
 exit-address-family
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
   ! distribute list, distance, and more defined under topolgoy base
   offset-list 1 out 1400000 GigabitEthernet0/0 
  exit-af-topology
  network 10.0.0.0
 exit-address-family
```

## Debug & Show Commands

```
! Normal EIGRP show command still work, but you can also use the show commands below to receive information about the eigrp AS
! These commands are similar to classic EIGRP, but instead of "show ip eigrp...", it's "show eigrp address-family ipv.."

R2#show eigrp address-family ipv4 1 interfaces
 EIGRP-IPv4 VR(A) Address-Family Interfaces for AS(1)
                              Xmit Queue   PeerQ        Mean   Pacing Time   Multicast    Pending
 Interface              Peers  Un/Reliable  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
 Gi0/1                    1        0/0       0/0           2       0/0           50           0

R2#show eigrp address-family ipv4 1 neighbors 
 EIGRP-IPv4 VR(A) Address-Family Neighbors for AS(1)
 H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                    (sec)         (ms)       Cnt Num
 0   10.1.2.1                Gi0/1                    11 03:34:31    2   100  0  28

R2#show eigrp address-family ipv4 topology
 EIGRP-IPv4 VR(A) Topology Table for AS(1)/ID(150.2.2.2)
 Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
        r - reply Status, s - sia Status  

 P 192.168.2.0/24, 1 successors, FD is 328990720
         via 10.1.2.1 (328990720/327761920), GigabitEthernet0/1
 P 4.4.4.4/32, 1 successors, FD is 984350720
         via 10.1.2.1 (984350720/983695360), GigabitEthernet0/1
 P 192.168.0.0/22, 1 successors, FD is 328990720
         via 10.1.2.1 (328990720/327761920), GigabitEthernet0/1

```

## References:

[Configure EIGRP Named Mode](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/200156-Configure-EIGRP-Named-Mode.html)


