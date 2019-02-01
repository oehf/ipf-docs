---
title: Advanced FHIR endpoint parameters
layout: single
permalink: /docs/ihe/fhirAdvanced/
classes: wide
---

Some more FHIR endpoint parameters are not required in all scenarios. 

All parameters are usually set on the FhirTransactionConfiguration (i.e. per
FHIR component type), but you may wish to overwrite these settings in your
endpoint URI.

## Parameters

| Parameter name         | Type                 | Default value | Short description                                                                    |
|:-----------------------|:---------------------|:--------------|:-------------------------------------------------------------------------------------|
| `fhirContext`          | FhirContext          | DSTU3 or R4   | Reference to a global HAPI FHIR Context. Otherwise a default FHIR contxt is used.
| `resourceProvider`     | FhirProvider         | n/a           | Reference to a custom resource provider, configurable per endpoint.
| `clientRequestFactory` | ClientRequestFactory | n/a           | reference to a custom ClientRequestFactory
| `consumerSelector`     | Predicate            | () -> true    | reference to a Predicate that selects a FhirConsumer

### fhirContext

The most commonly used of these is `fhirContext` (e.g. `?fhirContext=#fhirContext`), 
which allows to reference a global FHIR Context instance. This is useful if you
expose several FHIR endpoints and want to use the same configured context.

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

MHD ITI-65 is a transaction request containing DocumentManifest, DocumentReference and Binary
resources. The request is defined with a `Bundle.meta.profile` value set to `http://ihe.net/fhir/tag/iti-65`,
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
        setStaticConsumerSelector(new BundleProfileSelector(Iti65Constants.ITI65_PROFILE));
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
        boolean result = bundle.getMeta().getProfile().stream()
                .map(UriType::getValueAsString)
                .anyMatch(profileUris::contains);
        return result;
    }
}
```