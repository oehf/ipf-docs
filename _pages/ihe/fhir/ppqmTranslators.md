---
title: Translation between FHIR and XACML 2.0 message models
layout: single
permalink: /docs/ihe/ppqmTranslators/
toc: true
toc_icon: align-left
toc_sticky: true
---

IPF provides utilities for translation between FHIR and XACML 2.0, thus giving the possibility to implement
CH:PPQm transactions on top of their CH:PPQ counterparts and to avoid redundancy in that way.

The supported transaction pairs are

| CH:PPQm transaction (FHIR/REST)           | CH:PPQ transaction (XACML 2.0/SOAP) |
|-------------------------------------------|-------------------------------------|
| Mobile Privacy Policy Feed (PPQ-3)        | Privacy Policy Feed (PPQ-1)         |
| Mobile Privacy Policy Bundle Feed (PPQ-4) | Privacy Policy Feed (PPQ-1)         |
| Mobile Privacy Policy Retrieve (PPQ-5)    | Privacy Policy Retrieve (PPQ-2)     |

Routines for translation of CH:PPQm requests into CH:PPQ ones are provided in the 
class `org.openehealth.ipf.commons.ihe.fhir.chppqm.translation.FhirToXacmlTranslator`:
* `translatePpq3To1Request()`
* `translatePpq4To1Request()`
* `translatePpq5To2Request()`

Routines for translation of CH:PPQ responses into CH:PPQm ones are provided in the
class `org.openehealth.ipf.commons.ihe.fhir.chppqm.translation.XacmlToFhirTranslator`:
* `translatePpq1To3Response()`
* `translatePpq1To4Response()`
* `translatePpq2To5Response()`
* `translateSoapFault()`
