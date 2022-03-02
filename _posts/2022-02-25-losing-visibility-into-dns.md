---
layout: post
---

# Sources of DNS information hiding <!-- omit in toc -->
One of the recurring discussions in the [NCAP](https://www.icann.org/en/announcements/details/icann-publishes-name-collision-analysis-project-ncap-study-2-documents-27-1-2022-en)
work is what visibility one can get into DNS from the root servers.

- [QNAME Minimization](#qname-minimization)
- [Aggressive NSEC](#aggressive-nsec)
- [LocalRoot](#localroot)
- [Caching](#caching)
  - [Todo](#todo)
  -

## QNAME Minimization
[DNS Query Name Minimisation to Improve Privacy](https://datatracker.ietf.org/doc/rfc9156/)
(originally published in 2016) only sends the minimum required part of the query name to the authoritative servers.


/Without/ QNAME Minimization (assuming an empty cache), a recursive server will send the full query name (www.foo.example.com) to the root servers, and the full query name to the .com authoritative servers, and the full query name  to the example.com authoritative servers. This leaks a potentially sensitive information about the query name to the root servers, and the .com servers. There is no reason that the root needs know anything further than the TLD, nor any reason that the .com servers need to know anything further than example.com.

With QNAME Minimization thw resolver limits the root query to only ask where .com is, and then it will ask the .com servers for the authoritative servers for example.com, before finally asking the example.com servers for the answer for www.foo.example.com.

This is a privacy win, but elides information that might be useful for researchers, and may also have a negative impact on performance (e.g if resolving www.example.co.za, if the same servers serve both .co.za and .za this may result in an additional lookup).

## Aggressive NSEC
[RFC 8198 - Aggressive Use of DNSSEC-Validated Cache](https://datatracker.ietf.org/doc/rfc8198/) increases performance, decreases and resource utilization on both authoritative and recursive servers, as well as improving privacy.

When talking about DNSSEC, people generally talk about proving that a specific record is "valid": that the answer 192.0.2.1 is the valid A record for www.example.com. What is less commonly discussed is that DNSSEC also provides a mechanism to cryptographically prove that a name does **not** exist, and it
accomplishes this without having to do online signing operations.

It accomplishes this seemingly magical feat using NSEC and NSEC3 records.
The way that this works is that the zone contains a sorted list of all of the
names that do exist, and then also signs the holes between these names.

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
exists between .post and .pr), and so, by definition, exmaple.foobar doesn't
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
the TTL - 172800 seconds). This caching means that any additional queries for
www.example.com, or foo.example.com, or **any other subdomains** of .com will
not be sent to the root servers.

In the case that a query is sent for a name that


### Todo
Additional causes - write these up.
  * [ ] Caching
  * [ ] Local Root
  * [ ] Local Authoritative
