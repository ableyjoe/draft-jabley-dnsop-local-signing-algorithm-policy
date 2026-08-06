---
title: "Supporting Quantum-Safe Algorithms in DNSSEC with Local Validation Policy"
abbrev: "Quantum-Safe Local Validation Policy for DNSSEC"
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
  FIPS204:
    title: "Module-Lattice-Based Digital Signature Standard"
    author:
      - org: "National Institute of Standards and Technology (NIST)"
    date: 2024-08
    seriesinfo:
      FIPS: PUB 204
    target: https://doi.org/10.6028/NIST.FIPS.204

--- abstract

Security-aware resolvers validate signatures, where available, in
order to protect their clients from inauthentic data.  DNSSEC treats
all algorithms as equal when it comes to validation, such that a
single valid signature is considered sufficient proof of authenticity,
and that data is only to be judged to be inauthentic if all available
signatures are found to be invalid. However, a resolver might have
a different local policy, e.g. in its handling of quantum-safe
signatures.  This document discusses such local policy and describes
a means to indicate to a client that specific local policy has been
applied to response validation.

--- middle

# Introduction {#intro}

DNS Security Extensions (DNSSEC) are specified in {{!RFC9364}}.

DNSSEC provides a mechanism to attach cryptographic signatures to
a subject RRSet.  Signatures are encoded in a DNS message alongside
the subject RRSet as an RRSIG RRSet with the same owner name, TTL
and class. Different RRSIG RRs in such an RRSet might correspond
to different signing keys and different signing algorithms.

In {{Section 5.3.3 of !RFC4035}} the process for determining the
authenticity of an RRSet with more than one associated signature (RRSIG)
is considered. The specification {{!RFC4035}} defers the matter of
whether every RRSIG needs to be checked individually to local policy:

>   If other RRSIG RRs also cover this RRset, the local resolver security
>   policy determines whether the resolver also has to test these RRSIG
>   RRs and how to resolve conflicts if these RRSIG RRs lead to differing
>   results.

This guidance is further clarified in {{Section 5.4 of !RFC6840}} as
follows:

>    This document specifies that a resolver SHOULD accept any valid RRSIG
>    as sufficient, and only determine that an RRset is Bogus if all
>    RRSIGs fail validation.
>
>    If a resolver adopts a more restrictive policy, there's a danger that
>    properly signed data might unnecessarily fail validation due to cache
>    timing issues.  Furthermore, certain zone management techniques, like
>    the Double Signature Zone Signing Key Rollover method described in
>    {{Section 4.2.1.2 of ?RFC6781}}, will not work reliably.  Such a
>    resolver is also vulnerable to malicious insertion of gibberish
>    signatures.

This document describes one such "more restrictive policy" relating
to the introduction of quantum-safe algorithms in DNSSEC, and
describes a signal that can be used to inform a relying party that
such a policy is in place.


# Conventions and Definitions

This document uses DNS terminology as described in {{!RFC9499}}.

This document applies the phrase "quantum-safe" to cryptographic
algorithms and signatures made by those algorithms in the loose
sense of offering acceptable protection against cryptanalyic attack
by a quantum computer. This usage is consistent with general,
contemporary discussion of so-called post-quantum cryptography.

This document uses the phrase "quantum-unsafe" to mean something
that is not quantum-safe.


# Deployment of Quantum-Safe Algorithms in DNSSEC {#local_policy}

Certain cryptographic algorithms have been identified as being to
weak to withstand cryptanalytic attack by a quantum computer.
Examples of such algorithms used in DNSSEC are ECDSA Curve P-256
with SHA-256 {{?RFC6605}} and RSA/SHA-256 {{?RFC5702}}. In this
document we refer to such algorithms as quantum-unsafe.

However, there exist other algorithms that are considered quantum-safe,
such as Module-Lattice-Based Digital Signature Standard {?FIPS204}
whose use in DNSSEC as ML-DSA-44 is described in
{{?I-D.westerbaan-dnssec-mldsa}}.

Quantum-safe algorithms are not widely-deployed in DNSSEC at the
time of writing, and there is no known prediction of their rapid
deployment. DNSSEC validators are required to ignore signatures
made using algorithms that they do not support.

To publish data in the DNS with broad integrity protection for
relying parties that includes the use of quantum-safe algorithms,
it is therefore necessary to include multiple signatures over the
subject data: signatures with widely-deployed, quantum-unsafe
algorithms for the legacy population of validators and also signatures
with poorly-deployed, quantum-safe algorithms intended for validators
that support them.  Signing with quantum-safe algorithms alone would
afford no integrity protection at all to the legacy validator
population.

The classsification of algorithms as quantum-safe or quantum-unsafe
imagines a future in which the ability to validate a signature made
by a quantum-unsafe algorithm no longer provides sufficient confidence
that the signed data is authentic. However, a signature over the same
data by a quantum-safe algorithm would provide that confidence, to
a validator that is equipped to use it.

Such a validator might therefore implement a local policy to ignore
signatures made using quantum-unsafe algorithms when signatures
made by quantum-safe algorithms are also available for the same
subject data. Such a policy would treat data with a valid quantum-safe
signature as authentic regardless of the validity of any other
signature, and treat data with an invalid quantum-safe signature
as inauthentic even if valid signatures made by quantum-unsafe
algorithms are available.

The operator of a security-aware resolver that adopted this local
policy would naturally pay close attention to the concerns expressed
in {{Section 4.2.1.2 of !RFC6781}}.

# Signals to Relying Parties {#filtered_ede}

The local policy described in {{local_policy}} would have the effect
of suppressing DNS responses that might otherwise have been returned
to a client in the specific case where validation of a quantum-unsafe
signature succeded while validation of a quantum-safe signature over
the same data did not.

To the end-user, this is an example of DNS response filtering and
existing mechanisms described in {{!I-D.ietf-dnsop-filtering-transparency}}
can be used. For example, a negative DNS response that follows a
failure to validate according to this local policy mnight include
an Extended DNS Error Code 6 ("DNSSEC Bogus") {{!RFC8914}} and
include indirect references to the specific policy in force encoded in
the EXTRA-TEXT field, for example:

~~~
{
  "fbds": [
    {
      "db": "resolver-operator-reference",
      "id": "quantum-safe-validation-policy"
    }
  ]
}
~~~


# Security Considerations

This document provides an example of local policy relating to DNSSEC
validation that could be employed by the operator of a security-aware
resolver to address weaknesses in quantum-unsafe signing algorithms.

Local policy is often no friend of interoperability. In this case
the parties affected by local policy decisions might well have made
an informed decision to use a resolver with known local policy, in
which case resulting differences in validation behaviour will
presumably be well-aligned with the relying parties. However,
particular resolvers are also commonly assigned to devices without
an informed decision-making process with an end user, in which case
differences in behaviour might be surprising.

The Extended DNS Error described in {{filtered_ede}}} provides a
mechanism for a resolver operator to communicate the existence of
local policy to a client such that the reason for the different
behaviour can be better understood.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

Your name here, etc.

