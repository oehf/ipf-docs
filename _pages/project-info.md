---
title: Project Info
layout: single
permalink: /project-info/
header:
  overlay_color: "#000"
  overlay_filter: "0.1"
  overlay_image: /assets/images/info2.jpg
toc: true
toc_icon: align-left  
---

The Open eHealth Integration Platform (IPF) provides interfaces for health-care related integration solutions.
An prominent example of an healthcare-related use case of IPF is the implementation of interfaces for transactions specified
in Integrating the Healthcare Enterprise ([IHE][ihe]) profiles.

IPF is built upon and extends the [Apache Camel](https://camel.apache.org) routing and mediation engine. 
It comes with comprehensive support for message processing and connecting
systems in the eHealth domain. IPF provides domain-specific languages (DSLs) for implementing
[Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/)
in general-purpose as well as healthcare-specific integration solutions.

## License

IPF code is Open Source and licensed under [Apache license][apache-license].

## Development

[Contribute][development] by reporting issues, suggesting new features, or forking the
Git repository on [GitHub][ipf-github] and provide some good pull requests!

## What's New

The current version of IPF is 6.0.0. It moves IPF to Jakarta EE 11, Spring Framework 7 and Spring Boot 4.1, and
removes functionality that has been deprecated in the IPF 5.x releases. The most prominent changes are:

* **Platform upgrade**: Spring Boot 4.1, Apache Camel 4.22, Apache CXF 4.2, Jackson 3 and HAPI FHIR 8.12.
  Java 17 remains the minimum; IPF 6.0 is also tested with Java 21 and 25.
* **Reimplemented mapping service**: a typed `Mappings` API and declarative, validated mapping formats
  (IPF XML, IPF YAML and FHIR ConceptMap R4). The previous `MappingService` and the Groovy `.map` format are
  deprecated. See [Defining and Loading Mappings].
* **Simplified extension mechanism**: custom mappings, HL7v2 model classes, DSL extensions and route builders
  are collected by their holders themselves, so the `Configurer` SPI and the `SpringConfigurationPostProcessor`
  are no longer needed. See [Dynamic registration of IPF features].
* **Tracing with Camel telemetry**: IPF's own HL7v2 tracing support is replaced by span decorators for Apache
  Camel's telemetry SPI, e.g. with `camel-opentelemetry2`. See [Tracing with Camel telemetry].
* **New and updated transactions**: PIXm Patient Identity Feed [ITI-104], MHD 4.2.4 including the Target
  Communities Option, and [FHIR Binary streaming uploads] for large documents with ITI-65 and ITI-105.
* **ATNA auditing** follows the latest DICOM CPs, and the audit transports deprecated in IPF 5.0 were removed.
  See [Audit Transmission Protocols].
* **Hardened XML processing** against XXE and SSRF attacks.
* **FHIR STU3 support has been removed**; use the FHIR R4 modules instead.

See the list of fixed Github issues for an overview:

* Fixes for [6.0.0](https://github.com/oehf/ipf/releases/tag/ipf-6.0.0)
* Fixes for [5.3.0](https://github.com/oehf/ipf/releases/tag/ipf-5.3.0)
* Fixes for [5.2.0](https://github.com/oehf/ipf/releases/tag/ipf-5.2.0)
* Fixes for [5.1.0](https://github.com/oehf/ipf/releases/tag/ipf-5.1.0)
* Fixes for [5.0.0](https://github.com/oehf/ipf/releases/tag/ipf-5.0.0)
* Fixes for [4.8.0](https://github.com/oehf/ipf/releases/tag/ipf-4.8.0)
* Fixes for [4.7.0](https://github.com/oehf/ipf/releases/tag/ipf-4.7.0)
* Fixes for [4.6.0](https://github.com/oehf/ipf/releases/tag/ipf-4.6.0)
* Fixes for [4.5.0](https://github.com/oehf/ipf/releases/tag/ipf-4.5.0)
* Fixes for [4.4.0](https://github.com/oehf/ipf/releases/tag/ipf-4.4.0)
* Fixes for [4.3.2](https://github.com/oehf/ipf/releases/tag/ipf-4.3.2)
* Fixes for [4.3.1](https://github.com/oehf/ipf/releases/tag/ipf-4.3.1)
* Fixes for [4.3.0](https://github.com/oehf/ipf/releases/tag/ipf-4.3.0)
* Fixes for [4.2.0](https://github.com/oehf/ipf/releases/tag/ipf-4.2.0)
* Fixes for [4.1.0](https://github.com/oehf/ipf/milestone/27?closed=1)
* Fixes for [4.0.0](https://github.com/oehf/ipf/milestone/13?closed=1)
* Fixes for [3.7.2](https://github.com/oehf/ipf/milestone/26?closed=1)
* Fixes for [3.7.1](https://github.com/oehf/ipf/milestone/25?closed=1)
* Fixes for [3.7.0](https://github.com/oehf/ipf/milestone/20?closed=1)

The IPF 6.x release supports Java 17 and up, Spring Boot 4.x, Camel 4.x and Jakarta EE 11. Major new features will
be developed primarily for IPF 6.

IPF 5.x supports Java 17 and up, Spring Boot 3.x, Camel 4.x and Jakarta EE 10. Applications that cannot move to
Spring Boot 4 yet can stay on IPF 5.3, but we recommend upgrading to IPF 6.

IPF 4.x supports Java 11 and up, Spring Boot 2.x as well as Camel 3.x.

IPF 3.7.x was the last release supporting Java 8 and Camel 2.x. The support for this branch has already ended
and we highly recommend the upgrade to a recent JDK and new IPF releases.

## Update Instructions

If you are using previous versions of IPF and want to update:

* IPF 6.0 is not backwards-compatible with IPF 5.x, primarily because it is now based on Jakarta EE 11, Spring Boot 4
  and Jackson 3. It also reimplements the mapping service, deprecates the `Configurer` SPI, replaces the tracing
  support and removes FHIR STU3. See [6.0 Update Instructions] for details.
* IPF 5.3 completes FHIR-based (BALP) ATNA auditing, renaming and deprecating a few audit classes, and deprecates all FHIR STU3 support. See [5.3 Update Instructions] for details.
* IPF 5.2 is backwards-compatible with IPF 5.1, but brings stricter validation for some FHIR transactions and major Camel and Groovy updates. See [5.2 Update Instructions] for details.
* IPF 5.1 consists primarily of bugfixes and dependency updates. See [5.1 Update Instructions] for details.
* IPF 5.0 is not backwards-compatible with IPF 4.x, primarily because it is now based on Jakarta EE 10. See [5.0 Update Instructions] for details.
* IPF 4.8 introduce some changes that are mention in [4.8 Update Instructions].
* IPF 4.4 has reimplemented IHE MLLP component. Read the [4.4 Update Instructions] for how to update from earlier versions
* IPF 4.x is not backwards-compatible with IPF 3.x. However, there is most likely not much to be changed, and this is mostly due to the JDK and Camel upgrades. Read the [4.0 Update Instructions] for how to update from IPF 3.7.x
* IPF 3.7.x comes with a few minor changes. Read the [3.7 Update Instructions] for how to update from IPF 3.6.x

When doing a upgrade to newer IPF version, we recommend to sync minor versions of Spring Boot and Apache camel within your application, with the versions used in the corresponding IPF release. Please see the dependency management of IPF releases to see which version you should consider.

## Issue Tracking

Issue tracking is done in github. For current issues check [https://github.com/oehf/ipf/issues](https://github.com/oehf/ipf/issues).
Questions? Please direct your issues to the [IPF User Google Group](https://groups.google.com/forum/#!forum/ipf-user). 


## Javadocs

The javadocs can be obtained [here](apidocs/index.html).


[apache-license]: https://www.apache.org/licenses/LICENSE-2.0
[development]: {{ site.baseurl }}{% link _pages/development.md %}
[ipf-github]: https://github.com/oehf/ipf
[ihe]: https://www.ihe.net
[Migration Instructions]: {{ site.baseurl }}{% link _pages/migration/migration.md %}
[3.1 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-3.1.md %}
[3.2 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-3.2.md %}
[3.4 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-3.4.md %}
[3.5 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-3.5.md %}
[3.6 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-3.6.md %}
[3.7 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-3.7.md %}
[4.0 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-4.0.md %}
[4.4 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-4.4.md %}
[4.8 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-4.8.md %}
[5.0 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-5.0.md %}
[5.1 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-5.1.md %}
[5.2 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-5.2.md %}
[5.3 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-5.3.md %}
[6.0 Update Instructions]: {{ site.baseurl }}{% link _pages/migration/migration-6.0.md %}
[Defining and Loading Mappings]: {{ site.baseurl }}{% link _pages/mappings.md %}
[Dynamic registration of IPF features]: {{ site.baseurl }}{% link _pages/dynamic.md %}
[Tracing with Camel telemetry]: {{ site.baseurl }}{% link _pages/migration/migration-6.0.md %}#tracing-with-camel-telemetry
[ITI-104]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti104.md %}
[FHIR Binary streaming uploads]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirStreamingUpload.md %}
[Audit Transmission Protocols]: {{ site.baseurl }}{% link _pages/audit.md %}#audit-transmission-protocols
