---
title: EIGRP Stub
date: 2024-03-18 16:30:00 -0500
categories: [Layer 3 Technologies,EIGRP]
tags: [eigrp,enarsi]     # TAG names should always be lowercase
---


Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!


## EIGRP Stubs

EIGRP Stub Options:

* Connected - The router will advertise connected networks, that are included in the eigrp process
* Leak-map - Can use a route-map to customize which networks are allowed to be advertised, and which are not
* Receive-only - The router will not advertise any networks, it will only receive networks from its neighbors
    * This option can only be used by itself
* redistributed - The router will advertise redistributed routes
* static - The router will advertise static routes
* Summary - The router will advertise summary routes (which are configured locally)

**The default stub options, if none are defined, are connected & summary**


Configuring a device as a stub will limit the query domain in the EIGRP AS. Routers will not send queries to EIGRP stub routers.


## EIGRP Stub sites

==EXPAND ON THIS SECTION LATER==

* Only available in EIGRP named mode
* Mutually exclusive with the EIGRP stub configuration

Interfaces pointed towards the hub are marked with "stub-site wan-interface" under af-interface config mode.

## Configuration

**Stub Configuration**

```
R1(config)#router eigrp 1
R1(config-router)#eigrp stub ?
  connected      Do advertise connected routes
  leak-map       Allow dynamic prefixes based on the leak-map
  receive-only   Set receive only neighbor
  redistributed  Do advertise redistributed routes
  static         Do advertise static routes
  summary        Do advertise summary routes
  <cr>

R1(config-router)#eigrp stub 

show run | s router
router eigrp 1
 eigrp stub connected summary
```


**Stub Site Configuration**

```
router eigrp A
 !
  address-family ipv4 unicast autonomous-system 1
  !
  af-interface GigabitEthernet0/0
   stub-site wan-interface
  exit-af-interface
  !
  eigrp stub-site 22:22
 exit-address-family
```

## Debugs & Show Commands

```
show ip protocols
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
     Stub, connected, summary
     Topology : 0 (base) 
       Active Timer: 3 min
       Distance: internal 90 external 170
       Maximum path: 4
       Maximum hopcount 100
       Maximum metric variance 1  

   Automatic Summarization: disabled
   Maximum path: 4
   Routing for Networks:
     0.0.0.0
   Routing Information Sources:
     Gateway         Distance      Last Update
     10.1.4.4              90      00:01:13
   Distance: internal 90 external 170

```

## References:

[EIGRP Stub Routing](https://learningnetwork.cisco.com/s/article/eigrp-stub-routing)

[EIGRP Stub Sites](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_eigrp/configuration/xe-3s/asr1000/ire-xe-3s-asr1000/ire-iwan-simpl.pdf)

[IP Routing: EIGRP Configuration Guide - Chapter: EIGRP IWAN Simplification](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_eigrp/configuration/15-mt/ire-15-mt-book/ire-iwan-simpl.html)