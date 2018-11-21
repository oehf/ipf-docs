---
title: HL7v3 based transactions
layout: single
permalink: /docs/ihe/hl7v3/
classes: wide
---

 
IPF adds support for a subset of HL7v3-based profiles by providing Camel components (hiding the 
 implementation details on transport level) and [translators] between the Hl7 v3 and HL7 v2 message models.

The following IHE transactions for HL7v3 are currently supported:

| Transaction             | Profile          | Description           | IPF Component          |  IPF Module |
|:------------------------|:-----------------|:----------------------|:-----------------------|:------------|
{% for hash in site.data.ihe -%}
  {% assign tx = hash[1] -%}
  {% if tx.format == "HL7v3" -%}
| [{{ tx.transaction }}](../{{ tx.link }}/)  | {{ tx.profile }} | {{ tx.description }}  | `{{ tx.component }}`  | `{{ tx.module }}` |
  {% endif -%}
{% endfor %}

**Spring Boot** 
There is a [Spring Boot starter] for HL7v3-based IHE transactions.
{: .notice--info}

[Spring Boot starter]: {{ site.baseurl }}{% link _pages/boot/boot-hl7v3.md %}
[translators]: {{ site.baseurl }}{% link _pages/ihe/hl7v3/hl7v3Translators.md %}