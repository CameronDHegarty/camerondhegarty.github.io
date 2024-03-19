---
title: Time Based Access-Lists
date: 2024-03-19 15:54:00 -0500
categories: [Infrastructure Security,Router Security]
tags: [routersecurity,cisco,enarsi]     # TAG names should always be lowercase
---

Note: Updates will continue to be made to this as I learn more about this topic and how it interacts with others. Additionally, if any mistakes are found, please email me and I will correct. Any corrections would be welcome!



## Time based ACL

* Can only use IPv6 ACLs or IPv4 extended ACLs as time based ACLs
* Created by associating a time range to an ACL permit or deny statement
* Time ranges can be configured with absolute times or periodic 

### Configuration:

```
! Create a periodic time range
R6(config)#time-range BUSSINESS-HOURS
R6(config-time-range)#?
Time range configuration commands:
  absolute  absolute time and date
  default   Set a command to its defaults
  exit      Exit from time-range configuration mode
  no        Negate a command or set its defaults
  periodic  periodic time and date

R6(config-time-range)#periodic ?
  Friday     Friday
  Monday     Monday
  Saturday   Saturday
  Sunday     Sunday
  Thursday   Thursday
  Tuesday    Tuesday
  Wednesday  Wednesday
  daily      Every day of the week
  weekdays   Monday thru Friday
  weekend    Saturday and Sunday

R6(config-time-range)#periodic weekdays ?
  hh:mm  Starting time

R6(config-time-range)#periodic weekdays 08:00 ?
  to  ending day and time

R6(config-time-range)#periodic weekdays 08:00 to ?
  hh:mm  Ending time - stays valid until beginning of next minute

R6(config-time-range)#periodic weekdays 08:00 to 17:00 ?
  <cr>

R6(config-time-range)#periodic weekdays 08:00 to 17:00


! Apply the time range to an ACL
ip access-list extended EXTENDED-ACL
 deny   tcp 10.3.6.0 0.0.0.255 host 6.6.6.6 eq www time-range BUSSINESS-HOURS
 permit ip any any


R6#show clock
*23:26:34.097 UTC Tue Mar 19 2024

R6#show access-lists
Extended IP access list EXTENDED-ACL
    10 deny tcp 10.3.6.0 0.0.0.255 host 6.6.6.6 eq www time-range BUSSINESS-HOURS (inactive)
    20 permit ip any any

*Mar 19 09:15:00.000: %SYS-6-CLOCKUPDATE: System clock has been updated from 23:27:08 UTC Tue Mar 19 2024 to 09:15:00 UTC Tue Mar 19 2024, configured from console by console.

R6#clock set 09:15:00 19 mar 2024

R6#show access-lists
Extended IP access list EXTENDED-ACL
    10 deny tcp 10.3.6.0 0.0.0.255 host 6.6.6.6 eq www time-range BUSSINESS-HOURS (active)
    20 permit ip any any


```


## Debugs & Show Commands

```
R6#show access-lists
Extended IP access list EXTENDED-ACL
    10 deny tcp 10.3.6.0 0.0.0.255 host 6.6.6.6 eq www time-range BUSSINESS-HOURS (active)
    20 permit ip any any

R6#show time-range
time-range entry: BUSSINESS-HOURS (active)
   periodic weekdays 8:00 to 17:00
   used in: IP ACL entry
```

## References


