# 1 Built the container and test it
```bash
docker build -t <your-image-name> .
docker run -p 8000:8000 <your-image-name>
```

# 2 Azure Preparation

## 2.1 Login to Azure using terminal
```bash
az login
```
New browser window is opened. Just login using your credentials.

### 2.1.1 (Optional) Select subscription
```bash
az account set --subscription "Your-Subscription-Name-Or-ID"
```

## 2.2 (Optional) Create an Azure Container Registry (ACR)
```bash
az acr create --resource-group myResourceGroup --name <RegistryName> --sku Basic
```

## 2.3 Login to your ACR

my repo: `mlregistry1987.azurecr.io`

```bash
az acr login --name <RegistryName>
```

Before you can push your image to the Azure Container Registry, you need to tag it with the full ACR login server name. First, find the login server name:
```bash
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

# 3 Tag your Docker image
```bash
docker tag ocr-api <acrLoginServer>/ocr-api:v1
```


# 4 Docker Push
```bash
docker push <acrLoginServer>/<your_image_name>:<your_tag>
```

# 5 Create Azure Container Instance
Can be done in Azure Portal or you can use this code in Terminal:

```bash
az container create --resource-group myResourceGroup --name ocrapi-container --image <acrLoginServer>/ocr-api:v1 --cpu 1 --memory 1Gi --registry-login-server <acrLoginServer> --registry-username <acrUsername> --registry-password <acrPassword> --dns-name-label ocrapi-yoursubdomain --ports 8000
```


## 5.1 Access the Application
```bash
az container show --resource-group myResourceGroup --name ocrapi-container --query "{FQDN:ipAddress.fqdn}" --out table
```


## 5.2 Monitor Logs
If you need to see the logs of your container instance:
```bash
az container logs --resource-group myResourceGroup --name ocrapi-container

```

# 6 Clean Up (if needed)
If you're done testing and want to delete resources to avoid charges:
```bash
az container delete --resource-group myResourceGroup --name ocrapi-container
az acr delete --name <RegistryName>
az group delete --name myResourceGroup
```
