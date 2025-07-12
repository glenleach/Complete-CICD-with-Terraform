# Demo App – DevOps Bootcamp Training (TechWorld with Nana)
This project is part of the DevOps Bootcamp from TechWorld with Nana.

# java-maven-app-with-terraform

This project demonstrates how to integrate **Terraform** into a CI/CD pipeline using **Jenkins**. It builds upon a previous use case where a Docker image was built in a pipeline and then deployed to a remote server. In this enhanced demo, **Terraform** is used to provision the remote server as part of the automated CI/CD process.

> **This project is part of the Terraform training series with Techworld with Nana DevOps Bootcamp. A Complete CI/CD with Terraform part 1, 2, and 3.**

## Key Features

- **Jenkins Pipeline Integration:**  
  The pipeline is triggered via the `jenkinsfile-ssh` branch in Jenkins, automating the entire workflow from infrastructure provisioning to application deployment.

- **Terraform Automation:**  
  Terraform scripts are used to provision the remote server dynamically during the pipeline execution, ensuring infrastructure is created or updated as needed.

- **End-to-End CI/CD:**  
  The pipeline builds a Docker image, provisions the required infrastructure with Terraform, and deploys the Docker image to the newly created remote server—all in a single automated process.

## Workflow Overview

1. **Pipeline Trigger:**  
   The Jenkins pipeline is triggered (typically by a push or PR to the `jenkinsfile-ssh` branch).

2. **Infrastructure Provisioning:**  
   Terraform is executed within the pipeline to provision or update the remote server.

3. **Application Build:**  
   The Java Maven application is built and packaged as a Docker image.

4. **Deployment:**  
   The Docker image is deployed to the remote server provisioned by Terraform.

## Use Case

This demo is ideal for teams looking to:

- Automate infrastructure provisioning alongside application deployment.
- Integrate Terraform into existing CI/CD workflows.
- Demonstrate Infrastructure as Code (IaC) best practices within a Jenkins-driven pipeline.

---

**Note:**  
Make sure your Jenkins instance is configured to recognize and execute the pipeline defined in the `jenkinsfile-ssh` branch, and that your credentials and Terraform backend are properly set up for remote provisioning.
