---
title: DICOM audit support
layout: single
permalink: /docs/audit/
toc: true
toc_icon: align-left  
---


The `ipf-commons-audit` module contains a reimplementation and improvement of the ATNA functionality 
contained in the `ipf-oht-atna-*` modules that originate from the Eclipse OpenHealthTools project.
This includes building [DICOM][]/[ATNA][]-compliant Audit Records as well as queueing and sending them to
an Audit Repository.

The most important improvements are:

* type-safe Audit Event builders, that enforce setting mandatory members and avoid setting
coded values into the wrong places (e.g. accidentally setting a 
`ParticipantObjectTypeCodeRole` code as `ParticipantObjectTypeCode` code).
* possibility to select an Audit Message XML schema that belongs to a specific [DICOM] version
* possibility to plug-in your own exception handler in case the Audit Repository is not reachable
* configuration is done via an `AuditContext` bean instead of a static global class. You
can create and use as many `AuditContext` configuration beans as you wish.


## Constructing an Audit Message

All of the IHE components implemented by IPF automatically use the audit support of this module
for ATNA auditing. You only need to configure an [AuditContext](../apidocs/org/openehealth/ipf/commons/audit/AuditContext.html) bean.
When you use one of the Spring Boot starter modules, this bean is already provided for you. 

In order to create and submit your own Audit Message, perform the following steps

1. Construct your Audit Message by using one of the builder classes contained in the
`org.openehealth.ipf.commons.audit.event` package. Note that the IHE 
modules (such as `ipf-commons-ihe-core` or `ipf-commons-ihe-xds`) inherit from some of
these builders to create IHE-style ATNA-compliant Audit messages more easily.
2. Call the `getMessage` or `getMessages` method to obtain the `AuditMessage` from the
builder.
3. Submit the Audit Message to the configured destination by calling 
`AuditContext#audit(AuditMessage...)` method.

Every Audit Message and Audit Message builder also has a `validate` method that check basic
restrictions defined by DICOM or IHE.

The `AuditMessageBuilder` implementations are modelled corresponding to the definitions of the
[DICOM Specific Audit Messages](http://dicom.nema.org/medical/dicom/current/output/html/part15.html#sect_A.5.3). The delegate builder class `IHEAuditMessageBuilder` has builder sub classes that correspond with the IHE [ITI specification, Volume 2a](http://ihe.net/uploadedFiles/Documents/ITI/IHE_ITI_TF_Vol2a.pdf), section 3.20.4.1. 

The DICOM serialization strategies produce XML files that validate against the schema of the respective revision of the [DICOM Audit Message Schema](http://dicom.nema.org/medical/dicom/current/output/html/part15.html#sect_A.5.1).

For more details on the API, please study the [javadocs](../apidocs/org/openehealth/ipf/commons/audit/package-frame.html).


## Example

Auditing an application start event

```java
auditContext.audit(
    new ApplicationActivityBuilder.ApplicationStart(EventOutcomeIndicator.Success)
        .setAuditSource(auditContext)
        .setApplicationParticipant(
                appName,
                null,
                appName,
                AuditUtils.getLocalHostName())
        .addApplicationStarterParticipant(System.getProperty("user.name"))
        .getMessage()
);
```

Auditing a change to a user account:

```java
auditContext.audit(
new SecurityAlertBuilder(EventOutcomeIndicator.Success, null, EventTypeCode.UserSecurityAttributesChanged)
                .addActiveParticipant(adminUserId, null, adminUserName, true, null, networkId)
                .addParticipantObjectIdentification(
                        ParticipantObjectIdTypeCode.UserIdentifier,
                        targetUserName,
                        null,
                        null,
                        targetUserId,
                        ParticipantObjectTypeCode.Person,
                        ParticipantObjectTypeCodeRole.User,
                        null, null)
                .getMessage()
);
```

## Hints

The `org.openehealth.ipf.commons.audit.utils.AuditUtils` class contains static methods to obtain
runtime information like process ID, current user, local host and IP address.


## Configuration

The `AuditContext` interface (and its [DefaultAuditContext](../apidocs/org/openehealth/ipf/commons/audit/DefaultAuditContext.html) implementation) 
is the only place to configure static details for auditing, e.g. whether auditing is activated, the location of the Audit Repository, or 
the transmission protocol. It also allows to setup strategies for serialization, whether to send synchronously or asynchronously, and how errors are handled.

### Generic properties

| Property                   | Default   | Description                                                  |
| -------------------------- | --------- | ------------------------------------------------------------ |
| `auditEnabled`             | false     | Whether audit is sent to the repository or not               |
| `auditRepositoryHost`      | localhost | Host name of the audit repository where audit records are sent to |
| `auditRepositoryPort`      | 514       | Port of the the audit repository where audit records are sent to |
| `auditRepositoryTransport` | UDP       | Transport protocol. See info about Audit Transmission Protocols below. |


### Content properties

| Property                          | Default   | Description                                                  |
| :-------------------------------- | --------- | ------------------------------------------------------------ |
| `sendingApplication`              | IPF       | Sending application for the Syslog header info               |
| `auditSourceId`                   | IPF       | Audit source ID for the source identification of the audit message |
| `auditEnterpriseSiteId`           | IPF       | Audit enterprise site ID for the source identification of the audit message |
| `auditSource`                     | 9 (Other) | Audit source type for the source identification of the audit message |
| `includeParticipantsFromResponse` | false     | Whether to include participant objects from a response into the audit message |

### Advanced properties

| Property                      | Default                                    | Description                                                  |
| ---------------------------   | :----------------------------------------- | ------------------------------------------------------------ |
| `auditTransmissionProtocol`   | Instance of `UDPSyslogSenderImpl`          | Transport implementation. Overrules `auditRepositoryTransport` |
| `auditMessageQueue`           | Instance of `SynchronousAuditMessageQueue` | Audit message dispatcher implementation                      |
| `serializationStrategy`       | Instance of `Current` (i.e. DICOM2017c)    | Serialization implementation                                 |
| `tlsParameters` (3.7)         | using system defaults                      | Parameters used for TLS-based transmission                   |
| `auditMetadataProvider` (3.7) | Instance of DefaultAuditMetadataProvider   | Provider for header data used for SYSLOG-based transmission  |
| `auditMessagePostProcessor`   | no-op                                      | Audit Message Postprocessing, called before audit message is dispatched |
| `auditExceptionHandler`       | Instance of `LoggingAuditExceptionHandler` | Handler to be called if the delivery of audit message to the audit repository has failed |
| `auditValueIfMissing`         | `MISSING`                                  | Value used if a mandatory audit attribute is not present

The default setup is to send Audit Messages via UDP to `localhost:514`, and handle delivery errors by just logging them.
For production usage, it is usually required to configure:

* a TLS connection to a remote Audit Record Repository
* some decent strategy for handling failed connections


### Audit Transmission Protocols

The transmission protocol determines the network protocol used for sending an audit record to the Audit Record Repository.

| `auditRepositoryTransport`  | `auditTransmissionProtocol` class          | Description                                                  |
| --------------------------- | :----------------------------------------- | ------------------------------------------------------------ |
| `UDP`                       | [UDPSyslogSenderImpl](../apidocs/org/openehealth/ipf/commons/audit/protocol/UDPSyslogSenderImpl.html)           | UDP transport as SYSLOG record without delivery guarantee. Failed delivery is ignored.
| `TLS`                       | [TLSSyslogSenderImpl](../apidocs/org/openehealth/ipf/commons/audit/protocol/TLSSyslogSenderImpl.html)           | Blocking TLS transport as SYSLOG record
| `NIO-TLS` or `NETTY-TLS`    | [NettyTLSSyslogSenderImpl](../apidocs/org/openehealth/ipf/commons/audit/protocol/NettyTLSSyslogSenderImpl.html) | Non-blocking TLS transport as SYSLOG record. Requires Netty library on the classpath
| `VERTX-TLS`                 | [VertxTLSSyslogSenderImpl](../apidocs/org/openehealth/ipf/commons/audit/protocol/VertxTLSSyslogSenderImpl.html) | Non-blocking TLS transport as SYSLOG record. Requires Vertx library on the classpath


### Audit Message Queues

There are a couple of implementations for "how" IPF handles audit records. Some implementations circumvent the 
Audit Transmission Protocol.

| `auditMessageQueue` class          |  Description                                                  |
| ---------------------------        |  ------------------------------------------------------------ |
| [SynchronousAuditMessageQueue](../apidocs/org/openehealth/ipf/commons/audit/queue/SynchronousAuditMessageQueue.html)     |  Synchronously pass the audit record to the `auditTransmissionProtocol` instance
| [AsynchronousAuditMessageQueue](../apidocs/org/openehealth/ipf/commons/audit/queue/AsynchronousAuditMessageQueue.html)   |  Asynchronously pass the audit record to the `auditTransmissionProtocol` instance. Must be initialized with an `ExecutorService`
| [JMSAuditMessageQueue](../apidocs/org/openehealth/ipf/commons/audit/queue/JMSAuditMessageQueue.html)                     |  Send the audit record to a JMS queue. SYSLOG header data is sent as JMS properties. Requires JMS API library.
| [BasicHttpAuditMessageQueue](../apidocs/org/openehealth/ipf/commons/audit/queue/BasicHttpAuditMessageQueue.html) (3.7)   |  Send the audit record to a HTTP service. SYSLOG header data is sent as HTTP properties.
| [LoggingAuditMessageQueue](../apidocs/org/openehealth/ipf/commons/audit/queue/LoggingAuditMessageQueue.html)             |  Just log the audit record to an SLF4J logger
| [CamelAuditMessageQueue](../apidocs/org/openehealth/ipf/commons/audit/queue/CamelAuditMessageQueue.html)                 |  Send the audit record via a Camel producer. SYSLOG header data is sent as Camel headers.
| [CompositeAuditMessageQueue](../apidocs/org/openehealth/ipf/commons/audit/queue/CompositeAuditMessageQueue.html)         |  Send the audit record sequentially using one of the implementations listed above


### TLS Parameters (3.7)

With TLS-based audit transmission, the default TLS parameters (as passed using the `javax.net.ssl.*` system properties)
are used. However, sometimes the TLS Audit connection requires a different TLS configuration. In this case, an instance
of [CustomTlsParameters]((../apidocs/org/openehealth/ipf/commons/audit/CustomTlsParameters.html)) can be used to specifically assign a different TLS setup
to the `AuditContext`.




[DICOM]: https://dicom.nema.org/medical/dicom/current/output/html/part15.html#sect_A.5
[ATNA]: http://ihe.net/uploadedFiles/Documents/ITI/IHE_ITI_TF_Vol2a.pdf