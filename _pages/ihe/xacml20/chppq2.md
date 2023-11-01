---
title: ch-ppq2 component
layout: single
permalink: /docs/ihe/chppq2/
toc: true
toc_icon: align-left
toc_sticky: true
---

{% assign tx = site.data.ihe["chppq2"] %}

The {{ tx.component }} component provides interfaces for actors of the *{{ tx.description }}* IHE transaction ({{ tx.transaction }}),
which is described in the [{{ tx.section }}]({{ tx.section-link }}).

### Actors


The transaction defines the following actors:

{% capture imagepath -%}
/assets/images/{{tx.link}}.svg
{% endcapture %}

{% include figure image_path="/assets/images/chppq2.svg" alt="CH-PPQ-2 actors" caption="CH-PPQ-2 transaction and actors " %}

Producer side corresponds to the *{{ tx.client-actor }}* actor.
Consumer side corresponds to both *{{ tx.server-actor }}* actor.

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

For automatic creation of error messages in case of exceptions occurred in the Camel route, the home community ID 
must be known.  It can be specified statically using the URL parameter `homeCommunityId` 
(e.g. `&homeCommunityId=urn:oid:1.2.3.4.5`), or dynamically — either in the Camel message header named
[`HomeCommunityUtils.HOME_COMMUNITY_ID_NAME`](../../apidocs/org/openehealth/ipf/platform/camel/ihe/core/HomeCommunityUtils.html#HOME_COMMUNITY_ID_NAME), 
or in the Camel exchange property with the same name.

The resulting URL of the exposed IHE Web Service endpoint depends on both the configuration of the [deployment container]
and the serviceName parameter provided in the Camel endpoint URI.

For example, when a Tomcat container on the host `eHealth.server.org` is configured in the following way:

```
port = 8888
contextPath = /IHE
servletPath = /policy/*
```

and serviceName equals to `{{ tx.link }}`, then the {{ tx.component }} consumer will be available for external clients under the URL
`http://eHealth.server.org:8888/IHE/policy/{{ tx.link }}`

Additional URI parameters are optional and control special features as described in the corresponding section below.

### Data Types

Messages exchanged in the `ch-ppq2` transaction are defined in XACML 2.0 standard:

 * Request message -- [`XACMLPolicyQueryType`](../../apidocs/org/openehealth/ipf/commons/ihe/xacml20/stub/xacml20/saml/protocol/XACMLPolicyQueryType.html) or 
 * Response message -- [`ResponseType`](../../apidocs/org/openehealth/ipf/commons/ihe/xacml20/stub/saml20/protocol/ResponseType.html)
 
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

### Advanced Web Service Component Features

* [Handling Protocol Headers]
* [Deploying custom CXF interceptors]
* [Handling automatically rejected messages]
* [Using CXF features]



[ATNA auditing]: {{ site.baseurl }}{% link _pages/ihe/atna.md %}
[Message validation]: {{ site.baseurl }}{% link _pages/ihe/messageValidation.md %}

[deployment container]: {{ site.baseurl }}{% link _pages/ihe/ws/wsDeployment.md %}
[Secure Transport]: {{ site.baseurl }}{% link _pages/ihe/ws/wsSecureTransport.md %}
[File-Based payload logging]: {{ site.baseurl }}{% link _pages/ihe/ws/wsPayloadLogging.md %}

[Handling Protocol Headers]: {{ site.baseurl }}{% link _pages/ihe/ws/wsProtocolHeaders.md %}
[Deploying custom CXF interceptors]: {{ site.baseurl }}{% link _pages/ihe/ws/wsCustomInterceptors.md %}
[Handling automatically rejected messages]: {{ site.baseurl }}{% link _pages/ihe/ws/wsHandlingRejected.md %}
[Using CXF features]: {{ site.baseurl }}{% link _pages/ihe/ws/wsCxfFeatures.md %}



