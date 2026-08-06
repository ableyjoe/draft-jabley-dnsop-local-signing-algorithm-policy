---
title: "Reflecting Local Policy on Signing Algorithms in DNSSEC Signature Validation"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-jabley-dnsop-local-signing-algorithm-policy-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Domain Name System Operations"
keyword:
 - dnssec
 - local policy
 - algorithm
 - PQC
venue:
  group: "Domain Name System Operations"
  type: "Working Group"
  mail: "dnsop@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dnsop/"
  github: "ableyjoe/draft-jabley-dnsop-local-signing-algorithm-policy"
  latest: "https://ableyjoe.github.io/draft-jabley-dnsop-local-signing-algorithm-policy/draft-jabley-dnsop-local-signing-algorithm-policy.html"

author:
 -
    fullname: "Joe Abley"
    organization: Cloudflare
    email: "jabley@cloudflare.com"

normative:

informative:

--- abstract

Security-aware resolvers validate signatures, where available, in
order to protect their clients from data that is known to be
inauthentic. DNSSEC treats all algorithms as equal when it comes
to validation, such that any single valid signature is sufficient
proof of authenticity, and that data is only to be judged to be
inauthentic if all available signatures are found to be invalid.

However, a resolver might have a different local policy, e.g. to
interpret a failure to validate a signature made with an algorithm
that is considered to be post-quantum-safe as evidence of inauthentic
data regardless of whether a valid signature by a weaker, non-PQ-safe
algorithm also exists.  This document discusses such local policy
and provides a means to indicate to a client that local policy has
been applied to response validation.

--- middle

# Introduction

DNS Security Extensions (DNSSEC) are specified in {{!RFC 9364}}.

DNSSEC provides a mechanism for a subject RRSet to have cryptographic
signatures attached. Signatures are encoded in a DNS message alongside
the subject RRSet as an RRSIG RRSet with the same owner name, TTL
and class. Different RRSIG RRs in such an RRSet might correspond
to different signing keys and different signing algorithms.

In {{!Section 5.3.3 of RFC4035}} the process for determining the
authenticity of an RRSet with more than one associated signature (RRSIG)
is considered. The specification {{!RFC4035}} defers the matter of
whether every RRSIG needs to be checked individually to local policy:

>   If other RRSIG RRs also cover this RRset, the local resolver security
>   policy determines whether the resolver also has to test these RRSIG
>   RRs and how to resolve conflicts if these RRSIG RRs lead to differing
>   results.

This guidance is further clarified in {{!Section 5.4 of RFC6840}} as
follows:

>    This document specifies that a resolver SHOULD accept any valid RRSIG
>    as sufficient, and only determine that an RRset is Bogus if all
>    RRSIGs fail validation.
> 
>    If a resolver adopts a more restrictive policy, there's a danger that
>    properly signed data might unnecessarily fail validation due to cache
>    timing issues.  Furthermore, certain zone management techniques, like
>    the Double Signature Zone Signing Key Rollover method described in
>    Section 4.2.1.2 of [RFC6781], will not work reliably.  Such a
>    resolver is also vulnerable to malicious insertion of gibberish
>    signatures.

This document describes one such "more restrictive policy" relating to the
introduction of post-quantum-safe algorithms in DNSSEC.

# Conventions and Definitions

#{::boilerplate bcp14-tagged}

This document uses DNS terminology as described in {{!RFC9499}}.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
