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

SysAdmin
CodeBeamer is shipped with a default system administrator user with the name bond and 007 password. 
https://codebeamer.com/cb/wiki/86294#section-Change+System+Administrator%27s+Name+and+Password

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

V2 - AKS
az aks install-cli
set PATH=%PATH%;"C:\Users\laurean.gheorghiu\.azure-kubectl" 
az aks get-credentials --resource-group codebeamer-lau-rg --name codebeamer-aks
kubectl get nodes

Create a storage class
kubectl apply -f azure-file-sc.yaml

Create a persistent volume claim
kubectl apply -f azure-file-pvc.yaml

create azure-secret.yaml
kubectl apply -f azure-secret.yaml

kubectl get storageclasses
create deployment.yaml
kubectl apply -f deployment.yaml

V3
kompose convert --out ./k8s
From:
volumes:
  codebeamer-db-data:
  codebeamer-app-logo:
  codebeamer-app-repository-docs:
  codebeamer-app-repository-search:
  codebeamer-app-logs:
  codebeamer-app-tmp:
To:
codebeamer-app-claim0               - kind: PersistentVolumeClaim

codebeamer-db-data                  - kind: PersistentVolumeClaim
codebeamer-app-logo                 - kind: PersistentVolumeClaim
codebeamer-app-repository-docs      - kind: PersistentVolumeClaim
codebeamer-app-repository-search    - kind: PersistentVolumeClaim
codebeamer-app-logs                 - kind: PersistentVolumeClaim
codebeamer-app-tmp                  - kind: PersistentVolumeClaim

codebeamer-app-deployment
codebeamer-app-service
codebeamer-db-deployment


Change in codebeamer-app-service
from
status:
  loadBalancer: {}
to
type: LoadBalancer

az aks get-credentials --resource-group codebeamer-lau-rg --name codebeamer-aks
kompose convert --out ./k8s
docker-compose build
kubectl apply -f k8s
kubectl delete -f k8s
kubectl get svc codebeamer-app

kubectl get nodes
kubectl get services
kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
codebeamer-app-6bf8b7c5f5-zq4ps   1/1     Running   0          14m
codebeamer-db-7cff855876-xd75j    1/1     Running   0          14m

kubectl logs codebeamer-app-6bf8b7c5f5-zq4ps

kubectl create -f codebeamer-app-configmap.yaml
kubectl create configmap [cm-name] --from-file=[path-to-file]

# Azure Resources
codebeamer-lau-rg
codebeamerACR

# Encode the certificate file
cat example.crt | base64
[convert]::ToBase64String((Get-Content -path "domain.crt" -Encoding byte))

Edit Secret: codebeamer-app-certificate-secret
domain.crt -> [convert]::ToBase64String((Get-Content -path "domain.p12" -Encoding byte))
domain.key -> U3dxYTEyMzQh

Bash
cat ../certificates/test_certs/domain.p12 | base64 -w 0

# Encode the key file
cat example.key | base64
[convert]::ToBase64String((Get-Content -path "domain.key" -Encoding byte))

# Create a Kubernetes Secret
kubectl create secret generic example-secret \
  --from-file=example.crt=path/to/encoded/certificate.crt \
  --from-file=example.key=path/to/encoded/key.key

kubectl create secret generic codebeamer-app-certificate-secret --from-file=domain.crt=C:\Users\laurean.gheorghiu\work\certificates\test_certs\domain.crt.b64.txt --from-file=domain.key=C:\Users\laurean.gheorghiu\work\certificates\test_certs\domain.key.b64.txt

# View a Kubernetes Secret
kubectl get secrets
kubectl get secret <secret-name> -o yaml

# Export secret to yaml file
kubectl get secret codebeamer-app-certificate-secret -o yaml > codebeamer-app-certificate-secret.yaml

# Changes in codebeamer-app-deployment
- name: TOMCAT_CONNECTOR_KEYSTORE_FILE
  value: /home/appuser/ssl/keystore.p12
- name: TOMCAT_CONNECTOR_KEYSTORE_PASS
  value: Swqa1234!

# Forward port to pod
- kubectl port-forward codebeamer-db-68c9bb4d7d-8r6zq 3306 

# Attach to pod
- kubectl exec -it codebeamer-app-65c64ff999-4tdbd -n default -- bash

# Check the persistance on host node
- eg /home/appuser/codebeamer/repository/ directory for laurean.txt

# Test CodeBeamer is working
- curl -I http://localhost:9000