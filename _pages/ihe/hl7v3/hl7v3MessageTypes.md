---
title: Supported data types in HL7v3-based IPF IHE components
layout: single
permalink: /docs/ihe/hl7v3MessageTypes/
classes: wide
---

### XML strings

HL7v3-based IPF components do not mandate any special data model. Web Service metadata (WSDL documents) reflects
this by defining all message parts as `xsd:anyType`.

What the components deliver to a Camel route, is always an HL7 v3 message as String. Groovy 
[XML processing mechanisms](https://groovy-lang.org/processing-xml.html) can be efficiently used to
parse and create request and response messages.

What the components expect to obtain from a Camel route (e.g. outgoing requests on producer side and outgoing responses
on consumer side) is an XML String containing an HL7 v3 message, or something that can be transformed into such String
by the means of [Camel type converters](https://camel.apache.org/type-converter.html) — e.g. byte array,
input stream, stream reader, DOM document, XSLT source, model class instance, etc.

### Java model classes

Starting with the version 4.1.0, IPF contains model classes for requests and responses of selected HL7v3 transactions,
divided into two categories:
* stub classes, whose structures resemble generic HL7v3 definitions and whose names correspond
  to root XML elements of HL7v3 messages, e.g. `PRPAIN201309UV02Type` for ITI-45 request.
* simplified model classes tailored to a particular IHE transaction, e.g. `PixV3QueryRequest` for ITI-45 request.

For the conversion between these two categories of model classes, as well as between them and XML strings, 
IPF provides corresponding [Camel type converters](https://camel.apache.org/type-converter.html).

Currently, the following stub model classes are available:

| Message                                                                   | Model class                                                         |
|---------------------------------------------------------------------------|---------------------------------------------------------------------|
| ITI-44 request (create)                                                   | `net.ihe.gazelle.hl7v3.prpain201301UV02.PRPAIN201301UV02Type`       |
| ITI-44 request (update)<br>ITI-46 request                                 | `net.ihe.gazelle.hl7v3.prpain201302UV02.PRPAIN201302UV02Type`       |
| ITI-44 request (merge)                                                    | `net.ihe.gazelle.hl7v3.prpain201304UV02.PRPAIN201304UV02Type`       |
| ITI-44 response<br>ITI-46 response<br>ITI-47 continuation cancel response | `net.ihe.gazelle.hl7v3.mcciin000002UV01.MCCIIN000002UV01Type`       |
| ITI-45 request                                                            | `net.ihe.gazelle.hl7v3.prpain201309UV02.PRPAIN201309UV02Type`       |
| ITI-45 response                                                           | `net.ihe.gazelle.hl7v3.prpain201310UV02.PRPAIN201310UV02Type`       |
| ITI-47 request<br>ITI-55 request                                          | `net.ihe.gazelle.hl7v3.prpain201305UV02.PRPAIN201305UV02Type`       |
| ITI-47 continuation request                                               | `net.ihe.gazelle.hl7v3.quqiin000003UV01.QUQIIN000003UV01Type`       |
| ITI-47 continuation cancel request                                        | `net.ihe.gazelle.hl7v3.quqiin000003UV01.QUQIIN000003UV01CancelType` |
| ITI-47 response<br>ITI-47 continuation response<br>ITI-55 response        | `net.ihe.gazelle.hl7v3.prpain201306UV02.PRPAIN201306UV02Type`       |

Simplified model classes:

| Message         | Model class                                                               |                                                             
|-----------------|---------------------------------------------------------------------------|
| ITI-45 request  | `org.openehealth.ipf.commons.ihe.hl7v3.core.requests.PixV3QueryRequest`   |
| ITI-45 response | `org.openehealth.ipf.commons.ihe.hl7v3.core.responses.PixV3QueryResponse` |

Both the model classes and the converters are contained in the following Maven artifact:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.commons</groupId>
        <artifactId>ipf-commons-ihe-hl7v3model</artifactId>
        <version>${ipf-version}</version>
    </dependency>
```

