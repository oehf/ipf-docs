---
title: HL7 v3 interactive response continuation in WebService-based IPF IHE components
layout: single
permalink: /docs/ihe/hl7v3InteractiveContinuation/
classes: wide
---


This feature is supported on both producer and consumer sides, and controlled by the following endpoint URI parameters:

## Parameters

| Parameter name                 | Type       | Default value | Short description |                                                             
|:-------------------------------|:-----------|:--------------|:-------------------------------------------------------------------------------------|
| `supportContinuation`          | boolean    | false         | whether interactive message continuation should be supported by the given endpoint |
| `defaultContinuationThreshold` | int        | -1            | default threshold (maximal count of data records per message) for interactive message continuation. Relevant on consumer side only, and only when the field `RCP-2-1` of the request message does not contain a parseable integer value. Values smaller than 1 lead to no continuation |
| `continuationStorage`          | String     | n/a           | Spring bean name of a storage for interactive continuation chains (relevant on consumer side only) |
| `autoCancel`                   | boolean    | false         | whether a "continuation cancel" message should be automatically sent to the server when the producer receives last data fragment. Relevant on producer side only |
| `validationOnContinuation`     | boolean    | false         | Whether the system should validate messages which are internally handled when performing HL7v3 interactive continuation |

As stated in the above table, support for interactive continuation can be enabled for an endpoint by setting its URL parameter
`supportContinuation` to `true`.

### Consumer

When interactive continuation is enabled, a consumer will apply transaction-specific rules to split the messages into fragments,
using threshold value from the field `PRPA_IN201305UV02/controlActProcess/queryByParameter/initialQuantity`, attribute `value`.

If the mentioned threshold field is not filled in the expected way, the value of the URL parameter
`defaultContinuationThreshold` will be used. When this parameter is not configured as well or its value is
less than 1, no message splitting will be performed.

Interaction steps performed by the consumer are shown on the diagram below:

{% include figure image_path="/assets/images/conti-consumer-ic.svg" alt="Consumer Interactive Continuation" caption="Consumer Interactive Continuation" width="75%" %}

Fragments are stored internally, whereby the user must provide a storage via the `continuationStorage`
URI parameter of the consumer endpoint. This bean must implement the interface
[`org.openehealth.ipf.commons.ihe.hl7v3.storage.Hl7v3ContinuationStorage`].
An Ehcache-based implementation is provided by the IPF.

Here is an example of how to configure the Spring context descriptor, supposed that "interactiveContinuationCache" is defined in Ehcache configuration file:

```xml
    <bean id="ehcacheManager" class="net.sf.ehcache.CacheManager" factory-method="create" />

    <bean id="myICStorage"
          class="org.openehealth.ipf.commons.ihe.hl7v3.storage.EhcacheHl7v3ContinuationStorage">
        <constructor-arg>
            <bean factory-bean="ehcacheManager" factory-method="getCache">
                <constructor-arg value="interactiveContinuationCache" />
            </bean>
        </constructor-arg>
    </bean>
```

The consumer endpoint URI can then contain parameters `"&supportContinuation=true&continuationStorage=#myICStorage"`.

Obsolete fragment chains can be removed from the storage either by means of a corresponding cancel message `QUQI_IN000003UV01`
sent by the client (as usual, such messages are automatically served by the IPF) or by proprietary mechanisms of the storage, if available.

### Producer

A WebService-based IPF IHE producer with enabled interactive continuation will automatically send Query Continue (QUQI_IN000003UV01)
requests after having received the first response if there are results left to be fetched. This step will be repeated for all subsequent fragments until 
the last fragment of the chain has arrived.
After that, the producer joins all collected data records together (using the same transaction-specific rules as the consumer used)
and delivers the cumulative response to the caller.

Optionally — i.e. when the endpoint URI parameter `autoCancel` is set to true — a "continuation cancel" message will be sent
in order to tell the data provider that the it can safely release its resources. The diagram below shows these interaction steps:

{% include figure image_path="/assets/images/conti-producer-ic.svg" alt="Producer Interactive Continuation" caption="Producer Interactive Continuation" width="75%" %}

All unexpected fragments will be passed through to the route without changes.
This rule applies to cancel messages which relate to non-existent interactive chains (represented by their query tags) as well.
