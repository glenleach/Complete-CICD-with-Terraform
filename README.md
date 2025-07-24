# Complete CICD with Terraform

## Project Overview
This project demonstrates how to implement a complete CI/CD pipeline using Jenkins, Docker, and Terraform to automate the build, test, infrastructure provisioning, and deployment of a Java Maven application with a PostgreSQL database on AWS EC2. The project emphasizes best practices in infrastructure as code, automation, and secure credential management.

## Prerequisites
- AWS account with programmatic access (Access Key ID and Secret Access Key)
- Docker Hub account for image storage
- Jenkins server with required plugins (Pipeline, Docker, SSH Agent, Credentials Binding)
- SSH key pair for EC2 access, registered in Jenkins credentials
- Terraform installed and configured
- Maven installed and configured in Jenkins

### Jenkins Credentials Setup
Before running the pipeline, you must add the following credentials in Jenkins (Manage Jenkins > Credentials):

1. **AWS Credentials**
   - Type: "Username with password" or "Secret text"
   - ID: `jenkins_aws_access_key_id` (for Access Key ID), `jenkins-aws_secret_access_key` (for Secret Access Key)
   - Used for: Allowing Terraform to authenticate with AWS for provisioning resources

2. **Docker Hub Credentials**
   - Type: "Username with password"
   - ID: `docker-hub-repo`
   - Used for: Authenticating with Docker Hub to push and pull images

3. **EC2 SSH Key**
   - Type: "SSH Username with private key"
   - ID: `server-ssh-key`
   - Username: `ec2-user`
   - Private Key: Paste the contents of your AWS EC2 PEM file (see below)
   - Used for: Allowing Jenkins to SSH into the EC2 instance for deployment

#### Setting Up the EC2 Key Pair
- When you provision your EC2 instance with Terraform, you must specify a key pair using the `key_name` attribute:
  ```hcl
  associate_public_ip_address = true
  key_name = "myapp-key-pair"
  ```
- Create this key pair in the AWS Console (EC2 > Key Pairs > Create key pair) and download the PEM file (e.g., `myapp-key-pair.pem`).
- Store the PEM file securely. You will need to use its contents as the private key in your Jenkins credential (see above).
- This key pair allows Jenkins to SSH into the EC2 instance for automated deployment. Only the holder of the private key (Jenkins, via the credential) can access the instance as `ec2-user`.

> **Note:** The credential IDs must match those referenced in the Jenkinsfile. Store all credentials securely and never hard-code secrets in your codebase.

---

## Jenkinsfile & Pipeline Overview
The Jenkinsfile defines a declarative pipeline with the following stages:

1. **Build App**
   - Uses a shared library function `buildJar()` to compile the Java Maven application and produce a JAR file.
2. **Build Image**
   - Uses shared library functions `buildImage()`, `dockerLogin()`, and `dockerPush()` to build a Docker image, log in to Docker Hub, and push the image.
3. **Provision Server**
   - Runs Terraform commands to provision an AWS EC2 instance. Retrieves the public IP of the new server for deployment.
4. **Deploy**
   - Waits for the EC2 server to initialize.
   - Uses the SSH Agent plugin to securely connect to the EC2 instance.
   - Copies the `docker-compose.yaml` and `server-cmds.sh` files to the server.
   - Executes the shell script to launch Docker Compose, which starts two containers: one for PostgreSQL and one for the Java Maven app.

### Why Use a Jenkins Shared Library?
A Jenkins shared library encapsulates common pipeline steps such as building the JAR, building and pushing Docker images, and Docker authentication. This promotes reusability and keeps the Jenkinsfile clean and maintainable. The shared library is referenced at the top of the Jenkinsfile and its functions are called within the pipeline stages.

---

## CI/CD with Jenkins and Credentials Setup
- Jenkins credentials are used to securely store sensitive information such as AWS keys and Docker Hub credentials.
- The pipeline references these credentials using the `credentials()` function in the environment block, ensuring secrets are not hardcoded.
- The SSH Agent plugin is used to provide the SSH private key for connecting to the EC2 instance during deployment.

---

## Deployment Process: EC2, SSH, Docker Compose
- **Terraform** provisions an EC2 instance and outputs its public IP.
- The pipeline waits for the server to initialize (using a sleep step).
- Using SSH, Jenkins copies the deployment scripts and Docker Compose file to the EC2 instance.
- The shell script is executed remotely, which runs Docker Compose to start two containers:
  - **PostgreSQL**: Provides the database backend for the application.
  - **Java Maven App**: The main application, built and packaged as a Docker image.

---

## Terraform Configuration Overview
**File: `terraform/main.tf`**

```hcl
required_version = ">= 0.12"
backend "s3" {
  bucket = "jenkins-backup-bucket-123456789"
  key    = "myapp/state.tfstate"
  region = "eu-west-2"
}
```

- **Terraform Version Requirement**: Ensures Terraform version 0.12 or higher is used for compatibility.
- **Remote State Backend (S3)**: Configures Terraform to store its state file in an AWS S3 bucket for team collaboration and state locking.

---

## Setting Up Terraform State in an S3 Bucket

To enable remote state management and collaboration, Terraform state should be stored in an S3 bucket. Follow these steps to set up and use S3 as your backend:

### 1. Create an S3 Bucket
- Go to the AWS Console > S3 > Create bucket.
- Choose a unique bucket name (e.g., `jenkins-backup-bucket-123456789`).
- Select the region (e.g., `eu-west-2`).
- Leave other settings as default or adjust as needed.

### 2. (Recommended) Enable State Locking with DynamoDB
- Go to AWS Console > DynamoDB > Create table.
- Table name: `terraform-lock` (or similar)
- Partition key: `LockID` (String)
- Update your backend config in `main.tf` to include DynamoDB table:
  ```hcl
  backend "s3" {
    bucket         = "jenkins-backup-bucket-123456789"
    key            = "myapp/state.tfstate"
    region         = "eu-west-2"
    dynamodb_table = "terraform-lock"
  }
  ```

### 3. Update IAM Permissions
- Ensure the IAM user or role used by Jenkins/Terraform has permissions for S3 (GetObject, PutObject, ListBucket) and DynamoDB (if used).

### 4. Initialize Terraform
- In the `terraform` directory, run:
  ```sh
  terraform init
  ```
- This will configure Terraform to use the S3 backend and migrate any local state to S3.

### 5. Run the Pipeline
- Once the backend is configured and initialized, the Jenkins pipeline will be able to provision infrastructure and manage state remotely.

---

## Jenkins, Docker, and Shared Library Integration
- **Docker Hub Repo ID Consistency**: The Docker Hub repository ID set up in Jenkins must match the one used in your shared library repository.
- **Jenkins Server Requirements**: Jenkins must have Terraform, Docker, and SSH agent available for provisioning and deployment.

---

## AWS Credentials Management for Jenkins
- **Manual Credential Setup**: Create AWS credentials in IAM and add them to Jenkins as credentials.
- **Environment Variable Usage**: Credentials are exposed as environment variables for Terraform and AWS CLI tools.
- **Security Note**: Never hard-code AWS credentials in your code or Terraform files.

---

## Jenkins Job Setup Instructions
- **Create a Multibranch Pipeline Job**: In Jenkins, create a new job and select Multibranch Pipeline. Configure the branch source and filter for the correct branch.
- **Maven Tool Configuration**: Ensure Maven is installed and named "Maven" in Jenkins Global Tool Configuration.

---

## Terminating Resources with Terraform
1. Navigate to the Terraform directory:
   ```sh
   cd terraform
   ```
2. Initialize Terraform (if not already initialized):
   ```sh
   terraform init
   ```
3. Check the current state and list managed resources:
   ```sh
   terraform state list
   ```
4. Destroy all managed resources:
   ```sh
   terraform destroy
   ```

---

## Summary
This project demonstrates a full DevOps workflow, from code build to automated infrastructure provisioning and application deployment. It leverages Jenkins, Docker, and Terraform to achieve a robust, repeatable, and secure CI/CD pipeline, following industry best practices learned through the TechWorld with Nana bootcamp.
