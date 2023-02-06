# IONOS Container Registry - How To
Sometimes when you work with local docker container images, you want them to be accessible and deployed elsewhere. To do so, you will need a registry service. You can use a public one like [Docker Hub](https://hub.docker.com/) or you can use IONOS's registry service.

This document explains how you can use and deploy any container image from your computer to the cloud using the IONOS's registry service. You can find more information for this service [here](https://docs.ionos.com/cloud/managed-services/container-registry).

## Requirements
You will need the following:
- IONOS account;
- Docker;
- Linux shell or PowerShell;

## Registry service creation
In order to deploy the container image into the IONOS's K8S, it is required to create a registry service. To do so, follow the instructions:
1) Access the [DataCenter Designer](https://dcd.ionos.com)
2) Go to `Container Registry Manager` and add a registry;
3) Create a token filling the mandatory fields and click `Save`:
- Name: you will use this field as the user name for your connection;
- Scopes:
    - Name: use exactly the same name that you used in the docker deployment. For instance: edc-consumer. You can also use a `*`;
    - Select Action: select the allowed tasks to be used for this connection;
4) You will be prompted with informations related to the new connection. Copy/paste them because you will loose access to them!

## Pushing the image
1) To push your image just do the following:

```bash
docker login -u="<SCOPE NAME>" -p="<PASSWORD GENERATED>" <YOUR REGISTRY FQDN>
docker push <YOUR REGISTRY FQDN>/<YOUR CONTAINER NAME>:<VERSION> 
```

Example:
```bash
docker login -u="pjctoken" -p="sdfdsgdfryt45645435tgrdfgc" pjccontainerregistry.cr.de-fra.ionos.com
docker push pjccontainerregistry.cr.de-fra.ionos.com/edc-consumer:latest
```
2) Using the DataCenter Designer and accessing your registry service you will see a new respository containing your docker container image. 

## Deploying into an IONOS Kubernetes
After having the docker image uploaded into your registry, you can now deploy it using the following steps:
1) Create a secret: required by the Kubernetes to access your registry;
```bash
kubectl create secret docker-registry <SECRET NAME> --docker-server=<YOUR REGISTRY FQDN> --docker-username=<SCOPE NAME> --docker-password=<PASSWORD GENERATED> --docker-email=<YOUR EMAIL>
```

Example:
```bash
kubectl create secret docker-registry pjcsecret --docker-server=pjccontainerregistry.cr.de-fra.ionos.com --docker-username=pjctoken --docker-password=sdfdsgdfryt45645435tgrdfgc --docker-email=paulo.cabrita@XXXXXXX
```

2) Deploying the connector: there's several ways to do this but we will use a simple one;
```bash
kubectl run <App Name> --image=<YOUR REGISTRY FQDN>/<YOUR CONTAINER NAME>:<VERSION> --overrides='{ "apiVersion": "v1", "spec": { "imagePullSecrets": [{"name": "<YOUR SECRET NAME>"}] } }'
```

Example:
```bash
kubectl run my-demo --image=pjccontainerregistry.cr.de-fra.ionos.com/edc-consumer:latest --overrides='{ "apiVersion": "v1", "spec": { "imagePullSecrets": [{"name": "pjcsecret"}] } }'
```

3) Depending on your context, you can now configure the deployed container access with: port-forward, ingress, loadbalancer or nodeport.
