### Introduction

This is example of CI/CD pipeline setup for backend and frontend solution for Azure Container Registry service

### Getting started

#### Requirements

- Basic knowledge of Azure
- Basic knowledge of Github

#### Problem

We have source code in one of our repositories, we need to deploy source code for deployment, pre-release and release scenarios.

#### Solution overviews

- Create necessary branches
- Create cloud resources
- Setup workflows

### Creating necessary branches

<img style="width: 100%" src="docs/assets/images/1.png" alt="Branches image"/>

These branches must be created 

- main
- release/** - created only when releasing
- staging
- dev

## Create cloud resources

In this example we will create resources in azure cloud. I used Azure CLI because it's faster and only one interface to create all necessary resources

Following resources will be created
- 2 ACR ( Azure Container Registry ), one for backend and one for frontend
- 2 ACA ( Azure Container App ), one for backend and one for frontend
- 2 CAE ( Container App Environment ), one for backend and one for frontend
- 2 LAW ( Log Analytics Workspace ), one for backend and one for frontend
 
### Prepare Azure CLI

- Open Azure CLI
- Create storage account if needed
- Choose bash for terminal

### Create resource group

```bash
az group create --name <rg-name> --location <location>
```

In our example it wil be

```bash
az group create --name azure-demo-pipeline --location eastus
```

Explanation :
- create recourse group with name `azure-demo-pipeline` in `eastus` location


### Create service principal

- Create Service Principal ( registered application ) with following command and copy result credentials in json. Those credentials will be used by github actions

```Bash
az ad sp create-for-rbac --name <app-name> --role contributor --scopes /subscriptions/<subscription-id>/resourceGroups/<rg-name> --json-auth --output json
```

In our example it will be 

```Bash
az ad sp create-for-rbac --name github-deploy-action --role contributor --scopes /subscriptions/0091a550-db6c-44bb-8c82-42e826b6f0ff/resourceGroups/azure-demo-pipeline --json-auth --output json
```

Explanation : 
- Create service principal with `contributor` role and `github-deploy-action` name
- For subscription `0091a550-db6c-44bb-8c82-42e826b6f0ff` and resource group `azure-demo-pipeline`
- Display service principal credentials as JSON, copy that credentials and save somewhere

### Create container registries

```bash
az acr create --resource-group <your-resource-group> --name <acr-name> --sku Basic 
```

In our example it will

```Bash
az acr create --resource-group azure-demo-pipeline --name azurepipelinebackend --sku Basic 
```

Explanation
- Create ACR within recourse group `azure-demo-pipeline` with name `azurepipelinebackend`
- 


### Create container app

```Bash
az containerapp up \
  --name <container-name> \
  --resource-group <rg-name> \
  --location <location> \
  --environment <environment-name> \
  --image mcr.microsoft.com/k8se/quickstart:latest \
  --target-port 8080 \
  --ingress 'external' \
  --query configuration.ingress.fqdn
```

In our example it will be

```Bash
az containerapp up \
  --name azure-pipeline-backend-app \
  --resource-group azure-demo-pipeline \
  --location eastus \
  --environment 'azure-pipeline-backend-app-env' \
  --image mcr.microsoft.com/k8se/quickstart:latest \
  --target-port 80 \
  --ingress 'external' \
  --query configuration.ingress.fqdn
```

This will create container app with example image, now need to update ingress port with the one that we will be using










### Create Azure Container Registry

- Go to "Resource Groups" page
- Create new resource and choose subscription and region
- Go to "Azure Container Registry" resource
- Create container registry resource group and region

### Create Azure Container App

<img style="width: 100%" src="docs/assets/images/create-container-app.png" alt="Branches image"/>

- Select resource group
- Enter container app name
- Select region and create new app environment

<img style="width: 100%" src="docs/assets/images/choose-container.png" alt="Branches image"/>

- Uncheck "Use quickstart image"
- Select Azure Container Registry as image source
- Select registry, image and image tag
- Choose consumption allocation

### Setting up workflows

Workflows will be set up for backend and frontend solutions separately

Development - **dev** branch

- any pull request or commit in pull request should trigger workflow to run tests
- any push should trigger workflow to build container and push to registry for development environment with "dev" tag

Pre-Release - **staging** branch

- any pull request should trigger workflow to run tests
- any push should trigger workflow to build container and push to registry for staging environment with "staging" tag

Release - **release/** branch

- any pull request should trigger workflow to run tests
- any push should trigger workflow to build container and push to registry for staging environment with "vx.x.x" tag

