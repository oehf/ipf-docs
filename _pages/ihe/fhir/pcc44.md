---
title: qedm-pcc44 component
layout: single
permalink: /docs/ihe/pcc44/
toc: true
toc_icon: align-left
toc_sticky: true
---

{% assign tx = site.data.ihe['pcc44'] %}

The {{ tx.component }} component provides interfaces for actors of the *{{ tx.description }}* IHE transaction ({{ tx.transaction }}),
which is described in the [{{ tx.section }}]({{ tx.section-link }})

## Actors

The transaction defines the following actors:

{% include figure image_path="/assets/images/pcc44.svg" alt="PCC-44 actors" caption="PCC-44 transaction and actors " %}

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

## Transaction Options

This transaction defines the following options; at least one of them must be chosen to be supported:
 
 * OBSERVATIONS
 * ALLERGIES
 * CONDITIONS
 * DIAGNOSTIC_REPORTS
 * MEDICATIONS
 * IMMUNIZATIONS
 * PROCEDURES
 * ENCOUNTERS
 * ALL (default)

*Note*: the PROVENANCE option is not yet supported
{: .notice--info}

The options are represented as enumeration in the class [Pcc44Options](../../apidocs/org/openehealth/ipf/platform/camel/ihe/fhir/pcc44/Pcc44Options.html).

Support for one or more options is configured using the optional `options` parameter, followed by a comma-separated list of
options to be supported. If `options` is provided, the default option (see above) is used.
The endpoint will reject resource types that are outside the scope of the configured option.

## Example

This is an example on how to use the component on the consumer side:

```java
    from("{{ tx.component }}://service?audit=true&iheOptions=OBSERVATIONS")
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
[deployment container]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirDeployment.md %}
[translators]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirPixPdqTranslators.md %}
[Advanced FHIR Parameters]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirAdvanced.md %}