---
title: FHIR based transactions
layout: single
permalink: /docs/ihe/fhir/
classes: wide
---


FHIR® – Fast Healthcare Interoperability Resources (hl7.org/fhir) – is a next generation standards framework created by HL7,
combining the best features of HL7v2, HL7v3, and CDA while leveraging the latest web standards and applying a tight focus on implementability.
 
IPF adds support for a subset of these profiles by providing Camel components (hiding the 
 implementation details on transport level) and translators, e.g. between the FHIR and HL7 v2 message models.

While the FHIR transactions in IHE ITI revision 13 (2016/2017) was based on FHIR [DSTU2](https://hl7.org/fhir/DSTU2/index.html),
the transactions have been migrated to [STU3](https://hl7.org/fhir/STU3/index.html) as of IHE ITI revision 14 (2017/2018) 
and further to [R4](https://hl7.org/fhir/R4/index.html) as of IHE ITI revision 16 (2019/2020).

ALl current support for FHIR-based IHE profiles is for FHIR [R4](https://hl7.org/fhir/R4/index.html)

{: .notice--info}

The following IHE transactions for FHIR are currently supported:

| Transaction             | Profile          | Description           | IPF Component          |  IPF Module |
|:------------------------|:-----------------|:----------------------|:-----------------------|:------------|
{% for hash in site.data.ihe -%}
  {% assign tx = hash[1] -%}
  {% if tx.format == "FHIR" -%}
| [{{ tx.transaction }}](../{{ tx.link }}/)  | {{ tx.profile }} | {{ tx.description }}  | `{{ tx.component }}`  | `{{ tx.module }}` |
  {% endif -%}
{% endfor %}
