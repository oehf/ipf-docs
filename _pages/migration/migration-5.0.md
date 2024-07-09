---
title: IPF 5.0 Migration Guide
layout: single
permalink: /docs/migration-5.0/
toc: true
toc_icon: align-left  
toc_sticky: true
---

IPF 5.0 comes with some changes that should be considered when upgrading from earlier IPF versions

## Renamed Classes and Modules 

* Interface `org.openehealth.ipf.commons.ihe.ws.cxf.audit.XuaProcessor` was renamed to
  `org.openehealth.ipf.commons.ihe.ws.cxf.audit.WsAuditDatasetEnricher`, because this
  mechanism can be used for all types of Web Service audit dataset enrichment and not 
  only for extracting data from XUA tokens.

* Module `ipf-commons-ihe-xua` (containing an implementation of the aforementioned interface) 
  was renamed to `ipf-commons-ihe-swissepr`, because its functionalities relate not only
  to extracting data from XUA tokens and are specific for the Swiss EPR.

