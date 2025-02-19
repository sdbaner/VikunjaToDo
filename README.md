# VikunjaToDo
Vikunja is an open-source, self-hostable ToDo App [Vikunja](https://vikunja.io) .From version 0.23.0 onwards, they have merged the frontend and the api into one component called vikunja and the database [Ref 0.23.0](https://vikunja.io/changelog/whats-new-in-vikunja-0.23.0/) .The latest version is 0.24.6 as of 19th February 2025.

## Problem Statement
You have been given the Vikunja ToDo List application, a microservices-based app that consists of three services: frontend, backend, and database. Your task is to automate the deployment of this application using Kubernetes, optimize the network configuration for performance and reliability, consider the deployment strategy for the database service, and optionally implement Identity and Access Management (IAM) using Keycloak for secure authentication and authorisation.

## Assumption 
Since the latest version has only two components, I am considering an older version 0.21.0 with three components i.e. frontend, api and database.

## Design Approach
I am using AWS and deploying the application on EKS. I have using VPC with subnet and using fargate instead of ec2 instanc eto deploy the application to save some cost.  


## Documentation

- [Network considerations](README-NetworkConsiderations.md)
- [Security Considerations](README-SecurityConsiderations.md)
- [Database Deployment](README-DatabaseDeployment.md)
- [CI CD](README-CiCd.md)
- [Keycloak](README-Keycloak.md)
- [Observabilty](README-Observabilty.md) 


## TODO

1. Create infrastructure using terraform.
