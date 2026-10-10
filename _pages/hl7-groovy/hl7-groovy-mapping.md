---
title: Mapping HL7 type values with Groovy
layout: single
permalink: /docs/hl7-groovy-mapping/ 
classes: wide
---


The [Mapping Service] is part of the IPF Core features. After all, although often used in HL7 processing, code system mapping
is not a feature that is inherently exclusive for HL7.

Since IPF 6.0, mappings are declared in file formats and looked up through the `Mappings` API, as
described in [Defining and Loading Mappings]. The mapping service and the Groovy `.map` syntax used
in the [legacy example](#example-with-the-groovy-mapping-dsl) are deprecated. The
extensions shown here work with either.

What remains specific to IPF's HL7 v2 support, however, is that the mapping extensions can be applied directly on all [HAPI] message types.


## Example

Given the following mapping file `hl7-example.mapping.xml`:

```xml
<mappings xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:noNamespaceSchemaLocation="http://openehealth.org/schema/ipf-commons-map.xsd">

    <mapping name="encounterType"
             keySystem="2.16.840.1.113883.12.4"
             valueSystem="2.16.840.1.113883.5.4">
        <entry key="E" value="EMER"/>
        <entry key="I" value="IMP"/>
        <entry key="O" value="AMB"/>
    </mapping>

    <mapping name="vip"
             keySystem="2.16.840.1.113883.12.99"
             valueSystem="2.16.840.1.113883.5.1075">
        <entry key="Y" value="VIP"/>
        <unmatched mode="provided"/>    <!-- an unknown code maps onto itself -->
    </mapping>

    <mapping name="messageType">
        <entry key="ADT^A01" value="PRPA_IN402001"/>
        <unmatched mode="fail"/>        <!-- an unknown message type is an error -->
    </mapping>

</mappings>
```

It is loaded into the `Mappings` bean, for example with a `CustomMappings` bean. In Spring Boot:

```java
@Bean
CustomMappings exampleMappings() {
    var customMappings = new CustomMappings();
    customMappings.setMappingResource(new ClassPathResource("hl7-example.mapping.xml"));
    return customMappings;
}
```

The mapping functions can be directly applied on composite or primitive field objects. A composite
field is looked up by its encoded form, e.g. `ADT^A01`:

```groovy

    // Mapping primitives
    assert msg.PV1.patientClass.value == 'I'
    assert msg.PV1.patientClass.map('encounterType') == 'IMP'
    assert msg.PV1.patientClass.mapEncounterType() == 'IMP'

    // Together with the HL7 v2 DSL, you can also write
    assert msg.PV1[2].mapEncounterType() == 'IMP'

    // To map a Composite field, you can write
    assert msg.MSH.messageType.mapMessageType() == 'PRPA_IN402001'
    assert msg.MSH[9].mapMessageType() == 'PRPA_IN402001'

```

For a message type without an entry, `mode="fail"` raises an `IllegalArgumentException`. The legacy
example below throws an `HL7Exception` with error code 207 instead. A mapping file cannot choose the
exception, so if you need that exact behavior, catch the exception in your route or check for the
code before mapping it.


## Example with the Groovy mapping DSL

Given the following mapping example:

```groovy

    mappings = {
        encounterType(['2.16.840.1.113883.12.4','2.16.840.1.113883.5.4'],
             E : 'EMER',
             I : 'IMP',
             O : 'AMB'
        )

        vip(['2.16.840.1.113883.12.99','2.16.840.1.113883.5.1075'],
             Y      : 'VIP',
             (ELSE) : { it }
        )

        messageType(
             'ADT^A01' : 'PRPA_IN402001',
             (ELSE) : { throw new HL7Exception("Invalid message type", 207) }
        )
    }

```

The code applying the mapping functions is the same as in the [XML mapping example](#example).

[HAPI]: https://hapifhir.github.io/hapi-hl7v2/
[Mapping Service]: {{ site.baseurl }}{% link _pages/mapping.md %}
[Defining and Loading Mappings]: {{ site.baseurl }}{% link _pages/mappings.md %}
