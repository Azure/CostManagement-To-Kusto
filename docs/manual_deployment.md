# Walkthrough
## Export Azure Cost Management to Kusto

## Process Overview
At a high-level the process will be:
1. Daily export your cost data using Azure Cost Management to blob storage
2. When its created, load the blob automatically into ADX
3. Ensure you don't introduce duplicate records with your ingestion

Eliminating duplicates is the most important part of this process. To understand why deduplication is important, you need to understand how Azure Cost Management export [works](https://learn.microsoft.com/azure/cost-management-billing/costs/tutorial-export-acm-data).

## Prerequisites
* A subscription with the resource providers `Microsoft.Storage`, `Microsoft.ContainerInstance`, and `Microsoft.EventGrid` already registered. [Register the providers](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types#azure-cli).
* A resource group is required to deploy in to. [Create a resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-cli#create-resource-groups). You must have 'Owner' rights on the resource group.
## Azure Cost Management Export Overview

With Azure Cost Management Exports, you can export your data daily. The export places a new csv file on azure blob storage daily with all the data upto that day, month-to-date, or weekly depending on your selection. For example, on the 1st of the month you'll have a csv file with one day's worth of data, and on the 20th day of the month you'll have another csv file with 20 days-worth of data. However, these costs aren't finalized until the end of the month, so your 1st day of the month may not match if you compare the exports of the 20th day for the 1st day expenditures. Therefore, if we do not handle duplicate records correctly the data will NOT make sense.

1. We can end up with a lot of duplicates for each day.
2. We end up keeping the wrong record instead of the latest.
3. An update to a record made later during the month doesn't appear (ie. missing data).

## Example Implementation
For this implementation the following are needed:
1. Azure Data Explorer (required for performance needs)
2. Azure Storage Account
3. Azure Data Factory (allows to easily tag extents)
4. Cost Management Export

[Proceeed to Step 1](step1.md) ▶️


## Considerations
In this walkthrough we utilized ADF to manage the ingestion to a Kusto Database and orchestrate elimination of duplicates. But that's not the only way to handle this workflow.

- An Azure Function could be used instead of ADF with [ADX Bindings for Azure Functions](https://techcommunity.microsoft.com/t5/azure-data-explorer-blog/azure-data-explorer-kusto-bindings-for-azure-functions-public/ba-p/3828472). However, that would be a code-first aproach, instead of using the low-code UI in ADF.
- A [KQL materialize view](https://learn.microsoft.com/azure/data-explorer/kusto/management/materialized-views/materialized-view-overview) could be used instead of [.drop extents](https://learn.microsoft.com/azure/data-explorer/kusto/management/drop-extents) command to eliminate duplicates. As long as you understand which columns make the records unique, add the tag as values in an additional column or build a *surrogate-key* to uniquely identify or filter by, then a KQL materialize view would work. You wouldn't need to delete data either. Consider that records may be updated throughout the month (ie. prices change, etc) the materialized view would need to utilize an [arg_max()](https://learn.microsoft.com/azure/data-explorer/kusto/query/max-aggfunction) instead of a [take_any()](https://learn.microsoft.com/azure/data-explorer/kusto/query/take-any-aggfunction) for the deduplication.

## Summary
After implementing this example, the cost information for your Azure subscription(s) will be refreshed daily into your ADX database - as of the previous day.

Having this data in ADX can unlock other scenarios such as:
- Join this information with other metadata to enrich and provide new insights.
- Running ad-hoc queries in seconds to look/dashboard over the historical spend.
- Build powerful visuals in your favorite tool such as ADX Dashboards, Power BI, Grafana, Tableau, Jupyter Notebooks, etc...
