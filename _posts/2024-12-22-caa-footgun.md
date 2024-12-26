---
layout: post
title: CAA Footgun
---

CAA is a DNS record type which specifies the Certificate Authorities who are allowed to issue certificates for a domain.

At my job, we had a CAA record that looked like this
```
example.com CAA 0 issue "letsencrypt.org"
```
This allows Let's Encrypt to issue `example.com` certs and `*.example.com` wildcard certs. *The `0` is a flag used for CA-specific features, we can ignore it.*

A bit later, we needed another CA to issue both single-domain and wildcard certificates on our domain. Without realising the `issue` directive allows for both, someone appended both `issue` and `issuewild` directives:

```
example.com CAA 0 issue     "letsencrypt.org"
            CAA 0 issue     "amazon.com"
            CAA 0 issuewild "amazon.com"
```

A bit redundant, but it looks like it should still work.

...until our existing Let's Encrypt wildcard certificates suddenly stopped renewing.

Turns out that the `issue` directive allows for wildcard issuance **only in the absence** of any `issuewild` directive[^1]. As soon as an `issuewild` directive is added, CAs with only `issue` are forbidden from issuing wildcards.

This is a bad design. An additive modification should not remove capabilities of unrelated, existing entries.

On the other hand, suppose you want only one CA to issue only single-domain certificates for your domain, then you must do something like
```
issue     "letsencrypt.org"
issuewild ";"
```

---

[^1]: <https://datatracker.ietf.org/doc/html/rfc8659#section-4.3-3>
