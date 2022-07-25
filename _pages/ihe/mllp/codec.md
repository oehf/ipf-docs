---
title: HL7v2 Mina Codec
layout: single
permalink: /docs/ihe/codec/
classes: wide
---

Some parameters defined in [camel-mina][] have fixed values in MLLP-based IPF IHE components. 
This means that these parameters are actually not configurable by the user anymore; 
values provided via endpoint URIs will be silently ignored. 

These parameters are:

## MINA Parameters

| Parameter name        | Type       | Constant value | 
|:----------------------|:-----------|:---------------|
| `sync`                | boolean    | true           |
| `lazySessionCreation` | boolean    | true           |
| `transferExchange`    | boolean    | false          | 
| `encoding`            | String     | corresponds to the charset name configured for the HL7 codec factory, as described below |

All other URI parameters defined in [camel-mina][] remain fully functional and configurable by the user.
Of particular interest are the following parameters:

| Parameter name        | Type       | Default        | Description                                                                                 |
|:----------------------|:-----------|:---------------|:--------------------------------------------------------------------------------------------|
| `disconnect`          | boolean    | false          | (producer,consumer): disconnect(close) from Mina session right after use. 
| `minaLogger`          | boolean    | false          | (producer,consumer): enable Apache MINA slf4j logging at INFO level to log all I/O.
| `timeout`             | long (ms)  | 30000          | (producer,consumer): timeout that specifies how long to wait for a response from a remote server
| `writeTimeout`        | long (ms)  | 10000          | (producer,consumer): maximum amount of time it should take to send data to the MINA session

## HL7 Codec Parameters

[camel-mina][] defines a parameter named `codec`, which is expected to contain the name of a bean that references a codec factory 
that translates the network stream into a suitable application protocol and vice versa. 
[camel-hl7][] comes with an implementation of an MLLP codec factory. MLLP-based IPF IHE components set `#hl7codec` as a default value 
for this parameter. The corresponding bean must always be defined:

```xml
    <bean id="hl7codec" class="org.apache.camel.component.hl7.HL7MLLPCodec">
        <property name="charset" value="iso-8859-1"/>
    </bean>
```

In case you need to set a custom `HapiContext` on the codec, you need to use the IPF implementation of the HL7 codec factory:

```xml
    <bean id="hl7codec" class="org.apache.camel.component.hl7.CustomHL7MLLPCodec">
        <property name="charset" value="iso-8859-1"/>
        <property name="hapiContext" ref="hapiContext"/>
    </bean>
```

The character set name for the HL7 codec factory will be automatically:

* propagated to the Camel component (see parameter `encoding` in the table above)
* stored in the `Exchange.CHARSET_NAME` property of each Camel exchange
* used in all data transformation activities


[camel-mina]: https://camel.apache.org/mina.html
[camel-hl7]: https://camel.apache.org/hl7.html