---
title: IPF 5.0 Migration Guide
layout: single
permalink: /docs/migration-5.0/
toc: true
toc_icon: align-left  
toc_sticky: true
---

IPF 5.0 comes with some changes that should be considered when upgrading from earlier IPF versions.

## Changes in ATNA audit dataset enrichment 

Older IPF versions contained the module `ipf-commons-ihe-xua` which provided routines to propagate 
particular elements of XUA authorization assertions into ATNA audit datasets.  In IPF 5.0, this 
module is not present anymore.  Instead, a concept of a "Web Service audit dataset enricher" 
is introduced.  

IPF provides two enricher implementations out of the box:
* `org.openehealth.ipf.commons.ihe.ws.cxf.audit.XuaWsAuditDatasetEnricher` — fulfills requirements 
   of the IHE [XUA](https://profiles.ihe.net/ITI/TF/Volume2/ITI-40.html) profile.
* `org.openehealth.ipf.commons.ihe.ws.cxf.audit.SwissEprWsAuditDatasetEnricher` — fulfills both IHE XUA 
  requirements and the requirements of the 
  [Swiss Electronic Patient Record](https://www.e-health-suisse.ch/en/technique/technical-interoperability/specifications-for-the-epr-implementation).

A Web Service audit dataset enricher can be configured in an Audit Context
by providing a Spring bean of the type `org.openehealth.ipf.commons.ihe.ws.cxf.audit.WsAuditDatasetEnricher`,
or by specifying an enricher's class name in the corresponding Spring Boot configuration property.

Example for Spring context XML configuration:
```xml
<bean id="auditContext" class="org.openehealth.ipf.commons.audit.DefaultAuditContext">
    <property name="auditEnabled" value="true"/>
    <property name="auditMessageQueue" ref="myMessageQueue"/>
    <property name="auditSourceId" value="ipfapp"/>
    <property name="wsAuditDatasetEnricher">
        <bean class="org.openehealth.ipf.commons.ihe.ws.cxf.audit.SwissEprWsAuditDatasetEnricher"/>
    </property>
</bean>
```

Example for Spring Boot YAML configuration:
```YAML
ipf:
  atna:
    audit-enabled: true
    audit-queue-class: my.project.atna.DevNullMessageQueue
    audit-source-id: ipfapp
    ws-audit-dataset-enricher-class: org.openehealth.ipf.commons.ihe.ws.cxf.audit.SwissEprWsAuditDatasetEnricher
```