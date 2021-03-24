---
title: "An Open Source Customer Data Integration Solution"
date: 2021-03-24T11:39:56Z
type: posts
draft: False 
tags: [Data, Integration, Opensource]
---
## Customer Data Integration

The first step even before you can begin to apply data science to a problem consists of loading customer data into your system.
Although, trivial when you are doing a one off batch load/read from files or databases. It can quickly become a non-trivial task when data updates often, 
leading to many data refreshes.

To resolve this problem, data science systems use what are known as data loader/integration platforms. Most systems use 3rd party platforms for data integration. e.g., Jitterbit, Talend, AWS glue, etc. This space is mostly dominated by commercial vendors, which tend to be expensive as you scale your system and quite often result in vendor lock-ins.

## Solution

### [Airbyte](https://airbyte.io/): An opensource data integration/loader platform

An opensource data integration/loader platform which comes with drag-n-drop UI for connecting into customer systems. Airbyte supports wide range of ERP and CRM systems. Although, still a relatively new tool in the space, their public roadmap is available on Github to see the supported data source platforms. Airbyte is a promising alternative in the space of data integration dominated by commercial organizations.

When comparing the cost of e.g., jitterbit, etc. they are very expensive and additionally you are charged based on connectors as well as data transfer usage. Airbyte is a cheaper alternative, as it is free to deploy on your own or choose their managed hosted service with rates cheaper than others.


#### Additional Resources

1. https://docs.airbyte.io/tutorials/toy-connector
2. https://preset.io/blog/airbyte-superset/