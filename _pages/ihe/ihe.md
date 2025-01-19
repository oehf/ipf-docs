---
title: IPF components for eHealth integration profiles
layout: single
permalink: /docs/ihe/
classes: wide
---


IPF provides support for several eHealth profiles, primarily from the [IHE][] ITI domain.
The basic idea is to offer a [Camel component](https://camel.apache.org/components.html) for each profile transaction.
These components ensure that the technical requirements of the profile are met by applications built on top of the
IPF eHealth Integration Profiles support.

### Context

Core concepts of IHE profiles are:

| Concept          | IHE Definition (Technical Framework, Volume 1) | Description  | Example |
|------------------|------------------------------------------------|--------------|--------|
| Actor            | Actors are information systems or components of information systems that produce, manage, or act on categories of information required by operational activities in the enterprise. | An application role in a distributed system | Patient Identity Cross-Reference Manager aka PIX Manager. |
| Transaction      | Transactions are interactions between actors that communicate the required information through standards-based messages. 	                                                         | A message exchange between actors           | Patient Identity Feed between the Patient Identity Source and the PIX Manager |
| Profile          | Each integration profile is a representation of a real-world capability that is supported by a set of actors that interact through transactions.                                    | A set of actors and transactions            | PIX profile |

IPF is a development framework with special support for the implementation of IHE concepts (i.e. profiles).
This is illustrated with an abstract IHE profile the figure below.

The profile defines three actors and two transactions. Transaction 1 is between Actor 1 and Actor 2 whereas Transaction 2 
is between Actor 1 and Actor 3.

![Abstract IHE Profile]({{ "/assets/images/ihe.png" | relative_url }})

With IPF, these IHE concepts are mapped to [Camel components](https://camel.apache.org/component.html), partly by reusing
or extending existing Camel components or by providing [custom components](https://camel.apache.org/writing-components.html).

| Concept          | Mapping to the IPF/Camel programming model |
|------------------|----------------------------------------------------------------------|
| Actor            | Producer or Consumer of a [Camel endpoint](https://camel.apache.org/endpoint.html) |
| Transaction      | Camel component, e.g. `pix-iti8` |
| Profile          | Group of Camel components, e.g. `pix`, `xds` |

## Supported Transactions

The most IPF eHealth components are named according to the profile and the transaction they implement
(transaction IDs and profiles relate to IHE, when not stated otherwise).
A special case is the MLLP dispatcher component which allows to accept requests for multiple MLLP-based transactions through a single TCP port.

IHE Profiles are grouped by their underlying technical foundation, particularly by their *message format* and
*transport protocol* into IPF modules that can be included as dependencies in the Maven project descriptor:

* [HL7v2-based transactions] (MLLP)
* [HL7v2-based SOAP transactions]
* [HL7v3-based transactions] (SOAP)
* [ebXML transactions]
* [FHIR transactions]
* [DSMLv2-based transactions]
* [XACML-based transactions]
* [SVS-based transactions]

The table below references all supported eHealth transactions. Click on the link in the first column for details about
required dependencies, usage and parameters.

| Transaction             | Profile          | Description           | IPF Component          | Transport and Message Format  | IPF Module |
|:------------------------|:-----------------|:----------------------|:-----------------------|:------------------------------|------------|
{% for hash in site.data.ihe -%}
  {% assign tx = hash[1] -%}
| [{{ tx.transaction }}]({{ tx.link }}/)  | {{ tx.profile }} | {{ tx.description }}  | `{{ tx.component }}`   | {{ tx.transport }}, {{ tx.format }} | `{{ tx.module }}` |
{% endfor %}


## Supported Transaction Parameters

The parameters usually depend on the IHE transaction; some parameters are valid for groups of transactions,
a few are even valid for all transactions (e.g. turning ATNA auditing on or off). Details are given in the respective pages
describing each component.

## Example

An implementation of a *Patient Identity Cross-Reference Manager* actor for the *Patient Identity Feed* (PIX ITI-8) transaction
shall be created. PIX ITI-8 is a HL7v2-based transaction using MLLP as transport protocol.
Receiving actors are implemented as a [Camel consumer](https://camel.apache.org/maven/current/camel-core/apidocs/org/apache/camel/Consumer.html).

Thus, the following dependency must be registered in `pom.xml`:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.platform-camel</groupId>
        <artifactId>ipf-platform-camel-ihe-mllp</artifactId>
        <version>${ipf-version}</version>
    </dependency>
```

The basic pattern for consumers is to specify the component name in the URI parameter of a `from`-clause at the beginning of a Camel route:

```java
    // IHEConsumer.java
    from("pix-iti8://0.0.0.0:8777?parameters....")
      .process(...)
      // process the incoming HL7v2 request and create a response
```

While stepping through the Camel route, a proper response (or an Exception) must be generated that is sent back to the caller.

On the other side of the transaction, sending actors (for ITI-8 the *Patient Identity Source*) are implemented as a
[Camel producer](https://camel.apache.org/maven/current/camel-core/apidocs/org/apache/camel/Producer.html).
The basic pattern for producers is to specify the component name in the URI parameter of a to-clause at the end of a Camel route:

```java
    // IHEProducer.java
    from(...)
      .process(...)
      // create a ITI-8 request (i.e. HL7v2 message)
      .to("pix-iti8://mpiserver.com:8888?parameters....")
      // optionally process the response
```

[IHE]: https://www.ihe.net
[HL7v2-based transactions]: {{ site.baseurl }}{% link _pages/ihe/mllp/hl7v2.md %}
[HL7v2-based SOAP transactions]: {{ site.baseurl }}{% link _pages/ihe/hl7v2ws/pcd01.md %}
[HL7v3-based transactions]: {{ site.baseurl }}{% link _pages/ihe/hl7v3/hl7v3.md %}
[ebXML transactions]: {{ site.baseurl }}{% link _pages/ihe/xds/xds.md %}
[FHIR transactions]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhir.md %}
[DSMLv2-based transactions]: {{ site.baseurl }}{% link _pages/ihe/hpd/hpd.md %}
[XACML-based transactions]: {{ site.baseurl }}{% link _pages/ihe/xacml20/xacml20.md %}
[SVS-based transactions]: {{ site.baseurl }}{% link _pages/ihe/svs/svs.md %}