---
title: mhd-iti68 component
layout: single
permalink: /docs/ihe/iti68/
toc: true
toc_icon: align-left
toc_sticky: true
---

{% assign tx = site.data.ihe['iti68'] %}

The {{ tx.component }} component provides interfaces for actors of the *{{ tx.description }}* IHE transaction ({{ tx.transaction }}),
which is described in the [{{ tx.section }}]({{ tx.section-link }}).

## Actors

The transaction defines the following actors:

{% include figure image_path="/assets/images/iti68.svg" alt="ITI-68 actors" caption="ITI-68 transaction and actors " %}

Producer side corresponds to the *{{ tx.client-actor }}* actor.
Consumer side corresponds to the *{{ tx.server-actor }}* actor.

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

### Producer

There is no producer created by this endpoint. The component can only be used on the server-side.

### Consumer

The endpoint URI format of `{{ tx.component }}` component consumers is:

```
{{ tx.component }}://serviceName[?parameters]
```

The resulting URL of the exposed FHIR REST Service endpoint depends on the configuration of the [deployment container].
Consider a Tomcat container on  host `eHealth.server.org` is configured in the following way:

```
port = 8888
contextPath = /IHE
servletPath = /fhir/*
```

Then the {{ tx.component }} consumer will be available for external clients under the URL 
`http://eHealth.server.org:8888/IHE/fhir`.

URI parameters controlling the transaction features are described below.

## Example

This is an example on how to use the component on the consumer side:

```java
    from("{{ tx.component }}://service?audit=true&secure=true")
      .process(myProcessor)
      // process the incoming request and create a response
```

## Basic Common Component Features

* [ATNA auditing][]

The ITI-68 endpoint forwards a preliminary [Iti68AuditDataset](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/iti68/Iti68AuditDataset.html) 
instance in a Camel message header named `AuditDataset`. The Camel consumer route has the possibility adding 
the unique document ID, and, if applicable in scenarios involving XDS, an optional patient ID, repository ID 
and home community ID in order to populate this AuditDataset with more information. This is because the 
document retrieval URL is completely unspecified with regard to this information.

## Remarks for this component
 
Although this component is part of a FHIR-specific module, it just responds to a plain servlet
request, which can be about any URL that is mapped on the Camel servlet (see [deployment container]).
The successful response does consist of a binary data stream, representing the document content.
{: .notice--info}


[ATNA auditing]: {{ site.baseurl }}{% link _pages/ihe/atna.md %}
[Security]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirSecurity.md %}
[deployment container]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirDeployment.md %}
