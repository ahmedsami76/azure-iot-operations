### Step 1: Optimize `k3s`
Run the following command to increase the user watch/instance limits.
```bash
echo fs.inotify.max_user_instances=8192 | sudo tee -a /etc/sysctl.conf
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

For better performance, increase the file descriptor limit:
```bash
echo fs.file-max = 100000 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Step 2: Arc-enable your cluster
Register the required resource providers in your subscription
```bash
az provider register -n "Microsoft.ExtendedLocation"
az provider register -n "Microsoft.Kubernetes"
az provider register -n "Microsoft.KubernetesConfiguration"
az provider register -n "Microsoft.IoTOperations"
az provider register -n "Microsoft.DeviceRegistry"
az provider register -n "Microsoft.SecretSyncController"
```

Use the az connectedk8s connect command to Arc-enable your Kubernetes cluster and manage it as part of your Azure resource group.
```bash
az login
```

```bash
CLUSTER_NAME=aiok3scluster
REGION=westus2
RESOURCE_GROUP=rg-aio
SUBSCRIPTION_ID=$(az account list --query "[?isDefault].id" --output tsv)


az group create --location $REGION --resource-group $RESOURCE_GROUP --subscription $SUBSCRIPTION_ID
az extension add --upgrade --name connectedk8s

az connectedk8s connect --name $CLUSTER_NAME -l $REGION --resource-group $RESOURCE_GROUP --subscription $SUBSCRIPTION_ID --enable-oidc-issuer --enable-workload-identity --disable-auto-upgrade
```

Get cluster issuer's URL
```bash
az connectedk8s show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query oidcIssuerProfile.issuerUrl --output tsv
```

Create a k3s config file.

```bash
sudo nano /etc/rancher/k3s/config.yaml
```
Add the following content to the config.yaml file, replacing the <SERVICE_ACCOUNT_ISSUER> placeholder with your cluster's issuer URL.
```bash
kube-apiserver-arg:
 - service-account-issuer=<SERVICE_ACCOUNT_ISSUER>
 - service-account-max-token-expiration=24h
```

Get the objectId of the Microsoft Entra ID application that the Azure Arc service uses in your tenant and save it as an environment variable. Run the following command exactly as written, without changing the GUID value.

```bash
export OBJECT_ID=$(az ad sp show --id bc313c14-388c-4e7d-a58e-70017303ee3b --query id -o tsv)
```

Use the az connectedk8s enable-features command to enable custom location support on your cluster. This command uses the objectId of the Microsoft Entra ID application that the Azure Arc service uses. Run this command on the machine where you deployed the Kubernetes cluster:

```BASH
az connectedk8s enable-features -n $CLUSTER_NAME -g $RESOURCE_GROUP --custom-locations-oid $OBJECT_ID --features cluster-connect custom-locations
```

Restart K3s.
```bash
systemctl restart k3s
```