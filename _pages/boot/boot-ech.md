---
title: Spring Boot support for eCH transactions
layout: single
classes: wide
permalink: /docs/boot-ech/
---

`ipf-ech-spring-boot-starter` sets up the infrastructure for [eCH-based transactions].

The dependency on the IPF [Spring Boot] eCH starter module is:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.boot</groupId>
        <artifactId>ipf-ech-spring-boot-starter</artifactId>
    </dependency>
```

`ipf-ech-spring-boot-starter` defines the property prefix `ipf.ech.`, but does not currently contribute any
application properties of its own. Its purpose is to pull in `ipf-platform-camel-ihe-ech` together with the
web service stack, so that the `ech-0213` and `ech-0214` components are ready to use.

See [ipf-spring-boot-starter] and [ipf-atna-spring-boot-starter] for additional properties. Note that the
eCH services do not produce ATNA audit records; the ATNA starter comes in as a transitive dependency of the
IHE infrastructure.

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
[eCH-based transactions]: {{ site.baseurl }}{% link _pages/ihe/ech/ech.md %}
