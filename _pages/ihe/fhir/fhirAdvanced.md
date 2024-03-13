---
title: Advanced FHIR endpoint parameters
layout: single
permalink: /docs/ihe/fhirAdvanced/
classes: wide
---

Some more FHIR endpoint parameters are not required in all scenarios. 

All parameters are usually set on the `FhirTransactionConfiguration` (i.e. per
FHIR component type), but you may wish to overwrite these settings in your
endpoint URI.

## Parameters

| Parameter name                   | Type                         | Default value | Short description                                                                 |
|:---------------------------------|:-----------------------------|:--------------|:----------------------------------------------------------------------------------|
| `fhirContext`                    | FhirContext                  | STU3 or R4    | Reference to a global HAPI FHIR Context. Otherwise a default FHIR contxt is used. |
| `resourceProvider`               | FhirProvider                 | n/a           | Reference to a custom resource provider, configurable per endpoint.               |
| `clientRequestFactory`           | ClientRequestFactory         | n/a           | reference to a custom ClientRequestFactory                                        |
| `hapiClientInterceptorFactories` | HapiClientInterceptorFactory | n/a           | reference to custom client interceptor factories                                  |
| `hapiServerInterceptorFactories` | HapiServerInterceptorFactory | n/a           | reference to custom server interceptor factories                                  |
| `consumerSelector`               | Predicate                    | () -> true    | reference to a Predicate that selects a FhirConsumer                              |
| `sort`                           | Boolean                      | false         | whether the endpoint shall try to sort a search result if requested by the client |

### fhirContext

The most commonly used of these is `fhirContext` (e.g. `?fhirContext=#fhirContext`), 
which allows to reference a global FHIR Context instance like the one provided by the 
[IPF Spring Boot FHIR starter]. This is useful if you expose several FHIR endpoints that 
shall all use same configured context.

### resourceProvider

This is to determine what the HAPI Resource Provider shall be used. Usually, every component
statically defines its own resource provider, but for custom searches etc. you may override
the default resource provider with this option.

### clientRequestFactory

This is to determine what kind of ClientRequestFactory the Producer endpoint shall
instantiate for constructing FHIR requests. 

### consumerSelector

The `consumerSelector` parameter can be used to determine whether the given endpoint
is selected for a FHIR payload or not. This is primarily useful for FHIR batch or
transaction requests. These requests are directed at the root URL of the FHIR servlet,
but if you need to support more than one type of batch requests via different endpoints
you can choose what type shall be handled by this endpoint.

Example:

MHD ITI-65 is a transaction request containing DocumentManifest (MHD 3.2), List, DocumentReference and Binary resources. The request is defined with a dedicated `Bundle.meta.profile` value (depending on the MHD version),
so the transaction definition of ITI-65 statically configures
`org.openehealth.ipf.commons.ihe.fhir.support.BundleProfileSelector` so that ITI-65 consumer endpoints
receive these requests:

```java
	public Iti65TransactionConfiguration() {
        super("mhd-iti65",
                "Provide Document Bundle",
                false,
                new Iti65ClientAuditStrategy(),
                new Iti65ServerAuditStrategy(),
                FhirVersionEnum.R4,
                BatchTransactionResourceProvider.getInstance(),      // Consumer side. accept registrations
                BatchTransactionClientRequestFactory.getInstance(),  // Formulate requests
                new Iti65Validator());
    setStaticConsumerSelector(new BundleProfileSelector(
            ITI65_LEGACY_METADATA_PROFILE,
            ITI65_COMPREHENSIVE_METADATA_PROFILE,
            ITI65_MINIMAL_METADATA_PROFILE,
            ITI65_COMPREHENSIVE_BUNDLE_PROFILE,
            ITI65_MINIMAL_BUNDLE_PROFILE,
            ITI65_UNCONTAINED_COMPREHENSIVE_BUNDLE_PROFILE));
```

If you need to alter this behavior (using a different or no profile for your ITI-65 endpoint), you
can do so by using `consumerSelector=#mySelector`, where `mySelector` is the name of a bean of type
`Predicate<Object>`. 

For demonstration purposes, here is the implementation of BundleProfileSelector:

```java
public class BundleProfileSelector implements Predicate<Object> {

    private final Set<String> profileUris;

    /**
     * @param profileUris Profile URIs expected in the Bundle's meta element
     */
    public BundleProfileSelector(String... profileUris) {
        this.profileUris = new HashSet<>(Arrays.asList(profileUris));
    }

    /**
     * @param object bundle
     * @return true if one of the {@link #profileUris} are present in the Bundle's meta.profile
     */
    @Override
    public boolean test(Object object) {
        Bundle bundle = (Bundle) object;
        return bundle.getMeta().getProfile().stream()
                .map(UriType::getValueAsString)
                .anyMatch(profileUris::contains);
    }
}
```

### sort

This parameter can be set to true if the FHIR consumer endpoint shall attempt server-side orting if the client requests to do so. Sorting must be explicitly supported by the FHIR endpoint's `FhirSearchParameters` implementation - it must extend `FhirSearchAndSortParameters` and implement the `comparatorFor(String)` method that returns a `Comparator` object for the provided sort parameter.

For demonstration purposes, here is the implementation of `Iti78SearchParameters` that supports selected sorting parameters:

```java

public class Iti78SearchParameters extends FhirSearchAndSortParameters<PdqPatient> {

....

    @Override
    public Optional<Comparator<PdqPatient>> comparatorFor(String paramName) {
        switch (paramName) {
            case PdqPatient.SP_BIRTHDATE: return Optional.of(CP_DATE);
            case PdqPatient.SP_FAMILY: return Optional.of(CP_FAMILY);
            case PdqPatient.SP_GIVEN: return Optional.of(CP_GIVEN);
        }
        return Optional.empty();
    }

    private static final Comparator<PdqPatient> CP_DATE = nullsLast(comparing(PdqPatient::getBirthDate));
    private static final Comparator<PdqPatient> CP_FAMILY = nullsLast(comparing(patient -> patient.getNameFirstRep().getFamily()));
    private static final Comparator<PdqPatient> CP_GIVEN = nullsLast(comparing(patient -> patient.getNameFirstRep().getGivenAsSingleString()));
    
}
```

Any "unknown" sorting parameters are ignored. The returned `Comparator`s must be capable of handling null values - in this case they are ordered after all non-null values.

[IPF Spring Boot FHIR starter]: {{ site.baseurl }}{% link _pages/boot/boot-fhir.md %}