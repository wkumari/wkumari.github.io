---
layout: post
---

# Fixing JUNOS next-hop resolution failures
{: .no_toc}

* TOC
{:toc}

## Overview
I've recently seen a number of instances where a Juniper router will simply not forward packets correctly, often onto a "LAN" (Ethernet with lots of hosts). Looking at the routing table seems to show that everything is fine, and pinging the host from the router works, but transit traffic appears to just disappear into a black hole. This can be really annoying and hard to debug.

These are my quick notes on troubleshooting and fixing this.
Note that the examples has been edited slightly to make it easier to read.

# Scenario / description of issue
Traffic coming in on `xe-1/0/1.0` destined for `192.0.2.48` and `192.0.2.49` on `xe-1/0/0.0` is being dropped. It isn't a firewall problem, and packets are definitely arriving at the router (trust me on this).

Pinging `192.0.2.48` from the router works just fine:
```
wkumari@r2> ping 192.0.2.48 count 1
PING 192.0.2.48 (192.0.2.48): 56 data bytes
64 bytes from 192.0.2.48: icmp_seq=0 ttl=64 time=2.085 ms

--- 192.0.2.48 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max/stddev = 2.085/2.085/2.085/0.000 ms
```
and that is indeed the correct ARP entry (confirmed on the device being pinged).
```
wkumari@r2> show arp no-resolve hostname 192.0.2.48
MAC Address       Address         Interface         Flags
aa:bb:bb:dd:11:7c 192.0.2.48  xe-1/0/0.0               none

wkumari@r2>
```


But, pinging from the "outside" doesn't work:
```
 ping -c 2 192.0.2.48
PING 192.0.2.48 (192.0.2.48) 56(84) bytes of data.

--- 192.0.2.48 ping statistics ---
2 packets transmitted, 0 received, +1 errors, 100% packet loss, time 1002ms
```


Looking at the routing table, everything is fine.

```
wkumari@r2> show route 192.0.2.48

inet.0: 813779 destinations, 1632343 routes (813775 active, 3 holddown, 1 hidden)
+ = Active Route, - = Last Active, * = Both

192.0.2.0/24   *[Direct/0] 2d 12:34:56
                    > via xe-1/0/0.0
                    [Static/5] 2d 12:35:14
                      Discard
                    [BGP/170] 2d 12:33:03, MED 0, localpref 200, from 192.168.254.12
                      AS path: I, validation-state: unverified
                    > to 192.0.2.20 via xe-1/0/0.0
```
and looking in the forwarding table everything also looks fine:
```
wkumari@r2> show route forwarding-table destination 192.0.2.48
Routing table: default.inet
Internet:
Destination        Type RtRef Next hop           Type Index    NhRef Netif
192.0.2.48/32  dest     0 aa:bb:bb:dd:11:7c  ucst      723     1 xe-1/0/0.0

...

wkumari@r2>
```


But, transit traffic still just disappears into a black hole...

## Cause / troubleshooting
Under some set of cases, the router and forwarding table will both be correctly programmed, complete with the next-hop, but the PFE isn't correctly programmed.

Example:
```
wkumari@r2# run show pfe route ip table index 0 prefix 192.0.2.48

Slot 2
================ fpc2 ================

IPv4 Route Table 0, default.0, 0x80000:
Destination                       NH IP Addr      Type     NH ID Interface
--------------------------------- --------------- -------- ----- ---------
192.0.2.48                                        Hold   631 RT-ifl 65543 xe-1/0/0.0 ifl 65543
```
The Next Hop IP Address in the PFE is stuck in Hold state. This means that, when the PFE is trying to forward the packet, it doesn't have a next-hop address to send it, and it (for some reason) isn't initializing a resolution.
The address can be pinged from the router the router itself has the next-hop resolved correctly (see above) and doesn't need to pass through the PFE.

## Resolution
This can be fixed by rebooting the router (erm...), or, more easily, by making the router redo the PFE configuration and resolution:
```
wkumari@r2# set route 192.0.2.48 retain next-hop 192.0.2.48
wkumari@r2# commit
wkumari@r2# delete route 192.0.2.48
wkumari@r2# commit
```
and now, looking in the PFE again, we see that the Next Hop IP Address is now resolved, and everything works correctly again:
```
wkumari@r2# show pfe route ip table index 0 prefix 192.0.2.48

Slot 2
================ fpc2 ================

IPv4 Route Table 0, default.0, 0x80000:
Destination                       NH IP Addr      Type     NH ID Interface
--------------------------------- --------------- -------- ----- ---------
192.0.2.48                    192.0.2.48   Unicast   631 RT-ifl 65543 xe-1/0/0.0 ifl 65543

```
and from the outside:
```
ping 192.0.2.48
PING 192.0.2.48 (192.0.2.48): 56 data bytes
64 bytes from 192.0.2.48: icmp_seq=0 ttl=44 time=56.667 ms
64 bytes from 192.0.2.48: icmp_seq=1 ttl=44 time=56.503 ms
64 bytes from 192.0.2.48: icmp_seq=2 ttl=44 time=56.648 ms
64 bytes from 192.0.2.48: icmp_seq=3 ttl=44 time=62.501 ms
^C
--- 192.0.2.48 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 56.503/58.080/62.501/2.553 ms
```
