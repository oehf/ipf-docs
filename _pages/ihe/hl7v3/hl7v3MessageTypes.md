---
title: Supported data types in HL7v3-based IPF IHE components
layout: single
permalink: /docs/ihe/hl7v3MessageTypes/
classes: wide
---

HL7v3-based components do not mandate any special data model. Web Service metadata (WSDL documens) reflects
this by declaring all message parts to be `xsd:anyType`.

What the components deliver to a Camel route, is always an HL7 v3 message as String. Groovy 
[DSL extensions for XML processing](https://camel.apache.org/groovy-dsl.html) can be efficiently used to
handle and prepare request and response messages.

In addition, starting with the version 4.1.0, IPF introduces model classes for request and response messages of 
selected HL7v3 transactions (ongoing work), as well as [Camel type converters](https://camel.apache.org/type-converter.html) 
for transforming instances of these classes into Strings and back again.  

In a Maven-based environment, the following dependency must be registered in `pom.xml`:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.commons</groupId>
        <artifactId>ipf-commons-ihe-hl7v3model</artifactId>
        <version>${ipf-version}</version>
    </dependency>
```


Currently, the following model classes are available:

| Message          | Model class
|------------------|--------------------------------------------------------------------------------------------------
| ITI-45 request   | org.openehealth.ipf.commons.ihe.hl7v3.core.requests.PixV3QueryRequest
| ITI-45 response  | org.openehealth.ipf.commons.ihe.hl7v3.core.responses.PixV3QueryResponse

What the components expect to obtain from a Camel route (e.g. outgoing requests on producer side and outgoing responses
on consumer side) is an XML String containing an HL7 v3 message, or something that can be transformed into such String
by the means of [Camel type converters](https://camel.apache.org/type-converter.html) — e.g. byte array,
input stream, stream reader, DOM document, XSLT source, model class instance, etc.
