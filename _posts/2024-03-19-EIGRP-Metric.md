---
title: EIGRP Metric
date: 2024-03-19 14:00:00 -0500
categories: [Layer 3 Technologies,EIGRP]
tags: [eigrp,enarsi]     # TAG names should always be lowercase
---


Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!


## EIGRP Metric

EIGRP uses metric to determine the best path to a destination. By default EIGRP uses bandwidth and delay to calculate the metric, but can be tuned to also include reliability, load, and MTU into it's metric calculation.

When EIGRP is determining the best path, the lower metric is preferred to higher metrics.


* The route with the lowest metric is the successor route
* Any routes with the same metric are added to the RIB for ECMP
* Any other routes with worse metrics, which pass the feasability condition are considered feasable successors (These can be added to the RIB as well for unequal cost load balancing by configuring the variance)



**Metric equation for classic metrics:**

metric = [K1 * bandwidth + (K2 * bandwidth) / (256 - load) + K3 * delay] * [K5 / (reliability + K4)]

**Metric equation for wide metrics:**

Wide metrics are 64-bit, however they must scaled down to 32-bit for EIGRP to use it. Scale can be adjusted with the "metric rib-scale" command.

Metric = [(K1*Minimum Throughput + {K2*Minimum Throughput} / 256-Load) + (K3*Total Latency) + (K6*Extended Attributes)]* [K5/(K4 + Reliability)]

== Will expand on wide metrics later. Honestly dont know very much about them.==


## K-Values

**Default K values:**

* K1 - 1 (Bandwidth)
* K2 - 0 (Load)
* K3 - 1 (Delay)
* K4 - 0 (Reliability) 
* K5 - 0 (MTU)


Bandwidth is the lowest bandwidth link in the path

Delay is the accumulated delay between this router and the destination network

#### Configuration:

```
! Under router configuration mode, the below command will set the K values
R1(config-router)#metric weights 0 1 0 1 0 0

! The first 0 refers to TOS, 0 is the only valid option here
! The following numbers are K1, K2, K3, K4, and K5 respectively

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

show ip eigrp topology [x.x.x.x/xx]

```

## References:

[EIGRP Wide Metrics](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_eigrp/configuration/15-s/ire-15-s-book/ire-wid-met.pdf)

[IP Routing EIGRP Configuration Guide - Chapter: EIGRP Wide Metrics](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_eigrp/configuration/15-sy/ire-15-sy-book/ire-wid-met.html)