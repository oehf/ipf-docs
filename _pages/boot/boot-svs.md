---
title: Spring Boot support for SVS transactions
layout: single
permalink: /docs/boot-svs/
classes: wide
---

`ipf-svs-spring-boot-starter` sets up the infrastructure for [SVS-based IHE transactions].

The dependency on the IPF [Spring Boot] IHE SVS starter module is:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.boot</groupId>
        <artifactId>ipf-svs-spring-boot-starter</artifactId>
    </dependency>
```

`ipf-svs-spring-boot-starter` provides the following application properties:

| Property (`ipf.svs.`) | Default | Description |
|-----------------------|---------|-------------|
|                       |         |             |

See [ipf-spring-boot-starter] and [ipf-atna-spring-boot-starter] for additional properties.

This starter module also transitively depends on [cxf-spring-boot-starter-jaxws] that sets up the CXF
web service stack including the Camel CXF servlet, so you don't have to care about this anymore.

`cxf-spring-boot-starter-jaxws` provides the following application properties:

| Property (`cxf.`)         | Default   | Description                                       |
|---------------------------|-----------|---------------------------------------------------|
| `path`                    | /services | Path that serves as the base URI for the services |
| `servlet.init`            | empty map | optional servlet init parameters                  |
| `servlet.load-on-startup` | -1        | startup order                                     |

[Spring Boot]: https://projects.spring.io/spring-boot/
[ipf-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot.md %}
[ipf-atna-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot-atna.md %}
[cxf-spring-boot-starter-jaxws]: https://cxf.apache.org/docs/springboot.html
[SVS-based IHE transactions]: {{ site.baseurl }}{% link _pages/ihe/svs/svs.md %}