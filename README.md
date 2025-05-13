# Project Overview: Jenkins, Terraform, Docker, and Shared Library Integration

This project automates infrastructure provisioning and application deployment using Terraform, Jenkins, Docker, and a shared Groovy library. Below you'll find a step-by-step explanation of the setup, requirements, and workflow.

---

## 1. Terraform Configuration Overview

**File: `terraform/main.tf`**

```hcl
required_version = ">= 0.12"
backend "s3" {
  bucket = "jenkins-backup-bucket-123456789"
  key    = "myapp/state.tfstate"
  region = "eu-west-2"
}
```

### Step-by-Step Explanation

- **Terraform Version Requirement**  
  Ensures Terraform version 0.12 or higher is used for compatibility.

- **Remote State Backend (S3)**  
  Configures Terraform to store its state file in an AWS S3 bucket for team collaboration and state locking:
  - `bucket`: S3 bucket name for state storage.
  - `key`: Path within the bucket for the state file.
  - `region`: AWS region of the bucket.

---

## 2. Jenkins, Docker, and Shared Library Integration

- **Docker Hub Repo ID Consistency**  
  The Docker Hub repository ID set up in Jenkins **must match** the one used in your shared library repository (where your Groovy scripts like `buildJar()` reside). This ensures Jenkins builds and pushes images to the correct Docker Hub repository.

- **Jenkins Server Requirements**
  - **Terraform Installed**: Jenkins (running in a container) must have Terraform installed to provision infrastructure.
  - **Docker Installed**: Docker must be available inside the Jenkins container for building and running images.
  - **SSH Agent**: Jenkins requires an SSH agent to connect to remote servers for deployment and Docker operations.

---

## 3. AWS Credentials Management for Jenkins

- **Manual Credential Setup**
  - You must manually create an AWS Access Key ID and Secret Access Key in AWS IAM.
  - These credentials are then added to Jenkins as "Secret Text" or "Username with password" credentials.

- **Environment Variable Usage**
  - In Jenkins, these credentials are exposed to the build environment using environment variable names (commonly `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`).
  - This allows Terraform (and other AWS CLI tools) running in Jenkins to authenticate and access AWS resources, such as the S3 bucket for state storage or provisioning new instances.

- **Security Note**
  - Never hard-code AWS credentials in your code or Terraform files. Always use Jenkins credentials and environment variables for secure access.

---

## 4. Typical Workflow

1. **Jenkins Pipeline Starts**
   - Loads shared Groovy library functions (e.g., `buildJar()`).
   - Ensures Docker and Terraform are available in the Jenkins environment.

2. **AWS Credentials Injection**
   - Jenkins injects AWS credentials as environment variables for the build.

3. **Terraform Operations**
   - Terraform uses the injected credentials to access the S3 backend and provision AWS resources.

4. **Docker Build & Push**
   - Jenkins builds Docker images and pushes them to Docker Hub, using the consistent repository ID.

5. **Remote Deployment**
   - Jenkins uses SSH agent to connect to target servers and deploy Docker containers using the images from Docker Hub.

---

## 5. Shared Library Usage for Groovy Scripts

- **Remote Shared Library**  
  The Groovy script used in the Jenkins pipeline is sourced from Nana's remote Git repository as a [Jenkins Shared Library](https://gitlab.com/twn-devops-bootcamp/latest/12-terraform/jenkins-shared-library.git). This allows for centralized management and reuse of pipeline functions (such as `buildJar()`) across multiple projects and Jenkinsfiles.

- **Alternative: Local Implementation**  
  Alternatively, the Groovy script and its resources could have been implemented directly within the same Git repository as your Terraform and application code. This approach can simplify setup for smaller projects but may reduce reusability and maintainability for larger or multi-project environments.

---

## 6. Jenkins Job Setup Instructions

- **Create a Multibranch Pipeline Job**
  - In Jenkins, create a new job and select **Multibranch Pipeline** as the job type.
  - Under **Branch Sources**, choose **Git** and enter your repository URL in the **Project Repository** field.
  - Use the **Filter by name (with wildcards)** option to specify the branch you want Jenkins to build from, e.g., `jenkinsfile-sshagent`.
  - This ensures Jenkins only triggers builds for the specified branch containing your Jenkinsfile.

- **Maven Tool Configuration**
  - In **Manage Jenkins** > **Global Tool Configuration**, ensure that Maven is installed and the tool name is set to **Maven**.
  - This name must match the `tool name` referenced in your Jenkinsfile or pipeline scripts.

---

## 7. Terminating Resources with Terraform

To safely destroy all resources created by Terraform, follow these steps:

1. **Navigate to the Terraform directory:**
   ```sh
   cd terraform
   ```

2. **Initialize Terraform (if not already initialized):**
   ```sh
   terraform init
   ```

3. **Check the current state and list managed resources:**
   ```sh
   terraform state list
   ```

4. **Destroy all managed resources:**
   ```sh
   terraform destroy
   ```
   - Review the plan and confirm when prompted.

---

## 8. Summary

- **Terraform**: Manages infrastructure state in S3, requires AWS credentials via environment variables.
- **Jenkins**: Orchestrates builds, requires Docker, Terraform, and SSH agent.
- **Docker Hub**: Repository IDs must be consistent across Jenkins and shared libraries.
- **AWS Credentials**: Managed securely in Jenkins, injected as environment variables for Terraform and AWS CLI access.
- **Groovy Shared Library**: Sourced from a remote repo for reusability, but could be local for simpler setups.
- **Jenkins Job Setup**: Use a Multibranch Pipeline, filter for the correct branch, and ensure Maven is configured as "Maven".

---

**If you need a sample Jenkinsfile, further details on credential setup, or an architecture diagram, please ask!**
