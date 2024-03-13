---
title: ATNA Auditing in IPF eHealth Components
layout: single
permalink: /docs/ihe/atna/
classes: wide
---


ATNA auditing functionality is fully integrated into the corresponding IPF IHE components and controlled by the 
`auditContext` endpoint URI parameter. This parameters references a bean of type `AuditContext` with the given name
that bundles all relevant details around ATNA auditing (e.g. whether auditing is enabled, where the Audit Repository 
is located, which wire protocol is to be used, etc.) 

| Parameter name | Values                     | Behavior                                      |  Default | Example                         |
|:---------------|:---------------------------|:----------------------------------------------|:---------|:--------------------------------|
| `auditContext` | `AuditContext` reference   | uses the referenced AuditContext for auditing | n/a      | `?auditContext=#myAuditContext` |
| `audit`        | `true` or `false`          | if true, uses a unique AuditContext bean for auditing    | `true`   | `?audit=false`                  |

You can have as many `AuditContext` beans as you wish (for auditing being turned on/off, using different queue implementations, etc.).
In this case, you _must_ use the `auditContext` parameter. 
If you only have one `AuditContext` bean, you can omit the `auditContext` parameter as this unique instance is
obtained from the context.


## Auditor Configuration

All details regarding ATNA auditing is configured with the `AuditContext`. This is how it
looks like in a Spring XML file:


```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- ATNA Auditing Infrastructure. Everything simply goes into the AuditContext -->

    <bean id="auditContext" class="org.openehealth.ipf.commons.audit.DefaultAuditContext">

        <!-- Whether auditing is enabled. Default is false -->
        <property name="auditEnabled" value="true" />
        <!-- Audit Source ID. Default is "IPF" -->
        <property name="auditSourceId" value="MySource" />
        <!-- Enterprise Site ID. Default is "IPF" -->
        <property name="auditEnterpriseSiteId" value="MyEnterprise" />
        <!-- Default is "localhost" -->
        <property name="auditRepositoryHost" value="arr.server.com" />
        <!-- Default is 514 -->
        <property name="auditRepositoryPort" value="6514" />
        <!-- Audit Transport (UDP, TLS). Default is "UDP" -->
        <property name="auditRepositoryTransport" value="TLS" />
        <!-- Audit Source Type. Default is "Other" -->
        <property name="auditSource">
            <value type="org.openehealth.ipf.commons.audit.codes.AuditSourceType">ApplicationServerProcess</value>
        </property>
        <!-- Sending Application for SYSLOG frame. Default is "IPF" -->
        <property name="sendingApplication" value="demo"/>

        <!-- Advanced -->

        <!-- Adding a custom sender implementation. Overrules auditRepositoryTransport.

        <property name="auditTransmissionProtocol" ref="#abc"/>
        -->

        <!-- Setting if the message is sent synchronously (default), asynchronously, via JMS,  or a custom
             implementation like org.openehealth.ipf.platform.camel.ihe.atna.util.CamelAuditMessageQueue.

        <property name="auditMessageQueue" ref="#def"/>
        -->

        <!-- Setting the strategy to serialize an AuditMessage. By default, the latest DICOM Audit
             standard is used, but you may choose an older one (e.g. Dicom2016c). Or write your own that
             is maybe not even DICOM based, e.g. rending FHIR AuditEvents.

        <property name="serializationStrategy">
            <bean class="org.openehealth.ipf.commons.audit.marshal.dicom.Current"/>
        </property>
        -->
        
        <!-- Object responsible for handling exceptions that occur while sending the audit records
             to an audit repository. The default handler simply writes a warning to the log.
              
        <property name="auditExceptionHandler" ref="#ghi"/>
        -->
        
        <!-- Setting this to true causes data from the response being added to the audit records, particularly
             patient identifiers. This is not envisaged by DICOM, but project or legal requirements may overrule
             this. The default, however, is false
             
        <property name="includeParticipantsFromResponse" value="true" />
         -->

        <!-- Sometimes, request parts that are mandatory for auditing are not present, e.g. due to
             bad or incomplete request data. IPF always tries to write proper audit events and
             fills up missing audit attributes with this value
             
        <property name="auditValueIfMissing" value="UNKNOWN" />
         -->
         
    </bean>
</beans>    
```

Spring Boot support in [ipf-atna-spring-boot-starter] instantiates an `AuditContext` which can be configured.
{: .notice--info}

## Configure custom audit event queueing

The delivery queue that is used as channel for the audit sender can be customized by using a different implementation 
of the interface [`AuditMessageQueue`](../../apidocs/org/openehealth/ipf/commons/audit/queue/AuditMessageQueue.html). 
There are implementations for synchronous and asynchronous delivery of string-serialized audit messages as well as a 
queues for logging an `AuditingMessage` to a file or sending it to a Camel Endpoint (as described below).

## Configure custom audit event exception handling

The exception handler that is called when transmitting audit records fails can be customized by using a different implementation 
of the interface [`AuditExceptionHandler`](../../apidocs/org/openehealth/ipf/commons/audit/handler/AuditExceptionHandler.html). 
The only existing implementation is to log the exception, but you can implement more elaborate solutions, 
e.g. resending, or using a fallback transmission protocol.

## Specifiying a DICOM version

[DICOM is versioned](http://www.dclunie.com/dicom-status/status.html), and each year a couple of updates are published. 
Some updates also affect the way Audit messages are serialized (as specified in DICOM
[Part15](http://dicom.nema.org/medical/dicom/current/output/html/part15.html)).

As IHE ATNA does not reference a specific DICOM version, in the past there have been interoperability issues where
the Application Node sends ATNA events that the repository actor was not (yet) capable to consume. To ensure
a certain degree of backwards compatibility, the `serializationStrategy` can be configured to use a certain
implementation version, where [`Current`](../../apidocs/org/openehealth/ipf/commons/audit/marshal/dicom/Current.html) 
always references the latest relevant version.
You can also provide a custom implementation of `org.openehealth.ipf.commons.audit.marshal.SerializationStrategy`
that creates other representations of an `AuditMessage` (there is e.g. an implementation creating JSON-serialized 
`AuditEvent` FHIR resources).    

## Configuring TLS details

If `TLS` is selected as `auditRepositoryTransport`, the standard [JSSE system properties] are used to customize keystore,
truststore, passwords, cipher suites and TLS protocol. Once a TLS connection is established, it is kept open as long as
it is usable.

## Routing audit messages to Camel endpoints

Instead of sending audit messages to a syslog server, 
they can also be sent to Camel endpoints. For that purpose, add the following dependency to the `pom.xml`:

```xml
    <dependencies>
       <dependency>
          <groupId>org.openehealth.ipf.platform-camel</groupId>
          <artifactId>ipf-platform-camel-ihe-atna</artifactId>
          <version>${ipf-version}</version>
       </dependency>
    </dependencies>
```

Now an instance of `CamelEndpointSender` can be configured in the application context:

```xml
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:camel="http://camel.apache.org/schema/spring"
           xmlns:ipf="http://openehealth.org/schema/ipf-commons-core"    
           xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://camel.apache.org/schema/spring
        http://camel.apache.org/schema/spring/camel-spring.xsd
        http://openehealth.org/schema/ipf-commons-core
        http://openehealth.org/schema/ipf-commons-core.xsd">

        <ipf:globalContext id="globalContext"/>

        <camel:camelContext id="camelContext">
           <camel:routeBuilder ref="..."/>
        </camel:camelContext>
    
        <bean id="auditContext" class="org.openehealth.ipf.commons.audit.DefaultAuditContext">
            <property name="auditEnabled" value="true"/>
            <property name="auditMessageQueue" ref="camelEndpointSender"/>
        </bean>
    
        <bean id="camelEndpointSender"
              class="org.openehealth.ipf.platform.camel.ihe.atna.util.CamelAuditMessageQueue" init-method="init" destroy-method="destroy">
            <property name="camelContext" ref="camelContext" />
            <property name="endpointUri" value="direct:input" />
        </bean>
    
    </beans>
```

The `CamelEndpointSender` instance is configured to send audit message objects to the `direct:input` endpoint of a Camel route. 
The audit message is sent as In-Only exchange to the endpoint, the message body contains the `AuditMessage` instance.
Additionally the following Camel message headers are populated:

* `org.openehealth.ipf.platform.camel.ihe.atna.datetime`: date and time when the audit event was generated
* `org.openehealth.ipf.platform.camel.ihe.atna.destination.address`: the audit repository IP address derived from the configured endpoint URI
* `org.openehealth.ipf.platform.camel.ihe.atna.destination.port`: the audit repository port derived from the configured endpoint URI


## XUA Support

ATNA audit routines of Web Service-based eHealth components expect to find a prepared XUA user name (see the IHE ITI-40 transaction) in the 
CXF request message context property defined in `org.openehealth.ipf.commons.ihe.ws.cxf.audit.AbstractAuditInterceptor#DATASET_CONTEXT_KEY`, 
which is supposed to be set by project-specific mechanisms (not defined in IPF).

When no such property exists, IPF tries to determine the XUA user name by means of processing the SAML2 assertion contained in the 
WS-Security SOAP header of the request message.

Note that when the XUA user name cannot be determined, IPF does neither throw any exceptions nor performs any validation of SAML2 assertions. 
In other words, the support for XUA is restricted to filling in the corresponding field in ATNA audit records.
{: .notice--warning}

## FHIR Basic Audit Log Patterns [BALP] support

As of IPF 4.8.0, support for IHE [BALP] has been added:

* `DefaultBalpAuditContext` was added as subclass of `DefaultAuditContext`. This context class carries some more audit-related configuration properties that are necessary for REST-based FHIR auditing
* `BalpJsonSerializationStrategy` and `BalpXmlSerializationStrategy` have been added as implementation of `SerializationStrategy`. This converts the DICOM-based audit data model into its serialized FHIR `AuditEvent` equivalent.
* `ApacheFhirRestTLSAuditRecordSender` and `MethanolFhirRestTLSAuditRecordSender` have been added as implementation of `AuditTransmissionProtocol`. This uses the configured Restful sender in HAPI FHIR context for sending the serialized `AuditEvent` to a FHIR repository.

[ipf-atna-spring-boot-starter]: {{ site.baseurl }}{% link _pages/boot/boot-atna.md %}
[JSSE system properties]:   https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#InstallationAndCustomization
[BALP]:   https://profiles.ihe.net/ITI/BALP/
