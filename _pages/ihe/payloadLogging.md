---
title: File-based Logging of Message Payload
layout: single
permalink: /docs/ihe/payloadLogging/
classes: wide
---


IPF provides the possibility to store payload of incoming or outgoing messages of eHealth transactions into files.

**Warning**:
This feature is intended for debug purposes and may lead to performance problems when used in highly loaded production environment without proper configuration.
Making saved messages available for non-authorized persons may cause data privacy risks.
{: .notice--warning}

This feature is implemented in form of MLLP and CXF interceptors, depending on the protocol. 

The configuration of these interceptors is uniform. Their constructors require a single String parameter that corresponds to the name pattern 
of the log files to be created. This pattern can contain absolute or relative path and is defined using the Spring Expression Language (SpEL), 
with square brackets as expressions placeholders. 

The following specific values can be used in the expressions:

| Value Name          | Value Type | Description |
|:--------------------|:-----------|:----------------------------------------------------------------|
| `applicationId`     | String     | Identifier of the application, can be configured via `org.openehealth.ipf.commons.ihe.core.payload.PayloadLoggingContext#setApplicationId()` &mdash; e.g. at the application start.  Default value is "ipf".   |
| `processId`         | String     | Identifier of the current OS process, consisting from the process number and the host name, divided by a dash, e.g. "31415-ipfSuperMainframe" |
| `sequenceId`        | String     | Internally generated sequence number |
| `interactionId`     | String     | Identifier of the interaction to which the message belongs |
| `date('format')`    | String     | Current date and time, formatted using [SimpleDateFormat](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html) according to the given specification |

The file name pattern configured for an interceptor can be changed at any time using the corresponding setter method. 
When the file denoted by the pattern does already exist, the new message payload will be appended at the end of the existing content.

The actual logging is delegated to subclasses of `org.openehealth.ipf.commons.ihe.core.payload.PayloadLoggerBase`. This base class provide
additional fields which can be set via logging interceptor instances:

| Field name          | Type            | Default Value   | Description                                                                    |
|:--------------------|:----------------|:----------------|:-------------------------------------------------------------------------------|
| `enabled`           | boolean         | true            | whether this particular interceptor instance is enabled or disabled | 
| `errorCountLimit`   | int             | -1              | Maximal allowed count of messages this interceptor instance has sequentially failed to handle. When this value has been reached, the interceptor gets deactivated until `resetErrorCount()` is called. Negative values (the default) mean "no limit" |

Global configuration is done via system properties:

| System Property                                                            | Type            | Default Value    | Description                                         |
|:---------------------------------------------------------------------------|:----------------|:-----------------|:----------------------------------------------------|
| `org.openehealth.ipf.commons.ihe.core.payload.PayloadLoggerBase.DISABLED`  | boolean         | true             | globally enable/disable logging of payload          | 
| `org.openehealth.ipf.commons.ihe.core.payload.PayloadLoggerBase.CONSOLE`   | boolean         | false            | whether message payload will be written using regular Java logging mechanisms instead of custom files         | 


Note that even the globally deactivated logging may lead to performance penalties because the corresponding interceptors remain still deployed 
and perform some non-avoidable activities on each message.
{: .notice--warning}