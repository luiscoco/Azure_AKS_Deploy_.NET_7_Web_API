# How to deploy to Azure Kubernetes AKS a Web API .NET 8

IMPORTANT NOTE!: for creating the the Web API .NET 8 (including the **DockerFile**, the **deployment.yml**, and **service.yml**) see this repo:

https://github.com/luiscoco/Kubernetes_Deploy_dotNET_8_Web_API

## 1. Prerequisites

Install **kubectl** command in Windows: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

Download and Install Docker Desktop: https://docs.docker.com/desktop/install/windows-install/

Install Azure CLI: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows

## 2. Create Azure Container Registry (ACR)

### 2.1. Login in to Azure:

```
az login
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/7bdf12b0-dfcb-4d3b-ad53-b53510adb19d)

### 2.2. Create a ResourceGroup:

```
az group create --name myRG --location westeurope
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/875aa42a-5dcd-44bf-98d7-f6c531a63c17)

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/2cf39089-c990-424a-95da-2dd99183267d)


### 2.3. Create an ACR instance (**Note**: only use **lowercase letters** for the ACR name):

```
az acr create --resource-group myRG --name myregistryluiscoco1974 --sku Basic --location westeurope
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/baa3e3d5-b644-4df4-b8ac-f09cef95ecd3)

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/68deca9e-5d3b-49fb-8cb8-b864186792ba)

### 2.4. Set the **Admin user** in the ACR and copy the username and password:

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/d11bcdb0-79dd-4dee-a6a1-448b9fa8784b)

## 3. Build and Push Docker image

### 3.1. Navigate to your project

```
cd path/to/your/project
```

### 3.2. Log in to ACR:

```
az acr login --name myregistryluiscoco1974
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/9cf9cb0d-2d98-40d4-be6c-17aed5b4e0db)

**NOTE**: if you cannot enter with this command run again "az login" and try again running the command "az acr login --name myregistryluiscoco1974" 

### 3.3. Build your Docker image:

```
docker build -t myregistryluiscoco1974.azurecr.io/mywebapi:v1 .
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/fc101461-c21f-4e26-8ff4-94f71b9a36f4)

### 3.4. Push the Image to ACR:

```
docker push myregistryluiscoco1974.azurecr.io/mywebapi:v1
```

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/2255b4e1-47ba-4666-91d9-40c32ffb2348)

## 4. Create Azure Kubernetes AKS Cluster

```
az aks create --resource-group myRG --name myAKSClusterluiscoco1974 --node-count 1 --enable-addons monitoring --generate-ssh-keys --attach-acr myregistryluiscoco1974 --location westeurope
```

## 5. Connect to Azure Kubernetes AKS Cluster

```
az aks get-credentials --resource-group myRG --name myAKSClusterluiscoco1974
```

## 6. Deploy your app to AKS using a Kubernetes manifest files

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

## 7. Access to the Web API endpoint

We navigate to the ResourceGroup "myRG", and Then we click in the Kubernetes service "myAKSClusterluiscoco1974":

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/e7d3f41d-c90f-4d3c-b4b1-3ea382a04a2a)

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/0592d64a-9fe1-4c8b-97c5-acc97c73113e)

We copy the Load Balancer External IP:

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/6d36ff4d-cd7c-478d-baa9-eb172d075540)

In the internet web browser we input the Load Balancer External IP followed by the controller name "weatherforecast":

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_7_Web_API/assets/32194879/1090830c-f2dc-4118-b7a4-92bb40fc7cb8)
