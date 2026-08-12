---
title: eCH based transactions
layout: single
permalink: /docs/ihe/ech/
classes: wide
---

Besides transactions from IHE and other international profiles, IPF supports the Swiss [eCH][] standards for
querying and managing the *EPR-SPID* — the sectoral personal identifier used in the Swiss Electronic Patient
Record (EPD/DEP/CEP). The corresponding web services are operated by the Central Compensation Office (ZAS)
as part of the UPI register.

As with IHE transactions, IPF provides a Camel component per service, hiding the implementation details on
transport level.

The following eCH services are currently supported:

| Transaction             | Profile          | Description           | IPF Component          |  IPF Module |
|:------------------------|:-----------------|:----------------------|:-----------------------|:------------|
{% for hash in site.data.ihe -%}
  {% assign tx = hash[1] -%}
  {% if tx.format == "eCH" -%}
| [{{ tx.transaction }}](../{{ tx.link }}/)  | {{ tx.profile }} | {{ tx.description }}  | `{{ tx.component }}`  | `{{ tx.module }}` |
  {% endif -%}
{% endfor %}

Note that these services are not IHE transactions: they define no IHE actors, and IPF does not generate
ATNA audit records or run message validation for them. Everything else that applies to
[Web Service based components][] — secure transport, payload logging, custom CXF interceptors — applies here
as well.

**Spring Boot**
There is a [Spring Boot starter] for eCH-based transactions.
{: .notice--info}

[eCH]: https://www.ech.ch
[Web Service based components]: {{ site.baseurl }}{% link _pages/ihe/ws/wsDeployment.md %}
[Spring Boot starter]: {{ site.baseurl }}{% link _pages/boot/boot-ech.md %}
