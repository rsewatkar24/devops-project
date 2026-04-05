# devops-project
Implementing CI/CD Pipelines for EKS workloads with GitHub Actions for Multi-Environments Approach
<img width="1050" height="483" alt="image" src="https://github.com/user-attachments/assets/15dd522b-febc-46cf-9e0f-5592ca9ab5f5" />
### Prerequisites:
Before we get into the good stuff, first we need to make sure we have the required services on our local machine or dev server, which are:

- AWS Account
- GitHub Account
- AWS CLI installed and configured.
- Docker installed locally and docker image.
- Terraform
- Salck account
- Sonarcloud account
- Basic familiarity with YAML and GitHub workflows.
- Any Browser for testing
# Requirements
- [x] #1 Provision an AWS EC2 instance with Terraform.
- [x] #2 Install eksctl utility on EC2 instance.

## local testing

In the project directory, you can run:

## You can install aws cli using the following command


```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

```

Next, configure your aws account in your computer using the following command:

```
aws configure
```
