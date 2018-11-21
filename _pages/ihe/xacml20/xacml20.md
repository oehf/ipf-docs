---
title: XACML20 based transactions
layout: single
permalink: /docs/ihe/xacml20/
classes: wide
---

 
IPF adds support for some XACML-based profiles by providing Camel components (hiding the 
 implementation details on transport level).

The following IHE transactions for XACML 2.0 are currently supported:

| Transaction             | Profile          | Description           | IPF Component          |  IPF Module |
|:------------------------|:-----------------|:----------------------|:-----------------------|:------------|
{% for hash in site.data.ihe -%}
  {% assign tx = hash[1] -%}
  {% if tx.format == "XACML 2.0" -%}
| [{{ tx.transaction }}](../{{ tx.link }}/)  | {{ tx.profile }} | {{ tx.description }}  | `{{ tx.component }}`  | `{{ tx.module }}` |
  {% endif -%}
{% endfor %}
