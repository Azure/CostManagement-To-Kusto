# Testing

Now it's time to test and make sure everything works as expected. Luckily this is pretty easy to test.

   * Got back to your export and execute it manually just like in Step 4

   * This should automatically trigger the ADF Pipeline. In ADF studio, click Monitoring on the left menu blade, then Pipeline runs, and you should see the newly created pipeline has been Triggered. Verify it ran and if it failed, you can click on it to see why.

<img alt="img" width="1010px" src="/docs/images/manual_deployment_24.png" />

   * If the pipeline ran successfully, proceed to your ADX query url, select your database and verify the data in the curated table shows up as expected. Below I simply took 10 records from the *UsagePreliminary* table.

<img alt="img" width="1010px" src="/docs/images/manual_deployment_25.png" />

   * For the logic to prevent duplication to work. Run the below command, verify the extents are tagged correctly using the following KQL    command:
```kql
.show table UsagePreliminary extents
| project Tags
``````

You should see the ADF Pipeline **RunId**, followed by the **month** (ex: 20230801-20230831)

<img alt="img" width="710px" src="/docs/images/manual_deployment_26.png" />

   * Test the deduplication logic by rerunning the export job and making sure that the extents get replaced by new extents with a new RunID    but the same date range.

[Proceed to Summary](manual_deployment.md#summary) 
