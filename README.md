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

Advantages with self managed database in kubernetes :
- low operational cost
- full control
- no network latency as data resides within the cluster

  ![Screen Shot 2025-02-26 at 11 30 36 AM](https://github.com/user-attachments/assets/d69b7868-6bb9-469a-b737-ca31ff19f1f9)


### Flow of request
1. User access the vikunja app at https://www.myvinkunja.app
2. Ideally the domain name "myvikunja.app" should be hosted in Cloudflare/Route53. The Route53 then resolves the domain name to the public IP of AWS ALB.SSL en sures secure https communication between users and ALB.
Note: This step has not been implemented.
3. The AWS ALB then forwards the request to ALB Ingress Controller.
4. It forwards the traffic to nginx ingress controller hosted in EKS
5. Based on path, it directs the traffic to the services frontend or api. 
6. The frontend/api service processes the request and sends a response to the nginx ingress controller.
7. It forwards the response to ALB.
8. ALB then sends it back to user over https.


## Further Documentations
[Network considerations](NetworkConsiderations.md)
[Database Migration](DatabaseMigration.md)
[Observabilty](Observabilty.md)
[CI CD](CICD.md)
[Keycloak](Keycloak.md)






## TODO

1. Create infrastructure using terraform.
2. Improvements - rbac, node affinity, network policy for postgres
3. Secure api endpoint with authorized requests
4. Secrets management

