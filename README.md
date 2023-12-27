# How to deploy to Azure Kubernetes AKS a Web API .NET 8

## 1. Prerequisites

Install **Kubectl** command in Windows: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

Download and Install **Docker Desktop**: https://docs.docker.com/desktop/install/windows-install/

Install **Azure CLI**: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows

## 2. Create Azure Container Registry (ACR)

### 2.1. Login in to Azure:

```
az login
```

### 2.2. Create a ResourceGroup:

```
az group create --name myRG --location westeurope
```

### 2.3. Create an ACR instance (Note: only use lowercase letters for the ACR name):

```
az acr create --resource-group myRG --name myregistryluiscoco1974 --sku Basic --location westeurope
```

### 2.4. Set the Admin user in the ACR and copy the username and password:




## 3. Build and Push Docker image

### 3.1. Navigate to your project

```
cd /to/your/project
```

### 3.2. Log in to ACR:

```
az acr login --name myregistryluiscoco1974
```


