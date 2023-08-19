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

With Azure Cost Management Exports, you can export your data daily. The export places a new csv file on azure blob storage daily with all the data upto that day, month-to-date, or weekly depending on your selection. For example, on the 1^st^ of the month you'll have a csv file with one day's worth of data, and on the 20^th^ day of the month you'll have another csv file with 20 days-worth of data. However, these costs aren't finalized until the end of the month, so your 1^st^ day of the month may not match if you compare the exports of the 20^th^ day for the 1^st^ day expenditures. Therefore, if we do not handle duplicate records correctly the data will NOT make sense.

1. We can end up with a lot of duplicates for each day.
2. We end up keeping the wrong record instead of the latest.
3. An update to a record made later during the month doesn't appear (ie. missing data).

## Example Implementation
For this implementation the following are needed:
1. Azure Data Explorer (required for performance needs)
2. Azure Storage Account
3. Azure Data Factory (allows to easily tag extents)
4. Cost Management Export

[Proceeed to Step 1](Step1.md) ▶️
