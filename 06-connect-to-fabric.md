### Step 1: Create a workspace
[Create a workspace](https://learn.microsoft.com/en-us/fabric/get-started/create-workspaces)

### Step 2: Create a lakehouse
[Create a lakehouse](https://learn.microsoft.com/en-us/fabric/onelake/create-lakehouse-onelake)
Note: ensure Lakehouse schemas (Public Preview) is **unchecked**


### Step 3: Enable SP access to Fabric
From Fabric Admin Settings enable the following settings:
- Service principals can use Fabric APIs
- Allow service principals to create and use profiles

On the Fabric Wrkspace grant contributor access to the System Managed Identity or SPN 

### Step 4: Grant the System-Assigend Managed Identity acccess to Fabric workspace
If using system-assigned managed identity, in Azure portal, go to your Azure IoT Operations instance and select Overview. 
Copy the name of the extension listed after Azure IoT Operations Arc extension. For example, azure-iot-operations-xxxx7. 
Your system-assigned managed identity can be found using the same name of the Azure IoT Operations Arc extension.

Go to Microsoft Fabric workspace you created, select Manage access > + Add people or groups.

Search for the name of your user-assigned managed identity set up for cloud connections or the system-assigned managed identity. 
For example, azure-iot-operations-xxxx7. Select Contributor as the role, then select Add. 
This gives the managed identity the necessary permissions to write to the Fabric lakehouse

### Step 5: Create dataflow endpoint for Microsoft Fabric OneLake
In the [operations experience](iotoperationsazure.com), select the Dataflow endpoints tab

Under Create new dataflow endpoint, select Microsoft Fabric OneLake > New.
Enter the following settings for the endpoint:
| Setting                | Description                                                                                                     |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------|               
| Host                   | The hostname of the Microsoft Fabric OneLake endpoint in the format onelake.dfs.fabric.microsoft.com.           |
| Lakehouse name	     | The name of the lakehouse where the data should be stored.                                                      |
| Workspace name	     | The name of the workspace associated with the lakehouse.                                                        |
| OneLake path type      | The type of path used in OneLake. Select Files or Tables.                                                       |
| Authentication method  | The method used for authentication. Choose System assigned managed identity or User assigned managed identity.  |
| Client ID	             | The client ID of the user-assigned managed identity. Required if using User assigned managed identity.          |
| Tenant ID	             | The tenant ID of the user-assigned managed identity. Required if using User assigned managed identity.          |

### Step 6: Generate Schema file for the data
In order to send data to Fabric Lakehouse you will need to have a schema file representing your data either in Delta format or JSON format.
To generate the schema from a sample data file, use the [Schema Gen Helper](https://azure-samples.github.io/explore-iot-operations/schema-gen-helper/)

The event body of the oven data we are using for the OPC simulator is as follows:

```json
{
  "Temperature": {
    "SourceTimestamp": "2024-11-15T21:40:28.5062427Z",
    "Value": 6416
  },
  "FillWeight": {
    "SourceTimestamp": "2024-11-15T21:40:28.5063811Z",
    "Value": 6416
  },
  "EnergyUse": {
    "SourceTimestamp": "2024-11-15T21:40:28.506383Z",
    "Value": 6416
  }
}
```

use the schema gen helper to generate and download the schema file for Delta format and store it locally.

### Step 7: Create a Dataflow for Fabric OneLake destination:
1. In the [operations experience](iotoperationsazure.com), select the Dataflows tab 
2. Click the "+ Create dataflow" button
3. Click "Select source" tile from the flowchart 
4. while the "Asset" tile is selected click to select the "oven" option under "Asset name" then click Apply
5. From the flowchart click the "Select dataflow endpoint" tile
6. From the Dataflow endpoint details blade select the fabric dataflow endpoint created above then click proceed
7. Type a Table or folder name 
8. Click the "+ Upload" button to upload the generated schema file
9. Make sure "Delta" is select as the Serialization format then click Apply
10. Click Save and give a name to the dataflow then click Apply then click save again

Wait until the dataflow is accepted and successful.

Check the Fabric lakehouse and make sure the Table is created and populated as expected