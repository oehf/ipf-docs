---
title: Dynamic Custom Mappings
layout: single
permalink: /docs/customMappings/
toc: true
toc_icon: align-left  
---

## Deprecated as of IPF 6.0

`CustomMappings` beans are still the way to add mappings to a Spring application, but they now feed
a `SpringMappings` bean rather than the deprecated `SpringBidiMappingService`, and their resources
may be in any mapping format whose loader is on the classpath — not just the Groovy `.map` scripts
shown below. See
[Defining and Loading Mappings]({{ site.baseurl }}{% link _pages/mappings.md %}), which also
contains migration instructions for non-Spring, Spring and Spring Boot usage.
{: .notice--warning}

Custom mapping scripts can be added to a global mapping service instance.
Define a `org.openehealth.ipf.commons.spring.map.config.CustomMappings` bean in a custom spring application context file which references
one or more mapping scripts that shall be picked up.

```
<bean id="customMapping1"
      class="org.openehealth.ipf.commons.spring.map.config.CustomMappings">
    <property name="mappingResources">
        <list>
            <value>classpath:config1.map</value>
            <value>classpath:config2.map</value>
        </list>
    </property>
</bean>

<bean id="customMapping2"
      class="org.openehealth.ipf.commons.spring.map.config.CustomMappings">
    <property name="mappingResource" value="classpath:config3.map" />
</bean>


```

Since IPF 6.0, these mapping definitions are picked up by the `SpringMappings` bean itself. Up to IPF 5.3, they are
picked up by the `CustomMappingsConfigurer` (registered with a `SpringConfigurationPostProcessor`) and automatically added
to the shared MappingService.