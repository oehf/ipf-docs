---
title: Custom Route Builders / Interceptors / Exception Handlers
layout: single
permalink: /docs/customRouteBuilders/
toc: true
toc_icon: align-left
---

With the custom route builders it is possible to:

* extend the base application functionality by adding additional route builders to the existing camel context
* extend the existing route builder functionality by injecting additional [interceptors](https://camel.apache.org/intercept.html) to this route builder
* extend the existing route builder exception handling by injecting additional [exception handlers](https://camel.apache.org/exception-clause.html) to this route builder

The abstract class [`org.openehealth.ipf.platform.camel.core.config.CustomRouteBuilder`](apidocs/org/openehealth/ipf/platform/camel/core/config/CustomRouteBuilder.html)  must be extended by all your
custom route builders (also the ones in the base application). If the `intercepted` property is set, it is assumed
that the custom route builder is an interceptor or an exception handler and the
[`CustomRouteBuilderConfigurer`](apidocs/org/openehealth/ipf/platform/camel/core/config/CustomRouteBuilderConfigurer.html) will try
to inject it to the referenced intercepted route builder.

Otherwise if the `intercepted` property is *not* set, the `CustomRouteBuilderConfigurer` will inject this custom route builder
to the existing camel context.

Route builders with an `intercepted` property are configured first. Since IPF 6.0, the `CustomRouteBuilderConfigurer`
collects all `CustomRouteBuilder` beans of the application context (including those of ancestor contexts) by itself.
Within each of the two groups, the order is determined by Spring's `@Order` annotation or the `Ordered` interface on the
route builder beans. Collecting happens when Spring starts its `Lifecycle` beans, i.e. after all singletons have been initialized (so that Groovy DSL
extensions contributed by a `SpringDynamicExtensionRegistrar` are available), but before the Camel context is started.

Up to IPF 5.3, the `CustomRouteBuilderConfigurer` must be listed in the `springConfigurers` property of a
`SpringConfigurationPostProcessor` (see the example below). This is deprecated since IPF 6.0 and no longer necessary;
see the [IPF 6.0 migration guide].


## Example

Here is a fragment of a base custom route builder class:

```groovy

    class BaseRoute extends CustomRouteBuilder {
        void configure() {
            ...
            from('seda:input')...
            ...
        }
    }

```

And the corresponding Spring application context shown below. Note that the custom route builder must not be explicitly associated to the Camel context.
The `camelContext` property of the configurer may be omitted if the application context contains exactly one Camel context.

```xml

  <camel:camelContext id="camelContext" />

  <!-- Picks up the custom route builders, e.g. the BaseRoute, and adds them to the Camel context -->
  <bean id="customRouteBuilderConfigurer"
        class="org.openehealth.ipf.platform.camel.core.config.CustomRouteBuilderConfigurer">
    <property name="camelContext" ref="camelContext" />
  </bean>

  <!-- Usually this is defined in a different Spring context file -->
  <bean id="baseRoute" class="org.openehealth.ipf.tutorial.config.route.BaseRoute" />

```

Up to IPF 5.3, the configurer must additionally be registered with a post processor:

```xml

  <bean id="postProcessor"
        class="org.openehealth.ipf.commons.spring.core.config.SpringConfigurationPostProcessor">
      <property name="springConfigurers" >
        <list>
          <ref bean="customRouteBuilderConfigurer" />
          <!-- potentially more configurers -->
        </list>
      </property>
  </bean>

```

A custom [interceptor](https://camel.apache.org/intercept.html) is defined in separate `CustomRouteBuilder` and extends
the functionality of the `BaseRoute` by intercepting the inputs from the `'seda:input'` endpoint:

```groovy

    class CustomInterceptingRoute extends CustomRouteBuilder{
        void configure() {

            onException(MyCustomTransmogrifierException)
               .handled(true)
               // do some more stuff to handle this exception

            interceptFrom('seda:input').transmogrify('customTransmogrifier')
        }
    }

```

The Spring beans definition is shown below, this time as Groovy script that is compiled at load time.
Note the `intercepted` property is set to the custom route builder to be intercept. If started in same application context,
this interceptor will intercept all incoming exchanges to the `"seda:input"` endpoint in the base route and translate
it with the logic implemented in `customTransmogrifier` bean (not shown in this sample). Also, the custom exception handler
is added to the `intercepted` route.

```xml

    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:lang="http://www.springframework.org/schema/lang"
           xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/lang
    http://www.springframework.org/schema/lang/spring-lang.xsd">

      <lang:groovy id="interceptorRoute"
          script-source="classpath:config/CustomInterceptingRoute.groovy" >
          <lang:property name="intercepted" ref="baseRoute" />
      </lang:groovy>

      <lang:groovy id="customTransmogrifier"
          script-source="classpath:config/CustomTransmogrifier.groovy" />

    </beans>

```


[IPF 6.0 migration guide]: {{ site.baseurl }}{% link _pages/migration/migration-6.0.md %}#removal-of-the-configurer-spi
