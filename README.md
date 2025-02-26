# VikunjaToDo
Vikunja is an open-source, self-hostable ToDo App [Vikunja](https://vikunja.io) .From version 0.23.0 onwards, they have merged the frontend and the api into one component called vikunja and the database [Ref 0.23.0](https://vikunja.io/changelog/whats-new-in-vikunja-0.23.0/) .The latest version is 0.24.6 as of 19th February 2025.

## Problem Statement
You have been given the Vikunja ToDo List application, a microservices-based app that consists of three services: frontend, backend, and database. Your task is to automate the deployment of this application using Kubernetes, optimize the network configuration for performance and reliability, consider the deployment strategy for the database service, and optionally implement Identity and Access Management (IAM) using Keycloak for secure authentication and authorisation.

## Assumption 
Since the latest version has only two components, I am considering an older version 0.21.0 with three components i.e. frontend, api and database.

## Design Approach
I am using AWS and deploying the application on EKS with managed worker nodes. I have used eksctl to quickly deploy the cluster with a vpc with a public and private subnets. More: [Network considerations](NetworkConsiderations.md)

Vikunja App has three main components i.e. frontend, api and database in the same namespace vikunja. Both the frontend and the api components are exposed on the internet. 
I am going with statefulset database. 

** Advantages with self managed database in kubernetes :**
- low operational cost
- full control
- no network latency as data resides within the cluster

  ![Screen Shot 2025-02-26 at 11 30 36 AM](https://github.com/user-attachments/assets/d69b7868-6bb9-469a-b737-ca31ff19f1f9)

### Flow of request
1. User access the vikunja app at https://www.myvinkunja.app
2. Ideally the domain name "vikunja.app" should be hosted in Cloudflare/Route53. The Route53 then resolves the domain name to the public IP of AWS ALB. Note: This step has not been implemented. This also requires setting up ssl certificates.
3. The AWS ALB then forwards the traffic to the nginx ingress controller in the EKS Cluster.
4. The controller then routes the request to the services frontend and api. 


## Further Documentations







## TODO

1. Create infrastructure using terraform.
2. Cost optimization
3. Try ArgoCD
4. How to secure api
5. Serets managment
6. Keykloak
7. observabilty
