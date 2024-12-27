---
title: Groovy Camel extensions
layout: single
permalink: /docs/groovy/camelExtensions/
toc: true
toc_icon: align-left
---

The [camel-groovy] module of Apache Camel extended to accept Groovy closures as arguments to the Camel DSL.

To include [camel-groovy], add the following dependency to the `pom.xml` file:

```xml
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-groovy</artifactId>
        <version>${camel-version}</version>
    </dependency>
```

Now it is possible to use [Groovy closures](https://www.groovy-lang.org/closures.html) as argument to Camel processors,
filters, transformers, etc.

Some examples:

```groovy

    // Processor closure
    from('direct:input1')
        .process {exchange ->
            exchange.in.body = exchange.in.body.reverse()
        }
        .to('mock:output')

    // Filter closure
    from('direct:input2')
        .filter {exchange ->
            exchange.in.body == 'blah'
        }
        .to('mock:output')

    // Transform closure
    from('direct:input3')
        .transform {exchange ->
            exchange.in.body.reverse()
        }
```

[camel-groovy]: https://camel.apache.org/groovy-dsl.html
