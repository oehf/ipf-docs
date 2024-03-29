---
title: "IPF Open eHealth Integration Platform"
layout: splash
permalink: /
header:
  overlay_color: "#000"
  overlay_filter: "0.2"
  overlay_image: /assets/images/title-unsplash.jpg
  cta_label: "Download"
  cta_url: "https://search.maven.org/search?q=ipf"
excerpt: "eHealth standards on steroids."
intro: 
  - excerpt: 'The Open eHealth Integration Platform (IPF) provides interfaces for health-care related integration solutions. An prominent example of an healthcare-related use case of IPF is the implementation of interfaces for transactions specified in Integrating the Healthcare Enterprise (IHE) profiles.'
feature_row_1:
  - title: "IHE Profile Support"
    excerpt: "A set of components for creating actor interfaces as specified in IHE and Continua integration profiles. IPF supports creation of actor interfaces for a large number of IHE profiles as well as for Continua profiles HRN and WAN."
    url: "docs/ihe/"
    btn_label: "Read More"
    btn_class: "btn--info"
  - title: "FHIR Support"
    excerpt: "FHIR® – Fast Healthcare Interoperability Resources (hl7.org/fhir) – is a next generation standards framework created by HL7 leveraging the latest web standards and applying a tight focus on implementability."
    url: "/docs/ihe/fhir/"
    btn_label: "Read More"
    btn_class: "btn--info"
  - title: "HL7v2 Messaging"
    excerpt: "Basis for HL7 message processing is the HL7v2 DSL. These provides the basis for implementing HL7 Message processing Camel routes as well as for processing HL7v2-based IHE transactions."
    url: "/docs/hl7-groovy/"
    btn_label: "Read More"
    btn_class: "btn--info"
feature_row_2:
  - title: "DICOM Audit"
    excerpt: "Support for constructing, serializing and sending DICOM audit messages."
    url: "/docs/audit/"
    btn_label: "Read More"
    btn_class: "btn--info"
  - title: "Spring Boot support"
    excerpt: "IPF comes with a number of Spring Boot Starters that support running eHealth applications in the Spring Boot runtime environment"
    url: "/docs/boot/"
    btn_label: "Read More"
    btn_class: "btn--info"
---

{% include feature_row id="intro" type="center" %}

{% include feature_row id="feature_row_1" %}

{% include feature_row id="feature_row_2" %}

**Note!** This is a revamp of the IPF documentation valid as of IPF 3.6.x. Some sections are still incomplete.
The old documentation is [still available here](https://oehf.github.io/ipf).
{: .notice--info}