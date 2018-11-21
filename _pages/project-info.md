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

The current version of IPF is 3.5.3.

See the list of fixed Github issues for an overview:

* Fixes for [3.5.3](https://github.com/oehf/ipf/milestone/16?closed=1)
* Fixes for [3.5.2](https://github.com/oehf/ipf/milestone/15?closed=1)
* Fixes for [3.5.1](https://github.com/oehf/ipf/milestone/14?closed=1)
* Fixes for [3.5](https://github.com/oehf/ipf/milestone/12?closed=1)

## Update Instructions

If you are using previous versions of IPF and want to update:

* IPF 3.5.x comes with some changes that must be considered when upgrading from IPF 3.4.x. Read the [3.5 Update Instructions] for how to update from IPF 3.4.x
* IPF 3.4.x comes with some changes that must be considered when upgrading from IPF 3.2.x or IPF 3.3.x to IPF 3.4.x Read the [3.4 Update Instructions] for how to update from IPF 3.1.x
* IPF 3.2.x comes with some changes that must be considered when upgrading from IPF 3.1.x to IPF 3.2.x Read the [3.2 Update Instructions] for how to update from IPF 3.1.x
* IPF 3.1.x introduces a few minor incompatibilities compared to IPF 3.0.x due to having less mandatory dependencies on the Spring framework. Read the [3.1 Update Instructions] for how to update from IPF 3.0.x
* IPF 3.0.x is not backwards-compatible with IPF 2.x. Read the [Migration Instructions] for how to migrate a IPF 2.x-based integration solution.

## Removed functionality

As XDS.a transactions have been retired by IHE, all functionality related with ITI-14, ITI-15, ITI-16 and ITI-17
have been removed. This includes the ebXML/ebRS 2.x model classes.
{: .notice--warning}
 
## Added modules

IPF 3.5.1 added the modules listed below:

 * `ipf-commons-ihe-xacml20`
 * `ipf-platform-camel-ihe-xacml20`
 
## Deprecated modules

IPF 3.5.1 deprecates the following modules:

 * `ipf-commons-ihe-fhir-dstu2`
 * `ipf-platform-camel-ihe-fhir-dstu2`

## Issue Tracking

Issue tracking is done in github. For current issues check [https://github.com/oehf/ipf/issues](https://github.com/oehf/ipf/issues).
Questions? Please direct your issues to the [IPF User Google Group](https://groups.google.com/forum/#!forum/ipf-user). 

These modules will be removed as soon as FHIR r4 support has been included.

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