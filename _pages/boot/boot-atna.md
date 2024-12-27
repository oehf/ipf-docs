---
title: Spring Boot ATNA support
layout: single
permalink: /docs/boot-atna/
classes: wide
---

`ipf-atna-spring-boot-starter` sets up the infrastructure for ATNA auditing.
 
The dependency on the IPF [Spring Boot] ATNA starter module is:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.boot</groupId>
        <artifactId>ipf-atna-spring-boot-starter</artifactId>
    </dependency>
```

All IHE-related Spring boot starter modules depend on this starter module, so if you use one of those you do not have to
explicitly depend on `ipf-atna-spring-boot-starter`.
{: .notice--info}

As of IPF 3.7, `ipf-atna-spring-boot-starter` does not depend on `ipf-spring-boot-starter` anymore. This makes it
usable in scenarios where no other IPF modules are used and does not drag in 3rd party dependencies like Apache Camel
or Groovy anymore.

`ipf-atna-spring-boot-starter` auto-configures by default:

* a configurable [`AuditContext`](../apidocs/org/openehealth/ipf/commons/audit/DefaultAuditContext.html) bean
* a basic listeners that write ATNA audit events upon application startup and shutdown 
  ([`ApplicationStartEventListener`](../apidocs/org/openehealth/ipf/boot/atna/ApplicationStartEventListener.html) 
  and [`ApplicationStopEventListener`](../apidocs/org/openehealth/ipf/boot/atna/ApplicationStopEventListener.html))
* a basic listener for login events if Spring Security is on the classpath ([`AuthenticationListener`](../apidocs/org/openehealth/ipf/boot/atna/AuthenticationListener.html)) 

The `AuditContext` instance can be customized by providing a [`AuditContextCustomizer`](../apidocs/org/openehealth/ipf/boot/atna/AuditContextCustomizer.html) bean:

```java

    @Bean
    public AuditContextCustomizer auditContextCustomizer() {
        return new AuditContextCustomizer() {
            
            public void customizeAuditContext(AuditContext auditContext) {
                // configure AuditContext here
            }
        };
    }
```

You can define your own @Beans of this type in order to override the defaults.

`ipf-atna-spring-boot-starter` provides the following application properties that configures the `AuditContext`
as described [here]({{ site.baseurl }}{% link _pages/ihe/atna.md %}).

| Property (`ipf.atna.`)         | Default               | Description                                                           |
|--------------------------------|-----------------------|-----------------------------------------------------------------------|
| `audit-enabled`                | false                 | Whether auditing is enabled                                           |
| `audit-repository-host`        | localhost             | Host of the ATNA repository to send the events to                     |
| `audit-repository-port`        | 514                   | Port of the ATNA repository to send the events to                     |
| `audit-repository-transport`   | UDP                   | Wire transport format (UDP, TLS, NETTY-UDP, NETTY-TLS, FHIR-REST-TLS) |
| `audit-source-id`              | `${spring.application.name}` | Source ID for ATNA events                                             |
| `audit-enterprise-site-id`     |                       | Enterprise Site ID for ATNA events                                    |
| `include-participants-from-response`| false            | Whether to include (patient) participants from responses as well      |
| `audit-source-type`            | 4 (ApplicationServerProcess) | Type of Audit Source                                                  |
| `audit-queue-class`            | `org.openehealth.ipf.commons.audit.queue.SynchronousAuditMessageQueue` | Queue implementation for auditing                                     |
| `audit-sender-class`           | as indicated by `audit-repository-transport` | ATNA sender implementation                                            |
| `audit-exception-handler-class`| `org.openehealth.ipf.commons.audit.handler.LoggingAuditExceptionHandler`| Exception handler impleemntation                                      |
| `audit-value-if-missing`       | `UNKNOWN`             | Value used for mandatory audit attributes that are not set            |
| `audit-message-post-processor-class` | none | Class of the audit message post-processor | 
| `ws-audit-dataset-enricher-class`    | none | Class of the audit dataset enricher for Web Service based transactions (IPF 5.0+) |  
| `fhir-audit-dataset-enricher-class`  | none | Class of the audit dataset enricher for FHIR based transactions (IPF 5.0+) |

Instead of specifying class names in configuration properties, you can provide Spring @Beans of the types `AuditMessageQueue`, 
`AuditMessagePostProcessor`, `AuditTransmissionProtocol`, `AuditMetadataProvider`, `AuditExceptionHandler`, `WsAuditDatasetEnricher`, 
and `FhirAuditDatasetEnricher`.  Moreover, you can provide an own bean of the type `TlsParameters`.

As of IPF 4.8.0, you can audit following the IHE Basic Audit Log Patterns (BALP). By setting `ipf.atna.balp` properties you can enable FHIR-based auditing.

| Property (`ipf.atna.balp.`)          | Default                                         | Description                                                  |
|--------------------------------------|-------------------------------------------------|--------------------------------------------------------------|
| `audit-repository-context-path`      | ""                                              | URL context path of the FHIR Audit Repository                |
| `audit-event-serialization-type`     | json                                            | Whether to encode the AuditEvent as `json` or `xml`          |
| `oauth.id-path`                      | jti                                             | Where to extract audit-relevant data from a JWT access token |
| `oauth.issuer-path`                  | issuer                                          |                                                              |
| `oauth.client-id-path`               | client_id, cid                                  |                                                              |
| `oauth.subject-path`                 | sub                                             |                                                              |
| `oauth.subject-name-path`            | extensions:ihe_iua:subject_name                 |                                                              |
| `oauth.subject-organization-path`    | extensions:ihe_iua:subject_organization         |                                                              |
| `oauth.subject-organization-id-path` | extensions:ihe_iua:subject_organization_id      |                                                              |
| `oauth.subject-role-path`            | extensions:ihe_iua:subject_role                 |                                                              |
| `oauth.purpose-of-use-path`          | extensions:ihe_iua:purpose_of_use               |                                                              |
| `oauth.home-community-id-path`       | extensions:ihe_iua:home_community_id            |                                                              |
| `oauth.national-provider-id-path`    | extensions:ihe_iua:national_provider_identifier |                                                              |
| `oauth.person-id-path`               | extensions:ihe_iua:person_id                    |                                                              |
| `oauth.patient-id-path`              | extensions:ihe_bppc:patient_id                  |                                                              |
| `oauth.doc-id-path`                  | extensions:ihe_bppc:doc_id                      |                                                              |
| `oauth.acp-path`                     | extensions:ihe_bppc:acp                         |                                                              |

[Spring Boot]: https://projects.spring.io/spring-boot/
