---
title: Spring Boot support for transactions based on XACML 2.0
layout: single
permalink: /docs/boot-xacml20/
classes: wide
---

`ipf-xacml20-spring-boot-starter` sets up the infrastructure for [XACML20-based IHE transactions].

The dependency on the IPF [Spring Boot] IHE XACML 2.0 starter module is:

```xml

<dependency>
    <groupId>org.openehealth.ipf.boot</groupId>
    <artifactId>ipf-xacml20-spring-boot-starter</artifactId>
</dependency>
```

`ipf-xacml20-spring-boot-starter` provides the following application properties:

| Property (`ipf.xacml20.`) | Default | Description |
|---------------------------|---------|-------------|
|                           |         |             |

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
[XACML20-based IHE transactions]: {{ site.baseurl }}{% link _pages/ihe/xacml20/xacml20.md %}