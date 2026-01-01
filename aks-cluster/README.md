### AKS cluster with AGIC (Application Gateway Ingress Controller) add-on 

```bash
RESOURCE_GROUP='learning-github-actions'
AKS_NAME='learning-github-actions-aks'
ACR_NAME='learninggithubactions'
APP_GATEWAY_NAME='learning-github-actions-appgw'
APP_GATEWAY_SUBNET_CIDR='10.225.0.0/16'
REGION='westeurope'

# creates resource group
az group create --name $RESOURCE_GROUP --location $REGION
# create an acr instance
az acr create --name $ACR_NAME --resource-group $RESOURCE_GROUP --sku basic
# Deploy a new AKS cluster with the AGIC add-on enabled
# It will also create a Standard_v2 SKU application gateway instance
az aks create \
--name $AKS_NAME \
--resource-group $RESOURCE_GROUP \
--network-plugin azure \
--enable-managed-identity \
--enable-addons ingress-appgw \
--node-count 1 \
--node-vm-size 'Standard_D2ps_v6' \
--appgw-name $APP_GATEWAY_NAME \
--appgw-subnet-cidr $APP_GATEWAY_SUBNET_CIDR \
--attach-acr $ACR_NAME
```

- The AKS cluster will be created in the RESOURCE_GROUP resource group
- The Application Gateway instance will be created in the node resource group, by default: `MC_<resource-group-name>_cluster-name_location`

### Permissions for AGIC

- Please ensure the identity used by AGIC has the proper permissions. 
- A list of permissions needed by the identity can be found here: https://learn.microsoft.com/en-us/azure/application-gateway/configuration-infrastructure#permissions
- We use here built-in `Network Contributor` role (in prod use a custom role with the above permissions)

```bash
# Get application gateway id from AKS addon profile
appGatewayId=$(az aks show -n $AKS_NAME -g $RESOURCE_GROUP -o tsv --query "addonProfiles.ingressApplicationGateway.config.effectiveApplicationGatewayId")

# Get Application Gateway subnet id
appGatewaySubnetId=$(az network application-gateway show --ids $appGatewayId -o tsv --query "gatewayIPConfigurations[0].subnet.id")

# Get AGIC addon identity
agicAddonIdentity=$(az aks show -n $AKS_NAME -g $RESOURCE_GROUP -o tsv --query "addonProfiles.ingressApplicationGateway.identity.clientId")

# Assign network contributor role to AGIC addon identity to subnet that contains the Application Gateway
az role assignment create --assignee $agicAddonIdentity --scope $appGatewaySubnetId --role "Network Contributor"
```

### Get credentials

```bash
az aks get-credentials -n $AKS_NAME -g $RESOURCE_GROUP
```


### Resources:

https://learn.microsoft.com/en-us/azure/application-gateway/ingress-controller-overview