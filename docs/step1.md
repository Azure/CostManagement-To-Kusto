# Step 1: Setting up ADX
This setup will contain the following in ADX
1. UsagePreliminaryIngest: Raw table where data is ingested (bronze)
2. UsagePreliminaryMapping: Raw table CSV mapping for data ingestion
3. UsagePreliminary: Curated table (silver) used for querying & reporting
4. IngestToUsagePreliminary: KQL function that does the minor transformation of the raw table and used by the update policy of the curated table
5. Update policy on table UsagePreliminary so data automatically flows transactionally from raw table *UsagePreliminaryIngest* into landing table *UsagePreliminary*
6. Retention policy on the raw table *UsagePreliminaryIngest* with a retention of 0, eliminating ADX storage costs, given data is transformed into *UsagePreliminary* immediately upon ingestion.

The below script can be utilized to create everything with the correct
schema.

````kql
.execute database script <|
//Create raw table for usage data
.create table UsagePreliminaryIngest (InvoiceSectionName:string, AccountName:string, AccountOwnerId:string, SubscriptionId:string, SubscriptionName:string, ResourceGroup:string, ResourceLocation:string, Date:datetime, ProductName:string, MeterCategory:string, MeterSubCategory:string, MeterId:string, MeterName:string, MeterRegion:string, UnitOfMeasure:string, Quantity:decimal, EffectivePrice:decimal, CostInBillingCurrency:decimal, CostCenter:string, ConsumedService:string, ResourceId:string, Tags:string, OfferId:string, AdditionalInfo:dynamic, ServiceInfo1:string, ServiceInfo2:string, ResourceName:string, ReservationId:string, ReservationName:string, UnitPrice:decimal, ProductOrderId:string, ProductOrderName:string,Term:string, PublisherType:string, PublisherName:string, ChargeType:string, Frequency:string, PricingModel:string, AvailabilityZone:string, BillingAccountId:string, BillingAccountName:string, BillingCurrencyCode:string, BillingPeriodStartDate:datetime, BillingPeriodEndDate:datetime, BillingProfileId:string, BillingProfileName:string, InvoiceSectionId:string, IsAzureCreditEligible:string, PartNumber:string, PayGPrice:decimal, PlanName:string, ServiceFamily:string)
//
//Create the ingestion mapping
.create-or-alter table Usage ingestion csv mapping
'UsagePreliminaryMapping'
'[{"Name":"InvoiceSectionName","DataType":"","Ordinal":"0","ConstValue":null},{"Name":"AccountName","DataType":"","Ordinal":"1","ConstValue":null},{"Name":"AccountOwnerId","DataType":"","Ordinal":"2","ConstValue":null},{"Name":"SubscriptionId","DataType":"","Ordinal":"3","ConstValue":null},{"Name":"SubscriptionName","DataType":"","Ordinal":"4","ConstValue":null},{"Name":"ResourceGroup","DataType":"","Ordinal":"5","ConstValue":null},{"Name":"ResourceLocation","DataType":"","Ordinal":"6","ConstValue":null},{"Name":"Date","DataType":"","Ordinal":"7","ConstValue":null},{"Name":"ProductName","DataType":"","Ordinal":"8","ConstValue":null},{"Name":"MeterCategory","DataType":"","Ordinal":"9","ConstValue":null},{"Name":"MeterSubCategory","DataType":"","Ordinal":"10","ConstValue":null},{"Name":"MeterId","DataType":"","Ordinal":"11","ConstValue":null},{"Name":"MeterName","DataType":"","Ordinal":"12","ConstValue":null},{"Name":"MeterRegion","DataType":"","Ordinal":"13","ConstValue":null},{"Name":"UnitOfMeasure","DataType":"","Ordinal":"14","ConstValue":null},{"Name":"Quantity","DataType":"","Ordinal":"15","ConstValue":null},{"Name":"EffectivePrice","DataType":"","Ordinal":"16","ConstValue":null},{"Name":"CostInBillingCurrency","DataType":"","Ordinal":"17","ConstValue":null},{"Name":"CostCenter","DataType":"","Ordinal":"18","ConstValue":null},{"Name":"ConsumedService","DataType":"","Ordinal":"19","ConstValue":null},{"Name":"ResourceId","DataType":"","Ordinal":"20","ConstValue":null},{"Name":"Tags","DataType":"","Ordinal":"21","ConstValue":null},{"Name":"OfferId","DataType":"","Ordinal":"22","ConstValue":null},{"Name":"AdditionalInfo","DataType":"","Ordinal":"23","ConstValue":null},{"Name":"ServiceInfo1","DataType":"","Ordinal":"24","ConstValue":null},{"Name":"ServiceInfo2","DataType":"","Ordinal":"25","ConstValue":null},{"Name":"ResourceName","DataType":"","Ordinal":"26","ConstValue":null},{"Name":"ReservationId","DataType":"","Ordinal":"27","ConstValue":null},{"Name":"ReservationName","DataType":"","Ordinal":"28","ConstValue":null},{"Name":"UnitPrice","DataType":"","Ordinal":"29","ConstValue":null},{"Name":"ProductOrderId","DataType":"","Ordinal":"30","ConstValue":null},{"Name":"ProductOrderName","DataType":"","Ordinal":"31","ConstValue":null},{"Name":"Term","DataType":"","Ordinal":"32","ConstValue":null},{"Name":"PublisherType","DataType":"","Ordinal":"33","ConstValue":null},{"Name":"PublisherName","DataType":"","Ordinal":"34","ConstValue":null},{"Name":"ChargeType","DataType":"","Ordinal":"35","ConstValue":null},{"Name":"Frequency","DataType":"","Ordinal":"36","ConstValue":null},{"Name":"PricingModel","DataType":"","Ordinal":"37","ConstValue":null},{"Name":"AvailabilityZone","DataType":"","Ordinal":"38","ConstValue":null},{"Name":"BillingAccountId","DataType":"","Ordinal":"39","ConstValue":null},{"Name":"BillingAccountName","DataType":"","Ordinal":"40","ConstValue":null},{"Name":"BillingCurrencyCode","DataType":"","Ordinal":"41","ConstValue":null},{"Name":"BillingPeriodStartDate","DataType":"","Ordinal":"42","ConstValue":null},{"Name":"BillingPeriodEndDate","DataType":"","Ordinal":"43","ConstValue":null},{"Name":"BillingProfileId","DataType":"","Ordinal":"44","ConstValue":null},{"Name":"BillingProfileName","DataType":"","Ordinal":"45","ConstValue":null},{"Name":"InvoiceSectionId","DataType":"","Ordinal":"46","ConstValue":null},{"Name":"IsAzureCreditEligible","DataType":"","Ordinal":"47","ConstValue":null},{"Name":"PartNumber","DataType":"","Ordinal":"48","ConstValue":null},{"Name":"PayGPrice","DataType":"","Ordinal":"49","ConstValue":null},{"Name":"PlanName","DataType":"","Ordinal":"50","ConstValue":null},{"Name":"ServiceFamily","DataType":"","Ordinal":"51","ConstValue":null}]'
//
// Create the landing table for the data.
// This is the table that will hold the data and you will query
.create table UsagePreliminary (InvoiceSectionName:string, AccountName:string, AccountOwnerId:string, SubscriptionId:string, SubscriptionName:string, ResourceGroup:string, ResourceLocation:string, Date:datetime, ProductName:string, MeterCategory:string, MeterSubCategory:string, MeterId:string, MeterName:string, MeterRegion:string, UnitOfMeasure:string, Quantity:decimal, EffectivePrice:decimal, CostInBillingCurrency:decimal, CostCenter:string, ConsumedService:string, ResourceId:string, Tags:dynamic, OfferId:string, AdditionalInfo:dynamic, ServiceInfo1:string, ServiceInfo2:string, ResourceName:string, ReservationId:string, ReservationName:string, UnitPrice:decimal, ProductOrderId:string, ProductOrderName:string, Term:string, PublisherType:string, PublisherName:string, ChargeType:string, Frequency:string, PricingModel:string, AvailabilityZone:string, BillingAccountId:string, BillingAccountName:string, BillingCurrencyCode:string, BillingPeriodStartDate:datetime, BillingPeriodEndDate:datetime, BillingProfileId:string, BillingProfileName:string, InvoiceSectionId:string, IsAzureCreditEligible:string, PartNumber:string, PayGPrice:decimal, PlanName:string, ServiceFamily:string)
//
//Create the function that will do the minor transformation from the raw table
.create-or-alter function  IngestToUsagePreliminary() {
    UsagePreliminaryIngest
    | extend Tags = todynamic(strcat('{', Tags, '}')), ResourceId=tolower(ResourceId)
}
//
// Create the update policy on the landing table so data will flow from raw table to landing table
.alter table UsagePreliminary policy update ```
[
  {
    "IsEnabled": true,
    "Source": "UsagePreliminaryIngest",
    "Query": "IngestToUsagePreliminary()",
    "IsTransactional": true,
    "PropagateIngestionProperties": true,
    "ManagedIdentity": null
  }
] 
```
//
// Set the retention on the landing table to 0 days
.alter table UsagePreliminaryIngest policy retention 
```
{
   "SoftDeletePeriod": "00:00:00",
   "Recoverability": "Disabled"
} ```

````

[Proceed to Step 2](step2.md) ▶️
