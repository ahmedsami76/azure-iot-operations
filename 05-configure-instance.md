### Step 1: Deploy OPC PLC simualtor
This quickstart uses the OPC PLC simulator to generate sample data. To deploy the OPC PLC simulator, run the following command:

```bash
sudo kubectl apply -f https://raw.githubusercontent.com/Azure-Samples/explore-iot-operations/main/samples/quickstarts/opc-plc-deployment.yaml
```

### Step 2: Configure Cluster
Adds an asset endpoint that connects to the OPC PLC simulator.
Adds an asset that represents the oven and defines the data points that the oven exposes.
Adds a dataflow that manipulates the messages from the simulated oven.
Creates an Azure Event Hubs instance to receive the data

```bash
RESOURCE_GROUP=<RESOURCE GROUP NAME>
SUBSCRIPTION_ID=$(az account list --query "[?isDefault].id" --output tsv)
CLUSTER_NAME=<K3S CLUSTER NAME>

wget https://raw.githubusercontent.com/Azure-Samples/explore-iot-operations/main/samples/quickstarts/quickstart.bicep -O quickstart.bicep

AIO_EXTENSION_NAME=$(az k8s-extension list -g $RESOURCE_GROUP --cluster-name $CLUSTER_NAME --cluster-type connectedClusters --query "[?extensionType == 'microsoft.iotoperations'].id" -o tsv | awk -F'/' '{print $NF}')
AIO_INSTANCE_NAME=$(az iot ops list -g $RESOURCE_GROUP --query "[0].name" -o tsv)
CUSTOM_LOCATION_NAME=$(az iot ops list -g $RESOURCE_GROUP --query "[0].extendedLocation.name" -o tsv | awk -F'/' '{print $NF}')

az deployment group create --subscription $SUBSCRIPTION_ID --resource-group $RESOURCE_GROUP --template-file quickstart.bicep --parameters clusterName=$CLUSTER_NAME customLocationName=$CUSTOM_LOCATION_NAME aioExtensionName=$AIO_EXTENSION_NAME aioInstanceName=$AIO_INSTANCE_NAME


```