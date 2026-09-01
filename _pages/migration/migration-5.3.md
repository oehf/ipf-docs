---
title: IPF 5.3 Migration Guide
layout: single
permalink: /docs/migration-5.3/
toc: true
toc_icon: align-left
toc_sticky: true
---

IPF 5.3 is a small, compatible release. Its main theme is FHIR-based ATNA auditing, which is now implemented
consistently across all supported FHIR transactions; the price is a handful of renamed and deprecated audit
classes. Everything else is dependency maintenance, with one behavioral change inherited from Apache CXF
that is worth understanding if you serve asynchronous Web Service transactions.

IPF 5.3 is also the release in which **FHIR STU3 support is deprecated**. See
[below](#fhir-stu3-support-is-deprecated) and the [outlook on IPF 6.0](#outlook-ipf-60).

## At a glance

| Change | Relevant if you… | What to do |
|--------|------------------|------------|
| [BALP audit classes renamed](#fhir-based-atna-auditing-balp) | reference `BalpJwt*` classes or `BalpAuditContext` | rename imports; move to `AuditContext` |
| [FHIR auditing completed](#fhir-based-atna-auditing-balp) | audit PIXm, PDQm or MHD transactions | re-check which audit records you now get |
| [POST query auditing](#search-criteria-of-post-queries-are-audited) | run FHIR queries as `POST [type]/_search` | none — the audit record simply becomes complete |
| [CXF decoupled destinations](#asynchronous-web-services-and-cxf-418) | serve async transactions (XCPD, XCA, XDR) | nothing in IPF; check any custom CXF endpoints |
| [FHIR STU3 deprecated](#fhir-stu3-support-is-deprecated) | depend on any `*-fhir-stu3-*` module | plan the move to R4 before IPF 6.0 |

## Environment

Unchanged: Java 17 baseline, and IPF builds and runs on JDK 21 and JDK 25 as of
[IPF 5.2]({{ site.baseurl }}{% link _pages/migration/migration-5.2.md %}).

## Third-party dependencies

IPF 5.3 only refreshes dependencies within their existing lines:

| Library | IPF 5.2.0 | IPF 5.3.0 |
|---------|-----------|-----------|
| Apache Camel | 4.18.2 | 4.18.4  |
| Apache CXF | 4.1.6 | 4.1.8   |
| Groovy | 5.0.6 | 5.0.8   |
| HAPI FHIR | 8.10.0 | 8.10.1  |
| Spring Boot | 3.5.14 | 3.5.16  |
| Jackson | 2.21.2 | 2.22.1  |
| Woodstox | 7.1.1 | 7.2.1   |
| Saxon-HE | 12.9 | 12.10   |

The CXF update carries a behavioral change, described below.

## FHIR-based ATNA auditing (BALP)

FHIR-based auditing — writing `AuditEvent` resources following the IHE
[BALP](https://profiles.ihe.net/ITI/BALP/index.html) profile, as an alternative to DICOM-based ATNA
auditing — is now implemented for **all** supported FHIR transactions
([#509](https://github.com/oehf/ipf/issues/509)): PIXm [ITI-83], PDQm [ITI-78] and [ITI-119], and MHD
[ITI-65], [ITI-66], [ITI-67], [ITI-68] and [ITI-105].

If you already use FHIR auditing, expect audit records for transactions that previously produced none, or
more complete ones where previously support was partial. Review what your FHIR audit repository receives.

### Renamed classes

The `Balp` prefix has been dropped where the functionality turned out not to be BALP-specific:

| Until IPF 5.2 | As of IPF 5.3 |
|---------------|---------------|
| `org.openehealth.ipf.commons.audit.BalpJwtExtractorProperties` | `org.openehealth.ipf.commons.audit.JwtExtractorProperties` |
| `org.openehealth.ipf.commons.ihe.fhir.audit.auth.BalpJwtClaimsExtractor` | `org.openehealth.ipf.commons.ihe.fhir.audit.auth.JwtClaimsExtractor` |
| `org.openehealth.ipf.commons.ihe.fhir.audit.events.BalpJwtUtils` | `org.openehealth.ipf.commons.ihe.fhir.audit.events.JwtUtils` |

Referencing code must be adapted.

### Deprecated audit contexts

`BalpAuditContext` and `DefaultBalpAuditContext` are deprecated and will be removed in the next release:

| Until IPF 5.2 | As of IPF 5.3 |
|---------------|---------------|
| `BalpAuditContext` | `AuditContext` |
| `DefaultBalpAuditContext` | `DefaultAuditContext` |

The reason is that neither of the two properties these types added is specific to BALP: the audit repository
context path belongs to the transport addressing it, and the token claim paths belong to the token. Both now
sit on `AuditContext` itself. If you declare a `DefaultBalpAuditContext` bean, change it to `DefaultAuditContext`; the properties keep their names.

### Correlating the two ends of a transaction

BALP asks for a correlation id shared by the audit records that both ends of a transaction write. IPF picks
it up from an HTTP header, configurable in the [ATNA Spring Boot starter]:

| Property (`ipf.atna.`)     | Default        | Description |
|----------------------------|----------------|-------------|
| `request-id-header-names`  | `X-Request-Id` | Header names that may carry the correlation id, most preferred first. The first header a request actually carries wins; an empty list disables the lookup. |
| `request-id-type`          | unset          | Participant object ID type to record the id under. Left unset, the type follows the header it was found in. |

A deployment that already propagates a trace can name its own headers instead — `traceparent` for W3C Trace
Context, or `b3` and `X-B3-TraceId` for the two B3 flavors.

## Search criteria of POST queries are audited

FHIR queries may be sent as `POST [type]/_search` with the search criteria in the request body rather than
in the URL. IPF used to leave the query string of the resulting audit record empty; the criteria are now
recorded ([#514](https://github.com/oehf/ipf/issues/514)). This affects, for example,
[ITI-78]({{ site.baseurl }}{% link _pages/ihe/fhir/iti78.md %}). No action is required — audit records
simply become complete. Tests that assert on an empty query string need updating.

## Asynchronous Web Services and CXF 4.1.8

Since CXF 4.1.8, WS-Addressing *decoupled destinations* — non-anonymous `wsa:ReplyTo` and `wsa:FaultTo`
endpoint references — are rejected by default, as a defense against SSRF. Without an opt-in, CXF answers
such a request with a `wsa:DestinationUnreachable` SOAP fault instead of dispatching the response to the
given URI.

Asynchronous responses over WS-Addressing are a regular part of several IHE transactions (XCPD, XCA, XDR).
IPF therefore pre-approves them per exchange, for those endpoints only whose transaction configuration declares
that it allows asynchrony.

Two things to keep in mind:

* CXF's URI scheme allowlist (`org.apache.cxf.ws.addressing.decoupled.allowedSchemes`, by default `http://`,
  `https://` and a few others) remains effective.
* Web Service endpoints you build outside IPF's IHE components are not covered by this and are subject to
  CXF's new default.

See [asynchronous Web Service transactions]({{ site.baseurl }}{% link _pages/ihe/ws/wsCxfAsync.md %}) for
how asynchrony works in IPF.

## New ITI-18 stored query

[ITI-18]({{ site.baseurl }}{% link _pages/ihe/xds/iti18.md %}) Registry Stored Query supports the
`FindDocumentsExclude` stored query and the corresponding option, following CP-ITI-1323
([#515](https://github.com/oehf/ipf/issues/515)). This is an addition; existing queries are unaffected. Note that CP-ITI-1323 has not yet
been included into the main text, so consider this as a preview only - and expect potential changes until 
it has become final.

## FHIR STU3 support is deprecated

All FHIR **STU3** support is deprecated as of IPF 5.3 and **will be removed in IPF 6.0**. This covers the
STU3 variants of the MHD, PIXm/PDQm and QEDm transactions, i.e. all modules whose artifact id contains
`fhir-stu3`, including `ipf-fhir-stu3-spring-boot-starter`.

The reason is that IHE has been publishing its FHIR-based profiles against R4 since ITI revision 16
(2019/2020); the STU3 variants have had no specification-side reason to exist for years. Keeping them
alive costs a second code path for every FHIR transaction.

If you still deploy STU3 endpoints, plan the move to R4 during the 5.3 lifetime:

* replace `ipf-fhir-stu3-spring-boot-starter` with `ipf-fhir-r4-spring-boot-starter`, or the corresponding
  `*-fhir-stu3-*` modules with their `*-fhir-r4-*` counterparts,
* set `fhirVersion` to `R4` where you configure it explicitly (see [FHIR deployment]),
* and adapt your code to the R4 resource model of the HAPI FHIR structures.

There is no functional replacement for STU3 within IPF; R4 is the target. The next FHIR version to be
included will probably be R6 (skipping R5).

## Outlook: IPF 6.0

The next major release, **IPF 6.0**, will move the platform to the current generation of its foundations:

* **Spring Framework 7** and **Spring Boot 4.x**
* **Jakarta EE 11**
* correspondingly current versions of Apache Camel, Apache CXF and the other core dependencies

Expect a migration effort comparable to
[IPF 5.0]({{ site.baseurl }}{% link _pages/migration/migration-5.0.md %}), which moved IPF to Jakarta EE 10,
rather than to a minor release. Most of the work will be in the third-party upgrades rather
than in IPF's own API. In addition, deprecated classes and methods — including all FHIR STU3 support — will
be removed.

The complete list of resolved issues is in the
[5.3.0 release notes](https://github.com/oehf/ipf/releases/tag/ipf-5.3.0).

[ATNA Spring Boot starter]: {{ site.baseurl }}{% link _pages/boot/boot-atna.md %}
[FHIR deployment]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirDeployment.md %}
[ITI-65]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti65.md %}
[ITI-66]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti66.md %}
[ITI-67]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti67.md %}
[ITI-68]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti68.md %}
[ITI-78]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti78.md %}
[ITI-83]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti83.md %}
[ITI-105]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti105.md %}
[ITI-119]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti119.md %}
