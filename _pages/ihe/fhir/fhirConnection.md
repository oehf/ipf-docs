---
title: Connection-related FHIR endpoint parameters
layout: single
permalink: /docs/ihe/fhirConnection/
classes: wide
---

A couple of FHIR endpoints parameters are available to set client-side connection properties. 

You can also provide a reference to an instance of `org.apache.http.impl.client.HttpClientBuilder` to influence every detail on how
the HTTP client is configured. In this case, the other parameters have no effect.

## Parameters

| Parameter name             | Type        | Default value | Short description                                                              |
|:---------------------------|:------------|:--------------|:-------------------------------------------------------------------------------|
| `connectionTimeout`        | Integer     | 10000         | initial connection timeout in milliseconds                                     |
| `connectionRequestTimeout` | Integer     | 10000         | initial connection timeout in milliseconds                                     |
| `timeout`                  | Integer     | 10000         | socket timeout for read/write operations in milliseconds                       |   
| `poolMax`                  | Integer     | 20            | initial connection timeout in milliseconds                                     |
| `disableServerValidation`  | boolean     | false         | whether the FHIR client shall not obtain a CapabilityStatement from the server |
| `httpClient`               | String      | n/a           | HTTP Client implementation to be used (`apache5`, `apache`, `methanol`)        |
