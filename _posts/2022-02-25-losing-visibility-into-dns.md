---
layout: post
---

# Sources of DNS information hiding
One of the recurring discussions in the 
[NCAP](https://www.icann.org/en/announcements/details/icann-publishes-name-colli
sion-analysis-project-ncap-study-2-documents-27-1-2022-en)
work is what visibility one can get into DNS from the root servers.

[TOC]

## QNAME Minimization
[DNS Query Name Minimisation to Improve 
Privacy](https://datatracker.ietf.org/doc/rfc9156/)
(originally published in 2016) only sends the minimum required part of the 
query name to the authoritative servers.


/Without/ QNAME Minimization (assuming an empty cache), a recursive server will 
send the full query name (www.foo.example.com) to the root servers, and the 
full query name to the .com authoritative servers, and the full query name  to 
the example.com authoritative servers. This leaks a potentially sensitive 
information about the query name to the root servers, and the .com servers. 
There is no reason that the root needs know anything further than the TLD, nor 
any reason that the .com servers need to know anything further than example.com.

With QNAME Minimization thw resolver limits the root query to only ask where 
.com is, and then it will ask the .com servers for the authoritative servers 
for example.com, before finally asking the example.com servers for the answer 
for www.foo.example.com.

This is a privacy win, but elides information that might be useful for 
researchers, and may also have a negative impact on performance (e.g if 
resolving www.example.co.za, if the same servers serve both .co.za and .za this 
may result in an additional lookup).

## Aggressive NSEC
[RFC 8198 - Aggressive Use of DNSSEC-Validated 
Cache](https://datatracker.ietf.org/doc/rfc8198/) increases performance, 
decreases and resource utilization on both authoritative and recursive servers, 
as well as improving privacy.

When talking about DNSSEC, people generally talk about proving that a specific 
record is "valid": that the answer 192.0.2.1 is the valid A record for 
www.example.com. What is less commonly discussed is that DNSSEC also provides a 
mechanism to cryptographially prove that a name does *not* exist.

