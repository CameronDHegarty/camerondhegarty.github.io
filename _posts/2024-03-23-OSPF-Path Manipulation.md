---
title: OSPF Path Manipulation & Additional features
date: 2024-03-23 14:54:00 -0500
categories: [Layer 3 Technologies,OSPF]
tags: [ospf]     # TAG names should always be lowercase
---

Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!

This article is a rough draft. Will come back in the future and polish.


## OSPF Path Manipulation

OSPF Path Selection:

1. Most specific route. Ex. /28 preferred over /24, which is preferred over /16, etc. Default route being the least specific you can get (summarization can be used influence which path is used)
2. Lowest AD
3. Intra-Area(O) > Inter-Area (O IA)
4. E1 > E2 > N1 > N2
5. Lowest cost

Only best path will be entered into the routing table, when prefix lengths are the same. If prefix lengths are different then two or more routes to the destination may be installed into the routing table. However, the most specific route will always be preferred.

### OSPF Max-metric

* Can be used to set this router's cost to the maximum for all LSAs. (Prefer any router more than this one)


```
router ospf 1
max-metric router-lsa [external-lsa|include-stub|on-startup|summary-lsa]


! Other option's description from CLI output:
R4(config-router)#max-metric router-lsa ?
  external-lsa  Override external-lsa metric with max-metric value
  include-stub  Set maximum metric for stub links in router-LSAs
  on-startup    Set maximum metric temporarily after reboot
  summary-lsa   Override summary-lsa metric with max-metric value
  <cr>

```

### Summarize Routes

* Can summarize addresses from one area into another. Can only be completed on an ABR
    * The routes that are being summarized must originating in the area that the summary command is referencing
* Summarization can also occur on the ASBR where external routes are redistributed into OSPF
    * Must be done on the router where the external route is redistributed into OSPF. Cannot be done on another ABR/ASBR


NOTE: Summary routes will supress the specific networks that are being summarized. (It doesn't appear that there's a way to leak the more specific routes)

```
! Summarizing routes from one AS to another
router ospf 1
area 2 range 192.168.0.0 255.255.0.0
! The above command creates the 192.168.0.0/16 summary route.


! Summarizes routes redistributed into OSPF
router ospf 1
summary-address 160.1.4.0 255.255.255.0

```


### Filter Routes

Routes can be filtered out using the "area x filter-list" command. 
* Preformed on an ABR from one area to another
* Does not affect external routes

The "in" and "out" option are in reference to the area specified in the command. Referencing area 0 in the outbound direction will filter the routes leaving area 0 into the other areas.

* Applying a filter-list in the outbound direction will affect more than 1 area, if configured. For example, if filtering out of area 0, but you have area's 2 & 3 on the abr, the routes will be filtered to both area 2 & 3.

```
ip prefix-list FILTER seq 5 deny 150.1.0.0/16 ge 30
ip prefix-list FILTER seq 10 deny 155.1.146.0/24
ip prefix-list FILTER seq 15 permit 0.0.0.0/0 le 32

router ospf 1
 area 0 filter-list prefix FILTER out

```


Routes can also be filtered out using the summarization commands, with the [not-advertise] option added.

```
! Filter LSAs from one area to another using summarization
area 2 range 192.168.0.0 255.255.0.0 not-advertise

! Filter out networks that have been redistributed into OSPF, using summary command
summary-address 160.1.4.0 255.255.255.0 not-advertise
```


Routes can also be filtered out between the LSDB and the routing table. With this option, the route will continue to be advertised to neighbors, but will not be installed into the local routing table.
 
* This is completed with the distribute-list command

```
router ospf 1
distribute-list prefix FILTER in 
```

Routes can be filtered in this same way, but for all or specific routes only from a specific neighbor (next hop)

```
! The below will block all routes with specific next hop IPs, which are specified in the prefix-list "BLOCK-R5-ROUTES"
router ospf 1
distribute-list gateway BLOCK-R5-ROUTES in

! Can use the prefix option followed by the gateway option in the same command. However, from my experience they work seperately. Both the prefixes AND routes from the specified next hop will both be blocked

```


Route-maps can also be used with "distribute-list" command to provide more match criteria. With a route-map, you can match on outgoing interface, ip add, next-hop, advertising router id, metric, route-type, and tags.

```
access-list 1 permit 150.1.7.7 ! Prefixes to deny
access-list 1 permit 150.1.9.9 ! Prefixes to deny
access-list 2 permit 155.1.58.5 ! Next HOP IP addresse

route-map FILTER deny 10
 match ip address 1
 match ip next-hop 2
route-map FILTER permit 20

router ospf 1
 distribute-list route-map FILTER in

```


Routes can also be filtered by modifying the AD of specific routes to 255. (AD can also be set to less than IGP to have them be preferred)

**NOTE**: The below does not seem to affect external routes

```
! The below command will set all routes from the advertising router of 150.1.5.5 to an AD of 255,
! thus they will not be installed into the routing table. Where 150.1.5.5 is the router ID of the
! neighbor, not the next hop IP address.
router ospf 1
distance 255 150.1.5.5 0.0.0.0


! An ACL can also be specified to only affect certain routes from the neighbor. 1 being a standard ACL (ONLY STANDARD ACLS CAN BE USED)
! The routes permited in the ACL will have the adjusted AD.
router ospf 1
distance 255 150.1.5.5 0.0.0.0 1
```

### OSPF Default routes

* Default routes created for stubs with the [no-summary] option are sent as type-3 LSA
* Default routes created anywhere in the network with "default-information originate" are advertised as type-5 LSA's, or type-7 if in an NSSA


When there are two or more exits to an NSSA, this can be used as one way to prefer traffic going out certain egresses.

* No-summary option on the NSSA command from one ABR will cause all LSA's to be supressed and advertise a type-3 default route. While the other router will still advertise the specific prefixes via type-3 LSAs. It will be more preferred for inter-area routes
* Default-information originate on the other router will advertise a default route into the NSSA as a type-7 LSA. This will be less preferred to the type-3 lsa default route generated by the other router. Thus, the other router will be preffered for the external routes, which were not advertised into the NSSA.




### Conditional default route advertisement

OSPF can be setup to advertise a default route based on the condition of another route being present in the routing table. 
 
* For reliable conditional default route advertisement, a placeholder static route could be created tracking an IP SLA. If the sla is succeeding then the placeholder static route will be in the routing table and the default route will be advertised in ospf. If the sla is failing, the static route will not be in the routing table and ospf will not advertise the default.


```
! The below sets OSPF up to originate a default route ONLY, if the network matched in the route-map 
!  is currently in the routing table. Just referncing a loopback ip via a prefix list in this example.

ip prefix-list LOOP seq 5 permit 150.1.8.8/32
route-map EXIST permit 10
 match ip address prefix-list LOOP


router ospf 1
default-information originate route-map EXIST


! When shutting down the loopback and losing the route to the loopback in the RIB, the default route disappers.
```


### OSPF Database-filter

* Configuration option applied directly to the interface
* Can only be applied in the outbound direction
* Stops all LSAs from being sent to neighbor. However, still receive LSA's from neighbor

```
interface gig 0/0
ip ospf database-filter all out

```

## References

Many of these topics were learned from [INE's](www.ine.com) professional level labs. These labs are by far the best and in depth labs on routing protocols I've seen studying for the CCNP. Definetly worth the money you'd spend on a subscription!
