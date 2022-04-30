---
layout: post
---

# Sources of DNS information hiding <!-- omit in toc -->
{: .no_toc}

One of the recurring discussions in the ICANN [NCAP](https://www.icann.org/en/announcements/details/icann-publishes-name-collision-analysis-project-ncap-study-2-documents-27-1-2022-en)
(Name Collision Analysis Project) is what relevant information about the global DNS namespace can be extracted from the root servers.

Increasingly the DNS hides information, both to improve end user privacy as well as to increase performance and reliably.

{:toc}

## QNAME Minimization
[DNS Query Name Minimisation to Improve Privacy](https://datatracker.ietf.org/doc/rfc9156/)
(originally published in 2016), only sends the minimum required part of the query name to the authoritative servers.

*Without* QNAME Minimization (assuming an empty cache), a recursive server will send the full query name (www.foo.example.com) to the root servers, and the full query name to the .com authoritative servers, and the full query name  to the example.com authoritative servers. This leaks potentially sensitive information about the query name to the root servers, and the .com servers. There is no reason that the root needs know anything further than the TLD, nor any reason that the .com servers need to know anything further than example.com.

With QNAME Minimization the resolver limits the root query to only asking where .com is, and then it will ask the .com servers for the authoritative servers for example.com, before finally asking the example.com servers for the answer for www.foo.example.com.

This is a privacy win, but elides information that might be useful for researchers, and may also have a negative impact on performance (e.g if resolving www.example.co.za, if the same servers serve both .co.za and .za this may result in an additional lookup).

## Aggressive NSEC
[RFC 8198 - Aggressive Use of DNSSEC-Validated Cache](https://datatracker.ietf.org/doc/rfc8198/) increases performance, decreases resource utilization on both authoritative and recursive servers, and improves privacy.

When talking about DNSSEC, people generally talk about proving that a specific record is "valid": that the answer 192.0.2.1 is the valid A record for www.example.com. What is less commonly discussed is that DNSSEC also provides a mechanism to cryptographically prove that a name does **not** exist, and it
accomplishes this without having to do online signing operations.

It accomplishes this seemingly magical feat using NSEC and NSEC3 records.
The way that this works is that the zone contains a sorted list of all of the
names that do exist and then also signs the holes between these names.

Example:
```
>$ dig +dnssec potato @b.root-servers.net

;; AUTHORITY SECTION:
.			86400	IN	SOA	a.root-servers.net. nstld.verisign-grs.com. 2022022601 1800 900 604800 86400
post.			86400	IN	NSEC	pr. NS DS RRSIG NSEC
.			86400	IN	NSEC	aaa. NS SOA RRSIG NSEC DNSKEY
.			86400	IN	RRSIG	SOA 8 0 86400 20220311170000 20220226160000 9799 . IprOYs0O... dl9MKg==
post.			86400	IN	RRSIG	NSEC 8 1 86400 20220311170000 20220226160000 9799 . dNxQV...vfQ==
.			86400	IN	RRSIG	NSEC 8 0 86400 20220311170000 20220226160000 9799 . SJHRSa...Z8Xg==

```

This means that a query for a name that is not in a delegated TLD (e.g example.potato) will not be sent to the root servers - in fact, the resolver won't
send any queries at all - it already knows that .potato doesn't exist (nothing
exists between .post and .pr), and so, by definition, example.foobar doesn't
exist either.

## LocalRoot
[RFC8806 - Running a Root Server Local to a Resolver](https://datatracker.ietf.org/doc/rfc8806/) describes a method for the operator of a recursive resolver to have a complete root zone locally and to hide queries for the root zone from outsiders. This mechanism is often called LocalRoot or "[Hyperlocal root service](https://www.icann.org/en/system/files/files/octo-027-25aug21-en.pdf)", and is implementable in most major recursive resolver software (BIND, Knot, Microsoft Windows Server 2012, Unbound, etc).

By design, a resolver that is configured to use LocalRoot will not send queries
to the root servers, and so queries sent to these resolvers will not be
visible in root server logs.


## Caching
One of the largest sources of DNS information hiding is the caching built into
the DNS protocol itself. When someone looks up a name (e.g www.example.com), the resolver will first check the cache for the answer. If it has the answer
in the cache, it will return it. If the answer is not already cached, the
resolver will try and resolve the name, starting from the most specific answer
that it **does** know, and will then cache all of the answers that it collected
while resolving the name. This will include caching the answers for .com (for
the TTL of 172800 seconds). This caching means that any additional queries for
www.example.com, or foo.example.com, or **any other subdomains** of .com will
not be sent to the root servers (until the TTL for .com expires).

The situation is a bit more complicated when the query is for a name that doesn't exist - the resolver will first check the cache for the answer. If it does not have the answer, it will query the authoritative servers for the domain (in the case of something like an undelegated TLD like foo.nonexistent this would be the root). Assuming that there is no QNAME Minimization or Aggressive NSEC, the resolver asks for the full name - foo.nonexistant. It gets back a response containing the response NXDOMAIN (no such domain) for the full name, and caches this for the negative TTL.


Example of resolving a name that doesn't exist:

```
$ dig foo.nonexistant @b.root-servers.net

; <<>> DiG 9.10.6 <<>> foo.nonexistant @b.root-servers.net
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 395
;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; QUESTION SECTION:
;foo.nonexistant.		IN	A

;; AUTHORITY SECTION:
.			86400	IN	SOA	a.root-servers.net. nstld.verisign-grs.com. 2022042700 1800 900 604800 86400
```

The resolver will cache the NXDOMAIN response for the full name, and, if it gets a query for any other subdomain of .nonexistant (e.g: bar.nonexistant), it will have to query for that name as well (and so be seen at the root). In the case of QNAME Minimization or Aggressive NSEC though, the resolver does not have to query for the other subdomains of .nonexistant. With QNAME Minimization, it will have cached the non-existence of .nonexistant; and, in the case of Aggressive NSEC, it doesn't even need to query for .nonexistant as it has cryptographic proof that it doesn't exist.

## Local Authoritative
One of the last major sources of information hiding is if there is a local resolver which is authoritative for a name. As an example, if an organization is using a local resolver to provide DNS services for a domain, it is not necessary to query the root servers for the domain. For example, a company using .corp within the organization (for example so that they can use accounting.corp as a name), any query for accounting.corp or foo.corp will be answered by the local resolver, and this will not be leaked into the public DNS. This is a very common deployment scenario, especially for Microsoft Active Directory.

## Summary
These are just a few of the mechanisms that hide information from DNS authoritative servers, and make it impossible to get a full picture of the DNS LANDSCAPE.

# Appendix A
## Disclaimer
Note that this is writeup has many oversimplifications. There are many more causes of information hiding in the DNS, the description of caching leaves out many important bits, etc. This document is intended to be a general overview of the causes of DNS information hiding, and not a comprehensive description of the DNS.

## Todo
Add sections:
  * [x] Caching
  * [x] Local Root
  * [x] Local Authoritative
  * [ ] Search Lists.
