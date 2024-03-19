---
title: Access-Lists
date: 2024-03-19 19:14:00 -0500
categories: [Infrastructure Security,Router Security]
tags: [routersecurity,cisco,enarsi]     # TAG names should always be lowercase
---

Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!


## Standard ACL

* Standard ACLs match only the source address
* Since they're only able to match on the source address they're typically applied closer to the destination, as to not unintentionally block traffic to other destinations

* Standard ACL's are numbered 1-99

### Configuration:

```
! Can configure with the "access-list x" command
access-list 1 deny   10.3.6.0 0.0.0.255
access-list 1 permit any

! or with the "ip access-list" command (this allows modifying the ACL later, without having to remove it entirely)
ip access-list standard STANDARD-ACL
 deny   10.3.6.0 0.0.0.255
 permit any

! Applied on the interface with the below command either inbound or outbound
ip access-group STANDARD-ACL in
```

## Extended ACL

* Extended ACLs can match on source and destination ip addresses, as well as source and destination ports
* Typically applied closer to the source since it's able to match on the destination address

* Extended ACL's are numbered 100-199

### Configuration:

```
! Can configure with the "access-list x" command
access-list 101 deny   tcp 10.3.6.0 0.0.0.255 host 6.6.6.6 eq www
access-list 101 permit ip any any

! or with the "ip access-list" command (this allows modifying the ACL later, without having to remove it entirely)
ip access-list extended EXTENDED-ACL
 deny   tcp 10.3.6.0 0.0.0.255 host 6.6.6.6 eq www
 permit ip any any

! Applied on the interface with the below command either inbound or outbound
ip access-group EXTENDED-ACL in
```

## Debugs & Show Commands

```
R6#show access-list
Standard IP access list 1
    10 deny   10.3.6.0, wildcard bits 0.0.0.255
    20 permit any
Standard IP access list STANDARD-ACL
    10 deny   10.3.6.0, wildcard bits 0.0.0.255
    20 permit any
Extended IP access list 101
    10 deny tcp 10.3.6.0 0.0.0.255 host 6.6.6.6 eq www
    20 permit ip any any
Extended IP access list EXTENDED-ACL
    10 deny tcp 10.3.6.0 0.0.0.255 host 6.6.6.6 eq www
    20 permit ip any any
```

## References


