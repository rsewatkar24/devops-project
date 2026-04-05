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


## we can install aws cli ,eksctl utility and kubectl on EC2 Instance as follow 


```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

```
