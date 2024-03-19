---
title: Console & VTY Lines
date: 2024-03-19 17:30:00 -0500
categories: [Infrastructure Services,Remote Access]
tags: [remotemanagement,console,vty,cisco,enarsi]     # TAG names should always be lowercase
---

## Console

* Configuration of access for admins directly connected to the console port

### Configuration:

* Not much configuration here. Generally not as protected as it relies on physical access to the device. If you limit access here it may stop you from being able to access the device if you get locked out via other login methods.

* Below commands are what I find useful in the lab at least

```
line con 0
 logging synchronous
 no domain-lookup
```

## VTY Lines

* Configuration of access for admins connecting to the device remotely via SSH or Telnet

### Configuration:

* ACL can be applied inbound to limit which ip addresses are able to connect to the device
* ACL can also be applied in the outbound direction. This is used to limit what IP addresses an admin can access who is telnetted into this router. 
    * This does not affect an admins ability to connect to another device who is connected to the router via another method than the VTY lines

* Transport command defines what protocals are able to connect to the VTY lines. Notable options are telnet, ssh, none, and all
    * Can also be specified for the outbound direction


```
access-list 1 permit 10.3.6.0 0.0.0.255
line vty 0 4
 access-class 1 in
 login local              ! Can also define AAA login method lists if AAA is enabled
 transport input telnet ssh

```

## Debug & Show Commands

```
! Display connected users:
R6#show users
    Line       User       Host(s)              Idle       Location
*  0 con 0                idle                 00:00:00
 578 vty 0     admin      idle                 00:00:09 10.3.6.3

  Interface    User               Mode         Idle     Peer Address

```

## References

[Security Configuration Guide - Chapter: Controlling Access to a Virtual Terminal Line](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_data_acl/configuration/15-mt/sec-data-acl-15-mt-book/sec-cntrl-acc-vtl.html)
