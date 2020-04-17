---
title: IPF 3.7 Migration Guide
layout: single
permalink: /docs/migration-3.7/
toc: true
toc_icon: align-left  
toc_sticky: true
---

IPF 3.7 comes with some changes that must be considered when upgrading from IPF 3.6.

## Package Changes

* Classes depending on Spring Caching were moved out of the Spring Boot submodules and so changed their
class names and packages to:
  * `org.openehealth.ipf.commons.ihe.fhir.SpringCachePagingProvider`
  * `org.openehealth.ipf.commons.ihe.hl7v2.storage.SpringCacheUnsolicitedFragmentationStorage`
  * `org.openehealth.ipf.commons.ihe.hl7v2.storage.SpringCacheInteractiveContinuationStorage`
  * `org.openehealth.ipf.commons.ihe.hl7v3.storage.SpringCacheHl7v3ContinuationStorage`
  * `org.openehealth.ipf.commons.ihe.hl7v3.storage.SpringCacheHl7v3ContinuationStorage`
  * `org.openehealth.ipf.commons.ihe.ws.correlation.SpringCacheAsynchronyCorrelator`

## Spring Boot Updates

* `ipf-atna-spring-boot-starter` does not depend on `ipf-spring-boot-starter` anymore. This makes it
usable in scenarios where no other IPF modules are used and does not drag in 3rd party dependencies like Apache Camel
or Groovy anymore.

## Other notable changes

* `Hl7v3NakFactory` does not return quantity attributes for XCPD error responses
* FHIR R4 profiles have been updated to the recent IHE specification


