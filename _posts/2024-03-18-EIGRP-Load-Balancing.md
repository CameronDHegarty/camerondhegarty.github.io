---
title: EIGRP Load Balancing
date: 2024-03-18 15:54:00 -0500
categories: [Layer 3 Technologies,EIGRP]
tags: [eigrp,enarsi]     # TAG names should always be lowercase
---

Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!

## EIGRP Load-Balancing

**Equal cost load balancing:**

* ECMP will happen by default when more than one path to a prefix have the same feasable distance.


**Unequal cost load balancing:**

* This is not enabled by default, but can be enabled by setting the variance.
* Will only work on feasable successors, which have passed the feasability condition.

Variance works by multiplying the feasable distance of the successor by the variance number. If the feasable distance of a feasable successor is lower than that number, it will be used for unequal cost load balancing.

Example:

```
! Prior to enabling variance:
      3.0.0.0/32 is subnetted, 1 subnets
D        3.3.3.3 [90/156160] via 10.1.4.1, 00:00:59, GigabitEthernet0/0

P 3.3.3.3/32, 1 successors, FD is 130816
        via 10.1.4.1 (156160/128256), GigabitEthernet0/0
        via 10.3.4.3 (168960/128256), GigabitEthernet0/1
```

We can see in the above example that there is only one route in the routing table for 3.3.3.3. However in the eigrp topology table we see that there is a feasable successor route available. By default, this feasable successor wont be added into the RIB because it has a higher metric than the successor route.

When we set the variance to 2, EIGRP will take the feasable distance of the successor route, multiply it by the variance and any feasable successors with a FD lower than the number calculated will be used. 

156160 * 2 = 312,320

312,320 > 168960

```
! After applying a variance of 2:
      3.0.0.0/32 is subnetted, 1 subnets
D        3.3.3.3 [90/168960] via 10.3.4.3, 00:00:06, GigabitEthernet0/1
                 [90/156160] via 10.1.4.1, 00:00:06, GigabitEthernet0/0

P 3.3.3.3/32, 2 successors, FD is 130816
        via 10.1.4.1 (156160/128256), GigabitEthernet0/0
        via 10.3.4.3 (168960/128256), GigabitEthernet0/1

! NOTE: The FD didn't change for either path, but they're both in the routing table now                
```



**Maximum Paths:**

* EIGRP can load balance across 4 paths by default, but can be configured for as low as 1 and up to as many as 32 paths.


## Configuration

```
! The below configuration sets the variance and maximum paths:
router eigrp 1
 maximum-paths 16
 variance 2
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
     Router-ID: 10.1.4.4
     Topology : 0 (base) 
       Active Timer: 3 min
       Distance: internal 90 external 170
       Maximum path: 16
       Maximum hopcount 100
       Maximum metric variance 2

   Automatic Summarization: disabled
   Maximum path: 16
   Routing for Networks:
     0.0.0.0
   Routing Information Sources:
     Gateway         Distance      Last Update
     10.1.4.1              90      00:02:36
     10.3.4.3              90      00:02:36
   Distance: internal 90 external 170

```


## References:

[How Does Unequal Cost Path Load Balancing (Variance) Work in IGRP and EIGRP?](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/13677-19.html)

[IP Routing: EIGRP Configuration Guide - Chapter: EIGRP](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_eigrp/configuration/15-mt/ire-15-mt-book/ire-enhanced-igrp.html)