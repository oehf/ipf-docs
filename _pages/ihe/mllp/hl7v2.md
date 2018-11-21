---
title: HL7v2 based transactions
layout: single
permalink: /docs/ihe/hl7v2/
classes: wide
---

 
IPF adds support for a subset of HL7v2-based profiles by providing Camel components (hiding the 
 implementation details on transport level).


The following IHE transactions for HL7v2 are currently supported:

| Transaction             | Profile          | Description           | IPF Component          |  IPF Module |
|:------------------------|:-----------------|:----------------------|:-----------------------|:------------|
{% for hash in site.data.ihe -%}
  {% assign tx = hash[1] -%}
  {% if tx.format == "HL7 v2.5" or tx.format == "HL7 v2.3.1" -%}
| [{{ tx.transaction }}](../{{ tx.link }}/)  | {{ tx.profile }} | {{ tx.description }}  | `{{ tx.component }}`  | `{{ tx.module }}` |
  {% endif -%}
{% endfor %}

**Spring Boot** 
There is a [Spring Boot starter] for HL7v2-based IHE transactions.
{: .notice--info}

[Spring Boot starter]: {{ site.baseurl }}{% link _pages/boot/boot-hl7.md %}