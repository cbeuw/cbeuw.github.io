---
layout: post
title: CAA Footgun
---

CAA is a DNS record type which specifies the Certificate Authorities who are allowed to issue certificates for a domain.

At my job, we had a CAA record that looked like this
```
issue "letsencrypt.org"
```
This allows Let's Encrypt to issue `example.com` certs and `*.example.com` wildcard certs.

A bit later, we needed another CA to issue both single-domain and wildcard certificates on our domain. Without realising the `issue` directive allows for both, someone appended both `issue` and `issuewild` directives:

```
issue     "letsencrypt.org"
issue     "amazon.com"
issuewild "amazon.com"
```

A bit redundant, but it looks like it should still work.

...until our existing Let's Encrypt wildcard certificates suddenly stopped renewing.

Turns out that the `issue` directive allows for wildcard issuance **only in the absence** of any `issuewild` directive[^1]. As soon as an `issuewild` directive is added, CAs with only `issue` are forbidden from issuing wildcards.

This is a bad design. An additive modification should not remove capabilities of unrelated, existing entries.

This is bad for security too. Suppose you want only one CA to issue  only single-domain certificates for your domain. Well, this isn't "natively" supported. The best you can do is
```
issue "ca-i-want.com"
issuewild "a-registered-domain-thats-not-and-never-will-be-a-ca.com"
```
---
{: data-content="Footnotes"}

[^1]: <https://datatracker.ietf.org/doc/html/rfc8659#section-4.3-3>