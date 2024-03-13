---
title: IPF 4.8 Migration Guide
layout: single
permalink: /docs/migration-4.8/
toc: true
toc_icon: align-left  
toc_sticky: true
---

IPF 4.8 comes with some changes that should be considered when upgrading from earlier IPF versions

## XDS module API changes

Some new query types have been added to facilitate writing DSUB filters. While doing so, a couple of internal API changes have been done.

* EbXML Request and Response classes now have a type parameter
* EbXML Request and Response transformer classes (e.g. FindSubmissionSetsQueryTransformer) now provide a getInstance() method while their constructors have been made private.
* Top level XDS validator classes (AdhocQueryRequestValidator etc.) also provide a getInstance() method while their constructors have been made private.

## Other XDS module changes

* EbXML response Classifiation objects are populated with an `objectType` attribute where applicable
* A basic validation on the syntax of a MimeType of XDS DocumentEntries is now performed 

## FHIR module changes

* For implementing FHIR [Basic Audit Log Patterns](https://profiles.ihe.net/ITI/BALP), the standard audit strategies for FHIR transactions have been improved. However, the DICOM ATNA output for these transactions may look slightly different than before. 
* This release supports MHD 4.2.1 FHIR transactions in addition to the obsolete MHD 3.2. As users may choose to stick with the old specification or even have to support both versions in parallel, we chose to extend the existing IHE FHIR components. Please refer to the documentation of ITI-65, ITI-66, ITI67 and ITI-68. 

