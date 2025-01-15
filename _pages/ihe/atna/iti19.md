---
title: ATNA ITI-19
layout: single
permalink: /docs/ihe/iti19/
toc: false
toc_icon: align-left
toc_sticky: true
---

# ATNA ITI-19 Support

Implementation of the ITI-19 transaction means that network communication shall be secured.
This is supported for all types of IPF Camel components, for both clients and servers:
* [Secure transport in MLLP transactions][MLLP SSL]
* [Secure transport in SOAP transactions][SOAP SSL]
* [Secure transport in FHIR transactions][FHIR SSL]


[MLLP SSL]: {{ site.baseurl }}{% link _pages/ihe/mllp/hl7v2SecureTransport.md %}
[SOAP SSL]: {{ site.baseurl }}{% link _pages/ihe/ws/wsSecureTransport.md %}
[FHIR SSL]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirSecurity.md %}