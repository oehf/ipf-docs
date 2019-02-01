---
title: fhir component
layout: single
permalink: /docs/ihe/fhirCustom/
toc: true
toc_icon: align-left
toc_sticky: true
---

{% assign tx = site.data.ihe['fhir'] %}

The {{ tx.component }} component allows for custom FHIR-based interfaces to benefit from all features that are available to
FHIR-based IHE transactions, e.g. ATNA auditing.
This is particularly valuable if you already use IHE transaction components but you need additional interfaces for
which there is no defined IHE profile.

## Actors

There is no mapping to a particular actor.

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

## Explicitly configured `{{ tx.component }}` component instances

As the *fhirTransactionConfig* is a property of the component and not of the endpoint, you cannot use two `{{ tx.component }}` endpoints
with different *fhirTransactionConfig* parameters. Instead, you need to create separate component beans and refer to them by their
scheme ID.

In the following example, we define two component instances as `fhir1` and `fhir2`, configured with their distinct *fhirTransactionConfig* :

```xml

    <bean id="fhir1" class="org.openehealth.ipf.platform.camel.ihe.fhir.core.custom.CustomFhirComponent">
        <property name="fhirTransactionConfiguration" ref="fhirTransactionConfig1"/>
    </bean>

    <bean id="fhir2" class="org.openehealth.ipf.platform.camel.ihe.fhir.core.custom.CustomFhirComponent">
        <property name="fhirTransactionConfiguration" ref="fhirTransactionConfig2"/>
    </bean>

```

In the route definition, they can be used like this:

```java

    from("fhir1://service1?audit=true")
      .process(myProcessor)
      // process the incoming request and create a response

    from("fhir2://service2?audit=true")
      .process(myProcessor)
      // process the incoming request and create a response
```

## Basic Common Component Features

* [ATNA auditing][]
* [Resource validation][Message validation]

## Basic FHIR Component Features

* [Message types and exception handling][]
* [Security][]

## Connection-related FHIR Component Features

* [Connection Parameters][]


[ATNA auditing]: {{ site.baseurl }}{% link _pages/ihe/atna.md %}
[Message validation]: {{ site.baseurl }}{% link _pages/ihe/messageValidation.md %}
[Message types and exception handling]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirMessageTypes.md %}
[Security]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirSecurity.md %}
[Connection Parameters]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirConnection.md %}
[deployment container]: {{ site.baseurl }}{% link _pages/ihe/fhir/fhirDeployment.md %}
