---
title: IPF 5.0 Migration Guide
layout: single
permalink: /docs/migration-5.0/
toc: true
toc_icon: align-left  
toc_sticky: true
---

IPF 5.0 comes with some changes that should be considered when upgrading from earlier IPF versions.

## Overall changes

IPF 5.x requires Java 17 and buids upon Jakarta EE 10. This primarily means moving from `javax.*` to
`jakarta.*` packages, including upgrades of almost all relevant dependencies.
Therefore, expect considerable migration effort. Migration instructions for individual libraries include:

* Camel 4: [here](https://camel.apache.org/manual/camel-4-migration-guide.html) and [here](https://camel.apache.org/manual/camel-4x-upgrade-guide.html)
* [Spring 6](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-6.x)
* [Spring Boot](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)
* [CXF](https://cxf.apache.org/docs/41-migration-guide.html)
* [HAPI FHIR](https://hapifhir.io/hapi-fhir/docs/interceptors/jakarta_upgrade.html)


## Changes in ATNA audit builder classes

The generic builder classes `PatientRecordEventEventBuilder`, `PHIExportBuider`, `PHIImportBuilder`,
and `QueryInformationBuilder` have been made abstract. Concrete implementation classes are available
prefixed with `Default`.


## Changes in ATNA audit dataset enrichment 

Older IPF versions contained the module `ipf-commons-ihe-xua` which provided routines to propagate 
particular elements of XUA authorization assertions into ATNA audit datasets.  In IPF 5.0, this 
module is not present anymore.  Instead, a concept of an "audit dataset enricher" is introduced
for Web Service based and FHIR based transactions. See 
[here]({{ site.baseurl }}{% link _pages/audit.md %}) and 
[here]({{ site.baseurl }}{% link _pages/boot/boot-atna.md %}) for details.

## HL7v3 models

HL7v3 models have been upgraded to the Jakarta XML Bind namespace and moved to the
`ìpf-commons-ihe-hl7v3model` module.

## Removal of deprecated classes and methods

As this is a major release, expect deprecated classes and methods to be removed