# DevOps Automation Pipeline
- Designed and implemented a complete end-to-end CI/CD automation pipeline using modern DevOps tools including Jenkins, Docker, Trivy, ArgoCD, and Kubernetes on AWS EKS, following GitOps best practices.
- Configured Jenkins to automate code integration, Docker image creation, vulnerability scanning with Trivy, and push to Docker Hub. Final stage of the pipeline triggers the CD workflow by syncing with the deployment repository.
- Leveraged ArgoCD to monitor and deploy the latest container images to AWS EKS, ensuring continuous delivery and environment consistency via GitOps. (Repo: [DevOps ArgoCD Project](https://github.com/Ashik-Domain/Devops_project_Argocd/blob/main/README.md) )
- Outcome: Successfully streamlined the DevOps workflow, enabling faster, secure, and scalable deployments for cloud-native applications. This setup reduces manual intervention, improves security posture, and ensures consistent, version-controlled deployments across environments.


# Project Guide for Jenkins, Docker, and Trivy

This README provides a step-by-step guide to set up Jenkins, Docker, and Trivy on a Linux system. Follow the commands and instructions carefully.

## Prerequisites
- An EC2 instance or a Linux machine.
- Access to the terminal with `sudo` privileges.
- Internet connection for downloading packages.

## Steps

### Install and Start Docker
1. Update the system:
   ```bash
   yum update -y
   ```
2. Install Docker:
   ```bash
   yum install -y docker
   ```
3. Start Docker service:
   ```bash
   systemctl start docker
   ```
4. Enable Docker service to start on boot:
   ```bash
   systemctl enable docker
   ```

### Install Jenkins
1. Install Java:
   ```bash
   yum install java-17-amazon-corretto -y
   ```
2. Add the Jenkins repository:
   ```bash
   wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
   rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
   ```
3. Install Jenkins:
   ```bash
   yum install -y jenkins
   ```
4. Start and enable Jenkins service:
   ```bash
   systemctl start jenkins
   systemctl enable jenkins
   ```
5. Check Jenkins status:
   ```bash
   systemctl status jenkins
   ```

### Access Jenkins
1. Retrieve the initial admin password:
   ```bash
   cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
2. Open Jenkins in a browser:
   ```
   http://<EC2-public-IP>:8080
   ```
3. If the page does not load, update the inbound security rules of your EC2 instance to allow port `8080`.

### Configure Docker for Jenkins
1. Add the `jenkins` user to the `docker` group:
   ```bash
   usermod -aG docker jenkins
   ```
2. Restart Jenkins service:
   ```bash
   systemctl restart jenkins
   ```

### Create a DockerHub Account and Login
1. Create an account on [DockerHub](https://hub.docker.com/).
2. Log in to Docker:
   ```bash
   docker login
   ```
   Enter your DockerHub username and password when prompted.

### Install Trivy
1. Navigate to the `/etc/yum.repos.d/` directory:
   ```bash
   cd /etc/yum.repos.d/
   ```
2. Add the Trivy repository:
   ```bash
   cat << EOF | sudo tee -a /etc/yum.repos.d/trivy.repo
   [trivy]
   name=Trivy repository
   baseurl=https://aquasecurity.github.io/trivy-repo/rpm/releases/\$basearch/
   gpgcheck=1
   enabled=1
   gpgkey=https://aquasecurity.github.io/trivy-repo/rpm/public.key
   EOF
   ```
3. Install Trivy:
   ```bash
   yum install -y trivy
   ```

## Jobs

### Job 1: Pull Dockerfile and Build the Image

#### Create a New Jenkins Job:
1. Go to the Jenkins Dashboard.
2. Click **New Item**, enter a name (e.g., `Build_Docker_Image`), and select **Freestyle project**.

#### Configure the Job:
- **Source Code Management**:
  - Select **Git** and add the repository URL:
    ```
    https://github.com/Ashik-Domain/Bookinfo_app.git
    ```
  - Use the branch `main`.
- **Build Steps**:
  - Add the following shell command in the Build section:
    ```bash
    cd src/productpage
    docker build -t productpage-app .
    ```
- **Post-Build Actions**:
  - Add **Build other projects** and specify the next job (e.g., `Test_Docker_Image`).

#### Save the Job.

### Job 2: Test the Docker Image with Trivy

#### Create a New Jenkins Job:
1. Name it `Test_Docker_Image`.

#### Configure the Job:
- **Add a Build Step**:
  ```bash
  trivy image --severity HIGH,CRITICAL productpage-app
  ```
- **Post-Build Actions**:
  - Add **Build other projects** and specify the next job (e.g., `Push_to_DockerHub`).

#### Save the Job.

### Job 3: Push the Image to DockerHub

#### Create a New Jenkins Job:
1. Name it `Push_to_DockerHub`.

#### Configure the Job:
- **Build Steps**:
  - Add the following shell command:
    ```bash
    docker tag productpage-app <your_dockerhub_username>/productpage-app:latest
    docker push <your_dockerhub_username>/productpage-app:latest
    ```
    Replace `<your_dockerhub_username>` and `<your_dockerhub_password>` with your DockerHub credentials. Alternatively, use the Jenkins credentials ID to inject them dynamically.

#### Save the Job.

### Connecting the Jobs

#### Set Upstream and Downstream Triggers:
1. Go to Job 1 (`Build_Docker_Image`) and configure its Post-Build Action to trigger Job 2.
2. Similarly, go to Job 2 (`Test_Docker_Image`) and configure its Post-Build Action to trigger Job 3.

## Additional Commands
- Check free memory:
  ```bash
  free -m
  ```
- List Docker images:
  ```bash
  docker images
  ```

## Notes
- Ensure that you have sufficient privileges when running these commands. Use `sudo` when necessary.
- Refer to the official [Trivy documentation](https://trivy.dev/latest/getting-started/installation/) for more details on installation.

## Next Steps

After completing the Jenkins pipeline, you can use ArgoCD to deploy the Docker image to a Kubernetes cluster and manage the application's lifecycle. For more details, refer to the [DevOps ArgoCD Project](https://github.com/Ashik-Domain/Devops_project_Argocd/blob/main/README.md).

## Bookinfo Repository

- **For the developer app**:  
  [Bookinfo GitHub Repository](https://github.com/Ashik-Domain/Bookinfo_app.git)


## History of Commands
The following commands were executed during the setup process:

```bash
# Update and install necessary packages
yum update -y
yum install -y docker
systemctl start docker
yum install java-17-amazon-corretto -y

# Configure Jenkins
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install -y jenkins
systemctl start jenkins
systemctl enable jenkins
systemctl status jenkins

# DockerHub login
docker login

# Configure Jenkins user permissions
usermod -aG docker jenkins
systemctl restart jenkins

# Install Trivy
cd /etc/yum.repos.d/
cat << EOF | sudo tee -a /etc/yum.repos.d/trivy.repo
[trivy]
name=Trivy repository
baseurl=https://aquasecurity.github.io/trivy-repo/rpm/releases/\$basearch/
gpgcheck=1
enabled=1
gpgkey=https://aquasecurity.github.io/trivy-repo/rpm/public.key
EOF
yum install -y trivy
```
