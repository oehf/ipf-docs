---
title: xcpd-iti56 component
layout: single
permalink: /docs/ihe/iti56/
toc: true
toc_icon: align-left
toc_sticky: true
---

{% assign tx = site.data.ihe["iti56"] %}

The {{ tx.component }} component provides interfaces for actors of the *{{ tx.description }}* IHE transaction ({{ tx.transaction }}),
which is described in the [{{ tx.section }}]({{ tx.section-link }}).

### Actors


The transaction defines the following actors:

{% capture imagepath -%}
/assets/images/{{tx.link}}.svg
{% endcapture %}

{% include figure image_path="/assets/images/iti56.svg" alt="ITI-56 actors" caption="ITI-56 transaction and actors " %}

Producer side corresponds to the *{{ tx.client-actor }}* actor.
Consumer side corresponds to the *{{ tx.server-actor }}* actor.

### Dependencies

In a Maven-based environment, the following dependency must be registered in `pom.xml`:

```xml
    <dependency>
        <groupId>org.openehealth.ipf.platform-camel</groupId>
        <artifactId>{{ tx.module }}</artifactId>
        <version>${ipf-version}</version>
    </dependency>
```

### Endpoint URI Format

#### Producer

The endpoint URI format of the `{{ tx.component }}` component producers is:

```
{{ tx.component }}://hostname:port/path/to/service[?parameters]
```

where *hostname* is either an IP address or a domain name, *port* is a port number, and *path/to/service*
represents additional path elements of the remote service.
URI parameters are optional and control special features as described in the corresponding section below.

#### Consumer

The endpoint URI format of `{{ tx.component }}` component consumers is:

```
{{ tx.component }}:serviceName[?parameters]
```

The resulting URL of the exposed IHE Web Service endpoint depends on both the configuration of the [deployment container]
and the serviceName parameter provided in the Camel endpoint URI.

For example, when a Tomcat container on the host `eHealth.server.org` is configured in the following way:

```
port = 8888
contextPath = /IHE
servletPath = /ws/*
```

and serviceName equals to `{{ tx.link }}`, then the {{ tx.component }} consumer will be available for external clients under the URL
`http://eHealth.server.org:8888/IHE/ws/{{ tx.link }}`

Additional URI parameters are optional and control special features as described in the corresponding section below.


### Example

This is an example on how to use the component on the consumer side:

```java
    from("{{ tx.component }}:{{ tx.link }}?audit=true")
      .process(myProcessor)
      // process the incoming request and create a response
```


### Basic Common Component Features

* [ATNA auditing]
* [Message validation]

### Basic Web Service Component Features

* [Secure transport]
* [File-Based payload logging]

### Basic HL7v3 Component Features

* [Message types]

### Advanced Web Service Component Features

* [Handling Protocol Headers]
* [Deploying custom CXF interceptors]
* [Handling automatically rejected messages]
* [Using CXF features]
* [Asynchronous Web Service exchange option]


[ATNA auditing]: {{ site.baseurl }}{% link _pages/ihe/atna.md %}
[Message validation]: {{ site.baseurl }}{% link _pages/ihe/messageValidation.md %}

[deployment container]: {{ site.baseurl }}{% link _pages/ihe/ws/wsDeployment.md %}
[Secure Transport]: {{ site.baseurl }}{% link _pages/ihe/ws/wsSecureTransport.md %}
[File-Based payload logging]: {{ site.baseurl }}{% link _pages/ihe/ws/wsPayloadLogging.md %}

[Message types]: {{ site.baseurl }}{% link _pages/ihe/hl7v3/hl7v3MessageTypes.md %}

[Handling Protocol Headers]: {{ site.baseurl }}{% link _pages/ihe/ws/wsProtocolHeaders.md %}
[Deploying custom CXF interceptors]: {{ site.baseurl }}{% link _pages/ihe/ws/wsCustomInterceptors.md %}
[Handling automatically rejected messages]: {{ site.baseurl }}{% link _pages/ihe/ws/wsHandlingRejected.md %}
[Using CXF features]: {{ site.baseurl }}{% link _pages/ihe/ws/wsCxfFeatures.md %}
[Asynchronous Web Service exchange option]: {{ site.baseurl }}{% link _pages/ihe/ws/wsCxfAsync.md %}


