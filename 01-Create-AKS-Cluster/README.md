# Create AKS Cluster

## Step-01: Introduction
- Understand about AKS Cluster
- Discuss about Kubernetes Architecture from AKS Cluster perspective


## Step-02: Create AKS Cluster
- Create Kubernetes Cluster
- **Basics**
  - **Subscription:** Free Trial
  - **Resource Group:** Creat New: aks-rg1
  - **Kubernetes Cluster Name:** aksdemo1
  - **Region:** (US) Central US
  - **Kubernetes Version:** Select what ever is latest stable version
  - **Node Size:** Standard DS2 v2 (Default one)
  - **Node Count:** 1
- **Node Pools**
  - leave to defaults
- **Authentication**
  - Authentication method: 	System-assigned managed identity
  - Rest all leave to defaults
- **Networking**
  - **Network Configuration:** Advanced
  - **Network Policy:** Azure
  - Rest all leave to defaults
- **Integrations**
  - Azure Container Registry: None
  - leave to defaults
- **Tags**
  - leave to defaults
- **Review + Create**
  - Click on **Create**


## Step-02: Create AKS Cluster

**AKS through CLI**

az group create --name myResourceGroup --location eastus

az aks create -g myResourceGroup -n aksdemo1 --enable-managed-identity --node-count 1 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys

## Step-03: Cloud Shell - Configure kubectl to connect to AKS Cluster
- Go to https://shell.azure.com
```
# Template
az aks get-credentials --resource-group <Resource-Group-Name> --name <Cluster-Name>

# Replace Resource Group & Cluster Name
az aks get-credentials --resource-group aks-rg1 --name aksdemo1

# List Kubernetes Worker Nodes
kubectl get nodes 
kubectl get nodes -o wide
```

## Step-04: Explore Cluster Control Plane and Workload inside that
```
# List Namespaces
kubectl get namespaces
kubectl get ns

# List Pods from all namespaces
kubectl get pods --all-namespaces

# List all k8s objects from Cluster Control plane
kubectl get all --all-namespaces
```

## Step-05: Explore the AKS cluster on Azure Management Console
- Explore the following features on high-level
- **Overview**
  - Activity Log
  - Access Control (IAM)
  - Security
  - Diagnose and solver problems
- **Settings**
  - Node Pools
  - Upgrade
  - Scale
  - Networking
  - DevSpaces
  - Deployment Center
  - Policies
- **Monitoring**
  - Insights
  - Alerts
  - Metrics
  - and many more 
- **VM Scale Sets**
  - Verify Azure VM Instances
  - Verify if **Enhanced Networking is enabled or not**  



## Step-06: Local Desktop - Install Azure CLI and Azure AKS CLI
```
# Install Azure CLI (MAC)
brew update && brew install azure-cli

# Login to Azure
az login

# Install Azure AKS CLI
az aks install-cli

# Configure Cluster Creds (kube config)
az aks get-credentials --resource-group aks-rg1 --name aksdemo1

# List AKS Nodes
kubectl get nodes 
kubectl get nodes -o wide
```
- **Reference Documentation Links**
- https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest
- https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest

## Step-07: Deploy Sample Application and Test
- Don't worry about what is present in these two files for now. 
- By the time we complete **Kubernetes Fundamentals** sections, you will be an expert in writing Kubernetes manifest in YAML.
- For now just focus on result. 
```
# Deploy Application
kubectl apply -f kube-manifests/

# Verify Pods
kubectl get pods

# Verify Deployment
kubectl get deployment

# Verify Service (Make a note of external ip)
kubectl get service

# Access Application
http://<External-IP-from-get-service-output>
```

## Step-07: Clean-Up
```
# Delete Applications
kubectl delete -f kube-manifests/
```

## References
- https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-macos?view=azure-cli-latest

## Why Managed Identity when creating Cluster?
- https://docs.microsoft.com/en-us/azure/aks/use-managed-identity



# Create AKS Cluster and Applicationgateway using cmd:

### Create Resource Group:

az group create --name aks-westus --location westus

### Create a VNET and a subnet for the AKS cluster:

az network vnet create \
    --resource-group aks-westus \
    --name aks-westus-vnet\
    --address-prefixes 192.168.0.0/18 \
    --subnet-name aks-westus-subnet \
    --subnet-prefix 192.168.1.0/24
    
    
## Create a subnet for APP Gateway:

az network vnet subnet create \
   --name aks-westus-appgw-subnet \
   --resource-group aks-westus \
   --vnet-name aks-westus-vnet   \
   --address-prefix 192.168.2.0/24
   
## Create a static public IP:

az network public-ip create \
   --resource-group aks-westus \
   --name aks-westus-cluster-appgw-pip \
   --allocation-method Static \
   --dns-name aks-westus-cluster-appgw \
   --sku Standard
### get public IP value and ID (we will need it later)

AKS_PUB_IP=$(az network public-ip show -g aks-westus -n aks-westus-cluster-appgw-pip --query ipAddress -o tsv)

AKS_PUB_IP_ID=$(az network public-ip show -g aks-westus -n aks-westus-cluster-appgw-pip --query id -o tsv)

### Deploy App gateway using our static IP and our subnet:

az network application-gateway create \
   --name aks-westus-appgw \
   --location westus \
   --resource-group aks-westus \
   --capacity 2 \
   --sku Standard_v2 \
   --public-ip-address aks-westus-cluster-appgw-pip\
   --vnet-name aks-westus-vnet\
   --subnet aks-westus-appgw-subnet
   --priority 100
   
 ### get application gateway id
 
APPGW_ID=$(az network application-gateway show -g aks-westus -n aks-westus-appgw --query id -o tsv)

AKS_WESTUS_SUBNET_ID=$(az network vnet subnet show --resource-group aks-westus --vnet-name aks-westus-vnet --name aks-westus-subnet --query id -o tsv)

### Deploy an AKS cluster using our subnet resource ID: 

az aks create -y \
    --resource-group aks-westus \
    --name aks-westus-cluster \
    --enable-managed-identity \
    --network-plugin azure \
    --vnet-subnet-id $AKS_WESTUS_SUBNET_ID \
    --docker-bridge-address 172.17.0.1/16 \
    --service-cidr 10.2.0.0/24 \
    --dns-service-ip 10.2.0.10 \
    --enable-managed-identity \
    --generate-ssh-keys \
    --enable-addons ingress-appgw \
    --appgw-id $APPGW_ID
   
