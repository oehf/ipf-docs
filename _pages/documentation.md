---
title: Documentation
layout: single
classes: wide
permalink: /docs/
header:
  overlay_color: "#000"
  overlay_filter: "0.1"
  overlay_image: /assets/images/documentation2.jpg
---

The following table summarizes the IPF features related to the eHealth domain:

| Feature                                              | Description                                                  |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| [Support for eHealth integration profiles][]         | A set of components for creating actor interfaces as specified in IHE and Continua integration profiles. IPF currently supports creation of actor interfaces for the IHE profiles XDS, PIX, PDQ, PIXv3, PDQv3, PIXm, PDQm, MHD, QED, QEDm, XCPD, XCA, XCA-I, XCF, XPID, PCD, as well as for Continua profiles HRN and WAN. |
| HL7 Message processing with [Groovy][] or [Kotlin][] | Basis for HL7 message processing is the HL7v2 DSL.           |
| [HL7 Message translation][]                          | Translation utilities for translating between HL7v3 and HL7v2 messages for corresponding IHE transactions |
| [FHIR Support][]                                     | FHIR® – Fast Healthcare Interoperability Resources (hl7.org/fhir) – is a next generation standards framework created by HL7 leveraging the latest web standards and applying a tight focus on implementability. |
| [DICOM Audit Support][]                              | Support for constructing, serializing and sending DICOM audit messages |
| [CDA/MDHT Support][]                                 | Wrapping a number of CDA/MDHT-related libraries                   |


Other IPF features provide part of the underlying foundation or supporting functionality:

| Feature                          | Description                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| [Core Features][]                | Domain-neutral message processors and DSL extensions usable for general-purpose message processing. |
| [Code System Mapping][]          | A simplistic mechanism for mapping code values between code systems |
| [Dynamic Feature Registration][] | Aids in building up modular integration solutions where each module contributes routes, services etc. to the overall application |


IPF comes with a number of [Spring Boot Starters][] that support running eHealth applications
in the Spring Boot runtime environment.


[Support for eHealth integration profiles]: {{ site.baseurl }}{% link _pages/ihe/ihe.md %}
[Groovy]: {{ site.baseurl }}{% link _pages/hl7-groovy/hl7-groovy.md %}
[Kotlin]: {{ site.baseurl }}{% link _pages/hl7-kotlin/hl7-kotlin.md %}
[HL7 Message translation]: {{ site.baseurl }}{% link _pages/ihe/hl7v3/hl7v3Translators.md %}
[DICOM Audit Support]: {{ site.baseurl }}{% link _pages/audit.md %}
[FHIR support]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhir.md %}
[CDA/MDHT Support]: {{ site.baseurl }}{% link _pages/mdht/mdhtSupport.md %}
[Core Features]: {{ site.baseurl }}{% link _pages/groovy/extensions.md %}
[Code System Mapping]: {{ site.baseurl }}{% link _pages/mapping.md %}
[Dynamic Feature Registration]: {{ site.baseurl }}{% link _pages/dynamic.md %}
[Spring Boot Starters]: {{ site.baseurl }}{% link _pages/boot/boot.md %}