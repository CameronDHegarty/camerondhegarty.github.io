---
title: IPv6 Traffic Filter
date: 2024-03-19 15:54:00 -0500
categories: [Infrastructure Security,Router Security]
tags: [routersecurity,ipv6,cisco,enarsi]     # TAG names should always be lowercase
---


Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!


## IPv6 Traffic Filters

* IPv6 traffic filters work almost the exact same way as IPv4 extended access lists
* They are created with names, there are no numberd IPv6 traffic filters

**Note:** Be very careful creating IPv6 traffic filters with a deny any at the end. Similar to IPv4, there is a implicit deny any at the end of IPv6 traffic filters. However, just before the implicit deny any, there are two "implicit" permits to allow ND-NA and ND-NS. Please see the below for the hidden rules at the end of every IPv6 traffic filter.

```
permit icmp any any nd-na
permit icmp any any nd-ns
deny ipv6 any any
```

If for whatever reason, you end up putting a deny any rule in an IPv6 traffic filter, make sure to put in the above permits before it. Failing to do so will break IPv6 operations. 

## Configuration

```
! Create the IPv6 ACL
ipv6 access-list FILTER
 deny ipv6 2001:DB8:1:1::/64 2001:DB8:2:2::/64
 permit ipv6 any any

! Apply it to an interface (it can be applied inbound our outbound)
interface GigabitEthernet0/0
 ipv6 traffic-filter FILTER in
```

## Debug and Show Commands

```
IPv6 access list FILTER
    deny ipv6 2001:DB8:1:1::/64 2001:DB8:2:2::/64 sequence 10
    permit ipv6 any any (3 matches) sequence 20
```

## References

[IPv6 Traffic Filtering Access List Configuration Example](https://www.cisco.com/c/en/us/support/docs/ip/ip-version-6/113126-ipv6-acl-00.html)


