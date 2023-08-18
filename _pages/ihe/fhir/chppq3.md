---
title: ch-ppq3 component
layout: single
permalink: /docs/ihe/chppq3/
toc: true
toc_icon: align-left
toc_sticky: true
---

{% assign tx = site.data.ihe['chppq3'] %}

The {{ tx.component }} component provides interfaces for actors of the *{{ tx.description }}* IHE transaction ({{ tx.transaction }}),
which is described in the [{{ tx.section }}]({{ tx.section-link }})

## Actors

The transaction defines the following actors:

{% include figure image_path="/assets/images/chppq3.svg" alt="PPQ-3 actors" caption="PPQ-3 transaction and actors " %}

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

### Producer

The endpoint URI format of `{{ tx.component }}` component producers is:

```
{{ tx.component }}://hostname:port/path/to/service[?parameters]
```

where *hostname* is either an IP address or a domain name, *port* is a number, and path/to/service represents additional path 
elements of the remote service.

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
* [Resource validation][Message validation]

## FHIR Component Features

* [Message types and exception handling][]
* [Security][]
* [Advanced FHIR Parameters][]

## Connection-related FHIR Component Features

* [Connection Parameters][]


[ATNA auditing]: {{ site.baseurl }}{% link _pages/ihe/atna.md %}
[Message validation]: {{ site.baseurl }}{% link _pages/ihe/messageValidation.md %}
[Message types and exception handling]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirMessageTypes.md %}
[Security]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirSecurity.md %}
[Connection Parameters]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirConnection.md %}
[Deployment container]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirDeployment.md %}
[Translators]: {{ site.baseurl }}{% link _pages/ihe/fhir/ppqmTranslators.md %}
[Advanced FHIR Parameters]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirAdvanced.md %}