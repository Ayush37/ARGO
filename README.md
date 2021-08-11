# ARGO
## ARGO CD and ARGO Workflow demo (Some ascpects of the code is reused from AWS and crossplane repos)

To setup crossplane on an existing EKS or local K8's cluster

 ``` 
kubectl create namespace crossplane-system
helm repo add crossplane-alpha https://charts.crossplane.io/alpha
helm install crossplane --namespace crossplane-system crossplane-alpha/crossplane --version 0.8.0 --set clusterStacks.aws.deploy=true --set clusterStacks.aws.version=v0.6.0 --disable-openapi-validation 

```
Make sure all the pods are functional

![image](https://user-images.githubusercontent.com/19201225/128902058-d5ff708d-bb6a-480b-a108-e6bdecba839b.png)

Now install ARGOCD

```
create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```
port-forward from your K8s cluster to fetch the UI

```
kubectl port-forward svc/argocd-server -n argocd 8080:443

```
Navigate to localhost:8080

Fetch the admin password from a secret

```
kubectl get secret argocd-initial-admin-secret -n argocd -o="jsonpath={.data.password}" | base64 -d
```
Login using

Username: admin
Password: <password>

  UI is as (Please ignore my already created applications)
  ![image](https://user-images.githubusercontent.com/19201225/128902483-4ecc8dba-e531-4672-87df-9d6f2f41f8ec.png)

  Now create the infra from this github repo
  
  Click on new application
  
  ```
  apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: infra-setup
spec:
  destination:
    name: ''
    namespace: wordpress-app
    server: 'https://kubernetes.default.svc'
  source:
    path: pathway/aws-argo/infra
    repoURL: 'https://github.com/Ayush37/ARGO.git'
    targetRevision: HEAD
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: false

 ```
 And click create
  This will create 2 separate EKS clusters on us-east-1 and us-west-2 with 1 node each
  
 ![image](https://user-images.githubusercontent.com/19201225/128902980-a0a25a68-bf3e-49c5-9fc5-8fa0e0d28cf3.png)

  ![image](https://user-images.githubusercontent.com/19201225/128903003-26e775c6-3223-42eb-8d6f-950fe2fc0780.png)
  ![image](https://user-images.githubusercontent.com/19201225/128903042-554bae48-34a0-4b2a-895f-85b734ee220e.png)

  
  After about 15 minutes you should see 2 EKS clusters secret created in the UI
  ![image](https://user-images.githubusercontent.com/19201225/128903126-28c25b22-7899-452f-8d69-d51ed5034943.png)

  ### VOILA 2 EKS clusters created succesfully
  
  ![image](https://user-images.githubusercontent.com/19201225/128903251-6d1774d8-3322-4131-a262-04f1ae69459c.png)

  ![image](https://user-images.githubusercontent.com/19201225/128903277-8fd0348e-08b6-4460-851f-afc48c2c956b.png)

 Now we will install a sample app in each using https://github.com/Ayush37/ARGO/tree/main/pathway/aws-argo/app-1 yamls
 
  
# Deploy sample app across multi-cloud clusters (In this case we are deploying it across 2 AWS zones)

 Wordpress needs php and mysql as basic pre-reqs which are mostly satisfied by our earlier infra deployments (We used a RDSInstanceClass for MySql)
 
Creating these resources causes Crossplane to provision an Amazon RDS instance, obtain the connection information, then inject it into the WordPress application 
that it deploys to our Amazon EKS cluster in us-west-2. 
When this process is complete, you should see a Secret appear in the Argo CD UI that is associated with the MySQLInstance.

Create a similar application and use pathway/aws-argo/app-1 as path:
 
After succesfull creation we should be able to see a Secret appear in the Argo CD UI that is associated with the MySQLInstance.
 
 ![image](https://user-images.githubusercontent.com/19201225/128952607-6002ac06-4795-4353-9862-9ad393590be4.png)

 
Gather its loadbalancer's hostname at the bottom of the YAML manifest.
 
 ![image](https://user-images.githubusercontent.com/19201225/128952588-0082a409-9191-4882-b199-975e970b15e9.png)



