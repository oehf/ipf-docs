---
title: IPF 5.2 Migration Guide
layout: single
permalink: /docs/migration-5.2/
toc: true
toc_icon: align-left
toc_sticky: true
---

IPF 5.2 is a compatible release: applications built against IPF 5.1 are expected to compile and run
unchanged. Most of the upgrade effort comes from the third-party updates — above all Apache Camel and
Groovy — rather than from IPF's own API.

Two areas deserve a closer look before you upgrade: FHIR-based transactions have gained stricter
validation, and a couple of TLS-related classes have moved to a different module.

## At a glance

| Change | Relevant if you… | What to do |
|--------|------------------|------------|
| [Apache Camel 4.10 → 4.18](#third-party-dependencies) | build Camel routes (everybody) | follow the Camel upgrade notes; align your own Camel version |
| [Groovy 4 → 5](#third-party-dependencies) | use the HL7v2 Groovy DSL or Groovy route builders | review the Groovy 5 release notes |
| [TLS classes moved](#tls-parameter-classes-moved) | reference `TlsParameters` or `CustomTlsParameters` directly | update the imports; the old classes still work but are deprecated |
| [PIXm/PDQm validation](#stricter-validation-for-pixm-and-pdqm) | serve or consume ITI-78, ITI-83 or ITI-119 | re-test with your real messages; previously accepted requests may now be rejected |
| [MHD 4.2.3](#mhd-support-updated) | serve or consume ITI-65, ITI-66, ITI-67, ITI-68 or ITI-105 | re-test; some classes changed in minor, incompatible ways |
| [`preferred` uniqueIds](#namingsystem-uniqueid-selection-honors-preferred) | resolve identifiers from `NamingSystem` resources | check which uniqueId your code expects |
| [Corrected codes](#corrected-codes-in-generated-responses-and-audit-records) | assert on generated NAKs or audit records in tests | update expected values |
| [Spring Boot SSL bundles](#spring-boot-ssl-bundles) | configure TLS in a Spring Boot application | optional: switch to `server.ssl.bundle` |

## Environment

IPF 5.2 keeps the Java 17 baseline, so no change is required to your toolchain. In addition, IPF now
builds and runs on **JDK 25** ([#493](https://github.com/oehf/ipf/issues/493)).

## Third-party dependencies

IPF 5.2 carries a large batch of dependency updates. The most relevant ones:

| Library | IPF 5.1.0 | IPF 5.2.0 |
|---------|-----------|-----------|
| Apache Camel | 4.10.6 | 4.18.2 |
| Groovy | 4.0.28 | 5.0.6 |
| HAPI FHIR | 8.2.1 | 8.10.0 |
| Apache CXF | 4.1.3 | 4.1.6 |
| Spring Boot | 3.5.5 | 3.5.14 |
| Kotlin | 2.2.0 | 2.2.21 |
| Jackson | 2.19.2 | 2.21.2 |
| Nimbus JOSE+JWT | 9.37.4 | 10.9 |
| Bouncy Castle | 1.81 | 1.84 |
| ipf-gazelle | 3.3.0 | 3.4.0 |

Two of these are more than routine patch bumps:

* **Apache Camel jumps eight minor versions**, from the 4.10 LTS line to 4.18. Work through the
  [Camel 4.x upgrade guide](https://camel.apache.org/manual/camel-4x-upgrade-guide.html) for the versions
  you skip. As always, we recommend aligning the Camel version used by your own application with the one
  IPF was built against.
* **Groovy moves to the 5.x line.** This affects you if you write HL7v2 processing code in Groovy or use
  Groovy-based route builders; see the [Groovy 5 release notes](https://groovy-lang.org/releasenotes/groovy-5.0.html).

## TLS parameter classes moved

`TlsParameters` and `CustomTlsParameters` are no longer specific to ATNA auditing and have moved from the
`ipf-commons-audit` module to `ipf-commons-core`
([#500](https://github.com/oehf/ipf/issues/500)):

| Until IPF 5.1 | As of IPF 5.2 |
|---------------|---------------|
| `org.openehealth.ipf.commons.audit.TlsParameters` | `org.openehealth.ipf.commons.core.ssl.TlsParameters` |
| `org.openehealth.ipf.commons.audit.CustomTlsParameters` | `org.openehealth.ipf.commons.core.ssl.CustomTlsParameters` |

The classes at the old coordinates still exist and still work — they now merely extend the new ones — but
they are annotated `@Deprecated(forRemoval = true)` and will disappear in a future release. If your code
references either type, change the import; nothing else needs to change.

## Spring Boot SSL bundles

Setting `ipf.commons.reuse-ssl-config` to `true` makes IPF derive its TLS beans from the Spring Boot
server configuration. As of IPF 5.2 this also understands
[SSL bundles](https://docs.spring.io/spring-boot/reference/features/ssl.html)
([#500](https://github.com/oehf/ipf/issues/500)): when `server.ssl.bundle` names a bundle, the resulting
`SSLContextParameters` and `TlsParameters` beans are created from that bundle instead of from the
individual `server.ssl.*` keystore properties. Without a bundle, the previous behaviour is unchanged.

This is an addition, so no migration is required. See the
[Spring Boot support]({{ site.baseurl }}{% link _pages/boot/boot.md %}) page for the properties involved.

## MHD support updated

The MHD classes and validators have been adapted to the **MHD 4.2.3** specification, which introduces
minor backward incompatibilities ([#497](https://github.com/oehf/ipf/issues/497)). Related fixes in the
same area:

* Display names of MHD `List` types have been corrected
  ([#499](https://github.com/oehf/ipf/issues/499)).
* ITI-67 supports the additional search parameter `creation`
  ([#502](https://github.com/oehf/ipf/issues/502)), see
  [ITI-67]({{ site.baseurl }}{% link _pages/ihe/fhir/iti67.md %}).

If you implement an MHD actor, re-run your integration tests against the new version rather than assuming
a drop-in replacement.

## Stricter validation for PIXm and PDQm

The PIXm and PDQm implementations have been reorganized and now come with proper validation rules
([#507](https://github.com/oehf/ipf/issues/507)). This is the change most likely to surface in an existing
deployment: requests that earlier IPF versions passed through may now be rejected as invalid.

Affected transactions are
[ITI-78]({{ site.baseurl }}{% link _pages/ihe/fhir/iti78.md %}),
[ITI-83]({{ site.baseurl }}{% link _pages/ihe/fhir/iti83.md %}) and
[ITI-119]({{ site.baseurl }}{% link _pages/ihe/fhir/iti119.md %}).
Test with the messages your partners actually send. General information about switching validation on and
off is on the [message validation]({{ site.baseurl }}{% link _pages/ihe/messageValidation.md %}) page.

Also in this area, the datatype of `mothersMaidenName` in `PdqmMatchInputPatient` has been corrected
([#491](https://github.com/oehf/ipf/issues/491)).

## FHIR AuditRecordTranslator converted to Java

The R4 `AuditRecordTranslator` has been rewritten from Groovy to Java
([#501](https://github.com/oehf/ipf/issues/501)). The translation logic is unchanged; the class is simply
no longer a Groovy class. If you subclass it or call it from Groovy code relying on dynamic dispatch,
recompile and re-test. This continues the alignment started in
[IPF 5.1]({{ site.baseurl }}{% link _pages/migration/migration-5.1.md %}).

## NamingSystem uniqueId selection honors "preferred"

When several uniqueIds of the same type are present in a `NamingSystem` resource, IPF now selects the one
marked as `preferred` instead of the first match
([#512](https://github.com/oehf/ipf/issues/512)). If your resources carry more than one uniqueId per type
and you relied on the previous behavior, verify which identifier your code now receives.

## Phone number components accept strings

Phone number components may now be given as strings instead of long integers
([#498](https://github.com/oehf/ipf/issues/498)) — useful for area or country codes with leading zeros.
This is a purely additive API change.

## Corrected codes in generated responses and audit records

Several codes emitted by IPF were wrong and have been fixed. None of these require code changes, but
tests that assert on generated messages may need updating:

| Fix | Issue |
|-----|-------|
| Error codes in automatically generated ITI-44 NAKs | [#505](https://github.com/oehf/ipf/issues/505) |
| Status code in automatically generated HL7v3 NAKs, plus a new `aborted` status | [#508](https://github.com/oehf/ipf/issues/508) |
| Detected-issue event code in generated HL7v3 NAKs | [#510](https://github.com/oehf/ipf/issues/510) |
| Case of HL7v3 `statusCode` values in transformers | [#511](https://github.com/oehf/ipf/issues/511) |
| ATNA `roleIDCode` translation | [#484](https://github.com/oehf/ipf/issues/484) |

In addition, ITI-80 audit records now store the `homeCommunityId` as required by CP-ITI-1014
([#503](https://github.com/oehf/ipf/issues/503)), see
[ITI-80]({{ site.baseurl }}{% link _pages/ihe/xds/iti80.md %}).

## Syslog parser

The syslog parser used for server-side parsing of ATNA events — rewritten without external dependencies in
IPF 5.1 — conforms more closely to RFC 5424
([#489](https://github.com/oehf/ipf/issues/489)) and now supports multiline messages
([#490](https://github.com/oehf/ipf/issues/490)).

## New features worth knowing about

These are additions rather than migration items:

* **Convenience methods for HTTP and SOAP headers**
  ([#496](https://github.com/oehf/ipf/issues/496)) make it easier to read and write protocol headers in
  Web Service based transactions. See
  [protocol headers]({{ site.baseurl }}{% link _pages/ihe/ws/wsProtocolHeaders.md %}).
* **`AuditApplicationEventMessageQueue`** (in `ipf-atna-spring-boot-starter`) publishes IPF audit messages
  as Spring Boot `AuditApplicationEvent`s, so they show up in Spring Boot's audit infrastructure. It is
  meant to be combined with the real audit queue in a `CompositeAuditMessageQueue`. See
  [ATNA auditing with Spring Boot]({{ site.baseurl }}{% link _pages/boot/boot-atna.md %}).
* `ApacheHttpRequest5` gained a `toString()` implementation
  ([#488](https://github.com/oehf/ipf/issues/488)) and no longer throws a `NullPointerException` when
  reading a request body from a stream ([#487](https://github.com/oehf/ipf/issues/487)).

The complete list of resolved issues is in the
[5.2.0 release notes](https://github.com/oehf/ipf/releases/tag/ipf-5.2.0).
