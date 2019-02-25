---
title: IPF 3.6 Migration Guide
layout: single
permalink: /docs/migration-3.6/
toc: true
toc_icon: align-left  
toc_sticky: true
---

IPF 3.6 comes with some changes that must be considered when upgrading from IPF 3.5.

## Package Changes

* `org.openehealth.ipf.commons.ihe.fhir.support.FhirAuditStrategy` (and its subclasses) have been
relocated from the version-specific FHIR modules to the `org.openehealth.ipf.commons.ihe.fhir.audit`
package in the `ipf-commons-ihe-fhir` module. The old classes have been deprecated.

## Removal of deprecated functionality in IPF 3.6

* All modules concerning FHIR DSTU2 version have been removed in favor of FHIR R4