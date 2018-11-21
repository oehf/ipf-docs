---
title: XDS based transactions
layout: single
permalink: /docs/ihe/xds/
classes: wide
---

 
IPF adds support for a subset of XDS-based profiles by providing Camel components (hiding the 
 implementation details on transport level) and a XDS data model.

The following IHE transactions for XDS are currently supported:

| Transaction             | Profile          | Description           | IPF Component          |  IPF Module |
|:------------------------|:-----------------|:----------------------|:-----------------------|:------------|
{% for hash in site.data.ihe -%}
  {% assign tx = hash[1] -%}
  {% if tx.format == "ebXML" -%}
| [{{ tx.transaction }}](../{{ tx.link }}/)  | {{ tx.profile }} | {{ tx.description }}  | `{{ tx.component }}`  | `{{ tx.module }}` |
  {% endif -%}
{% endfor %}


**Spring Boot** 
There is a [Spring Boot starter] for ebXML/XDS-based IHE transactions.
{: .notice--info}

[Spring Boot starter]: {{ site.baseurl }}{% link _pages/boot/boot-xds.md %}