---
title: IPF 4.0 Migration Guide
layout: single
permalink: /docs/migration-4.0/
toc: true
toc_icon: align-left  
toc_sticky: true
---

IPF 4.0 comes with some major changes that must be considered when upgrading from IPF 3.7.

## Java Runtime

* IPF 4.0 requires Java 11. No, it won't work with Java 8.


## 3rd party library changes

* Most notably, [Camel](https://camel.apache.org) has been upgraded to version 3.7.x. Please consult the [Camel migration guide](https://camel.apache.org/manual/latest/migration-and-upgrade.html) for details. "Normal" usage of Camel routes should not be affected, but
 functionality has been moved to individual Camel modules and there have been various package changes.
* [Ehcache](https://ehcache.org) was updated to 3.x. If you have used this as transitive dependency, be aware of the changed API in Ehcache.

## Other notable changes

* It is not possible anymore to map a string using the `String.mapXXX()` or `String.mapReverseXXX()` metaclass extensions 
  (where `XXX` is the mapping key). Use `String.map('XXX')` or `String.mapReverse('XXX')`instead.
* `FhirConsumer#test` does not expect the FHIR resource but an instance of `RequestDetails` as parameter


## Deprecations in IPF 4.0

* The Vert.x audit message senders have been deprecated


## Removal of deprecated functionality in IPF 3.7

* Deprecated modules in 3.7 have been removed

  * `ipf-modules-cda-oht`


* Deprecated methods in 3.7 have been removed
  
  * `AuditContext.setAuditEnabled`
  * `IHEAuditMessageBuilder.makeDocumentDetail`
  * `PayloadLoggerBase.isLocallyEnabled/setLocallyEnabled`
  * `PayloadLoggerBase.isGloballyEnabled/setGloballyEnabled`
  * `DocumentEntry.get/setAuthor`
  * `SubmissionSet.getAuthor`
  * `BidiMappingService.addMappingScript/addMappingScripts`
  * `MessageUtils.pipeEncode`
  * `Hl7ExtensionModule.ack/nak/dump`
  * `ValidatorAdapter.profile`
  