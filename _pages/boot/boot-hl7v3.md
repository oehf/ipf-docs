---
title: Spring Boot HL7v3 support
layout: single
permalink: /docs/boot-hl7v3/
classes: wide
---

`ipf-hl7v3-spring-boot-starter` sets up the infrastructure for [HL7v3-based IHE transactions].
 
The dependency on the IPF [Spring Boot] IHE HL7v3 starter module is:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.boot</groupId>
        <artifactId>ipf-hl7v3-spring-boot-starter</artifactId>
    </dependency>
```

Furthermore, if a single `org.springframework.cache.CacheManager` bean is available and the application
property `ipf.hl7v3.caching` is set to true, the following caching storage beans are set up:

* `cachingAsynchronyCorrelator` for interactive continuation

The actual [cache implementation](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html)
being used is the one that Spring Boot finds on the classpath.

`ipf-hl7v3-spring-boot-starter` provides the following application properties:

| Property (`ipf.hl7v3.`)     | Default        | Description                                         |
|----------------------------|-----------------|-----------------------------------------------------|
| `caching`                  | false           | Whether to set up a cache for paging requests |

See [ipf-spring-boot-starter] and [ipf-atna-spring-boot-starter] for additional properties.

This starter module also transitively depends on [cxf-spring-boot-starter-jaxws] that sets up the CXF
web service stack including the Camel CXF servlet, so you don't have to care about this anymore.

`cxf-spring-boot-starter-jaxws` provides the following application properties:

| Property (`cxf.`)          | Default                | Description                                         |
|----------------------------|------------------------|-----------------------------------------------------|
| `path`                     | /services              | Path that serves as the base URI for the services |
| `servlet.init`             | empty map              | optional servlet init parameters |
| `servlet.load-on-startup`  | -1                     | startup order |


[HL7v3-based IHE transactions]: {{ site.baseurl }}{% link _pages/ihe/hl7v3/hl7v3.md %}
[Spring Boot]: https://projects.spring.io/spring-boot/
[ipf-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot.md %}
[ipf-atna-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot-atna.md %}
[cxf-spring-boot-starter-jaxws]: https://cxf.apache.org/docs/springboot.html