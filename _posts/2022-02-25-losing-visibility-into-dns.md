---
layout: post
---

# Sources of DNS information hiding
One of the recurring discussions in the [NCAP](https://www.icann.org/en/announcements/details/icann-publishes-name-collision-analysis-project-ncap-study-2-documents-27-1-2022-en)
work is what visibility one can get into DNS from the root servers.

[TOC]

## QNAME Minimization
[DNS Query Name Minimisation to Improve Privacy](https://datatracker.ietf.org/doc/rfc9156/)
(originally published in 2016) only sends the minimum required part of the query name to
the authoritative servers. This means that, if you are trying to resolve
www.example.com, instead of sending the full query name (www.foo.example.com) to
the root servers, and the .com authoritative servers, and the .example.com authoritative
servers, it will only send the bits needed. So (assuming that the cache is cold/empty),
the recursive will only ask the root servers where .com is, and then will ask the
.com servers where .example.com is, and then will ask the .example.com servers where
www.example.com is.  This means that the only information available to the root servers
is the TLD being asked for, and the variety of second (and third, forth, etc) level
domains is not intentionally not visible.

## Aggressive NSEC
[RFC 8198 - Aggressive Use of DNSSEC-Validated Cache](https://datatracker.ietf.org/doc/rfc8198/)
increases performance, decreases and resource utilization on both authoritative and recursive servers, as well as improving privacy.
