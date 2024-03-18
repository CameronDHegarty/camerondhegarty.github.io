---
title: EIGRP Filtering & Summarization
date: 2024-03-18 15:54:00 -0500
categories: [Layer 3 Technologies,EIGRP]
tags: [eigrp]     # TAG names should always be lowercase
---

## EIGRP Filtering

**Distribute-list**

* Filters advertisements coming in from neighbor based on standard ACL, extended ACL, prefix list matching the route, prefix list matching the gateway address, or a route-map

### Configuration:

```
! EIGRP routes from routing table from test router
      3.0.0.0/32 is subnetted, 1 subnets
D        3.3.3.3 [90/168960] via 10.3.4.3, 00:35:23, GigabitEthernet0/1
                 [90/156160] via 10.1.4.1, 00:35:23, GigabitEthernet0/0

D     192.168.1.0/24 [90/156160] via 10.1.4.1, 00:00:53, GigabitEthernet0/0
D     192.168.2.0/24 [90/156160] via 10.1.4.1, 00:00:46, GigabitEthernet0/0
D     192.168.3.0/24 [90/156160] via 10.1.4.1, 00:00:36, GigabitEthernet0/0


! Removes all routes learned on gig 0/0 except for what is matched in the ACL (192.168.2.0/24) (Standard ACL)
ip access-list standard STANDARD-ACL
 permit 192.168.2.0 0.0.0.255
router eigrp 1
 distribute-list STANDARD-ACL in GigabitEthernet0/0

! Will come back to investigate and complete configuration for distribute-lists with extended ACLs

! Removes all routes with a gateway of 10.1.4.1, which in this case is all EIGRP routes except for the (Gateway prefix list)
! route to 3.3.3.3 via gig 0/1. May be useful in some implementations where eigrp doesn't set next hop to self.
ip prefix-list GATEWAY seq 5 deny 10.1.4.1/32
ip prefix-list GATEWAY seq 10 permit 0.0.0.0/0 le 32
router eigrp 1
 distribute-list gateway GATEWAY in GigabitEthernet0/0

! Removes just the 192.168.3.0/24 network and allows everything else (Pefix list)
ip prefix-list NET3 seq 5 deny 192.168.3.0/24
ip prefix-list NET3 seq 10 permit 0.0.0.0/0 le 32
router eigrp 1
 distribute-list prefix NET3 in GigabitEthernet0/0


! Removes all networks identified in the route-map 
ip prefix-list NET1 seq 5 permit 192.168.1.0/24
route-map FILTER deny 10
 match ip address prefix-list NET1
route-map FILTER permit 20
router eigrp 1
 distribute-list route-map FILTER in GigabitEthernet0/0
```

## EIGRP Summarization

* Creates a summary route prefix that is advertised to neighbors
    * Local route added pointed to NULL0
    * Local route has an AD of 5
* There needs to be at least one network inside the RIB that is matched by the summary route in order for the summary route to be advertised. 


### Configuration:

```
! Applied under interface configuration mode. This summarizes a 192.168.0.0/22 network to its neighbor connected to gig 0/0
interface GigabitEthernet0/0
 ip summary-address eigrp 1 192.168.0.0 255.255.252.0
```

## Debugs & Show Commands

```
Show ip protocols
Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 Protocol for AS(1)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0
    Soft SIA disabled
    NSF-aware route hold timer is 240
    Router-ID: 10.1.4.1
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1

  Automatic Summarization: disabled
  Address Summarization:
    192.168.0.0/22 for Gi0/0
      Summarizing 3 components with metric 128256
  Maximum path: 4
  Routing for Networks:
    0.0.0.0
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.1.4.4              90      00:03:48
  Distance: internal 90 external 170

```


## References:

[IP Routing EIGRP Configuration Guide - Chapter: EIGRP Support for Route Map Filtering](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_eigrp/configuration/15-sy/ire-15-sy-book/ire-sup-routemap.html)