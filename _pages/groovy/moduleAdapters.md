---
title: DSL extensions for IPF module adapters
layout: single
permalink: /docs/groovy/moduleAdapters/
toc: true
toc_icon: align-left
---

The [`org.openehealth.ipf.commons.core.modules.api`][modules-api] package,
which is part of the `ipf-commons-core` component, defines a
number of interfaces representing typical message processors like

* [`Validator`][Validator],
* [`Aggregator`][Aggregator]
* [`Transmogrifier`][Transmogrifier] (a transformer)

The intention of these interfaces was to define message processors that are independent of integration infrastructures
like Apache Camel in order to increase their reusability in other contexts.

The integration of these message processors into IPF routes is done via generic module adapters.
They translate from Camel-specific interfaces to the interfaces and are provided by the `ipf-platform-camel-core` component.

*Note*:
The Groovy DSL extensions described on this page date back to Camel 1.x, when the Camel DSL was far less expressive
than it is today. They are thin wrappers: each one just creates a module adapter, which is an ordinary Camel
`Processor`, `DataFormat` or `AggregationStrategy`. The adapters can therefore be used from the Java DSL (or as beans
from XML/YAML routes) as well, and for most use cases Camel itself offers an equivalent, e.g. via [bean integration],
lambdas or dedicated components. Each section below closes with a *Without the Groovy DSL* subsection describing these alternatives.

This section is organized in the following way.

* Detailed description of the `transmogrify` extension. Many concepts described here also apply to other module adapter extensions.
* Overview of the remaining module adapter extensions. This overview only covers topics that haven't been discussed for the transmogrify extension.

## Transmogrifier

The easiest way to describe the DSL extensions for IPF module adapters is to start with an example.
Let's use the [`org.openehealth.ipf.commons.core.modules.api.Transmogrifier`][Transmogrifier]
interface for that purpose.
Inspired by [Calvin and Hobbes](https://en.wikipedia.org/wiki/Calvin_hobbes),
a transmogrifier converts anything into whatever you like.

Transmogrification is accompanied by a loud *zap*:

```java
    public interface Transmogrifier<S, T> {

        T zap(S object, Object... params);

    }
```

Implementations of `Transmogrifier` are used for message transformation. Transformation input is given by the object parameter
and optionally some additional params. The transformation result is the return value of the `zap` method.
To include a `Transmogrifier` instance into a Camel route we use the `transmogrify` DSL extension:

```groovy

    // this is a Groovy route
    Transmogrifier transmogrifier = new MyTransmogrifier()
    from('direct:input')
        .transmogrify(transmogrifier)
        .to('mock:output')

```

### Inclusion Pattern

There are three different ways of including a transmogrifier into a Camel route:

* Pass a transmogrifier object as argument to the `transmogrify` method. This has already been shown in the example route above.
* Pass the name of a transmogrifier bean as argument to the `transmogrify` method (see below). A bean with that name must exist in the Camel registry.
* Define a transmogrifier logic inline using a Closure (see below). This is comparable to implement an anonymous Transmogrifier class.

```groovy

    from('direct:input1')
        .transmogrify('myTransmogrifierBean')
        .to('mock:output')

    from('direct:input2')
        .transmogrify { body, headers ->
            def result = ... // create result from input body and headers
            return result    // return the transformation result
        }
        .to('mock:output')
```

This pattern also applies to all other DSL extensions for IPF module adapters: `verify`, `parse`, `render`, and `aggregationStrategy`.

### Input and Parameters

Default arguments to the `Transmogrifier.zap(S object, Object... params)` method are:

* The in-message body for the object parameter.
* The in-message headers for the params parameter.

A transmogrify closure may define a one or two parameters:

* The first parameters corresponds to the object parameter of the `zap` method. The default argument is the in-message body.
* The second parameter corresponds to the params parameter of the `zap` method. The default argument is a Map containing the in-message headers.

Input to the zap method as well as the transmogrify closure can be customized via the following modifiers:

* `input` : Camel expression or closure to be used as input
* `params`: Camel expression or closure to be used as params
* `staticParams`: Object with static params

The following snippet causes the transmogrifier's `zap` method to be called with the message's *foo*-header as first argument
and the messages *bar*-header as the second argument:

```groovy
    from('direct:input')
        .transmogrify { fooHeader, barHeader ->
            // ...
        }
        .input { it.in.headers.foo }
        .params { it.in.headers.bar }
        .to('mock:output')
```

The `params` DSL extension also supports predefined expressions. These are accessible by calling `params` without arguments.
The following predefined expressions are currently supported as part of the DSL:

* `headers()`: message headers (default for transmogrifiers)
* `header('foo')`: message foo-header
* `builder()`: a Groovy XML builder
* `headersAndBuilder()`: message headers and a Groovy XML builder (params array of length 2)

Thus, the example above could be rewritten to:

```groovy
    from('direct:input')
        .transmogrify { fooHeader, barHeader ->
            // ...
        }
        .input { it.in.headers.foo }
        .params().header('bar')
        .to('mock:output')
```

The `staticParams` extension can be used to pass constant values to the transmogrifier or transmogrifier closure.
This extension method defines a variable argument parameter. For example to pass a String array with elements 'a', 'b' and 'c':

```groovy
    from('direct:input')
        .transmogrify { body, stringArray ->
            // ...
        }
        .staticParams('a', 'b', 'c')
        .to('mock:output')
```

### Output

The return value of the `Transmogrifier.zap(S object, Object... params)` method or the return value of the transmogrify closure
is written to the `org.apache.camel.Exchange` object from which the input was taken. It depends on the exchange pattern
to which exchange message the result is written.

If the exchange is out-capable (i.e. `exchange.getPattern().isOutCapable()` returns true) then the result is written to the exchange's
out-message body, otherwise, it is written to the in-message body. Furthermore, if the exchange is out-capable, the in-message
is copied onto the out-message before the result is written (this is useful e.g. for preserving message headers along a precessing chain).

#### Transmogrifier implementations

IPF provides three Transmogrifier implementations out of the box:

* [`org.openehealth.ipf.commons.xml.XsltTransmogrifier`][XsltTransmogrifier] for xslt transforming XML documents
* [`org.openehealth.ipf.commons.xml.SchematronTransmogrifier`][SchematronTransmogrifier] for creating Schematron validation reports from XML documents
* [`org.openehealth.ipf.commons.xml.XqjTransmogrifier`][XqjTransmogrifier] for xquering XML documents or collections

#### XSLT and Schematron

The XSLT implementations are "by-products" for Schematron validation, but they can be used independently as well.
Compared to Camel's [xslt endpoint](https://camel.apache.org/xslt.html), the IPF counterpart

* can use variable stylesheets
* caches XSLT templates for better performance
* accepts explicit XSLT parameters (not just as Camel message header)

The input is automatically converted into a `StreamSource`. By default, all Camel headers are added as parameters which
are available in the stylesheet unless you define parameters by either using `params(...)` or `staticParams(...)`.

```groovy

    from('direct:input1')
        .transmogrify().xslt().staticParams('path/to/stylesheet') // static stylesheet
        .to('mock:output')

    from('direct:input1')
        .transmogrify().xslt().staticParams('path/to/stylesheet', parameterMap) // static stylesheet with parameters
        .to('mock:output')

    from('direct:input3')
        .setHeader('stylesheet', constant('path/to/stylesheet'))
        .transmogrify().xslt().params().header('stylesheet') // dynamic stylesheet
        .to('mock:output')

    // In most cases you will need the SchematronValidator, which scans the Schematron report
    // for failed assertions. Use only if you require custom processing of the report in the
    // route.

    from('direct:input3')
        .transmogrify().schematron().staticParams('path/to/rules', options ) // static rules
        .to('mock:output')

    from('direct:input4')
        .setHeader('rules', constant('path/to/rules'))
        .transmogrify().schematron().params().header('rules') // dynamic rules
        .to('mock:output')

```

By default, XSLT transformations return a `javax.xml.transform.Result` object, which is, however, not very useful for
further processing. IPF's XSLT-related transmogrifiers therefore return a `String` by default. This can be influenced by
using a Class parameter to the xslt()/schematron() extensions or with a subsequent call to `convertBodyTo(Class)`:

```groovy

    from('direct:input1')
        .transmogrify().xslt(InputStream.class).staticParams('path/to/stylesheet')
        .to('mock:output')

    from('direct:input2')
        .transmogrify().xslt().staticParams('path/to/stylesheet')
        .convertBodyTo(InputStream.class)
        .to('mock:output')
```

#### XQuery

Additionally, an [XQuery](https://www.w3.org/TR/xquery/) implementation is provided on top of the [XQJ API](https://www.jcp.org/en/jsr/detail?id=225).
In the actual state it uses the Saxon's XQJ implementation. The `XqjTransmogrifier` is implemented similar to the `XsltTransmogrifier`
and can be used to execute xqueries to the Apache Camel's exchange body.

All extensions used with the `org.openehealth.ipf.commons.xml.XsltTransmogrifier` like `params` and `staticParams`
can be used also with the XQuery Transmogrifier. The main difference with the Apache Camel endpoing is that the
`XqjTransmogrifier` uses the IPF's `org.openehealth.ipf.commons.xml.ClasspathUriResolver`. This allows to reference external document nodes or
collections directly from the classpath.

```groovy
    from('direct:input24') // using a dedicated XqjTransmogrifier bean
        .convertBodyTo(StreamSource.class)
        .transmogrify('xqj').staticParams('xquery/extract.xq', [id:'someid'])
        .to('mock:output')

    from('direct:input29') // passing classpath parameters as stream
        .transmogrify().xquery()
        .staticParams('xquery/extract-map-document.xq', [id:'someid',
            map: new StreamSource(new ClassPathResource('xquery/mapping.xml').getInputStream())])
        .to('mock:output')

    from('direct:input29') // passing parameters as map
        .transmogrify().xquery().params{
            [(XqjTransmogrifier.RESOURCE_LOCATION) : 'xquery/extract-map-document.xq',
              map: new StreamSource(new ClassPathResource('xquery/mapping.xml').getInputStream()),
              id:'someid']}
        .to('mock:output')
```

### Without the Groovy DSL

`transmogrify` creates a [`TransmogrifierAdapter`][TransmogrifierAdapter], which is a plain Camel `Processor`.
It can be used in Java routes with `process(...)`, and offers the same `input(...)`, `params(...)` and `staticParams(...)`
modifiers, taking Camel expressions instead of closures. Note that the adapter itself has no default params:
to pass the message headers like the Groovy DSL does, set them explicitly.

```java
    import static org.openehealth.ipf.platform.camel.core.util.Expressions.headersExpression;
    ...
    from("direct:input1")
        .process(new TransmogrifierAdapter(transmogrifier)
            .params(headersExpression()))  // headers as params, like the Groovy DSL default
        .to("mock:output");

    from("direct:input2")
        .process(new TransmogrifierAdapter(transmogrifier)
            .input(header("foo"))
            .params(header("bar")))
        .to("mock:output");

    from("direct:input3")
        .process(new TransmogrifierAdapter(transmogrifier)
            .staticParams("a", "b", "c"))
        .to("mock:output");
```

Without any IPF adapter, a transmogrifier is just a function producing a new message body, so Camel's `transform()`
with a lambda does the same job. The same applies to transformation logic that isn't implemented as `Transmogrifier`
at all, which can also be integrated via [bean integration].

```java
    from("direct:input1")
        .transform().body(String.class, body -> transmogrifier.zap(body))
        .to("mock:output");

    from("direct:input2")
        .transform().message(m -> transmogrifier.zap(m.getHeader("foo"), m.getHeader("bar")))
        .to("mock:output");
```

#### XSLT, Schematron and XQuery

The `xslt()`, `schematron()` and `xquery()` extensions only preset the transmogrifier and convert the input
into a `StreamSource`. In Java routes this is done explicitly:

```java
    import static org.apache.camel.builder.Builder.bodyAs;
    ...
    from("direct:input1")
        .process(new TransmogrifierAdapter(new XsltTransmogrifier<>(String.class))
            .input(bodyAs(StreamSource.class))
            .staticParams("path/to/stylesheet", parameterMap))
        .to("mock:output");

    from("direct:input2")
        .process(new TransmogrifierAdapter(new XqjTransmogrifier<>(String.class))
            .input(bodyAs(StreamSource.class))
            .staticParams("xquery/extract.xq", Map.of("id", "someid")))
        .to("mock:output");
```

Camel's own components cover these use cases as well:

* [`xslt-saxon`](https://camel.apache.org/components/latest/xslt-saxon-component.html) (or `xslt`) caches the compiled
  stylesheet by default. Message headers, exchange properties and exchange variables are passed as stylesheet parameters;
  there is no separate parameter map per call. A stylesheet determined at runtime is used via `toD(...)`, which creates
  and caches one endpoint per stylesheet location.
* [`schematron`](https://camel.apache.org/components/latest/schematron-component.html) leaves the message body
  untouched and stores the SVRL report in the `CamelSchematronValidationReport` header, and the outcome (`SUCCESS`
  or `FAILED`) in the `CamelSchematronValidationStatus` header.
* [`xquery`](https://camel.apache.org/components/latest/xquery-component.html) (from `camel-saxon`) makes
  message headers available as external variables named `in.headers.<name>` and exchange properties under their plain names.
  The query result is a DOM node by default.

```java
    from("direct:input1")
        .to("xslt-saxon:path/to/stylesheet")          // static stylesheet, headers as parameters
        .to("mock:output");

    from("direct:input2")
        .toD("xslt-saxon:${header.stylesheet}")       // dynamic stylesheet
        .to("mock:output");

    from("direct:input3")
        .to("schematron:path/to/rules")
        .setBody(header(Constants.VALIDATION_REPORT)) // the SVRL report, like SchematronTransmogrifier
        .to("mock:output");

    from("direct:input4")
        .to("xquery:xquery/extract.xq")               // declare variable $in.headers.id external;
        .convertBodyTo(String.class)
        .to("mock:output");
```

## Validator

The API in the `ipf-commons-core` module defines an
[`org.openehealth.ipf.commons.core.modules.api.Validator`][Validator]
interface for message validation.

```java
public interface Validator<S, P> {

    void validate(final S message, final P profile);

}
```

It defines a single `validate` method that validates a message against a profile. If validation fails, an
[`org.openehealth.ipf.commons.core.modules.api.ValidationException`][ValidationException] is thrown.

The validator is included into Camel routes via the `verify` DSL extension. The `verify` extension accepts either a validator object,
a validator bean name or a validator closure as argument. If a closure is used, a failed validation is either indicated by throwing an
`org.openehealth.ipf.commons.core.modules.api.ValidationException` or by just returning false. If false is returned,
IPF generates a ValidationException internally.

Some examples:

```groovy

    // validation will fail if the in-message body doesn't equal 'blah'
    from('direct:input1')
         .verify {body -> body == 'blah'}
         .to('mock:output')

    // validation will fail because a ValidationException is thrown directly
    from('direct:input2')
         .verify { throw new ValidationException('always fail') }
         .to('mock:output')

    // a second parameter is used for passing a validation profile. By default it is null but it can be customized
    // via the staticProfile DSL extension
    from('direct:input3')
         .verify {body, profile ->
             body == profile
         }
         .staticProfile('blah')
         .to('mock:output')

    // input is used to pass the in-message's foo-header as first argument to the validation closure.
    // If the foo-header doesn't equal 'abcd' validation will fail.
    from('direct:input4')
         .verify {fooHeader, profile ->
             fooHeader == profile
         }
         .input { it.in.headers.foo }
         .staticProfile('abcd')
         .to('mock:output')

    // a validation profile is obtained from the in-messages's customProfile header using the profile() DSL extension and a closure
    from('direct:input5')
         .verify(...)
         .input(...)
         .profile { exchange ->
             exchange.in.headers.customProfile
         }
         .to('mock:output')

    // can also use validator objects
         ...
         .verify(new MyCustomValidator())
         ...

    // can use validator beans
         ...
         .verify('myValidatorBean')
         ...
```

### Skipping Validation

It is possible to switch off validation by means of a Camel message header.
When there is input message header named `org.openehealth.ipf.platform.camel.core.adapter.ValidatorAdapter#NEED_VALIDATION_HEADER_NAME`
and its value equals to `Boolean.FALSE`, the validation step will be skipped. This gives the possibility to validate conditionally
based of user-defined properties and/or make it configurable e.g. via JMX.

Please be aware that the aforementioned Camel message header will remain untouched and deactivate subsequent validation steps as well.


###  XSD and Schematron validation

IPF provides a Validator implementation that validates an XML Source against an *W3C XML Schema*.
The schema location value can be either a URL or a non-URL string, in the latter case the classpath is searched for the schema resource.

```groovy
    from('direct:input1')
         .verify().xsd().staticProfile('schema location')
         .to('mock:output')
```

IPF also provides a `Validator` implementation that validates an XML Source against a set of *Schematron* rules.

```groovy
    import org.openehealth.ipf.commons.xml.SchematronProfile;
    ...
    from('direct:input1')
         .verify().schematron().staticProfile(new SchematronProfile('rules location', options))
         .to('mock:output')
```

Note that you have to provide an instance of `SchematronProfile`, not just the plain Schematron rules location.
The rules location value can be either a URL or a non-URL string, in the latter case the classpath is searched for the schema resource.

The options parameter is optional. If present, it must be of type `Map<String, Object>`. Its purpose is to configure
Schematron's validation process. Please refer to the [Schematron website](http://www.schematron.com) for more details.

| option key          | description                                                                                                       | values                | default |
|---------------------|-------------------------------------------------------------------------------------------------------------------|-----------------------|---------|
| phase               | Select the phase for validation. Schematron allows for staged validation by assigning phases to validation rules. | NMTOKEN or #ALL       | #ALL    |
| allow-foreign       | Pass non-Schematron elements and rich markup to the generated stylesheet                                          | 'true' or 'false'     | 'false' |
| diagnose            | Add the diagnostics to the assertion test in reports                                                              | 'true' or 'false'     | 'true'  |
| property            | Experimental: Add properties to the assertion test in reports                                                     | 'true' or 'false'     | 'true'  |
| generate-paths      | Generate the @location attribute with XPaths                                                                      | 'true' or 'false'     | 'true'  |
| sch.exslt.imports   | semi-colon delimited string of filenames for some EXSLT implementations                                           | string                | ''      |
| optimize            | Use only when the schema has no attributes as the context nodes                                                   | 'visit-no-attributes' | ''      |
| generate-fired-rule | Generate fired-rule elements. Significantly increases report size 	                                                | 'true' or 'false'     | 'true'  |

### Without the Groovy DSL

`verify` creates a [`ValidatorAdapter`][ValidatorAdapter], which can be used in Java routes with `process(...)`.
It still throws `ValidationException` and honors the header for skipping validation. Set the profile before calling
`input(...)` or `params(...)`, as those return the more general `ProcessorAdapter` type. For XSD and Schematron
validation, the conversion into a `StreamSource` must be configured explicitly:

```java
    import static org.apache.camel.builder.Builder.bodyAs;
    ...
    from("direct:input1")
        .process(new ValidatorAdapter(new MyCustomValidator()).staticProfile("blah"))
        .to("mock:output");

    from("direct:input2")
        .process(new ValidatorAdapter(new MyCustomValidator())
            .profile(header("customProfile"))
            .input(header("foo")))
        .to("mock:output");

    from("direct:input3")
        .process(new ValidatorAdapter(new XsdValidator())
            .staticProfile("schema location")
            .input(bodyAs(StreamSource.class)))
        .to("mock:output");

    from("direct:input4")
        .process(new ValidatorAdapter(new SchematronValidator())
            .staticProfile(new SchematronProfile("rules location", options))
            .input(bodyAs(StreamSource.class)))
        .to("mock:output");
```

Camel provides the [validate](https://camel.apache.org/components/latest/eips/validate-eip.html) EIP for checking
a predicate, as well as the [`validator`](https://camel.apache.org/components/latest/validator-component.html) (XSD)
and [`schematron`](https://camel.apache.org/components/latest/schematron-component.html) components. Note that these
throw Camel exceptions (`PredicateValidationException`, `SchemaValidationException` and the Camel
`SchematronValidationException`, respectively) instead of IPF's `ValidationException`, so `onException(...)` clauses
must be adapted accordingly. They also ignore the header for skipping validation; use e.g. a `filter` or `choice` for
conditional validation.

```java
    from("direct:input1")
        .validate(body().isEqualTo("blah"))
        .to("mock:output");

    from("direct:input2")
        .to("validator:schema location")
        .to("mock:output");

    from("direct:input3")
        .to("schematron:rules location?abort=true")  // without abort=true, only the result headers are set
        .to("mock:output");
```

## Parser

The API in the `ipf-commons-core` module defines an
[`org.openehealth.ipf.commons.core.modules.api.Parser`][Parser] interface for parsing
an external representation of information into an internal model.

Examples:

```groovy

    from('direct:input1')
        .parse(new MyParser())
        .to('mock:output')

    from('direct:input2')
        .parse('myParserBean')
        .input { it.in.headers.foo }
        .params { it.in.headers.bar }
        .to('mock:output')
```

*Note*: Prefer using Camel's [Data Format](https://camel.apache.org/data-format.html) or [bean integration] for this purpose.

### Without the Groovy DSL

A [`DataFormatAdapter`][DataFormatAdapter] wraps a parser into a Camel `DataFormat` that can be used with `unmarshal(...)`.
Alternatively, `parse` creates a [`ParserAdapter`][ParserAdapter] processor, which accepts a String, `InputStream`,
`Reader` or `Source` as input. Without IPF adapters, the parser is called from a `transform()` lambda.

```java
    from("direct:input1")
        .unmarshal(new DataFormatAdapter(parser))
        .to("mock:output");

    from("direct:input2")
        .process(new ParserAdapter(parser)
            .input(header("foo"))
            .params(header("bar")))
        .to("mock:output");

    from("direct:input3")
        .transform().body(String.class, s -> parser.parse(s))
        .to("mock:output");
```

## Renderer

The API in the `ipf-commons-core` module defines an
[`org.openehealth.ipf.commons.core.modules.api.Renderer`][Renderer] interface for
creating an external representation of an internal model.

Examples:

```groovy

    from('direct:input1')
        .render(new MyRenderer())
        .to('mock:output')

    from('direct:input2')
        .render('myRendererBean')
        .input { it.in.body[0] }
        .params { it.in.headers.bar }
        .to('mock:output')
```

*Note*: Prefer using Camel's [Data Format](https://camel.apache.org/data-format.html) or [bean integration] for this purpose.

### Without the Groovy DSL

Analogous to the parser, a renderer is used via a [`DataFormatAdapter`][DataFormatAdapter] with `marshal(...)`,
via a [`RendererAdapter`][RendererAdapter] processor, or from a `transform()` lambda:

```java
    from("direct:input1")
        .marshal(new DataFormatAdapter(renderer))
        .to("mock:output");

    from("direct:input2")
        .process(new RendererAdapter(renderer)
            .params(header("bar")))
        .to("mock:output");

    from("direct:input3")
        .transform().body(MyModel.class, model -> renderer.render(model))
        .to("mock:output");
```

If a parser and a renderer are implemented by the same class, `new DataFormatAdapter(parser, renderer)` creates a
single data format for both directions.

## Aggregator

The [`org.openehealth.ipf.commons.core.modules.api.Aggregator`][Aggregator]
interface is a `transmogrifier` that combines/aggregates a
collection of input object into a result object. The result object is the return value of the `zap` method.

```java

    public interface Transmogrifier<S, T> {
        T zap(S object, Object... params);
    }

    public interface Aggregator<S, T> extends Transmogrifier<Collection<S>, T>
```

The `aggregationStrategy` DSL extension can be used to include an `Aggregator` into Camel routes.
The extension supports an aggregator object, a bean name or a closure as argument. The created Camel `AggregationStrategy`
can then be used with e.g. `enrich` or `pollEnrich`, which always provide the original exchange to aggregate with.
It does not support the first aggregation step of `multicast`, `split` or `aggregate`, where there is no previous
exchange yet.

### Without the Groovy DSL

`aggregationStrategy` just creates an [`AggregatorAdapter`][AggregatorAdapter], which can be instantiated directly.
As `AggregationStrategy` is a functional interface, a lambda is a plain Camel alternative:

```java
    from("direct:input1")
        .enrich("direct:resource", new AggregatorAdapter(aggregator))
        .to("mock:output");

    from("direct:input2")
        .enrich("direct:resource", (oldExchange, newExchange) -> {
            oldExchange.getMessage().setBody(aggregator.zap(List.of(
                oldExchange.getMessage().getBody(),
                newExchange.getMessage().getBody())));
            return oldExchange;
        })
        .to("mock:output");
```


[bean integration]: https://camel.apache.org/bean-integration.html
[modules-api]: ../apidocs/org/openehealth/ipf/commons/core/modules/api/package-summary.html
[Aggregator]: ../apidocs/org/openehealth/ipf/commons/core/modules/api/Aggregator.html
[Parser]: ../apidocs/org/openehealth/ipf/commons/core/modules/api/Parser.html
[Renderer]: ../apidocs/org/openehealth/ipf/commons/core/modules/api/Renderer.html
[Transmogrifier]: ../apidocs/org/openehealth/ipf/commons/core/modules/api/Transmogrifier.html
[ValidationException]: ../apidocs/org/openehealth/ipf/commons/core/modules/api/ValidationException.html
[Validator]: ../apidocs/org/openehealth/ipf/commons/core/modules/api/Validator.html
[SchematronTransmogrifier]: ../apidocs/org/openehealth/ipf/commons/xml/SchematronTransmogrifier.html
[XqjTransmogrifier]: ../apidocs/org/openehealth/ipf/commons/xml/XqjTransmogrifier.html
[XsltTransmogrifier]: ../apidocs/org/openehealth/ipf/commons/xml/XsltTransmogrifier.html
[AggregatorAdapter]: ../apidocs/org/openehealth/ipf/platform/camel/core/adapter/AggregatorAdapter.html
[DataFormatAdapter]: ../apidocs/org/openehealth/ipf/platform/camel/core/adapter/DataFormatAdapter.html
[ParserAdapter]: ../apidocs/org/openehealth/ipf/platform/camel/core/adapter/ParserAdapter.html
[RendererAdapter]: ../apidocs/org/openehealth/ipf/platform/camel/core/adapter/RendererAdapter.html
[TransmogrifierAdapter]: ../apidocs/org/openehealth/ipf/platform/camel/core/adapter/TransmogrifierAdapter.html
[ValidatorAdapter]: ../apidocs/org/openehealth/ipf/platform/camel/core/adapter/ValidatorAdapter.html
