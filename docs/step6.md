# Step 6: Create the ADF Pipeline

Create a new Pipeline in ADF called "ingest_usage_preliminary". You may copy the pipeline's JSON definition from Will Brown's repo or proceed defining it manually using the following instructions.

   * Add two pipeline Parameters of type String (BlobPath and BlobName)

<img alt="img" width="1010px" src="/docs/images/manual_deployment_9.png" />

   * Add a "Copy data" activity from the Activities pane on the left, expand "Move and Transform" section, or type Copy data into the search and double click it or drag it onto the blank canvas. I've set the "General" tab as shown:

<img alt="img" width="610px" src="/docs/images/manual_deployment_10.png" />

   * Click the **Source** tab of this activity, to create a new dataset. Choose Azure Blob Storage and CSV. Configure the dataset properties:

<img alt="img" width="610px" src="/docs/images/manual_deployment_11.png" />

   * Open the Source dataset and set the **Escape character** to be double quote (")

<img alt="img" width="610px" src="/docs/images/manual_deployment_12.png" />

<img alt="img" width="610px" src="/docs/images/manual_deployment_13.png" />

   * Next, set the **File path type** to **Prefix** and click Add dynamic content

<img alt="img" width="810px" src="/docs/images/manual_deployment_14.png" />

   * In the popup box paste the following text:

```javascript
@{concat(pipeline().parameters.BlobPath, '/',
pipeline().parameters.BlobName)}
```

* Configure the Sink tab of the Copy Data activity:
  - Sync dataset: Select the ADX linked service from the drop down
  - Table: "UsagePreliminaryIngest"
    - Ingestion mapping name: "UsagePreliminaryMapping"
    - Additional properties
      - click **Add dynamic content** and paste in the following text:

```javascript
@concat('{"tags":"["',last(split(pipeline().parameters.BlobPath, '/')),'","drop-by:', pipeline().RunId, '"]"}')
```

*Note, this effectively adds a custom tag to our data once ingested into ADX (Kusto). The extents will be tagged a concatenated string of the BlobPath (month of the export) and the key-value "drop-by:" plus the ADF pipeline runid (unique id) which ingested the data.*

   * Click on the "Mapping" tab of our Copy Data activity. Choose the option to "Import schemas". Use the exported CSV created in Step 4 and paste in the BlobPath and BlobName

<img alt="img" width="1210px" src="/docs/images/manual_deployment_15.png" />

   * Verify that the mapping comes across as you'd expect

<img alt="img" width="910px" src="/docs/images/manual_deployment_16.png" />
  
   * Under the Activities pane on the left, add the "Azure Data Explorer Command" activity by typing it into the search and double clicking it or drag it to the canvas. It should be under the "Azure Data Explorer" Activities section. Have it execute on "Success" after the "Copy Data" activity on the canvas.

<img alt="img" width="910px" src="/docs/images/manual_deployment_17.png" />

   * Under the General Tab give the activity a name

<img alt="img" width="610px" src="/docs/images/manual_deployment_18.png" />

   * In the "Connection" tab select your ADX linked service

<img alt="img" width="810px" src="/docs/images/manual_deployment_19.png" />

   * On the "Command" tab click **Add dynamic content** and paste the
    following text:
```javascript
@concat('.drop extents <|
.show table UsagePreliminary extents
| extend Tags = split(Tags, "rn")
| where set_has_element(Tags, "', last(split(pipeline().parameters.BlobPath, '/')),'") and not(set_has_element(Tags, "drop-by:', pipeline().RunId,'"))')
```

*Note, this will run the ".drop extents" command but only drop extents for the same month that doesn't have the pipeline().RunId of the current
run.*

Below is what the Command dynamic content should look like.

<img alt="img" width="610px" src="/docs/images/manual_deployment_20.png" />

   * Next, add the trigger to schedule our pipeline. Click on "Add trigger" and choose "New/Edit".

<img alt="img" width="610px" src="/docs/images/manual_deployment_21.png" />

   * Create a new "BlobEventsTrigger" as shown:

<img alt="img" width="610px" src="/docs/images/manual_deployment_22.png" />

   * Click next until you get to the Trigger Run Parameters

| Variable Name | Value |
| --- | --- |
| BlobPath | ```javascript @replace(trigger().outputs.body.folderPath, 'usage-preliminary/', '')``` |
| BlobName | ```javascript @trigger().outputs.body.fileName``` |

   * Click **Save** and **Publish all** of your changes

<img alt="img" width="810px" src="/docs/images/manual_deployment_23.png" />

[Proceed to Testing](testing.md) ▶️
