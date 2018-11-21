---
title: Dynamic registration of IPF features
layout: single
permalink: /docs/dynamic/
classes: wide  
---


IPF application developers are often facing the task to add some extended functionality to pre-packaged applications.
This should be possible without modifying these applications.

A closely related challenge is to build up modular integration solutions where each module contributes routes,
services etc. to the overall application without having a single definition that actually references all these contributions.

IPF provides an dynamic registration mechanism to help developers to overcome this problem.
The mechanism currently depends on the Spring dependency injection framework and allows developers to

 * assemble routes, mappings, HL7 model classes etc. into a general context
 * add routes, mappings, HL7 model classes etc. to existing IPF applications without modifying any configuration or source files of those applications

Here is a brief overview of supported extension points:

* [Custom Mappings] - add additional mappings between code systems, i.e. from one set of codes into a corresponding set of codes
* [Custom HL7 Model Classes] - add support for non-standard HL7 "dialects" which are not covered by default HL7 specification
* [Custom Groovy Extension Modules] - activate additional Groovy Extension Modules
* [Custom Route Builders] - add additional route builders, interceptors or exception handlers into an existing Camel context

The IPF extension mechanism uses the bean lifecycle interfaces of Spring framework to perform the customization, 
so it can be used in Spring based applications only.

{% include figure image_path="/assets/images/extension-mechanism.png" alt="Extension mechanism" caption="Extension mechanism " %}

On application startup the dynamic registration mechanism searches for any contribution present in the Spring application
context and extends/customizes the application with that contribution.

In order to use the capabilities of IPF dynamic registration mechanism it is necessary to add some Spring configuration
to the application.


### Configuration

To activate the IPF dynamic registration mechanism in your application, all desired *configurers* must be
registered with a Spring-based *post processor* bean like shown on the spring beans definition below.


```xml

    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:camel="http://camel.apache.org/schema/spring"
           xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://camel.apache.org/schema/spring
    >

      <!-- Some basic beans -->

      <camel:camelContext id="camelContext" />

      <bean id="mappingService" class="org.openehealth.ipf.commons.spring.map.SpringBidiMappingService"/>

      <!-- Picking up custom mappings -->
      <bean id="customMappingsConfigurer"
            class="org.openehealth.ipf.commons.map.spring.config.CustomMappingsConfigurer">
          <property name="mappingService" ref="mappingService" />
      </bean>

      <bean id="customModelClassFactory"
            class="org.openehealth.ipf.modules.hl7.parser.CustomModelClassFactory" />

      <!-- For HL7 model scripts compiled at runtime, use:
      <bean id="customModelClassFactory"
            class="org.openehealth.ipf.modules.hl7.parser.GroovyCustomModelClassFactory" />
      -->

      <!-- Picking up custom HL7v2 model class factories -->
      <bean id="customModelClassesConfigurer"
            class="org.openehealth.ipf.modules.hl7.config.CustomModelClassFactoryConfigurer">
        <property name="customModelClassFactory" ref="customModelClassFactory" />
      </bean>

      <!-- Picking up custom route builders -->
      <bean id="customRouteBuilderConfigurer"
            class="org.openehealth.ipf.platform.camel.core.config.CustomRouteBuilderConfigurer">
        <property name="camelContext" ref="camelContext" />
      </bean>

      <!-- Picking up dynamic extension modules -->
      <bean id="customExtensionConfigurer"
            class="org.openehealth.ipf.commons.core.extend.config.DynamicExtensionConfigurer">
      </bean>


      <bean id="postProcessor"
            class="org.openehealth.ipf.commons.spring.core.config.SpringConfigurationPostProcessor">
          <property name="springConfigurers" >
            <list>
              <ref bean="customMappingsConfigurer" />
              <ref bean="customModelClassesConfigurer" />
              <ref bean="customRouteBuilderConfigurer" />
              <ref bean="customExtensionConfigurer" />
            </list>
          </property>
      </bean>
    ...
    </beans>

```

### Extension order

It is possible to assign the order property to each of defined configurers. This property defines the initialization
priority, where an `order` property value of 1 has higher priority than a value of 2.

The one with higher priority (i.e. lower `order` number) is initialized *before* the ones with lower priority.
The default order value of every configurer is Integer.MAX_VALUE what basically means the lowest priority.
The only exception is the `DynamicExtensionConfigurer` with order value set to 2 which is necessary in order to activate
all extension modules by default before any other configuration takes place.



[Custom Mappings]: {{ site.baseurl }}{% link _pages/customMappings.md %}
[Custom HL7 Model Classes]: {{ site.baseurl }}{% link _pages/hl7-groovy/hl7-groovy-customModelClasses.md %}
[Custom Groovy Extension Modules]: {{ site.baseurl }}{% link _pages/groovy/customExtensions.md %}
[Custom Route Builders]: {{ site.baseurl }}{% link _pages/customRouteBuilders.md %}
