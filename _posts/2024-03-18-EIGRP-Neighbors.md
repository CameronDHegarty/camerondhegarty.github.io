---
title: EIGRP communication & EIGRP Neighbors
date: 2024-03-18 11:30:00 -0500
categories: [Layer 3 Technologies,EIGRP]
tags: [eigrp,enarsi]     # TAG names should always be lowercase
---

Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!

## Basics of EIGRP

* EIGRP is protocol number 88
* It is an "enhanced" distance vector protocol which, by default, uses bandwidth and delay to calculate a metric

## Neighbor Communication

* IPv4 EIGRP hello messages are sent to multicast address 224.0.0.10.
* IPv6 EIGRP hello messages are sent to multicast address FF02::A

Two EIGRP routers become neighbors when they receive hello messages from each other on a common network segment.

By default EIGRP sends hellos every 5 seconds on fast links and every 60 seconds on slow links. 

EIGRP also uses a hold timer to track its neighbors state. By default the hello timer is 15 seconds on fast links and 180 seconds on slow links (typically 3x the hello time). The hold timer ticks down until a hello message is received from the neighbor. When a hello message is received, the hold timer resets back to its maximum value. If no hello message is received before the hold timer reaches 0, the neighber will be declared dead. 

==COME BACK TO AND RUN PCAP TO VERIFY IF ALL COMMUNICATION IS MULTICAST OR IF SOME ARE UNICAST AFTER NEIGHBORSHIP IS FORMED - ALSO REVIST and EXPAND on EIGRP MESSAGE TYPES==

EIGRP message types:

* Hello - Used for neighbor discovery and as a keep alive.
* Query - Sent to neighbors to request alternate paths to a prefix that has been lost via another neighbor.
* Reply - Unicast message sent as responses to query packets. Confirms weather there's an alternate path to the prefix or not.
* Update - Sends information to neighbors about know prefixes and their reachability.
* Ack - Unicast message to acknowledge an update, query, and reply Messages. No acknowlegement is sent for hello messages


## Matching values between neighbors

The below values must match between routers in order for EIGRP neighborships to form:

* Must be on the same subnet
* Must be in the same AS
* K-values
* EIGRP authentication

NOTE: Hello & hold timers do not need to match for neighbor adjacency to form. The hold timer configured on a router is what the neighbor router uses as its hold timer for the router. There is a great write up on this [here.](https://dorreke.wordpress.com/2008/05/25/eigrp-timers-solution/)

## Debugs & Show Commands

```
show ip eigrp neighbor
 H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                    (sec)         (ms)       Cnt Num
 0   10.1.4.4                Gi0/0                    12 00:00:54    1  3000  0  1
show ip eigrp neighbor detail
 EIGRP-IPv4 Neighbors for AS(1)
 H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                    (sec)         (ms)       Cnt Num
 0   10.1.4.4                Gi0/0                    11 00:01:36    1  3000  0  1
    Version 20.0/2.0, Retrans: 1, Retries: 0
    Topology-ids from peer - 0 
    Topologies advertised to peer:   base

 Max Nbrs: 0, Current Nbrs: 0

 BFD sessions
  NeighAddr         Interface
  10.1.4.4            GigabitEthernet0/0

Debug eigrp packets
```

## References:

[Understand and Use the Enhanced Interior Gateway Routing Protocol](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/16406-eigrp-toc.html)

