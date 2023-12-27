# How to deploy to Azure Kubernetes AKS a Web API .NET 8

https://github.com/luiscoco/SpringBoot_Sample5-deploy-WebAPI-to-Azure_Kubernetes_AKS/commits/main

## 1. Prerequisites

Install **Kubectl** command in Windows: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

Download and Install **Docker Desktop**: https://docs.docker.com/desktop/install/windows-install/

Install **Azure CLI**: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows

## 2. Create Azure Container Registry (ACR)

```
az account set --subscription "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```

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

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_8_Web_API/assets/32194879/7241ba40-7fd9-4ee7-80e2-fefbcd6867b1)

## 3. Build and Push Docker image

### 3.1. Navigate to your project

```
cd /to/your/project
```

### 3.2. Log in to ACR:

```
az acr login --name myregistryluiscoco1974
```

**NOTE**: if you cannot enter with this command run again "az login" and try again running the command "az acr login --name myregistryluiscoco1974"

### 3.3. Build your Docker image:

Create a Dockerfile image:

```
#See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER app
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY [".NET 8 Web API Kubernetes.csproj", "."]
RUN dotnet restore "./././.NET 8 Web API Kubernetes.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "./.NET 8 Web API Kubernetes.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./.NET 8 Web API Kubernetes.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", ".NET 8 Web API Kubernetes.dll"]
```

```
docker build -t myregistryluiscoco1974.azurecr.io/mywebapi:v1 .
```

### 3.4. Push the Image to ACR:

```
docker push myregistryluiscoco1974.azurecr.io/mywebapi:v1
```

## 4. Create Azure Kubernetes AKS Cluster

```
az aks create --resource-group myRG --name myAKSClusterluiscoco1974 --node-count 1 --enable-addons monitoring --generate-ssh-keys --attach-acr myregistryluiscoco1974.azurecr.io
```

## 5. Connect to Azure Kubernetes AKS Cluster

```
az aks get-credentials --resource-group myRG --name myAKSClusterluiscoco1974
```

## 6. Access to the Web API endpoint

We navigate to the ResourceGroup "myRG", and Then we click in the Kubernetes service "myAKSClusterluiscoco1974":



We copy the Load Balancer External IP:



In the internet web browser we input the Load Balancer External IP followed by the controller name "weatherforecast":




