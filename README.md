# azmeta-pipeline

This ARM template creates an environment for automatically ingesting Azure usage data in to an Azure Data Explorer cluster. This is useful in large accounts with millions of usage records that need to be explored and/or processed. It is also useful for any size account where custom processing is required for [cost allocation and allotment](https://github.com/wpbrown/azmeta-docs) beyond what is possible in the Cost Management portal with tags, scopes, and other discriminators.

The pipeline compresses and stores the [Usage Detail v2](https://docs.microsoft.com/en-us/azure/cost-management-billing/manage/consumption-api-overview#usage-details-api) CSV data in to a storage account for archival purposes and loads it to Azure Data Explorer for online analysis.

## Using the Data

This database is meant to be used with the azmeta ecosystem of tools such as an [azmeta-codespace](https://github.com/wpbrown/azmeta-codespace).

You can also connect to your new azmeta database using Azure Monitor's [Workbooks](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/workbooks-overview) and Azure Data Explorer's own [native tools](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/tools/) such as the [Azure Data Explorer Web UI](https://dataexplorer.azure.com/), [Azure Data Studio](https://docs.microsoft.com/en-us/sql/azure-data-studio/notebooks-kqlmagic?view=sql-server-ver15#kqlmagic-with-azure-monitor-logs), [Kusto Explorer](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/tools/kusto-explorer), or the [Kusto CLI](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/tools/kusto-cli). 

## Limitations

Currently the Enterprise Agreement [billing account type](https://docs.microsoft.com/en-us/azure/cost-management-billing/cost-management-billing-overview#billing-accounts) are supported. The new Microsoft Customer Agreement type is not yet supported.

# Architecture

The system consists of an Azure Data Explorer cluster, an Azure Data Factory instance, and a storage account. An admin using a command line tool or Azure Cost Management automatic export push usage data in to blob containers in the storage account. Azure Data Factory will automatically push new data in to Azure Data Explorer as it arrives in the blob containers.

![img](docs/images/usage-pipeline.svg)

## Data Flows

Usage data for a closed billing period will be automatically imported in to Azure Data Explorer once it is exported by the Azure Cost Management on the 5th day of the month. This data will be imported in to the `Usage` table and is considered an immutable record. This Azure Data Factory pipeline will compress and archive the usage data in the storage account before it is loaded to Azure Data Explorer.

Month-to-date usage data for the currently open billing period will be automatically import on a nightly schedule as it exported. This data will be imported in to the `UsagePreliminary` table and all records are subject to change until the billing period has closed. 

Each month, after the closed version of a month has been committed to the `Usage` table, the open version of that month will be dropped from the `UsagePreliminary` table. The `UsagePreliminary` table is a sliding window of open billing data. For example:

```
On 2020-02-28:
  Usage contains closed billing months:   ... [2019-11] [2019-12] [2020-01]
  UsagePreliminary contain open billing days: [2020-02-01 - 2020-02-27]
On 2020-03-01:
  Usage contains closed billing months:   ... [2019-11] [2019-12] [2020-01]
  UsagePreliminary contain open billing days: [2020-02-01 - 2020-02-28]
On 2020-03-04:
  Usage contains closed billing months:   ... [2019-11] [2019-12] [2020-01]
  UsagePreliminary contain open billing days: [2020-02-01 - 2020-02-28] [2020-03-01 - 2020-03-03]
On 2020-03-05:
  Usage contains closed billing months:   ... [2019-11] [2019-12] [2020-01] [2020-02]
  UsagePreliminary contain open billing days: [2020-03-01 - 2020-03-04]
On 2020-03-08:
  Usage contains closed billing months:   ... [2019-11] [2019-12] [2020-01] [2020-02]
  UsagePreliminary contain open billing days: [2020-03-01 - 2020-03-07]
```

If you are writing queries that should only deal with closed billing data, simply query against the `Usage` table. If you would like to write queries that span closed and open data you can join the tables:

```kql
Usage 
| union UsagePreliminary 
| summarize sum(CostInBillingCurrency) by Date
```

## Prerequisites

* A subscription with the resource providers `Microsoft.Storage`, `Microsoft.ContainerInstance`, and `Microsoft.EventGrid` already registered. [Register the providers](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types#azure-cli).
* A resource group is required to deploy in to. [Create a resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-cli#create-resource-groups). You must have 'Owner' rights on the resource group.
* A user-assigned managed identity is required during deployment. [Create a user-assigned managed identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli). This can be created in the resource group mentioned above, however it is not required to be located there. *This resource can be deleted after deployment of the template is complete.*
  * The identity must have 'Contributor' or higher rights on the resource group being deployed to.
  * The identity must have 'Managed Identity Operator' or higher rights on the resource group it belongs to (*if* it is not located in the resource group being deployed to).
* A service principal with a password/key is required for Azure Data Factory to connect to Azure Data Explorer. [Create a service principal](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest#password-based-authentication). Azure Data Factory does not currently support connecting Azure Data Explorer via managed identity.


# Installation

This process requires an Azure subscription, resource group, and service principal in the same Azure tenant as the users with access to the billing account. 

To implement this method via automation then click [here](/docs/template_deployment.md). A walkthrough of setting this up manually is also provided [here](/docs/manual_deployment.md). 


