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

## Building

IPF 5.x will require Java 17 for both compile time and runtime.
IPF 4.x builds with Java 11 for both compile time and runtime.
IPF 3.x requires Java 8 for both compile time and runtime.

IPF builds using Maven 3.x. IPF and its dependencies are available at [Maven Central], 
except for the MDHT dependencies that are provided at [https://projects.suisse-open-exchange.healthcare/artifactory/releases](https://projects.suisse-open-exchange.healthcare/artifactory/releases).

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
    mvn site (-DskipTests) -rf :ipf
```

Documentation is maintained in Markdown in the `ipf-docs` repository. Pushing changes to github will
automatically render the documentation at `https://oehf.github.io/ipf-docs`. The generated Javadoc
must be copied from `target/site/apidocs` into the `_pages/apidocs` directory.

Javadocs artifacts are uploaded to Maven Central, which makes them also available online under
https://www.javadoc.io. The javadocs are located under `https://www.javadoc.io/<maven-group-id>/<maven-artifact-id>/<version>`, e.g.
[https://javadoc.io/doc/org.openehealth.ipf.commons/ipf-commons-audit/4.8.0/index.html](https://javadoc.io/doc/org.openehealth.ipf.commons/ipf-commons-audit/4.0.0).


### How to build and test the documentation locally

* Install Ruby and Jekyll as described [here](https://jekyllrb.com/docs/installation/)
* Clone the [ipf-docs](https://github.com/oehf/ipf-docs) repository
* Install `bundler`: `gem install bundler`  
* In the project root directory, run `bundle install` to download and install all dependencies
* Run `bundle exec jekyll serve`

You can now browse the documentation at [http://127.0.0.1:4000/ipf-docs/](http://127.0.0.1:4000/ipf-docs/). If you update one of the
Markdown files of the project, the server regenerates its content on the fly.


### How to upload the documentation to github

* Simply commit the changes. Github Pages will regenerate the docs automatically

## IDE

IPF depends on Maven, [Groovy](https://www.groovy-lang.org/), [Kotlin](https://kotlinlang.org/) and [Lombok](https://projectlombok.org/).
Depending on the choice of your IDE, you may need to install corresponding plugins.

## Continuous Integration

IPF is continuously built using github actions. Snapshot artifacts are uploaded to the 
[Sonatype](https://oss.sonatype.org/content/repositories/snapshots/org/openehealth/ipf/) snapshot repository.

## Issue Tracking

Issue tracking is done in github. For current issues check [https://github.com/oehf/ipf/issues](https://github.com/oehf/ipf/issues).
Questions? Please direct your issues to the [IPF Development Google Group](https://groups.google.com/forum/#!forum/ipf-dev). 


[Maven Central]: https://search.maven.org