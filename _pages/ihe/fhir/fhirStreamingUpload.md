---
title: FHIR Binary streaming uploads
layout: single
permalink: /docs/ihe/fhirStreamingUpload/
toc: true
toc_icon: align-left
toc_sticky: true
---

Since IPF 6.0, large document uploads with [ITI-65] and [ITI-105] no longer need several times the
document size on the heap.

## How it works

A servlet filter, `Base64SplittingFilter`, decodes the base64 document content of
`Binary.data` and `DocumentReference.content.attachment.data` into memory or a temporary file
before the body reaches the FHIR parser, and leaves a short marker in its place. The FHIR parser
then only sees a few kilobytes of metadata.

## Accessing the document content

Everything downstream of the filter sees the marker instead of the document. Consumers exchange
the marker for the content through `StagedContentRegistry`:

```java
var content = StagedContentRegistry.forParameters(parameters)
        .flatMap(registry -> registry.resolve(binary.getData()));
if (content.isPresent()) {
    DataHandler document = content.get().dataHandler(binary.getContentType());
    ...
} else {
    // not split: binary.getData() is the document content itself
}
```

Both branches are needed: a request below the size threshold is never split, so an empty
`Optional` is the normal case and means that the value at hand already is the document. IPF's own
translators do not resolve markers, so a route or translator that processes the document content
has to do this itself. `StagedContent` also provides `getSize()` and `getSha1Hex()`, measured while
the content was streamed, so a client-supplied hash can be verified without reading the document
again.

The staged content lives only as long as the request: temporary files are deleted as soon as the
response has been produced. Consume the content within the request, and do not hand it to
asynchronous processing.

## Configuration

The feature is off by default. In Spring Boot it is enabled and configured through the properties
of the [IPF Spring Boot FHIR starter]:

| Property (`ipf.fhir.streaming-upload.`) | Default             | Description                                            |
|-----------------------------------------|---------------------|--------------------------------------------------------|
| `enabled`                               | `false`             | split large uploads before they reach the FHIR parser  |
| `engage-above-bytes`                    | 256 KiB             | only request bodies larger than this are split         |
| `memory-threshold-bytes`                | 512 KiB             | extracted content up to this size is kept in memory    |
| `max-document-size-bytes`               | 128 MiB             | maximum size of a single extracted document            |
| `temp-directory`                        | JVM temp directory  | directory for temporary document files                 |
| `inline-document-resources`             | `DocumentReference` | resource types whose create carries a document inline  |
| `filter-order`                          | `-90`               | position of the filter, directly after Spring Security |

## Without Spring Boot

Outside Spring Boot, create the `Base64SplittingFilter` yourself and register it as a servlet filter
in front of the FHIR servlet. This is what the Spring Boot starter does as well. The filter extends
Spring's `OncePerRequestFilter`, and `ipf-commons-ihe-fhir-core` declares `spring-web` only as an
optional dependency, so add it to your application:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
</dependency>
```

The filter takes the tuning parameters as `StreamingUploadOptions`, the `FhirContext` and the path
of the FHIR servlet below the context path. `StreamingUploadOptions.defaults()` yields the defaults
listed above; construct the record for other values. The filter has no default constructor, so it
cannot be declared in `web.xml` directly. Register it programmatically instead, e.g. in a
`ServletContainerInitializer` or `WebApplicationInitializer`:

```java
var options = new StreamingUploadOptions(
        256 * 1024,                         // engageAboveBytes
        512 * 1024,                         // memoryThresholdBytes
        128L * 1024 * 1024,                 // maxDocumentSizeBytes
        Path.of("/var/tmp/ipf-uploads"),    // tempDirectory
        StreamingUploadOptions.MHD_INLINE_DOCUMENT_RESOURCES);

var filter = new Base64SplittingFilter(options, fhirContext, "/fhir");
servletContext.addFilter("base64SplittingFilter", filter)
        .addMappingForUrlPatterns(null, false, "/*");
```

In a Spring application context without Spring Boot, you can declare the filter as a bean instead
and reference it from `web.xml` through a `DelegatingFilterProxy` with the same name.

A second constructor additionally takes a `UnaryOperator<String>`, which is applied to the
diagnostics of a rejected request before the filter sends them, and a `StreamedDocumentListener`,
which is notified about every document streamed out of a request body.

Mind the filter order: the filter must run _after_ authentication, so that unauthenticated requests
are never spooled to disk, and _before_ any request logging, which then sees the slimmed-down body.

## Disk space

Documents larger than `memory-threshold-bytes` are written to `temp-directory` while the request is
processed. Make sure the partition holding this directory is large enough: in the worst case it
must hold `max-document-size-bytes` for every document of every concurrent upload. Depending on
document sizes and load, the partition may need to be increased in size, or `temp-directory`
pointed to a larger one.
{: .notice--warning}

[ITI-65]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti65.md %}
[ITI-105]: {{ site.baseurl }}{% link _pages/ihe/fhir/iti105.md %}
[IPF Spring Boot FHIR starter]: {{ site.baseurl }}{% link _pages/boot/boot-fhir.md %}
