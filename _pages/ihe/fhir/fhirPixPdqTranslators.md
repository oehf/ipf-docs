---
title: Translation between FHIR and HL7 v2 message models
layout: single
permalink: /docs/ihe/fhirPixPdqTranslators/
toc: true
toc_icon: align-left
toc_sticky: true
---

IPF provides utilities for translation between FHIR and HL7v2, thus giving the possibility to implement FHIR-based [IHE] transactions
on top of their HL7 v2 counterparts and to avoid redundancy in that way.

Currently supported transaction pairs are

* PIX(m) Query ([ITI-9]/[ITI-83])
* PDQ(m)       ([ITI-21]/[ITI-78])
* PIX(m) Feed  ([ITI-8]/[ITI-104])

## Dependencies

In a Maven-based environment, the following dependencies should be registered in `pom.xml`:

```xml
<dependency>
    <groupId>org.openehealth.ipf.commons</groupId>
    <artifactId>ipf-commons-spring</artifactId>
    <version>${ipf-version}</version>
</dependency>
<dependency>
    <groupId>org.openehealth.ipf.platform-camel</groupId>
    <artifactId>ipf-platform-camel-ihe-fhir-r4-pixpdq</artifactId>
    <version>${ipf-version}</version>
</dependency>
```

This depends transitively on the required module:

```xml
<dependency>
    <groupId>org.openehealth.ipf.commons</groupId>
    <artifactId>ipf-commons-ihe-fhir-r4-pixpdq</artifactId>
    <version>${ipf-version}</version>
</dependency>
```


## Configuring the URI Mapper

For translation of FHIR messages, an instance of [`UriMapper`](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/translation/UriMapper.html) is required
in order to map FHIR URIs into OIDs and vice versa. IPF provides an implementation ([`NamingSystemUriMapper`](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/support/translation/NamingSystemUriMapper.html))
that uses an instance of [`NamingSystemService`](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/support/NamingSystemService.html) under the hood. 

The default implementation is [`DefaultNamingSystemServiceImpl`](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/support/DefaultNamingSystemServiceImpl.html), 
which expects a Bundle of FHIR [NamingSystem resources].
In addition, for code system mapping, a [Mapping Service] bean must be available.
Here is a snippet of the required Spring XML configuration:

```xml

    <bean id="fhirContext" class="ca.uhn.fhir.context.FhirContext" factory-method="forR4"/>

    <bean id="mappingService" class="org.openehealth.ipf.commons.spring.map.SpringBidiMappingService">
        <property name="mappingResources">
            <list>
                 <value>classpath:META-INF/map/fhir-hl7v2-translation.map</value>
            </list>
        </property>
    </bean>

    <!-- Use NamingSystemService for the URI Mapper -->

    <bean id="namingSystemService" class="org.openehealth.ipf.commons.ihe.fhir.support.DefaultNamingSystemServiceImpl">
        <constructor-arg ref="fhirContext"/>
        <property name="namingSystemsFromXml" value="classpath:identifiers.xml"/>
    </bean>

    <bean id="uriMapper" class="org.openehealth.ipf.commons.ihe.fhir.support.translation.NamingSystemUriMapper">
        <constructor-arg ref="namingSystemService"/>
        <constructor-arg value="identifiers"/>
    </bean>
    
```

Note that Spring Boot applications can depend on `ipf-fhir-spring-boot-starter`, which already auto-configures these beans for you. 
{: .notice--info}

An example for a bundle of [NamingSystem resources] (referenced to be contained in the `identifiers.xml` file in the example above)
looks like this:

```xml

<Bundle xmlns="http://hl7.org/fhir" >
    <id value="identifiers"/>
    <type value="collection"/>

    <entry>
        <resource>
            <NamingSystem>
                <id value="fhir1"/>
                <name value="FHIR1 Patient Identifier Namespace"/>
                <status value="active"/>
                <kind value="identifier"/>
                <date value="2015-07-31"/>
                <type>
                    <coding>
                        <system value="http://hl7.org/fhir/identifier-type"/>
                        <code value="PI"/>
                    </coding>
                </type>
                <uniqueId>
                    <type value="oid"/>
                    <value value="1.2.3.4"/>
                </uniqueId>
                <uniqueId>
                    <type value="other"/>
                    <value value="fhir1"/>
                </uniqueId>
                <uniqueId>
                    <type value="uri"/>
                    <value value="http://org.openehealth/ipf/commons/ihe/fhir/1"/>
                    <preferred value="true"/>
                </uniqueId>
            </NamingSystem>
        </resource>
    </entry>

    <entry>
        <resource>
            <NamingSystem>
                <id value="fhir2"/>
                <name value="FHIR2 Patient Identifier Namespace"/>
                <status value="active"/>
                <kind value="identifier"/>
                <date value="2015-07-31"/>
                <type>
                    <coding>
                        <system value="http://hl7.org/fhir/identifier-type"/>
                        <code value="PI"/>
                    </coding>
                </type>
                <uniqueId>
                    <type value="oid"/>
                    <value value="1.2.3.4.5.6"/>
                </uniqueId>
                <uniqueId>
                    <type value="other"/>
                    <value value="fhir2"/>
                </uniqueId>
                <uniqueId>
                    <type value="uri"/>
                    <value value="http://org.openehealth/ipf/commons/ihe/fhir/2"/>
                    <preferred value="true"/>
                </uniqueId>
            </NamingSystem>
        </resource>
    </entry>

</Bundle>

```

Of course you are free to include your own implementations of `UriMapper` and/or `MappingService`.

Since IPF 6.0 the mapping service is deprecated. The bean above becomes a `SpringMappings` bean, and
the mapping IPF provides ships as `classpath:META-INF/map/fhir-hl7v2-translation.mapping.xml`.
`DefaultUriMapper` takes `Mappings` directly. See [Defining and Loading Mappings].

## Translators

The packages `org.openehealth.ipf.commons.ihe.fhir.iti78`, `...iti83` and `...iti104` contain the set of translators
that is able to translate between corresponding IHE transactions. Each of them takes the `UriMapper` as constructor argument.

From a *Patient identity Cross Reference Manager* 's perspective, there are **inbound** translators:

| FHIR transaction | FHIR-to-HL7v2 request                                                                                                                                | HL7v2-Transaction  | HL7v2-to-FHIR response                                                                                                                                                 |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PDQm [ITI-78]    | [`iti78.PdqmRequestToPdqQueryTranslator`](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/iti78/PdqmRequestToPdqQueryTranslator.html) | PDQ       [ITI-21] | [`iti78.PdqResponseToPdqmResponseTranslator`](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/iti78/PdqResponseToPdqmResponseTranslator.html)           |
| PIXm [ITI-83]    | [`iti83.PixmRequestToPixQueryTranslator`](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/iti83/PixmRequestToPixQueryTranslator.html) | PIX Query [ITI-9]  | [`iti83.PixQueryResponseToPixmResponseTranslator`](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/iti83/PixQueryResponseToPixmResponseTranslator.html) |
| PIXm [ITI-104]   | [`iti104.PixmFeedRequestToPixFeedTranslator`](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/iti104/PixmFeedRequestToPixFeedTranslator.html) | PIX Feed [ITI-8]   | [`iti104.PixFeedResponseToPixmFeedResponseTranslator`](../../apidocs/org/openehealth/ipf/commons/ihe/fhir/iti104/PixFeedResponseToPixmFeedResponseTranslator.html) |

Each translator has a set of configurable properties. Their descriptions can be taken from javadoc of the
corresponding classes. Below there's an example of a Spring application context defining translator beans: 

```xml

<!-- Example for PIXm Query -->

<bean name="pixmRequestTranslator"
      class="org.openehealth.ipf.commons.ihe.fhir.iti83.PixmRequestToPixQueryTranslator">
    <constructor-arg ref="uriMapper" />
</bean>

<bean name="pixmResponseTranslator"
      class="org.openehealth.ipf.commons.ihe.fhir.iti83.PixQueryResponseToPixmResponseTranslator">
    <constructor-arg ref="uriMapper" />
</bean>

<!-- Example for PDQm -->

<bean name="pdqmRequestTranslator"
      class="org.openehealth.ipf.commons.ihe.fhir.iti78.PdqmRequestToPdqQueryTranslator">
    <constructor-arg ref="uriMapper" />
</bean>

<bean name="pdqmResponseTranslator"
      class="org.openehealth.ipf.commons.ihe.fhir.iti78.PdqResponseToPdqmResponseTranslator">
    <constructor-arg ref="uriMapper" />
</bean>

<!-- Example for PIXm Feed -->

<bean name="pixmFeedRequestTranslator"
      class="org.openehealth.ipf.commons.ihe.fhir.iti104.PixmFeedRequestToPixFeedTranslator">
    <constructor-arg ref="uriMapper" />
    <property name="addReviseEventType" value="A08" />
</bean>

<bean name="pixmFeedResponseTranslator"
      class="org.openehealth.ipf.commons.ihe.fhir.iti104.PixFeedResponseToPixmFeedResponseTranslator"/>

```

## Translating the PIXm Feed

The translation of the Patient Identity Feed FHIR ([ITI-104]) into the PIX Feed ([ITI-8], HL7 v2.3.1) follows
these rules:

| ITI-104 message           | ITI-8 message                              | PID-3                                                                                            |
|---------------------------|--------------------------------------------|--------------------------------------------------------------------------------------------------|
| Add/Revise Patient        | `ADT^A08^ADT_A01` (see `addReviseEventType`) | The identifier of the condition in the first repetition, all other identifiers of the Patient in subsequent repetitions |
| Resolve Duplicate Patient | `ADT^A40^ADT_A39`                          | The surviving patient from the `replaced-by` link; the subsumed patient goes into MRG-1           |
| Remove Patient            | none                                       | Rejected with `501 Not Implemented`, as PIX Feed has no equivalent                                |

* **Event type.** The Patient Identity Source does not know whether its conditional update creates the patient,
  so Add/Revise Patient is sent as A08 by default. `addReviseEventType` configures A01, A04 or A05 instead, and
  subclasses can override `eventType(Patient, Map)` to decide per request.
* **Demographics.** For Add/Revise Patient, the PID segment is populated from the Patient as the inverse of
  `PdqResponseToPdqmResponseTranslator`: name, mother's maiden name, birth date, gender, address, home and work
  telecom, language, marital status, religion, birth place, multiple birth, citizenship and death indicators.
  Religion, birth place and citizenship are taken from the custom elements of `PdqmPatient`, or from the corresponding
  extensions of a plain Patient.
* **Merge.** An A40 is not meant to update the patient, so it only carries minimal demographics: the surviving
  patient in PID-3 and the name in PID-5, which HL7 v2.3.1 requires. The other identifiers of the Patient belong
  to the subsumed patient and are not sent. Both patients must be in the same patient identifier domain.
* **Identifiers.** An identifier whose system the `UriMapper` cannot map to an assigning authority is skipped,
  except for the identifier of the condition, which rejects the request with `400 Bad Request`.
* **Coded values** are translated with the `fhir2hl7v2-*` mappings of `META-INF/map/fhir-hl7v2-translation.mapping.xml`.
  They are separate from the `hl7v2fhir-*` mappings, whose inverses are chosen for backwards compatibility rather
  than for being the most suitable HL7v2 code.

The response translator turns the acknowledgement into a `MethodOutcome`. The patient counts as created only if
the event type was A01, A04 or A05. Errors on PID-3 result in `400 Bad Request`, errors on MRG (unknown subsumed patient)
in `404 Not Found`, other errors and rejections in `422 Unprocessable Entity`.

A Patient Identifier Cross-reference Manager that proxies ITI-104 to ITI-8 does not support the Remove Patient Option,
and should use the `Iti104UpdateResourceProvider`, so that conditional delete is not advertised in its CapabilityStatement
(see [ITI-104]).

## Using the translators

A translator instance can be used two ways:

* directly from a Java or Groovy application (not discussed here)
* from a Camel route using ´.process()`

The module `ipf-platform-camel-ihe-fhir-r4-pixpdq`, being the basis for the PIXm/PDQm FHIR transactions' implementation,
provides processors that can be used to embed HL7 translation functionality into a Camel route.

There are two processor implementations, each taking a `translator` instance as parameter for the desired translation:

* `FhirCamelTranslators.translateFhir(translator)`
* `FhirCamelTranslators.translateToFhir(translator, Message.class)`


## Example

Here is a sample Camel route that bridges PIXm requests (ITI-83) to an HL7 v2-based Patient Identifier
Cross-Reference Manager (ITI-9), and does the same in reverse direction for responses.


```java

import ca.uhn.hl7v2.model.Message;
import org.apache.camel.builder.RouteBuilder;
import org.openehealth.ipf.commons.ihe.fhir.translation.FhirTranslator;
import org.openehealth.ipf.commons.ihe.fhir.translation.ToFhirTranslator;

import static org.openehealth.ipf.platform.camel.ihe.fhir.core.FhirCamelTranslators.translateFhir;
import static org.openehealth.ipf.platform.camel.ihe.fhir.core.FhirCamelTranslators.translateToFhir;

public class Iti83TestRouteBuilder extends RouteBuilder {

    private final FhirTranslator<Message> requestTranslator;
    private final ToFhirTranslator<Message> responseTranslator;

    public Iti83TestRouteBuilder(FhirTranslator<Message> requestTranslator, 
                                 ToFhirTranslator<Message> responseTranslator) {
        super();
        this.requestTranslator = requestTranslator;
        this.responseTranslator = responseTranslator;
    }

    @Override
    public void configure() throws Exception {
        from("pixm-iti83:translation?audit=true")
                // Translate into ITI-9
                .process(translateFhir(requestTranslator))
                        // Send to the PIX Manager
                .to("pix-iti9://${pixManagerUri}")
                        // Translate back into FHIR
                .process(translateToFhir(responseTranslator, Message.class));
    }
}

```

[ITI-8]: {{ site.baseurl }}{% link _pages/ihe/mllp/iti8pix.md %}
[ITI-9]: {{ site.baseurl }}{% link _pages/ihe/mllp/iti9.md %}
[ITI-21]: {{ site.baseurl }}{% link _pages/ihe/mllp/iti21.md %}
[ITI-78]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti78.md %}
[ITI-83]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti83.md %}
[ITI-104]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti104.md %}

[Mapping Service]: {{ site.baseurl }}{% link _pages/mapping.md %}
[Defining and Loading Mappings]: {{ site.baseurl }}{% link _pages/mappings.md %}
[NamingSystem resources]: https://www.hl7.org/fhir/namingsystem.html

[IHE]: https://www.ihe.net