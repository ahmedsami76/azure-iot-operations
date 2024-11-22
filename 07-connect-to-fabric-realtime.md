### Step 1: Enable Secure Settings
In order to setup Fabric Real-time intelligence as destination in a dataflow you will need to enable secure settings first (so far we were on testing settings)

Set up secrets management
1. Create an Azure key vault that's used to store secrets, and give your user account permissions to manage secrets with the `Key Vault Secrets Officer` role.
2. Create a user-assigned managed identity for the Secret Store extension.
    1. Sign in to the Azure portal.
    2. In the search box, enter Managed Identities. Under Services, select Managed Identities.
    3. Select Add, and enter values in the following boxes in the Create User Assigned Managed Identity pane: Subscription, ResourceGroup, Region and Name
    4. Select Review + create to review the changes.
    5. Select Create.

3. Use the `az iot ops secretsync enable` command to set up the Azure IoT Operations instance for secret synchronization:
```bash
# Variable block
AIO_INSTANCE_NAME="<AIO_INSTANCE_NAME>"
RESOURCE_GROUP="<RESOURCE_GROUP>"
USER_ASSIGNED_MI_NAME="<USER_ASSIGNED_MI_NAME>"
KEYVAULT_NAME="<KEYVAULT_NAME>"

#Get the resource ID of the user-assigned managed identity
USER_ASSIGNED_MI_RESOURCE_ID=$(az identity show --name $USER_ASSIGNED_MI_NAME --resource-group $RESOURCE_GROUP --query id --output tsv)

#Get the resource ID of the key vault
KEYVAULT_RESOURCE_ID=$(az keyvault show --name $KEYVAULT_NAME --resource-group $RESOURCE_GROUP --query id --output tsv)

#Enable secret synchronization
az iot ops secretsync enable --instance $AIO_INSTANCE_NAME --resource-group $RESOURCE_GROUP --mi-user-assigned $USER_ASSIGNED_MI_RESOURCE_ID --kv-resource-id $KEYVAULT_RESOURCE_ID
```

### Step 2: Retrive the Kafka endpoint connection details
1. From Fabric workspace create an eventstream
2. Add custom endpoint data as a source
    1. To add a custom endpoint source, on the get-started page, select Use custom endpoint. Or, if you already have a published eventstream and you want to add custom endpoint data as a source, switch to edit mode. On the ribbon, select Add source > Custom endpoint
    2. In the Custom endpoint dialog, enter a name for the custom source under Source name, and then select Add.
    3. After you create the custom endpoint source, it's added to your eventstream on the canvas in edit mode. To implement the newly added data from the custom app source, select Publish.
3. Get endpoint details on the Details pane:
    1. The Details pane has three protocol tabs: Event Hub, AMQP, and Kafka. Select the Kafka tab from the Details pane
    2. Click the "SAS Key Authentication" tab from the lower pane
    3. Make note of the (Bootstrap server, Topic name and Connection string-primary key) values as they will be needed to configure the dataflow endpoint



### Step 3: Create a Microsoft Fabric Real-Time Intelligence dataflow endpoint
1. In the IoT Operations portal, select the Dataflow endpoints tab.
2. Under Create new dataflow endpoint, select Microsoft Fabric Real-Time Intelligence > New.

| Setting	                            |   Description                                         |
| ---------                             |   ------------                                        |
| Name	                                |   The name of the dataflow endpoint.
| Host	                                |   The hostname of the Event Stream Custom Endpoint in the format *.servicebus.windows.net:9093. Use the bootstrap server address noted previously.
| Authentication method                 |	SASL is the currently the only supported authentication method.
| SASL type	                            |   Choose Plain
| Synced secret name                    |	Name of secret that will be synced to the Kubernetes cluster. You can choose any name.
| Username reference of token secret    |	Create a new or choose an existing Key Vault reference. The secret value must be the literal string $ConnectionString. It isn't an environment variable.
| Password reference of token secret    |	Create a new or choose an existing Key Vault reference. The secret value must be the Custom Endpoint connection string noted earlier.

### Step 4: Create a dataflow
1. In the [operations experience](iotoperationsazure.com), select the Dataflows tab 
2. Click the "+ Create dataflow" button
3. Click "Select source" tile from the flowchart 
4. while the "Asset" tile is selected click to select the "oven" option under "Asset name" then click Apply
5. From the flowchart click the "Select dataflow endpoint" tile
6. From the Dataflow endpoint details blade select the fabric realtime endpoint created above then click proceed
7. In the Dataflow endpoint details blade and under "Topic" type or paste the topic value of the Kafka custom endpoint then click Apply
8. Click Save and give a name to the dataflow then click Apply then click save again

Wait until the dataflow is accepted and successful.

Check the Fabric eventstream and make sure that there's streaming data flowing into it.