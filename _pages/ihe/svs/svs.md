---
title: SVS based transactions
layout: single
permalink: /docs/ihe/svs/
classes: wide
---

IPF supports SVS-based profiles (Sharing Value Sets) by providing Camel components (hiding the 
implementation details on transport level).

The following IHE transactions for SVS are currently supported:

| Transaction             | Profile          | Description           | IPF Component          |  IPF Module |
|:------------------------|:-----------------|:----------------------|:-----------------------|:------------|
{% for hash in site.data.ihe -%}
  {% assign tx = hash[1] -%}
  {% if tx.format == "SVS" -%}
| [{{ tx.transaction }}](../{{ tx.link }}/)  | {{ tx.profile }} | {{ tx.description }}  | `{{ tx.component }}`  | `{{ tx.module }}` |
  {% endif -%}
{% endfor %}

**Spring Boot** 
There is a [Spring Boot starter] for SVS-based IHE transactions.
{: .notice--info}

[Spring Boot starter]: {{ site.baseurl }}{% link _pages/boot/boot-svs.md %}