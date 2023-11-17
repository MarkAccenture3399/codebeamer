# Install Docker locally

- Run multi-container application locally to make sure both codebeamer and the db work. 
- Edit the sample docker-compose.yaml file from here (https://codebeamer.com/cb/wiki/18076086)
to add appropiate certificates and build the container images.
- Check the logs in Docker to make sure containers have started up and are ready to handle requests
- docker-compose up --build -d

### Check codebeamer instance
- https://localhost:9000/

### Stop the application and remove the containers.
- docker-compose down

### See the created images:
- docker images

### Tag the images
- docker tag intland/mysql:debian-8.0.23-utf8mb4 codebeameracr.azurecr.io/codebeamer-db
- docker tag intland/codebeamer:21.09-lts codebeameracr.azurecr.io/codebeamer-app

### Install or update Azure CLI using CMD (admin privileges) 
- az --version
- az --upgrade

### Create a resource group
- az group create --name codebeamer-lau-rg --location westeurope

### Create an Azure container registry 
- The container registry name must be unique within Azure, and contain 5-50 alphanumeric characters.
- az acr create --resource-group codebeamer-lau-rg --name codebeamerACR --sku Basic

### Log in to container registry
- az acr login --name codebeamerACR

- If tenant not found
- az login --tenant <tenant_id>

### Push images to container registry
- docker-compose push

### Notes
- CodeBeamer is shipped with a default system administrator user (SysAdmin) with the name bond and 007 password. 
https://codebeamer.com/cb/wiki/86294#section-Change+System+Administrator%27s+Name+and+Password

# Deploy Codebeamer to Azure

## 0. Azure Resources
- codebeamer-lau-rg
- codebeamerACR
- codebeamerstorageaccount
- codebeamer-fs

## 1. Using Azure Container Instances (ACI)
### Create Azure context. 
- This context associates Docker with an Azure subscription and resource group 
- docker login azure --tenant-id 2f075e73-4163-45e2-a1a9-b702c43b8701
- docker context create aci codebeamer_aci_context

### View current context
- docker context ls

### Deploy application to Azure Container Instances (ACI)
- docker context use codebeamer_aci_context

### Use the alternate _docker-compose.yaml file
- docker compose up

- Note: Docker Compose's integration for ECS and ACI will be retired in November 2023. Learn more: 
- https://docs.docker.com/go/compose-ecs-eol/

## 2. Using Azure Kubernetes Service (AKS)

### Install AKS CLI
- az aks install-cli
- set PATH=%PATH%;"C:\Users\laurean.gheorghiu\.azure-kubectl" 

### Link CLI to AKS
- az aks get-credentials --resource-group codebeamer-lau-rg --name codebeamer-aks

### Get AKS nodes
- kubectl get nodes

### Get AKS services
- kubectl get services

### Get AKS pods
- kubectl get pods

### Create a storage class
- kubectl apply -f azure-file-sc.yaml

### Create a persistent volume claim
- kubectl apply -f azure-file-pvc.yaml

### Create Azure secret
- kubectl apply -f azure-secret.yaml

### Retrieve storage classes
- kubectl get storageclasses

### Create deployment.yaml
- kubectl apply -f deployment.yaml

### Convert docker-compose.yaml to AKS deployment yaml files
- Go to docker-compose location
- kompose convert --out ./k8s
- Will generate a k8s folder will all the yaml files
- Manual editing of the files is required
- eg. pv, pvc, deployment, service, but no ingress

### Make service externally available
- Change in codebeamer-app-service from
status:
  loadBalancer: {}
to
type: LoadBalancer

### Reconnect to AKS
- az aks get-credentials --resource-group codebeamer-lau-rg --name codebeamer-aks

- kompose convert --out ./k8s
- docker-compose build
- kubectl apply -f k8s
- kubectl delete -f k8s
- kubectl get svc codebeamer-app

- kubectl get nodes
- kubectl get services
- kubectl get pods

- kubectl logs codebeamer-app-6bf8b7c5f5-zq4ps

- kubectl create -f codebeamer-app-configmap.yaml
- kubectl create configmap [cm-name] --from-file=[path-to-file]

### Encode the certificate file
- Using powershell:
- -  cat example.crt | base64
[convert]::ToBase64String((Get-Content -path "domain.crt" -Encoding byte))

- Edit Secret: codebeamer-app-certificate-secret
domain.crt 
- Using powershell:
- -  [convert]::ToBase64String((Get-Content -path "domain.p12" -Encoding byte))
- Using bash:
- - cat ../certificates/test_certs/domain.p12 | base64 -w 0
- eg. domain.key -> U3dxYTEyMzQh

### Encode the key file
- Using powershell:
- - cat example.key | base64
[convert]::ToBase64String((Get-Content -path "domain.key" -Encoding byte))

### Create a Kubernetes Secret
- kubectl create secret generic example-secret \
  --from-file=example.crt=path/to/encoded/certificate.crt \
  --from-file=example.key=path/to/encoded/key.key

- kubectl create secret generic codebeamer-app-certificate-secret --from-file=domain.crt=C:\Users\laurean.gheorghiu\work\certificates\test_certs\domain.crt.b64.txt --from-file=domain.key=C:\Users\laurean.gheorghiu\work\certificates\test_certs\domain.key.b64.txt

### View a Kubernetes Secret
- kubectl get secrets
- kubectl get secret <secret-name> -o yaml

### Export secret to yaml file
- kubectl get secret codebeamer-app-certificate-secret -o yaml > codebeamer-app-certificate-secret.yaml

### Changes in codebeamer-app-deployment
- name: TOMCAT_CONNECTOR_KEYSTORE_FILE
  value: /home/appuser/ssl/keystore.p12
- name: TOMCAT_CONNECTOR_KEYSTORE_PASS
  value: Swqa1234!

### Forward port to pod
- kubectl port-forward codebeamer-db-68c9bb4d7d-8r6zq 3306 

### Attach to pod
- kubectl exec -it codebeamer-app-65c64ff999-4tdbd -n default -- bash

### Check the persistance on host node
- eg /home/appuser/codebeamer/repository/ directory for laurean.txt

### Test CodeBeamer is working from inside the pod
- curl -I http://localhost:8090

### View nodes in your cluster along with their resource utilization
- kubectl top nodes
- - DS2_v2 had 7Gb RAM and crashed at login, now using D4s that has 16Gb RAM, using 13Gb
- Codebeamer usage 

NAME | CPU(cores) | CPU% | MEMORY(bytes) | MEMORY%
--- | --- | ---| --- | ---
aks-agentpool-36809342-vmss000000 | 215m | 5%| 12490Mi | 99%
aks-agentpool-36809342-vmss000000 | 1281m | 33% | 4281Mi | 34%
aks-agentpool-36809342-vmss000000 | 210m  | 5%  |  3853Mi | 30%
aks-agentpool-36809342-vmss000000 | 1434m | 37% | 809Mi   | 54% 


