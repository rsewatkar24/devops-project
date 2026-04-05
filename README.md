# Project - End-to-End Kubernetes Implementation 
<img width="1050" height="483" alt="image" src="https://github.com/user-attachments/assets/24d744f7-d20f-4473-a6e3-e7d4bc8826f7" />
### Prerequisites:
Before we get into the good stuff, first we need to make sure we have the required services on our local machine or dev server, which are:

- AWS Account
- GitHub Account
- AWS CLI installed and configured.
- Docker installed locally
- Terraform
- Basic familiarity with YAML and GitHub workflows.
- Any Browser for testing

# Requirements

- [x] #1 Create AWS Access Keys
- [x] #2 Provision an AWS EC2 instance with terraform
- [x] Setting up a CI/CD pipeline using Terraform :tada:
- [x] Terraform Cloud Configuration
- [x] Check result in your AWS Management Console :tada:

## You can deploy EC2 instance with below Terraform code
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

# ---------------- VPC ----------------
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "ramesh_vpc"
  }
}

# ---------------- Subnet ----------------
resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet"
  }
}

# ---------------- Internet Gateway ----------------
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

# ---------------- Route Table ----------------
resource "aws_route_table" "rt" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route" "internet_access" {
  route_table_id         = aws_route_table.rt.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.igw.id
}

resource "aws_route_table_association" "rta" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.rt.id
}

# ---------------- Security Group ----------------
resource "aws_security_group" "ec2_sg" {
  name   = "ec2-security-group"
  vpc_id = aws_vpc.main.id

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # restrict later
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# ---------------- Get Latest AMI (FIX) ----------------
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

##Add Ubuntu AMI Data Block

data "aws_ami" "ubuntu" {
  most_recent = true

  owners = ["099720109477"]  # Canonical (Ubuntu)

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# ---------------- EC2 Instance ----------------
resource "aws_instance" "my_ec2" {
  ami           = data.aws_ami.ubuntu.id   # dynamic AMI
  instance_type = "t3.medium"

  subnet_id              = aws_subnet.public_subnet.id
  vpc_security_group_ids = [aws_security_group.ec2_sg.id]

  tags = {
    Name = "ramesh-terraform-EC2"
  }
}

# ---------------- Output ----------------
output "public_ip" {
  value = aws_instance.my_ec2.public_ip
}

```

<img width="782" height="118" alt="image" src="https://github.com/user-attachments/assets/ec495d8a-8f04-4f80-883a-c76e2c82b55e" />

## we can install aws cli ,eksctl utility and kubectl on EC2 Instance as follow 


```
 apt-get update
 apt install curl unzip
 curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
 unzip awscliv2.zip
 sudo ./aws/install
 yes
 exit
 apt-get update
 ls
 unzip awscliv2.zip
 sudo ./aws/install
 sudo ./aws/install --update 
 aws --version 
 curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp  
 sudo mv /tmp/eksctl /usr/local/bin
 eksctl version
 curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.9/2020-08-04/bin/linux/amd64/kubectl
 chmod +x ./kubectl  
 sudo mv ./kubectl /usr/local/bin
 kubectl version --short --client
 eksctl create cluster   --name cluster-demo   --region us-east-1   --version 1.34   --nodegroup-name my-nodes   --node-type t3.medium   --nodes 2   --nodes-min 1   --nodes-max 2   --managed

```
<img width="956" height="295" alt="image" src="https://github.com/user-attachments/assets/b64b36b1-214f-491c-a48d-2b1e1c161af1" />
<img width="964" height="418" alt="image" src="https://github.com/user-attachments/assets/cece40f8-613e-4be1-8111-7842efcfe4b2" />

<img width="959" height="260" alt="image" src="https://github.com/user-attachments/assets/8c3d00c3-0bb0-48ad-94f7-19bcca5143ad" />

<img width="947" height="368" alt="image" src="https://github.com/user-attachments/assets/6a339c48-42d7-4845-a0c9-1ac29840b194" />



## Next we can install  ArgoCD with below commands.

```
 kubectl create namespace argocd
 kubectl apply -n argocd   -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml   --server-side
 kubectl get svc -n argocd
 kubectl get pod -A
 kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
 kubectl get pod -A
 kubectl get svc -n argocd
 kubectl port-forward svc/argocd-server -n argocd 9092:443 --address 0.0.0.0

```
## access ArgoCD Application with Public IP of ekscluster node and the port from web browser .

<img width="955" height="483" alt="image" src="https://github.com/user-attachments/assets/85bdb88a-c26e-4e33-81fd-d96d40ed5668" />


# Github Workflows Terraform Pipeline Provision To Deploy to AWS EKS 
  Name: Deploy Pipeline --> Terraform CI/CD pipeline To AWS EKS Cluster - Enterprise
   DEV STAGE -->SonarQube Scan-->TRIVY SCAN--> Slack notify --> QA STAGE --> PROD STAGE
```
name: Deploy Pipeline

on:
  push:
    branches:
      - main

jobs:
  # ================= DEV STAGE =================
  deploy-dev:
    name: Deploy DEV
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: SonarQube Scan
        uses: SonarSource/sonarcloud-github-action@v3
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        with:
          args: >
            -Dsonar.projectKey=rsewatkar24_devops-project
            -Dsonar.organization=rsewatkar24
            -Dsonar.host.url=https://sonarcloud.io

      - name: Build Docker Image
        run: docker build -f .github/Dockerfile -t rsewatka/myapp:${{ github.sha }} .

      - name: Install Trivy
        run: |
          set -euxo pipefail
          sudo apt-get update
          sudo apt-get install -y wget apt-transport-https gnupg lsb-release
          wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
          echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
          sudo apt-get update
          sudo apt-get install -y trivy
          trivy --version

      - name: TRIVY SCAN
        run: |
          docker images | head -n 50
          docker image inspect rsewatka/myapp:${{ github.sha }}
          trivy image --severity HIGH,CRITICAL --exit-code 0 rsewatka/myapp:${{ github.sha }}

      - name: Login to DockerHub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Push Docker Image
        run: docker push rsewatka/myapp:${{ github.sha }}

      - name: Slack notify (DEV)
        if: always()
        uses: slackapi/slack-github-action@v2.0.0
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
          webhook-type: incoming-webhook
          payload: |
            {
              "text": "*DEV* deploy finished with status: *${{ job.status }}*\nRepository: ${{ github.repository }}\nBranch: ${{ github.ref_name }}\nCommit: ${{ github.sha }}\nRun: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
            }

  # ================= QA STAGE =================
  deploy-qa:
    name: Deploy QA
    needs: deploy-dev
    runs-on: ubuntu-latest
    environment: qa
    steps:
      - name: Deploy to QA
        run: echo "Deploying to QA"

      - name: Slack notify (QA)
        if: always()
        uses: slackapi/slack-github-action@v2.0.0
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
          webhook-type: incoming-webhook
          payload: |
            {
              "text": "*QA* deploy finished with status: *${{ job.status }}*\nRepository: ${{ github.repository }}\nBranch: ${{ github.ref_name }}\nCommit: ${{ github.sha }}\nRun: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
            }

  # ================= PROD STAGE =================
  deploy-prod:
    name: Deploy PROD
    needs: deploy-qa
    runs-on: ubuntu-latest
    environment: prod
    steps:
      - name: Deploy to PROD
        run: echo "Deploying to PROD"

      - name: Slack notify (PROD)
        if: always()
        uses: slackapi/slack-github-action@v2.0.0
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
          webhook-type: incoming-webhook
          payload: |
            {
              "text": "*PROD* deploy finished with status: *${{ job.status }}*\nRepository: ${{ github.repository }}\nBranch: ${{ github.ref_name }}\nCommit: ${{ github.sha }}\nRun: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
            }

```
# we can use SonarQube and Slack account for the image scan and notifications
<img width="965" height="434" alt="image" src="https://github.com/user-attachments/assets/9ad3df05-d4d7-46b9-ad09-f83a27dd20ba" />


<img width="956" height="456" alt="image" src="https://github.com/user-attachments/assets/03da1881-606f-4e48-b8d8-db77c6833922" />




