### Step 0: Make sure you are logged in using the right acccount and scope
```bash
az login
```

### Step 1: 
The Azure IoT Operations extension for Azure CLI. Use the following command to add the extension or update it to the latest version:
```bash
az extension add --upgrade --name azure-iot-ops

```

### Step 2: Create Storage account
```bash
STORAGE_ACCOUNT=<STORAGE_ACCOUNT_NAME>
SCHEMA_REGISTRY=<SCHEMA_REGISTRY_NAME>
SCHEMA_REGISTRY_NAMESPACE=<SCHEMA_REGISTRY_NAMESPACE>
LOCATION=westus2
RESOURCE_GROUP=<RESOURCE GROUP NAME>
SUBSCRIPTION_ID=$(az account list --query "[?isDefault].id" --output tsv)
CLUSTER_NAME=<K3S CLUSTER NAME>
REGION=westus2

az storage account create --name $STORAGE_ACCOUNT --location $LOCATION --resource-group $RESOURCE_GROUP --enable-hierarchical-namespace
```

### Step 3: Deploy IoTOPS with testing settings
Create a schema registry which will be used by Azure IoT Operations components after the deployment
```bash
az iot ops schema registry create  --subscription $SUBSCRIPTION_ID  -g  $RESOURCE_GROUP -n $SCHEMA_REGISTRY  --registry-namespace $SCHEMA_REGISTRY_NAMESPACE  --sa-resource-id /subscriptions/$SUBSCRIPTION_ID/resourcegroups/$RESOURCE_GROUP/providers/Microsoft.Storage/storageAccounts/$STORAGE_ACCOUNT  --sa-container schema
```

Prepare the cluster for Azure IoT Operations deployment
```bash
az iot ops init  --subscription $SUBSCRIPTION_ID  -g rg-aio  --cluster $CLUSTER_NAME
```

Deploy Azure IoT Operations
```bash
az iot ops create  --subscription $SUBSCRIPTION_ID  -g rg-aio  --cluster $CLUSTER_NAME  --custom-location aiok3scluster-cl-8778  -n aiok3scluster-ops-instance  --sr-resource-id /subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.DeviceRegistry/schemaRegistries/$SCHEMA_REGISTRY  --broker-frontend-replicas 2  --broker-frontend-workers 2  --broker-backend-part 2  --broker-backend-workers 2  --broker-backend-rf 2  --broker-mem-profile Medium
```

Check Deployment
```bash
az iot ops check
```
