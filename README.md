# How to deploy to Azure Kubernetes AKS a .NET7 Web API

## 1. Prerequisites

Install **kubectl** command in Windows: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

Download and Install Docker Desktop: https://docs.docker.com/desktop/install/windows-install/

Install Azure CLI: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows

## 2. Create Azure Container Registry (ACR)

Login in to Azure:

```
az login
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/7bdf12b0-dfcb-4d3b-ad53-b53510adb19d)

Create a ResourceGroup:

```
az group create --name myRG --location westeurope
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/875aa42a-5dcd-44bf-98d7-f6c531a63c17)

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/2cf39089-c990-424a-95da-2dd99183267d)


Create an ACR instance (**Note**: only use **lowercase letters** for the ACR name):

```
az acr create --resource-group myRG --name myregistryluiscoco1974 --sku Basic --location westeurope
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/baa3e3d5-b644-4df4-b8ac-f09cef95ecd3)

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/68deca9e-5d3b-49fb-8cb8-b864186792ba)

Set the **Admin user** in the ACR and copy the username and password:

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/d11bcdb0-79dd-4dee-a6a1-448b9fa8784b)

## 3. Build and Push Docker image

Navigate to your project

```
cd path/to/your/project
```

Log in to ACR:

```
az acr login --name myregistryluiscoco1974
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/9cf9cb0d-2d98-40d4-be6c-17aed5b4e0db)

**NOTE**: if you cannot enter with this command run again "az login" and try again running the command "az acr login --name myregistryluiscoco1974" 

Build your Docker image:

```
docker build -t myregistryluiscoco1974.azurecr.io/mywebapi:v1 .
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/fc101461-c21f-4e26-8ff4-94f71b9a36f4)

Push the Image to ACR:

```
docker push myregistryluiscoco1974.azurecr.io/mywebapi:v1
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/2255b4e1-47ba-4666-91d9-40c32ffb2348)

## 4. Create Azure Kubernetes AKS Cluster

```
az aks create --resource-group myRG --name myAKSClusterluiscoco1974 --node-count 1 --enable-addons monitoring --generate--ssh-keys --attach-acr myregistryluiscoco1974
```





