---
title: HL7v2 Netty Codec
layout: single
permalink: /docs/ihe/nettycodec/
classes: wide
---

Some parameters defined in [camel-netty][] have fixed values in MLLP-based IPF IHE components. 
This means that these parameters are actually not configurable by the user anymore; 
values provided via endpoint URIs will be silently ignored. 

These parameters are:

## Netty Parameters

| Parameter name        | Type       | Constant value | 
|:----------------------|:-----------|:---------------|
| `sync`                | boolean    | true           |
| `lazySessionCreation` | boolean    | true           |
| `transferExchange`    | boolean    | false          | 
| `encoding`            | String     | corresponds to the charset name configured for the HL7 codec factory, as described below |

All other URI parameters defined in [camel-netty][] remain fully functional and configurable by the user.
Of particular interest are the following parameters:

| Parameter name                | Type       | Default        | Description                                                                                     |
|:------------------------------|:-----------|:---------------|:------------------------------------------------------------------------------------------------|
| `disconnect`                  | boolean    | false          | (producer,consumer): disconnect(close) from Mina session right after use. |
| `timeout` or `requestTimeout` | long (ms)  | 30000          | (producer): timeout that specifies how long to wait for a response from a remote server |
| `connectTimeout`              | long (ms)  | 10000          | (producer): time to wait for a socket connection to be available. Value is in milliseconds |

## HL7 Codec Parameters

[camel-netty][] defines a parameter named `encoders` and `decoders`, which are expected to contain the name of the beans that translate the network stream into a suitable application protocol and vice versa. 
[camel-hl7][] comes with an implementation of an MLLP encoder and decoder factory. MLLP-based IPF IHE components set `#hl7decoder` and `#hl7encoder` as a default values 
for these parameter. The corresponding beans must always be defined (choose your favorite charset/encoding):

```xml
<bean id="hl7decoder" class="org.apache.camel.component.hl7.HL7MLLPNettyDecoderFactory">
    <property name="charset" value="iso-8859-1"/>
    <!-- Produce a byte array so that charset can be detected dynamically -->
    <property name="produceString" value="false"/>
</bean>

<bean id="hl7encoder" class="org.apache.camel.component.hl7.HL7MLLPNettyEncoderFactory">
    <property name="charset" value="iso-8859-1"/>
</bean>
```

The character set name for the HL7 codec factory will be automatically:

* propagated to the Camel component (see parameter `encoding` in the table above)
* stored in the `Exchange.CHARSET_NAME` property of each Camel exchange
* used in all data transformation activities

Note that this default charset can be overridden by the message itself, see below.

## HL7 Charset Detection

In HL7v2 messages, the field MSH-18 can be used to indicate the character set with which the message is encoded. As of IPF 4.4, both the decoding and encoding process uses this message-specific charset with higher priority than the configured default charset, i.e. if MSH-18 is empty or the value in MSH-18 is unknown, the configured default charset is used.

Valid character sets are:

| HL7 charset (Table 0211) in MSH-18 | Java charset equivalent |
|:-----------------------------------|:------------------------|
| 8859/1                             | ISO-8859-1              | 
| 8859/2                             | ISO-8859-2              |
| 8859/3                             | ISO-8859-3              |
| 8859/4                             | ISO-8859-4              |
| 8859/5                             | ISO-8859-5              |
| 8859/1                             | ISO-8859-6              |
| 8859/1                             | ISO-8859-7              |
| 8859/1                             | ISO-8859-8              |
| 8859/1                             | ISO-8859-9              |
| ASCII                              | US-ASCII                |
| BIG-5                              | Big5"                   |
| CNS 11643-1992                     | ISO-2022-CN             |
| ISO IR14                           | ISO-2022-JP             |
| ISO IR159                          | EUC-JP                  |
| ISO IR87                           | EUC-JP                  |
| KS X 1001                          | EUC-KR                  |
| UNICODE                            | UTF-8                   |
| UNICODE UTF-16                     | UTF-16                  |
| UNICODE UTF-32                     | UTF-32                  |
| UNICODE UTF-8                      | UTF-8                   |


## HL7 Connection Multiplexing

Normally, if messages are sent concurrently over the same MLLP endpoint, a pool of producer instances (and hence: connections) is used under the hood. This is also the default behavior. The size and eviction strategies of this pool can be configured using `producerPool*` endpoint parameters (see the [camel-netty] page). 

A new feature with MLLP producer endpoints based on [camel-netty] is multiplexing.
Using this feature, the *one and the same* connection can be used to concurrently transfer requests *and* correctly associate the respective responses. This can be useful when e.g. the receiving side only allows one connection per source but can handle several requests on this connection at the same time.

There are, however, some prerequisites for this to work correctly:

* The message IDs in MSH-10 of outgoing HL7 messages *must* be unique
* The acknowleged message IDs in MSA-2 of response HL7 messages *must always correlate* with MSH-10 of the request
* The receiving side must be able to receive concurrent requests on the same connection
* A Spring bean of type `org.openehealth.ipf.platform.camel.ihe.mllp.core.Hl7CorrelationManager` must be instantiated. The Spring Boot auto configuration in the `ipf-hl7-spring-boot-starter` module defines such a bean with the id `hl7Correlation`.
* The producer endpoint URL must be enhanced with the following two parameters: `producerPoolEnabled=false&correlationManager=#hl7Correlation`

The alternative would be to set `producerPoolMaxTotal=1`; however, this would effectively serialize requests to be sent. 

## DNS address caching

The `cachedAddress` parameter (for caching InetAddress instances after a DNS lookup) is not available in camel-netty. This parameter was useful for container setups where hosts may change their virtual IPs. Instead, Netty uses the global JDK resolution mechanism that can be configured by setting the security property `networkaddress.cache.ttl` in the file `JAVA_HOME/conf/security/java.security` to a few seconds only or 0 to disable caching completely. The default is usually 30 seconds. 

You can override this value by providing this in a separate properties file and start your JVM with the system property `-Djava.security.properties=<URL of file>`. The following example would reduce the time a DNS lookup is cached to 5 seconds:

```
networkaddress.cache.ttl=5
```

[camel-netty]: https://camel.apache.org/netty.html
[camel-hl7]: https://camel.apache.org/hl7.html