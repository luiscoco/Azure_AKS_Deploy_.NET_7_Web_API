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

Create a ResourceGroup:

```
az group create --name myRG --location westeurope
```

Create an ACR instance:

```
az acr create --resource-group myRG --name myRegistryluiscoco1974 --sku Basic
```

## 3. Build and Push Docker image








