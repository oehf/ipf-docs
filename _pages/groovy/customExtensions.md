---
title: Custom DSL extensions definition
layout: single
permalink: /docs/groovy/customExtensions/
toc: true
toc_icon: align-left
---

Custom DSL extensions implement the
[`org.openehealth.ipf.commons.core.extend.config.DynamicExtension`](../apidocs/org/openehealth/ipf/commons/core/extend/config/DynamicExtension.html)
 marker interface in order to be picked up and activated by a
[`SpringDynamicExtensionRegistrar`](../apidocs/org/openehealth/ipf/commons/spring/core/extend/SpringDynamicExtensionRegistrar.html)
bean (up to IPF 5.3: a `DynamicExtensionConfigurer` registered with a `SpringConfigurationPostProcessor`; see
[Dynamic registration of IPF features]).

The following Groovy script `CustomModelExtension.groovy` defines a simple extension to the Camel DSL:

```groovy

    package org.openehealth.ipf.tutorial.config.extend

    import org.apache.camel.builder.RouteBuilder
    import org.openehealth.ipf.commons.core.extend.config.DynamicExtension

    class CustomModelExtensionModule implements DynamicExtension {

        static RouteBuilder direct(RouteBuilder delegate, String endpointName) {
            delegate.from('direct:' + endpointName)
        }

    }

```

In addition a bean needs to be defined in the custom spring context file, running in the same Spring application context as the base application:

```xml

    ...
    <lang:groovy id="customExtension"
         script-source="classpath:org/openehealth/ipf/tutorial/config/extend/CustomModelExtension.groovy" />
    ...

```

Since IPF 6.0, the `SpringDynamicExtensionRegistrar` registers the extensions once all singletons of the Spring
application context have been instantiated, i.e. before the Camel context is started and builds its routes. The extensions
are therefore available to all Camel route builders, including static ones that are directly configured in the
`CamelContext` bean. They are not available during bean initialization, though, e.g. in a `@PostConstruct` method.

Up to IPF 5.3, the Groovy meta class extension is loaded *after* the complete Spring application context has been
initialized.
Thus, the extensions are not available during that initialization process, so static Camel route builders - i.e.
those that are directly configured in the `CamelContext` bean - are not able to depend on them. Instead, these Camel route builders
must be initialized as [Custom Route Builders] as well.


[Custom Route Builders]: {{ site.baseurl }}{% link _pages/customRouteBuilders.md %}
[Dynamic registration of IPF features]: {{ site.baseurl }}{% link _pages/dynamic.md %}
