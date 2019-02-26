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


`ipf-hl7-spring-boot-starter` auto-configures by default:

* a `org.apache.camel.component.hl7.HL7MLLPCodec`
* a `org.openehealth.ipf.modules.hl7.parser.CustomModelClassFactory` covering IHE-specific PIX and PDQ message structures
* a `ca.uhn.hl7v2.conf.store.ProfileStore` referencing predefined IHE Gazelle profiles
* a `ca.uhn.hl7v2.validation.ValidationContext` without any specific validation rules
* a configurable `ca.uhn.hl7v2.parser.ParserConfiguration`

Furthermore, if a single `org.springframework.cache.CacheManager` bean is available and the application
property `ipf.hl7v2.caching` is set to true, the following caching storage beans are set up:

* `interactiveContinuationStorage` for [interactive continuation]
* `unsolicitedFragmentationStorage` for [unsolicited fragmentation]

You can define your own beans of this type in order to override the defaults.

The module instantiates a mandatory and correctly configured `ca.uhn.hl7v2.HapiContext`. 
The instance can, however, be customized by providing a 
[`HapiContextCustomizer`](../apidocs/org/openehealth/ipf/boot/hl7v2/HapiContextCustomizer.html) bean:

```java

    @Bean
    public HapiContextCustomizer hapiContextCustomizer() {
        return new HapiContextCustomizer() {
            
            public void customizeHapiContext(HapiContext fhirContext) {
                // configure HapiContext here
            }
        }
    }
```


The actual [cache implementation](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html)
being used is the one that Spring Boot finds on the classpath.

`ipf-hl7-spring-boot-starter` provides the following application properties:

| Property (`ipf.hl7v2.`)    | Default               | Description                                        |
|----------------------------|-----------------------|----------------------------------------------------|
| `charset`                  | UTF-8                 | Charset for HL7v2 messages
| `convert-line-feed`        | false                 | Whether to convert line feeds to proper segment separators before parsing starts
| `caching`                  | false                 | Whether to set up caches for paging and unsolicited fragmentation
| `generator`                | file                  | ID generator for HL7 messages. One of "file", "uuid", "nano".


In case you use the file-based ID generator, you configure it as follows:

| Property (`ipf.hl7v2.id-generator`)    | Default               | Description                                        |
|----------------------------------------|-----------------------|----------------------------------------------------|
| `lo`                                   | 100                   | How many IDs to be generated internally before incrementing the file value
| `directory`                            | Value of `Home.getHomeDirectory().getAbsolutePath();` | Directory of the ID file
| `file-name`                            | id_file               | Name of the file
| `never-fail`                           | true                  | If set to false retrieving a new ID may fail if the ID file can not be written/read. If set to true, failures will be ignored, which means that IDs may be repeated after a JVM restart.
| `minimize-reads`                       | false                 | If set to true, the generator minimizes the number of disk reads by caching the last read value. This means one disk read less per X number of IDs generated, but also means that multiple instances of this generator may clobber each other's values.


The instantiated `ca.uhn.hl7v2.parser.ParserConfiguration` can be immediately configured with properties starting with `ipf.hl7v2.parser`

| Property (`ipf.hl7v2.parser`)    | Default               | Description                                        |
|----------------------------------|-----------------------|----------------------------------------------------|
| `allow-unknown-versions`         | false                 | If the HL7 parser accepts unknown HL7 versions     |
| ... further properties of ParserConfiguration   | ...    | ...                                                |

 
See [ipf-spring-boot-starter] and [ipf-atna-spring-boot-starter] for additional properties.


[ipf-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot.md %}
[ipf-atna-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot-atna.md %}
[HL7v2-based IHE transactions]: {{ site.baseurl }}{% link _pages/ihe/mllp/hl7v2.md %}
[Spring Boot]: https://projects.spring.io/spring-boot/
[interactive continuation]: {{ site.baseurl }}{% link _pages/ihe/mllp/hl7v2InteractiveContinuation.md %}
[unsolicited fragmentation]: {{ site.baseurl }}{% link _pages/ihe/mllp/unsolicitedFragmentation.md %}