---
title: EIGRP Authentication
date: 2024-03-18 12:00:00 -0500
categories: [Layer 3 Technologies,EIGRP]
tags: [eigrp,enarsi]     # TAG names should always be lowercase
---

Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!


## EIGRP Authentication

* Authentication between neighbors is used to ensure that only authorized devices are allowed to participate in the EIGRP autonomous system.
* Authentication is configured with a pre-shared key
* Only MD5 is availble in EIGRP classic mode, but in EIGRP named mode, both MD5 and HMAC-SHA-256 are available

## Configuration

**Configuration is done in two steps:**

* Configure a key chain
* Apply the key chain to be used by the eigrp process
  * If using EIGRP named mode with HMAC-SHA-256, a key chain is not required



```
!Key Chain Configuration:
key chain key
 key 1
  key-string cisco

!Apply the key chain to the EIGRP process under interface configuration mode (Classic mode):
interface GigabitEthernet0/0
 ip authentication mode eigrp 1 md5
 ip authentication key-chain eigrp 1 key

!Apply the key chain to the EIGRP process under af-interface configuration mode (Named Mode):
router eigrp A
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface GigabitEthernet0/2
   authentication mode md5
   authentication key-chain key
  exit-af-interface


! Configure HMAC-SHA-256 authentication under af-interface configuration mode (Named Mode):
router eigrp A
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface GigabitEthernet0/2
   authentication mode hmac-sha-256 CCNP 
   authentication key-chain key
  exit-af-interface

!Because this password "CCNP" is applied to the auth mode command, this can be configured with the key-chain or without it

```

## Debugs & Show Commands


```
! EIGRP CLASSIC MODE:
Show ip eigrp interfaces detail gig 0/0
 EIGRP-IPv4 Interfaces for AS(1)
                               Xmit Queue   PeerQ        Mean   Pacing Time   Multicast    Pending
 Interface              Peers  Un/Reliable  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
 Gi0/0                    1        0/0       0/0        1943       0/0         7772           0
   Hello-interval is 5, Hold-time is 15
   Split-horizon is enabled
   Next xmit serial <none>
   Packetized sent/expedited: 2/0
   Hello's sent/expedited: 556/3
   Un/reliable mcasts: 0/1  Un/reliable ucasts: 2/4
   Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
   Retransmissions sent: 2  Out-of-sequence rcvd: 1
   Topology-ids on interface - 0 
   Authentication mode is md5,  key-chain is "key"
   BFD is enabled
   Topologies advertised on this interface:  base
   Topologies not advertised on this interface:


! EIGRP NAMED MODE:
show eigrp address-family ipv4 interfaces detail gig 0/2
 EIGRP-IPv4 VR(A) Address-Family Interfaces for AS(1)
                               Xmit Queue   PeerQ        Mean   Pacing Time   Multicast    Pending
 Interface              Peers  Un/Reliable  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
 Gi0/2                    1        0/0       0/0           3       0/0           50           0
   Hello-interval is 5, Hold-time is 15
   Split-horizon is enabled
   Next xmit serial <none>
   Packetized sent/expedited: 5/0
   Hello's sent/expedited: 619/6
   Un/reliable mcasts: 0/9  Un/reliable ucasts: 5/6
   Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
   Retransmissions sent: 1  Out-of-sequence rcvd: 5
   Topology-ids on interface - 0 
   Authentication mode is HMAC-SHA-256, key-chain is "key"
   Topologies advertised on this interface:  base
   Topologies not advertised on this interface:


debug eigrp packets
```

## References:


[EIGRP Message Authentication Configuration Example](https://www.cisco.com/c/en/us/support/docs/ip/enhanced-interior-gateway-routing-protocol-eigrp/82110-eigrp-authentication.html)