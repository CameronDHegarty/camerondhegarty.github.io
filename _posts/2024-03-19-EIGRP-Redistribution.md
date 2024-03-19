---
title: EIGRP Redistribution
date: 2024-03-19 10:54:00 -0500
categories: [Layer 3 Technologies,EIGRP]
tags: [eigrp,enarsi]     # TAG names should always be lowercase
---

Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!

## Redistributing into EIGRP

When redistributing into EIGRP, the metric must be defined via the redistirbution command or by setting the default metric. If the metric is not manually set, it will be set to infinity and the route will not be inserted into the RIB.

The exception to this is if one EIGRP AS is being redistributed into another EIGRP AS, the metric will be carried over from the original AS.

## Configuration

```
! Below is the command to set the default metric to be used for redistribution
! When default metric is set, it does not need to be specified on the redistribution command
router eigrp 1
 default-metric 100000 100 255 1 1500
 redistribute connected
 redistribute ospf 1

! Without the default-metric command, the metric must be defined on the redistribution command
router eigrp 1
 redistribute connected metric 100000 100 255 1 1500
 redistribute ospf 1 metric 100000 100 255 1 1500

```

## References:

[Configure Protocol Redistribution for Routers](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/8606-redist.html#toc-hId-760653558)
