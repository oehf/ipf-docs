---
title: Spring Boot HL7v2 support
layout: single
permalink: /docs/boot-hl7/
classes: wide
---

`ipf-hl7-spring-boot-starter` sets up the infrastructure for [HL7v2-based IHE transactions].
 
The dependency on the IPF [Spring Boot] IHE HL7 starter module is:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.boot</groupId>
        <artifactId>ipf-hl7-spring-boot-starter</artifactId>
    </dependency>
```


`ipf-hl7-spring-boot-starter` auto-configures 

* `org.apache.camel.component.hl7.HL7MLLPCodec`
* a correctly configured `ca.uhn.hl7v2.HapiContext` 

Furthermore, if a single `org.springframework.cache.CacheManager` bean is available and the application
property `ipf.hl7v2.caching` is set to true, the following caching storage beans are set up:

* `interactiveContinuationStorage` for [interactive continuation]
* `unsolicitedFragmentationStorage` for [unsolicited fragmentation]

The actual [cache implementation](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html)
being used is the one that Spring Boot finds on the classpath.

`ipf-hl7-spring-boot-starter` provides the following application properties:

| Property (`ipf.hl7v2.`)    | Default               | Description                                        |
|----------------------------|-----------------------|-----------------------------------------------------|
| `charset`                  | UTF-8                 | Charset for HL7v2 messages
| `convert-line-feed`        | false                 | Whether to convert line feeds to proper segment separators before parsing starts
| `caching`                  | false                 | Whether to set up caches for paging and unsolicited fragmentation

See [ipf-spring-boot-starter] and [ipf-atna-spring-boot-starter] for additional properties.


[ipf-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot.md %}
[ipf-atna-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot-atna.md %}
[HL7v2-based IHE transactions]: {{ site.baseurl }}{% link _pages/ihe/mllp/hl7v2.md %}
[Spring Boot]: https://projects.spring.io/spring-boot/
[interactive continuation]: {{ site.baseurl }}{% link _pages/ihe/mllp/hl7v2InteractiveContinuation.md %}
[unsolicited fragmentation]: {{ site.baseurl }}{% link _pages/ihe/mllp/unsolicitedFragmentation.md %}