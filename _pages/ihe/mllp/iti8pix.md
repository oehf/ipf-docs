---
title: pix-iti8 component
layout: single
permalink: /docs/ihe/iti8pix/
toc: true
toc_icon: align-left
toc_sticky: true
---

{% assign tx = site.data.ihe["iti8pix"] %}

The {{ tx.component }} component provides interfaces for actors of the *{{ tx.description }}* IHE transaction ({{ tx.transaction }}),
which is described in the [{{ tx.section }}]({{ tx.section-link }}).

## Actors

The transaction defines the following actors:

{% capture imagepath -%}
/assets/images/{{tx.link}}.svg
{% endcapture %}

{% include figure image_path="/assets/images/iti8pix.svg" alt="ITI-8 actors" caption="ITI-8 transaction and actors " %}

Producer side corresponds to the *{{ tx.client-actor }}* actor.
Consumer side corresponds to both *{{ tx.server-actor }}* actor.

## Dependencies

In a Maven-based environment, the following dependency must be registered in `pom.xml`:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.platform-camel</groupId>
        <artifactId>{{ tx.module }}</artifactId>
        <version>${ipf-version}</version>
    </dependency>
```

## Endpoint URI Format

The endpoint URI format of the `{{ tx.component }}` component is identical for producers and consumers:

```
{{ tx.component }}://hostname:port[?parameters]
```

where *hostname* is either an IP address or a domain name, and *port* is a number. For the consumer side, the host name
`0.0.0.0` allows the access from any remote host.
These two obligatory URI parts represent the address of the {{ tx.transport }} endpoint which is to be served by the given consumer or
accessed by the given producer. URI parameters controlling the transaction features are described below.


## HL7v2 Codec

All HL7v2-based transactions are realized using the [camel-mina2](https://camel.apache.org/mina2.html) and [camel-hl7](https://camel.apache.org/hl7.html)
components and requires that an [HL7v2 Codec][] is available in the Camel registry.

## Example

This is an example on how to use the component on the consumer side:

```java
    from("{{ tx.component }}://0.0.0.0:8777?audit=true&secure=true")
      .process(myProcessor)
      // process the incoming request and create a response
```

## Basic Common Component Features

* [ATNA auditing][]
* [Message validation][]

## Basic MLLP Component Features

* [Message types and exception handling][]
* [Secure transport][]
* [File-Based payload logging][]

## Advanced MLLP Component Features

* [Interceptor chain configuration][]
* [Segment fragmentation][]
* [Unsolicited request message fragmentation][]


## Remarks for this component

**Watch out!** {{ tx.transaction }} is the only HL7v2-based transaction that uses HL7 v2.3.1, while most of the others use HL7 v2.5
{: .notice--info}


[ATNA auditing]: {{ site.baseurl }}{% link _pages/ihe/atna.md %}
[Message validation]: {{ site.baseurl }}{% link _pages/ihe/messageValidation.md %}
[HL7v2 Codec]: {{ site.baseurl }}{% link _pages/ihe/mllp/codec.md %}
[Message types and exception handling]: {{ site.baseurl }}{% link _pages/ihe/mllp/hl7v2MessageTypes.md %}
[Secure transport]: {{ site.baseurl }}{% link _pages/ihe/mllp/hl7v2SecureTransport.md %}
[File-Based payload logging]: {{ site.baseurl }}{% link _pages/ihe/mllp/hl7v2PayloadLogging.md %}
[Interceptor chain configuration]: {{ site.baseurl }}{% link _pages/ihe/mllp/hl7v2InterceptorChain.md %}
[Segment fragmentation]: {{ site.baseurl }}{% link _pages/ihe/mllp/segmentFragmentation.md %}
[Unsolicited request message fragmentation]: {{ site.baseurl }}{% link _pages/ihe/mllp/unsolicitedFragmentation.md %}

