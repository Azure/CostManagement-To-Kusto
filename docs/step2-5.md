# Step 2: Storage Account Container

On your blob storage account create a container called "usage-preliminary". This is where you will export the cost management data. 

<img alt="img" width="610px" src="/docs/images/manual_deployment_1.png" />


# Step 3: Grant the ADF System Assigned Identity Permission

*  We are going to use the ADF System Assigned Identity to read the blob from our storage account and to write the results to ADX. Therefore, we need to make sure it has Storage Blob Data Reader RBAC permissions on this container.

<img alt="img" width="610px" src="/docs/images/manual_deployment_2.png" />

*  Grant the ADF System Assigned Identity Database Admin permission on your ADX Database. This is required because we are dropping extents and need more permission than just ingestor.

<img alt="img" width="810px" src="/docs/images/manual_deployment_3.png" />


# Step 4: Create the Azure Cost Management Export

At this point, I recommend creating the Azure Cost Management Export, so that we'll have a csv file in storage to author the ADF pipeline mapping. If this is your first time working with an export in Azure Cost Management check out the article [here](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-export-acm-data?tabs=azure-portal).

* Below you can see how we configure the Cost Management Export which will affect the values in the ADF pipeline:

<img alt="img" width="610px" src="/docs/images/manual_deployment_4.png" />


* Click Run now to trigger the export and generate a CSV file in the storage account, which we'll to utilize during the creation of the ADF Pipeline.

<img alt="img" width="610px" src="/docs/images/manual_deployment_5.png" />

# Step 5: Create your Linked Services in Azure Data Factory

Create two linked services in ADF. One to the storage account and one to ADX (Kusto). If you haven't done this in the past, please read the [Linked Service with UI](https://learn.microsoft.com/azure/data-factory/concepts-linked-services?tabs=data-factory#linked-service-with-ui) and [Management Hub](https://learn.microsoft.com/azure/data-factory/author-management-hub) articles.

You should end up with the following two Linked Services:

<img alt="img" width="610px" src="/docs/images/manual_deployment_6.png" />


Both linked services are using the ADF System Assigned Identity

Blob Linked Service

<img alt="img" width="610px" src="/docs/images/manual_deployment_7.png" />

ADX Linked Service

<img alt="img" width="610px" src="/docs/images/manual_deployment_8.png" />

### [Proceed to Step 6](step6.md) ▶️
