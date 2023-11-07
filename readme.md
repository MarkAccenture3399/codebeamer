Install Docker locally

Run multi-container application locally to make sure both codebeamer and the db work. 
Edit the sample docker-compose.yaml file from here (https://codebeamer.com/cb/wiki/18076086)
to add appropiate certificates and build the container images. Check the logs in Docker to make sure containers have started up and are ready to handle requests
docker-compose up --build -d

Check codebeamer instance
https://localhost:9000/

After trying the local application, run the following command to stop the application and remove the containers.
docker-compose down

When completed, use the following command to see the created images:
docker images

Tag the images
docker tag intland/mysql:debian-8.0.23-utf8mb4 codebeameracr.azurecr.io/codebeamer-db
docker tag intland/codebeamer:21.09-lts codebeameracr.azurecr.io/codebeamer-app

Install or update Azure CLI using CMD with Administrator privileges
az --version
az --upgrade

Create a resource group
az group create --name codebeamer-lau-rg --location westeurope

Create an Azure container registry. The container registry name must be unique within Azure, and contain 5-50 alphanumeric characters.
az acr create --resource-group codebeamer-lau-rg --name codebeamerACR --sku Basic

Log in to container registry
az acr login --name codebeamerACR

If tenant not found
az login --tenant <tenant_id>

Push images to container registry
docker-compose push

Deploy Codebeamer instances to Azure

V1 - ACI
Create Azure context. This context associates Docker with an Azure subscription and resource group 
docker login azure --tenant-id 2f075e73-4163-45e2-a1a9-b702c43b8701
docker context create aci codebeamer_aci_context

View current context
docker context ls

Deploy application to Azure Container Instances
docker context use codebeamer_aci_context

Use _docker-compose.yaml file
docker compose up

