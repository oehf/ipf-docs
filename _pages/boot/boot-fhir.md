---
title: Spring Boot FHIR support
layout: single
permalink: /docs/boot-fhir/
classes: wide
---

`ipf-fhir-spring-boot-starter` sets up the infrastructure for FHIR-based IHE transactions
 
The dependency on the IPF [Spring Boot] IHE FHIR starter module is for FHIR STU3:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.boot</groupId>
        <artifactId>ipf-fhir-dstu3-spring-boot-starter</artifactId>
    </dependency>
```

and for FHIR R4:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.boot</groupId>
        <artifactId>ipf-fhir-r4-spring-boot-starter</artifactId>
    </dependency>
```


Both `ipf-fhir-spring-boot-starter` modules auto-configure by default:
 
* the FHIR Servlet ([`IpfFhirServlet`](../apidocs/org/openehealth/ipf/boot/fhir/IpfBootFhirServlet.html) )
* a [`NamingSystemService`](../apidocs/org/openehealth/ipf/commons/ihe/fhir/support/DefaultNamingSystemServiceImpl.html) instance
* mappings for translating FHIR requests into PIX Query or PDQ requests and vice versa

You can define your own beans of this type in order to override the defaults.

The modules define a mandatory instance of [`ca.uhn.fhir.context.FhirContext`](https://hapifhir.io/apidocs/ca/uhn/fhir/context/FhirContext.html) 
in the correct FHIR version, the bean name is `fhirContext`. The instance can, however, be customized by providing a 
[`FhirContextCustomizer`](../apidocs/org/openehealth/ipf/boot/fhir/FhirContextCustomizer.html) bean:

```java

    @Bean
    public FhirContextCustomizer fhirContextCustomizer() {
        return new FhirContextCustomizer() {
            
            public void customizeFhirContext(FhirContext fhirContext) {
                // configure FhirContext here
            }
        }
    }
```

Furthermore, if a single `org.springframework.cache.CacheManager` bean is available and the application
property `ipf.fhir.caching` is set to true, the following caching storage beans are set up:

* a `IPagingProvider` for [paging results](http://hapifhir.io/doc_rest_server.html#Paging_Providers)


`ipf-fhir-spring-boot-starter` modules do *not*  transitively depend on the respective Camel-dependent IHE FHIR
modules as these have been split into support for IHE MHD, PIXm/PDQm, QEDm and RESTful ATNA, respectively. 
So, e.g. in order to provide STU3 MHD endpoints, you have to include

```xml
        <dependency>
            <groupId>org.openehealth.ipf.platform-camel</groupId>
            <artifactId>ipf-platform-camel-ihe-fhir-stu3-mhd</artifactId>
        </dependency>
```

into your project descriptor.


`ipf-fhir-spring-boot-starter` modules provides the following application properties:

| Property (`ipf.fhir.`)                | Default         | Description                                        |
|---------------------------------------|-----------------|----------------------------------------------------|
| `caching`                             | false           | Whether to set up a cache for paging |
| `path`                                | /fhir           | Path that serves as the base URI for the FHIR services |
| `fhir-version`                        | (depends on module) | FHIR Version | 
| `identifier-naming-systems`           |                 | Resource containing a bundle of FHIR NamingSystem resources used for mapping from FHIR URIs to OIDs and namespaces |
| `servlet.init`                        |                 | init parameters for the FHIR servlet |
| `servlet.load-on-startup`             | 1               | Load on startup priority of the FHIR servlet |
| `servlet.name`                        | FhirServlet     | Name of the FHIR servlet |
| `servlet.paging-requests`             | 50              | Number of concurrent paging requests that can be handled |
| `servlet.default-page-size`           | 50              | Default number of result entries to be returned if no _count parameter is specified in a search |
| `servlet.max-page-size`               | 100             | Maximum number of result entries to be returned even if the _count parameter of a search demands for more |
| `servlet.distributed-paging-provider` | false  | Whether the Paging Provider cache is expected to be distributed, so that serialization of result bundles is necessary. In this case, FHIR endpoints must not use lazy-loading of results. |
| `servlet.logging`                     | false           | Whether server-side request logging is enabled |
| `servlet.pretty-print`                | true            | Whether pretty-printing responses is enabled |
| `servlet.response-highlighting`       | true            | Whether color-coding responses queried from a Web Browser is enabled |
| `servlet.strict`                      | false           | Whether FHIR resource parsing is strict |


See [ipf-spring-boot-starter] and [ipf-atna-spring-boot-starter] for additional properties.


The starter module does *not* set up a Camel servlet for serving MHD [ITI-68] (Retrieve Document) transactions.
Camel provides a Spring boot starter module for this:

```xml
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-servlet-starter</artifactId>
        </dependency>
```

[camel-servlet-starter] provides the following application properties:

| Property (`camel.component.servlet.mapping.`) | Default                | Description                                        |
|-----------------------------------------------|------------------------|----------------------------------------------------|
| `enabled`                                     | true                   | Enables the automatic mapping of the servlet component into the Spring web context |
| `contextPath`                                 | /camel/*               | Context path used by the servlet component for automatic mapping |
| `servletName`                                 | CamelServlet           | The name of the Camel servlet |


[Spring Boot]: https://projects.spring.io/spring-boot/
[ipf-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot.md %}
[ipf-atna-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot-atna.md %}
[ITI-68]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti68.md %}
[camel-servlet-starter]: https://github.com/apache/camel/blob/master/components/camel-servlet/src/main/docs/servlet-component.adoc