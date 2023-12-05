---
title: Sorting and Pagination in ITI-58 and CH:CIQ
layout: single
permalink: /docs/ihe/hpdQueryControls/
toc: true
toc_icon: align-left
toc_sticky: true
---

Starting from the version 4.3, IPF supports server-side sorting and paging in HPD and CPI queries.
Both producer and consumer support is provided.
Elements of request and response messages which control the sorting and paging functionality, 
are called "controls".


## Sorting

Normative reference: [RFC 2891 "LDAP Control Extension for Server Side Sorting of Search Results"](https://www.ietf.org/rfc/rfc2891.txt).

### Producer Side

The only thing a client application must do to request server-side sorting of search result entries, 
is to put an instance of 
[`SortControl2`](../../apidocs/org/openehealth/ipf/commons/ihe/hpd/controls/sorting/SortControl2.html),
rendered as DSMLv2, into the list field `control`of 
[`SearchRequest`](../../apidocs/org/openehealth/ipf/commons/ihe/hpd/stub/dsmlv2/SearchRequest.html), e.g.:

```groovy
BatchRequest batchRequest = new BatchRequest(
    requestID: '1',
    batchRequests: [
        new SearchRequest(
            requestID: '2',
            control: [
                ControlUtils.toDsmlv2(new SortControl2(true, new SortKey('givenName', false, 'std_i'))),
            ],
            .....
        ), 
```

This control shall specify a set of sorting keys, each consisting from an attribute name, search order inversion 
flag, and sorting rule ID ("matching algorithm" in LDAP speech).   

### Consumer Side

To enable automatic server-side sorting of search result entries, the consumer endpoint URI parameter 
`supportSorting` shall be set to `true`:

```groovy
from('hpd-iti58:hpd-service?...&supportSorting=true&...')
```

The current implementation supports only one sorting key in request controls
([`SortControl2`](../../apidocs/org/openehealth/ipf/commons/ihe/hpd/controls/sorting/SortControl2.html)).

Moreover, the sorting rule specified by the client must be supported by the IPF application.
The following sorting rules are supported out-of-the box:

| Sorting rule name | Description |
|:------------------|:------------|
| `std`    | Case-sensitive string comparison, no whitespace normalization
| `std_i`  | Case-insensitive string comparison, no whitespace normalization
| `std_w`  | Case-sensitive string comparison with whitespace normalization
| `std_iw` | Case-insensitive string comparison with whitespace normalization

Application developers may extend this list by adding instances of `java.util.Comparator<String>` 
into the static map `COMPARATORS` of
[`SearchResponseSorter`](../../apidocs/org/openehealth/ipf/commons/ihe/hpd/controls/sorting/SearchResponseSorter.html),
with sorting rule names as keys.

It is not required that all Search Requests of a Batch Request contain the same Sort Control or contain one at all.  
Each Search Response is processed individually as shown on the UML diagram below:

{% include figure image_path="/assets/images/hpd-query-sorting-consumer.svg" 
alt="Consumer-side sorting of HPD search result entries" 
caption="Consumer-side sorting of HPD search result entries" %}

First, using the specified sorting rule, values of the specified attribute in each search result entry are sorted; 
then, entries are sorted based on the first values of the specified attribute. 
                          

## Paging

Normative reference: [RFC 2696 "LDAP Control Extension for Simple Paged Results Manipulation"](https://www.ietf.org/rfc/rfc2696.txt).

### Producer Side

On the producer side, the server endpoint URI parameter `supportPagination` shall be set to `true`:

```groovy
    .to('hpd-iti58:server.com:7888/hpd-service?...&supportPagination=true&...')
```

Furthermore, an instance of `javax.naming.ldap.PagedResultsControl` rendered as DSMLv2, shall be inserted 
into the list field `control`of
[`SearchRequest`](../../apidocs/org/openehealth/ipf/commons/ihe/hpd/stub/dsmlv2/SearchRequest.html), e.g.:

```groovy
BatchRequest batchRequest = new BatchRequest(
    requestID: '3',
    batchRequests: [
        new SearchRequest(
            requestID: '4',
            control: [
                ControlUtils.toDsmlv2(new PagedResultsControl(10, null, true)),
            ],
            .....
        ), 
```

This control shall specify the requested page size and contain no cookie.

The IPF producer automatically handles response controls, requests subsequent pages, and creates an aggregated
response to be returned into the application Camel route, as shown on the UML diagram below:

{% include figure image_path="/assets/images/hpd-query-paging-producer.svg"
alt="Producer-side support for paging of HPD search result entries"
caption="Producer-side support for paging of HPD search result entries" %}

### Consumer Side

To enable automatic paging of search result entries, the URI of the consumer endpoint shall contain two parameters —
`supportPagination` set to `true`, and `paginationStorage` set to the name of a bean implementing the interface
[`PaginationStorage`](../../apidocs/org/openehealth/ipf/commons/ihe/hpd/controls/pagination/PaginationStorage.html).
For example:

```groovy
from('hpd-iti58:hpd-service?...&supportPagination=true&paginationStorage=#myStorage&...')
```
IPF provides an implementation of
[`PaginationStorage`](../../apidocs/org/openehealth/ipf/commons/ihe/hpd/controls/pagination/PaginationStorage.html) —
the class
[`EhcachePaginationStorage`](../../apidocs/org/openehealth/ipf/commons/ihe/hpd/controls/pagination/EhcachePaginationStorage.html),
which is backed up by Ehcache or any other javax.cache API (JSR-107) implementation and optionally 
supports JAXB serialization of DSMLv2 Search Result Entries which is required for persistent caches.

Consumer-side handling of paging controls is shown on the UML diagram below:

{% include figure image_path="/assets/images/hpd-query-paging-consumer.svg"
alt="Consumer-side paging of HPD search result entries"
caption="Consumer-side paging of HPD search result entries" %}


## Combination of Controls

Sorting and paging controls can be present in the same Search Request at the same time.
In this case, the IPF consumer performs sorting first, and then paginates over the sorted collection.
Please be aware that some third-party Healthcare Provider Directories have restrictions in regard to allowed
combination of controls.

