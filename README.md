# How to deploy SpringBoot WebAPI to Azure Kubernetes (AKS)

## Summary

1. Create a SpringBoot WebAPI in VSCode: https://github.com/luiscoco/SpringBoot_Sample2-created-WebAPI-with-VSCode

2. Create the Dockerfile

```
# Start with a base image containing Java runtime
FROM openjdk:11-jdk-slim as build

# Add Maintainer Info
LABEL maintainer="your_email@example.com"

# Add a volume pointing to /tmp
VOLUME /tmp

# Make port 8080 available to the world outside this container
EXPOSE 8080

# The application's jar file
ARG JAR_FILE=target/demoapi-0.0.1-SNAPSHOT.jar

# Add the application's jar to the container
ADD ${JAR_FILE} demoapi.jar

# Run the jar file
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/demoapi.jar"]
```

3. Create Azure Container Registry ACR

```
az acr create --name myAzureContainerRegistry --resource-group myResourceGroup --sku Basic
```

4. Create a Service Principal

```
az ad sp create-for-rbac ^
    --name service-principal-name ^
    --scopes /subscriptions/SubscriptionID/resourceGroups/ResourceGroupName/providers/Microsoft.ContainerRegistry/registries/myregistryluiscoco1974 ^
    --role acrpull ^
    --query "password" ^
    --output tsv
```

5. Docker login

```
docker login myregistryluiscoco1974.azurecr.io -u ApplicationID -p SecretValue
```

6. Build your Docker image:

```
docker build -t myregistryluiscoco1974.azurecr.io/mywebapi:v1 .
```

7. Push the Image to ACR:

```
docker push myregistryluiscoco1974.azurecr.io/mywebapi:v1
```

8. Assign a role "Contributor" to an Azure Active Directory application

```
az role assignment create --assignee ApplicationID ^
--scope /subscriptions/SubscriptionID/resourceGroups/ResourceGroupName/providers/Microsoft.ContainerRegistry/registries/myregistryluiscoco1974 ^
--role Contributor
```

9. Push the Docker image to Azure ACR

```
docker push myregistryluiscoco1974.azurecr.io/mywebapi:v1
```

10. Run the Docker container in local

```
docker run -p 8080:8080 myregistryluiscoco1974.azurecr.io/mywebapi:v1
```

11. Create the Kubernetes manifest files (deployment.yml and service.yml)

**deployment.yml** 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demoapi-deployment
spec:
  replicas: 1  # The number of Pods to run
  selector:
    matchLabels:
      app: demoapi
  template:
    metadata:
      labels:
        app: demoapi
    spec:
      containers:
        - name: demoapi
          image: myregistryluiscoco1974.azurecr.io/mywebapi:v1  # Replace with your Docker image, e.g., "username/demoapi:latest"
          ports:
            - containerPort: 8080
```

**service.yml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: demoapi-service
spec:
  type: LoadBalancer  # Exposes the service externally using a load balancer
  selector:
    app: demoapi
  ports:
    - protocol: TCP
      port: 80  # The port the load balancer listens on
      targetPort: 8080  # The port the container accepts traffic on
```

12. Create Azure Kubernetes cluster (AKS)

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

13. Deploy the SpringBoot WebAPI to AKS:

```
kubectl apply -f deployment.yml
```

and 

```
kubectl apply -f service.yml
```

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

This command is used in Azure to create a Service Principal with the Azure Container Registry Pull (**acrpull**) role. 

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

The **deployment.yaml** and **serivce.yml** files define how your app should run and what image to use

**deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demoapi-deployment
spec:
  replicas: 1  # The number of Pods to run
  selector:
    matchLabels:
      app: demoapi
  template:
    metadata:
      labels:
        app: demoapi
    spec:
      containers:
        - name: demoapi
          image: myregistryluiscoco1974.azurecr.io/mywebapi:v1  # Replace with your Docker image, e.g., "username/demoapi:latest"
          ports:
            - containerPort: 8080
```

**serivce.yml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: demoapi-service
spec:
  type: LoadBalancer  # Exposes the service externally using a load balancer
  selector:
    app: demoapi
  ports:
    - protocol: TCP
      port: 80  # The port the load balancer listens on
      targetPort: 8080  # The port the container accepts traffic on
```

Assuming you have a manifest file (e.g., deployment.yaml), use the following command to deploy the Kubernetes cluster:

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

We navigate to the **ResourceGroup** "myRG", and Then we click in the **Kubernetes** service "myAKSClusterluiscoco1974":

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_8_Web_API/assets/32194879/8aed65f5-f2cb-4924-ae78-6a5251d34110)

We copy the **Load Balancer External IP**:

![image](https://github.com/luiscoco/Azure_AKS_Deploy_.NET_8_Web_API/assets/32194879/eeb2c01e-b070-416c-b90c-8ba8ab1a5703)
