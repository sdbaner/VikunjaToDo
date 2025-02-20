# VikunjaToDo
Vikunja is an open-source, self-hostable ToDo App [Vikunja](https://vikunja.io) .From version 0.23.0 onwards, they have merged the frontend and the api into one component called vikunja and the database [Ref 0.23.0](https://vikunja.io/changelog/whats-new-in-vikunja-0.23.0/) .The latest version is 0.24.6 as of 19th February 2025.

## Problem Statement
You have been given the Vikunja ToDo List application, a microservices-based app that consists of three services: frontend, backend, and database. Your task is to automate the deployment of this application using Kubernetes, optimize the network configuration for performance and reliability, consider the deployment strategy for the database service, and optionally implement Identity and Access Management (IAM) using Keycloak for secure authentication and authorisation.

## Assumption 
Since the latest version has only two components, I am considering an older version 0.21.0 with three components i.e. frontend, api and database.

## Design Approach
I am using AWS and deploying the application on EKS. I have using VPC with subnet and using fargate instead of ec2 instanc eto deploy the application to save some cost.  

```
eksctl create cluster --name myDemoEKS \
--region eu-central-1 \
--fargate
```

Create fargate cluster config
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: fargate-cluster
  region: eu-central-1
nodeGroups:
  - name: ng-1
    instanceType: t3.small
    desiredCapacity: 1
fargateProfiles:
  - name: fp-default
    selectors:
      - namespace: default
      - namespace: kube-system
  - name: fp-dev
    selectors:
      # All workloads in the "dev" Kubernetes namespace matching the following
      # label selectors will be scheduled onto Fargate:
      - namespace: vikunja
        labels:
          env: dev
```

Create fargate profile
```
eksctl create cluster -f myFargateCluster.yaml
```

associate an oidc provider in the cluster
```
eksctl utils associate-iam-oidc-provider --region eu-central-1 --cluster myDemoEKS --approve
```

Delete cluster
```
eksctl delete cluster --name myDemoEKS --region eu-central-1
```
### Why Choose EKS with Fargate?
Amazon EKS simplifies the management of Kubernetes clusters, while Fargate abstracts the underlying infrastructure, allowing you to focus on deploying applications without worrying about the servers. Key benefits include:

Scalability: Automatically scales compute resources based on your application’s needs.
Security: Isolates each pod, enhancing security and reducing attack surfaces.
Cost Efficiency: Pay only for the resources your applications use, avoiding the need for over-provisioning.


## Documentation

- [Network considerations](README-NetworkConsiderations.md)
- [Security Considerations](README-SecurityConsiderations.md)
- [Database Deployment](README-DatabaseDeployment.md)
- [CI CD](README-CiCd.md)
- [Keycloak](README-Keycloak.md)
- [Observabilty](README-Observabilty.md)
- [Secrets Mannagement](README-Secrets.md) 



Network Design

Deploy Amazon EKS cluster with Fargate-Linux nodes through eksctl


## TODO

1. Create infrastructure using terraform.

Adding GitHub Actions steps to build and push Docker images to DockerHub
Creating a Helm chart for the Kubernetes deployment
Exploring the Helm chart structure and values.yaml configuration
Automating docker image tag update to be picked up by ArgoCD
Setting up ArgoCD for GitOps: Deploying ArgoCD with Helm using Terraform
Accessing ArgoCD dashboard and logging in with default credentials
Writing the ArgoCD application YAML (argocd-app.yaml) to link the Helm chart
Configuring ArgoCD to monitor GitHub repository using a Personal Access Token (PAT)
Deploying the ArgoCD application and enabling auto-sync
Testing the pipeline with a Python app update
Accessing the updated Python app using port forwarding
Best practices for CI/CD and GitOps pipelines in production environments
