---
title: EIGRP Path Maniuplation
date: 2024-03-19 10:50:00 -0500
categories: [Layer 3 Technologies,EIGRP]
tags: [eigrp,enarsi]     # TAG names should always be lowercase
---


Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!

## EIGRP Path Manipulation

There are a few ways of manipulating which path is selected as the best path:

* Modify interface delay
* Use offset-list
* Use summarization (more specific path is chosen)
* Use summarization with with leak maps
* Modify AD
* Route filtering


## Configuration

**Interface Delay:**

```
! The below sets the delay on delay to 10000 usec. This will only affect this routers metric to its neighbors routes.
! If you want to also affect the routes as seen by the neighbor, the delay must be set on the neighbors interface as well
interface GigabitEthernet0/0
 delay 1000
```

**Offset-list**

```
! Below applies an outgoing offset out gig 0/0 for all networks matching acl 1
! Both standard and extended ACLs can be applied to offset lists
access-list 1 permit 10.1.2.0 0.0.0.255
router eigrp A
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
   offset-list 1 out 1400000 GigabitEthernet0/0 
  exit-af-topology
```

**Summarization**


Please see post on route filtering and summarization. [Here](https://camerondhegarty.github.io/posts/EIGRP-Filtering-Summarization/)

**Summarization with leak map**

```
! An excerpt from R2's routing table
D     192.168.1.0/24 [90/2570240] via 10.1.2.1, 00:04:45, GigabitEthernet0/1
D     192.168.2.0/24 [90/2570240] via 10.1.2.1, 00:04:45, GigabitEthernet0/1
D     192.168.3.0/24 [90/2570240] via 10.1.2.1, 00:04:45, GigabitEthernet0/1

! On R1 we will create a summary route towards R2, but will leak the 192.168.2.0/24 network
ip prefix-list NET2 seq 5 permit 192.168.2.0/24
route-map LEAK permit 10
 match ip address prefix-list NET2
interface GigabitEthernet0/1
 ip summary-address eigrp 1 192.168.0.0 255.255.252.0 leak-map LEAK

! R2's routing table after the summarization has been applied.
D     192.168.0.0/22 [90/2570240] via 10.1.2.1, 00:00:04, GigabitEthernet0/1
D     192.168.2.0/24 [90/2570240] via 10.1.2.1, 00:09:31, GigabitEthernet0/1
```

**Modify AD**

```
! The below configuration will set EIGRP internal routes to an AD of 80 and EIGRP external routes to an AD of 160
router eigrp A
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
   distance eigrp 80 160


! The below configuration sets all routes with a next hop of 10.1.2.1 to an AD of 8. 
! The wildcard of 0.0.0.0 can be updated to match a range of next hop IPs, can also apply an ACL to this command
router eigrp A
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
   distance 8 10.1.2.1 0.0.0.0

! Before
D     192.168.0.0/22 [80/2570240] via 10.1.2.1, 00:02:37, GigabitEthernet0/1
D     192.168.2.0/24 [80/2570240] via 10.1.2.1, 00:02:37, GigabitEthernet0/1

! After
D     192.168.0.0/22 [8/2570240] via 10.1.2.1, 00:00:02, GigabitEthernet0/1
D     192.168.2.0/24 [8/2570240] via 10.1.2.1, 00:00:02, GigabitEthernet0/1
```

**Route Filtering**

Please see post on route filtering and summarization. [Here](https://camerondhegarty.github.io/posts/EIGRP-Filtering-Summarization/)




## Debugs & Show Commands

```
show ip route
show ip eigrp topology [x.x.x.x/xx]
```

## References:

[Configure EIGRP to Influence Path Selection](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/221548-configure-eigrp-to-influence-path-select.html#toc-hId-228257578)
