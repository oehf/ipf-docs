---
title: IPF 4.4 Migration Guide
layout: single
permalink: /docs/migration-4.4/
toc: true
toc_icon: align-left  
toc_sticky: true
---

IPF 4.4 comes with some major changes that must be considered when upgrading from IPF 4.3.

## MLLP component

The underlying NIO component of IHE MLLP components was changed from Apache Mina to Netty, using camel-netty instead
of camel-mina as Camel component. Although the inbound and outbound data formats remained the same, some changes were
necessary when configuring the endpoint:

* The endpoint parameters `options` and `optionProvider` have been renamed to `iheOptions` and `iheOptionProvider`, respectively, for all affected components, because `options` is already used in camel-netty.
* The `cachedAddress` parameter (for caching InetAddress instances after a DNS lookup) is not available with camel-netty. This parameter was useful for container setups where hosts tend to change their virtual IPs. Netty uses the JDK resolution mechanism that can be configured by setting the security property `networkaddress.cache.ttl` in JVM `java.security` to a few seconds only (defaults to 30s usually). You can override this value by providing this in a separate properties file and start your JVM with the system property `-Djava.security.properties=<URL of file>`.
* The `hl7codec` bean (for the MLLP codec) must be replaced by `hl7decoder` and `hl7encoder` beans of type `org.apache.camel.component.hl7.HL7MLLPNettyEncoderFactory` and `org.apache.camel.component.hl7.HL7MLLPNettyDecoderFactory`, respectively. `ipf-hl7-spring-boot-starter` has been been updated accordingly.

A new feature is _connection multiplexing_. Concurrent MLLP requests and responses can be transferred over a single connection; responses are associated with their requests via a correlation strategy that relies an MSH-10/MSA-2 values. This can be achieved by setting `producerPoolEnabled=false&correlationManager=#hl7Correlation` on the producer side, where hl7CorrelationManager is a bean instance of type `org.openehealth.ipf.platform.camel.ihe.mllp.core.Hl7CorrelationManager`.

Starting with this release, the value in field MSH-18 can determine the charset used for parsing and rendering HL7 messages. The `hl7decoder` bean must be configured to pass on the decoded MLLP message as a byte array to the HL7 parser; this can be achieved by setting the property `produceString` to `false`.

## API changes

* `Interceptor*` classes in the package `org.openehealth.ipf.platform.camel.ihe.core` don't use an `Endpoint` type parameter anymore. 

## 3rd party library changes

* Most notably, [Camel](https://camel.apache.org) has been upgraded to version 3.14.x. Please consult the [Camel migration guide](https://camel.apache.org/manual/latest/migration-and-upgrade.html) for details. "Normal" usage of Camel routes should not be affected, but functionality has been moved to individual Camel modules and there have been various package changes.