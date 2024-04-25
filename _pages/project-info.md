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

The current version of IPF is 4.8.0

See the list of fixed Github issues for an overview:

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

The upcoming IPF 5.x release will support java 17 and up, Spring-boot 3.x, Camel 4.x. Major new features will be developed
primarily for IPF 5.

IPF 4.x supports Java 11 and up, Spring-boot 2.x as well as Camel 3.x. We will support this branch for some more
patch releases to support the folks that are not able (yet) to upgrade to a recent JDK.

IPF 3.7.x was the last release supporting Java 8 and Camel 2.x. The support for this branch has already ended
and we highly recommend the upgrade to a recent JDK and new IPF releases.

## Update Instructions

If you are using previous versions of IPF and want to update:

* IPF 4.8 introduce some changes that are mention in [4.8 Update Instructions].
* IPF 4.4 has reimplemented IHE MLLP component. Read the [4.4 Update Instructions] for how to update from earlier versions
* IPF 4.x is not backwards-compatible with IPF 3.x. However, there is most likely not much to be changed, and this is mostly due to the JDK and Camel upgrades. Read the [4.0 Update Instructions] for how to update from IPF 3.7.x
* IPF 3.7.x comes with a few minor changes. Read the [3.7 Update Instructions] for how to update from IPF 3.6.x

When doing a upgrade to newer IPF version, we recommend to sync minor versions of Spring-Boot and Apache camel within your application, with the versions used in the corresponding IPF release. Please see the dependency management of IPF releases to see which version you should consider.

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