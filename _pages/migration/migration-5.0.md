---
title: IPF 5.0 Migration Guide
layout: single
permalink: /docs/migration-5.0/
toc: true
toc_icon: align-left  
toc_sticky: true
---

IPF 5.0 comes with some changes that should be considered when upgrading from earlier IPF versions.

## Changes in ATNA audit dataset enrichment 

Older IPF versions contained the module `ipf-commons-ihe-xua` which provided routines to propagate 
particular elements of XUA authorization assertions into ATNA audit datasets.  In IPF 5.0, this 
module is not present anymore.  Instead, a concept of an "audit dataset enricher" is introduced
for Web Service based and FHIR based transactions. See 
[here]({{ site.baseurl }}{% link _pages/audit.md %}) and 
[here]({{ site.baseurl }}{% link _pages/boot/boot-atna.md %}) for details.
