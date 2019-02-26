---
title: Development
layout: single
permalink: /development/
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/development2.jpg
toc: true
toc_icon: align-left
---

## Sources

IPF uses [git](https://git-scm.com/) for source code management. The IPF git repository is located at
[https://github.com/oehf/ipf](https://github.com/oehf/ipf).

Additionally, there are the following support projects:

* `ipf-docs`, which is the source of this documentation. It is located at [https://github.com/oehf/ipf-docs](https://github.com/oehf/ipf-docs).
* `ipf-gazelle`, which provides conformance profiles for HL7v2 based IHE transactions.
It may be released independently and is located at [https://github.com/oehf/ipf-gazelle](https://github.com/oehf/ipf-gazelle).
* `ipf-oht-atna`, which provides infrastructure for IHE audit trails and node authentication via TLS.
It may be released independently and is located at [https://github.com/oehf/ipf-oht-atna](https://github.com/oehf/ipf-oht-atna).

**Watch out!** `ipf-oht-atna` libraries have been deprecated, and IPF has removed all dependencies targeting at `ipf-oht-atna`.
{: .notice--warning}


## Building

IPF requires Java 8 for both compile time and runtime.
IPF does not yet support Java 9+, this is aimed for with IPF 4.0.

IPF builds using Maven 3.6.0. IPF is available at [Maven Central], so no custom repositories need to
be added to the `settings.xml` configuration file.

Before building, adjust the `MAVEN_OPTS` environment variable to assign Maven more heap space.

```
    set MAVEN_OPTS=-Xmx1024m
    mvn clean install
```

## Documentation

In order to generate the site documentation, Java stubs from Groovy and Lombok
sources must be generated for proper Javadoc creation during the `site` phase.

```
    set MAVEN_OPTS=-Xmx1024m
    mvn -Pgenerate-stubs generate-sources 
    mvn site (-DskipTests)
```

Documentation is maintained in Markdown in the `ipf-docs` repository. Pushing changes to github will
automatically render the documentation at `https://oehf.github.io/ipf-docs`.

Javadocs artifacts are uploaded to Maven Central, which makes them also available online under
https://www.javadocs.io.


## IDE

IPF depends on Maven, [Groovy](https://www.groovy-lang.org/), [Kotlin](https://kotlinlang.org/) and [Lombok](https://projectlombok.org/).
Depending on the choice of your IDE, you may need to install corresponding plugins.

## Continuous Integration

IPF is continuously built on [Travis](https://travis-ci.org/oehf). Snapshot artifacts are uploaded to the 
[Sonatype](https://oss.sonatype.org/content/repositories/snapshots/org/openehealth/ipf/) snapshot repository.

## Issue Tracking

Issue tracking is done in github. For current issues check [https://github.com/oehf/ipf/issues](https://github.com/oehf/ipf/issues).
Questions? Please direct your issues to the [IPF Development Google Group](https://groups.google.com/forum/#!forum/ipf-dev). 


[Maven Central]: https://search.maven.org