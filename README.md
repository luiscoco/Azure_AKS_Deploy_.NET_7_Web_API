# How to deploy SpringBoot WebAPI to Azure Kubernetes (AKS)

## 1. Prerequisites

Install **Kubectl** command in Windows: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

Download and install **Docker Desktop**: https://docs.docker.com/desktop/install/windows-install/

Install **Azure CLI**: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows

## 2. Create a Service Principal

This Service Principal's credentials can then be used in various automated workflows to securely pull images from the registry.

```
az ad sp create-for-rbac ^
    --name service-principal-name ^
    --scopes /subscriptions/SubscriptionID/resourceGroups/ResourceGroupName/providers/Microsoft.ContainerRegistry/registries/myregistryluiscoco1974 ^
    --role acrpull ^
    --query "password" ^
    --output tsv
```

After creating the a service principal we get the secret value as output

**SecretValue**: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX

This command is used in Azure to create a Service Principal with the Azure Container Registry Pull (**acrpull**) role. Let's break down each part of the command to understand what it does:

Command Overview:

**az ad sp create-for-rbac**: This is an Azure CLI command to create a new Azure Active Directory (Azure AD) Service Principal. 

Service Principals are used in Azure to provide a security identity for applications or automated tools to access specific Azure resources.

Parameters:

**--name service-principal-name**: This specifies the name of the new Service Principal. 

You should replace service-principal-name with a name you choose for your Service Principal.

**--scopes** /subscriptions/SubscriptionID/resourceGroups/ResourceGroupName/providers/Microsoft.ContainerRegistry/registries/myregistryluiscoco1974: This defines the scope of access for the Service Principal. 

The scope is set to a specific Azure Container Registry within a subscription and resource group. 

You should replace SubscriptionID with your Azure subscription ID, ResourceGroupName with the name of your resource group, and myregistryluiscoco1974 with the name of your container registry.

**--role acrpull**: This assigns the role of acrpull to the Service Principal. 

The acrpull role allows the Service Principal to pull images from the Azure Container Registry. 

It's a read-only permission specific to Azure Container Registry.

**--query "password"**: This query parameter is used to extract only the password of the created Service Principal from the command's output. 

It's useful if you want to capture or use this password immediately after creation.

**--output tsv**: This outputs the result in Tab-Separated Values (TSV) format. It's a simple, unformatted output that's easy to use in scripts.

**Use Case**:

The primary use case for this command is when you need to automate the deployment of applications that use images stored in an Azure Container Registry. 

The Service Principal created by this command can be used in your CI/CD pipelines or from within Kubernetes (as an image pull secret) to authenticate and pull images from the registry.

In summary, this Azure CLI command creates a new Service Principal with limited permissions (only to pull images) scoped to a specific Azure Container Registry. 

We also can verify the service principal we created in Azure Portal. Navigate to 

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_8_Web_API/assets/32194879/45c98d51-1e80-496f-8028-a048bd5ae97d)

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_8_Web_API/assets/32194879/85af68a0-93d9-491c-8863-ee14cce1be95)

## 3. Get the ApplicationID

Use this command for gettin the Application ID, we also can get this value from Azure Portal in **Microsoft Entra ID**

```
az ad sp list --display-name service-principal-name --query "[].appId" --output tsv
```

We have to set the **ApplicationID**: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

## 4. Login in Azure Container Registry ACR

Run this command for log in to Azure ACR

```
docker login myregistryluiscoco1974.azurecr.io -u ApplicationID -p SecretValue
```

As parameter we have to set:

The Azure Continer Registry ACR name: myregistryluiscoco1974.azurecr.io

The **ApplicationID** and the **SecretValue**

## 5. Assign a role "Contributor" to an Azure Active Directory application

The command is used to assign a role (**Contributor** and **acrpush**) to an Azure Active Directory (AAD) application or service principal within Azure

```
az role assignment create --assignee ApplicationID ^
--scope /subscriptions/SubscriptionID/resourceGroups/ResourceGroupName/providers/Microsoft.ContainerRegistry/registries/myregistryluiscoco1974 ^
--role acrpush
```

```
az role assignment create --assignee ApplicationID ^
--scope /subscriptions/SubscriptionID/resourceGroups/ResourceGroupName/providers/Microsoft.ContainerRegistry/registries/myregistryluiscoco1974 ^
--role Contributor
```

## 6. Push the Docker image to Azure Container Registry ACR

Push the Docker image to Azure Container Registry ACR

```
docker push myregistryluiscoco1974.azurecr.io/mywebapi:v1
```

## 7. Run the Docker image

Run the Docker Container 

```
docker run -p 8080:8080 myregistryluiscoco1974.azurecr.io/mywebapi:v1
```

**IMPORTANT NOTE**: for creating the the Web API .NET 8 (including the **DockerFile**, the **deployment.yml**, and **service.yml**) see this repo:

https://github.com/luiscoco/Kubernetes_Deploy_dotNET_8_Web_API


## 8. Create Azure Container Registry (ACR)

### 8.1. Login in to Azure

```
az login
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/7bdf12b0-dfcb-4d3b-ad53-b53510adb19d)

### 8.2. Create a ResourceGroup

```
az group create --name myRG --location westeurope
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/875aa42a-5dcd-44bf-98d7-f6c531a63c17)

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/2cf39089-c990-424a-95da-2dd99183267d)


### 8.3. Create an ACR instance (**Note**: only use **lowercase letters** for the ACR name)

```
az acr create --resource-group myRG --name myregistryluiscoco1974 --sku Basic --location westeurope
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/baa3e3d5-b644-4df4-b8ac-f09cef95ecd3)

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/68deca9e-5d3b-49fb-8cb8-b864186792ba)

### 8.4. Set the **Admin user** in the ACR and copy the username and password

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/d11bcdb0-79dd-4dee-a6a1-448b9fa8784b)

## 9. Build and Push Docker image

### 9.1. Navigate to your project

```
cd path/to/your/project
```

### 9.2. Log in to ACR:

```
az acr login --name myregistryluiscoco1974
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/9cf9cb0d-2d98-40d4-be6c-17aed5b4e0db)

**NOTE**: if you cannot enter with this command run again "az login" and try again running the command "az acr login --name myregistryluiscoco1974" 

### 9.3. Build your Docker image:

```
docker build -t myregistryluiscoco1974.azurecr.io/mywebapi:v1 .
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/fc101461-c21f-4e26-8ff4-94f71b9a36f4)

### 9.4. Push the Image to ACR:

```
docker push myregistryluiscoco1974.azurecr.io/mywebapi:v1
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/2255b4e1-47ba-4666-91d9-40c32ffb2348)

## 10. Create Azure Kubernetes AKS Cluster

```
az aks create --resource-group myRG --name myAKSClusterluiscoco1974 --node-count 1 --enable-addons monitoring --generate-ssh-keys --attach-acr myregistryluiscoco1974 --location westeurope
```

```
az aks create ^
    --resource-group myRG ^
    --name myAKSClusterluiscoco1974 ^
    --node-count 1 ^
    --enable-addons monitoring ^
    --generate-ssh-keys ^
    --attach-acr myregistryluiscoco1974 ^
    --location westeurope
```

## 11. Connect to Azure Kubernetes AKS Cluster

```
az aks get-credentials --resource-group myRG --name myAKSClusterluiscoco1974
```

## 12. Deploy your app to AKS using a Kubernetes manifest files

The **deployment.yaml** and **serivce.yml** files define how your app should run and what image to use. 

Assuming you have a manifest file (e.g., deployment.yaml), use the following command:

```
kubectl apply -f deployment.yml
```

and 

```
kubectl apply -f service.yml
```

Verify the Deployment. To ensure your deployment is running, use:

```
kubectl get deployments
```

For detailed information on the deployed pods, use:

```
kubectl get pods
```

To see the LoadBalancer IP address run this command:

```
kubectl get services
```

## 13. Access to the Web API endpoint

We navigate to the ResourceGroup "myRG", and Then we click in the Kubernetes service "myAKSClusterluiscoco1974":

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/e7d3f41d-c90f-4d3c-b4b1-3ea382a04a2a)

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/0592d64a-9fe1-4c8b-97c5-acc97c73113e)

We copy the Load Balancer External IP:

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/6d36ff4d-cd7c-478d-baa9-eb172d075540)

In the internet web browser we input the Load Balancer External IP followed by the controller name "weatherforecast":

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/1090830c-f2dc-4118-b7a4-92bb40fc7cb8)
