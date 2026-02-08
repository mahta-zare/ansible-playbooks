# Ansible Configuration Management Project

This project demonstrates Infrastructure as Code (IaC) using **Terraform** for infrastructure provisioning and **Ansible** for configuration management. The project provisions AWS EC2 instances and automatically configures them with various services and applications.

## 📋 Overview

This repository combines:
- **Terraform**: Provisions AWS infrastructure (VPC, EC2 instances, security groups)
- **Ansible**: Configures and manages software on provisioned servers
- **Integration**: Terraform provisioner automatically triggers Ansible playbooks after instance creation

## 🏗️ Architecture

```
Terraform (Infrastructure) → EC2 Instance → Ansible (Configuration) → Configured Services
```

1. Terraform creates EC2 instances in AWS
2. Terraform provisioner triggers Ansible playbook execution
3. Ansible configures the instances with required software and services

## 📁 Project Structure

```
ansible/
├── main.tf                    # Terraform configuration for AWS infrastructure
├── terraform.tfvars          # Terraform variable values
├── ansible.cfg               # Ansible configuration
├── hosts                     # Ansible inventory file
├── project-vars              # Ansible variable file
├── my-playbook.yaml          # Basic nginx playbook example
├── deploy-node.yaml          # Node.js application deployment
├── deploy-docker.yaml         # Docker and Docker Compose installation
├── deploy-docker-newuser.yaml # Docker setup with new user creation
├── deploy-nexus.yaml         # Nexus Repository Manager installation
└── README.md                # This file
```

## 📄 File Descriptions

### Terraform Files

#### `main.tf`
Terraform configuration that provisions:
- VPC with custom CIDR block
- Public subnet in specified availability zone
- Internet Gateway and Route Table
- Security Group (SSH on port 22, HTTP on port 8080)
- EC2 instance (Amazon Linux 2023)
- **Terraform provisioner** that automatically runs Ansible after instance creation

**Key Features**:
- Uses `local-exec` provisioner to run Ansible playbook
- Outputs server public IP address
- Automatically triggers `deploy-docker-newuser.yaml` after instance creation

#### `terraform.tfvars`
Variable definitions for Terraform:
- VPC and subnet CIDR blocks
- Availability zone
- Environment prefix
- Instance type
- SSH key locations
- Your IP address for security group

### Ansible Configuration Files

#### `ansible.cfg`
Ansible configuration settings:
- Disables host key checking (for dynamic inventory)
- Sets default inventory file to `hosts`

#### `hosts`
Ansible inventory file defining target servers:
- `[nexus_server]`: Server for Nexus Repository Manager
- `[docker_server]`: Server for Docker deployments
- Includes SSH connection details (user, private key)

#### `project-vars`
Ansible variable file containing:
- Linux user name
- Application location and version
- User home directory
- Docker password (should be kept secure)

### Ansible Playbooks

#### `my-playbook.yaml`
**Purpose**: Basic example playbook for nginx management

**Features**:
- Demonstrates package management with `apt`
- Shows service management with `service` module
- Example of uninstalling and stopping nginx

**Usage**:
```bash
ansible-playbook -i hosts my-playbook.yaml
```

---

#### `deploy-node.yaml`
**Purpose**: Deploys Node.js application to a server

**What it does**:
1. Updates package cache and installs Node.js and npm
2. Creates a new Linux user for running the Node.js app
3. Deploys the application:
   - Unpacks application tarball
   - Installs npm dependencies
   - Starts the Node.js server
   - Verifies application is running

**Key Features**:
- Uses `vars_files` to load variables
- Demonstrates `become_user` for running tasks as specific user
- Uses `async` and `poll` for background process execution
- Includes debugging output

**Usage**:
```bash
ansible-playbook -i hosts deploy-node.yaml
```

---

#### `deploy-docker.yaml`
**Purpose**: Installs Docker and Docker Compose, then starts containers

**What it does**:
1. Installs Docker using `dnf` package manager
2. Starts and enables Docker daemon
3. Installs Docker Compose v5.0.2 as a CLI plugin
4. Adds `ec2-user` to docker group
5. Copies docker-compose.yaml file to server
6. Logs into Docker registry
7. Starts containers using docker-compose

**Key Features**:
- Uses `community.docker` collection for Docker operations
- Demonstrates user group management
- Uses `meta: reset_connection` after group changes
- Supports Docker registry authentication

**Usage**:
```bash
ansible-playbook -i hosts deploy-docker.yaml
```

---

#### `deploy-docker-newuser.yaml`
**Purpose**: Enhanced Docker deployment with new user creation

**What it does**:
1. Waits for SSH connection to be available
2. Installs Docker and Docker Compose
3. Creates a new Linux user with docker group membership
4. Deploys Docker containers as the new user

**Key Features**:
- Includes connection wait logic for newly created instances
- Creates dedicated user for Docker operations
- Runs Docker operations as the new user (better security)
- Used by Terraform provisioner for automated deployment

**Usage**:
```bash
ansible-playbook -i hosts deploy-docker-newuser.yaml
```

---

#### `deploy-nexus.yaml`
**Purpose**: Installs and configures Nexus Repository Manager

**What it does**:
1. Installs Java 21 and net-tools
2. Downloads and unpacks Nexus 3.88.0-08
3. Creates `nexus` user and group
4. Sets proper ownership of Nexus directories
5. Configures Nexus to run as nexus user
6. Starts Nexus service
7. Verifies Nexus is running (checks process and network ports)

**Key Features**:
- Handles conditional logic (checks if Nexus already exists)
- Demonstrates file ownership management
- Uses `lineinfile` for configuration file modification
- Includes verification steps

**Usage**:
```bash
ansible-playbook -i hosts deploy-nexus.yaml
```

## 🚀 Getting Started

### Prerequisites

1. **AWS Account** with appropriate permissions
2. **Terraform** installed (version 1.0+)
3. **Ansible** installed (version 2.9+)
4. **Ansible Collections**:
   ```bash
   ansible-galaxy collection install community.docker
   ```
5. **SSH Key Pair**:
   - Public key at `~/.ssh/id_rsa.pub`
   - Private key at `~/.ssh/id_rsa`
6. **AWS CLI** configured with credentials

### Setup Steps

1. **Clone the repository**:
   ```bash
   cd /Users/mahtazare/Documents/Learning/ansible
   ```

2. **Configure variables**:
   - Update `terraform.tfvars` with your values:
     - Your IP address for security group
     - SSH key locations
     - VPC and subnet CIDR blocks
   - Update `project-vars`:
     - Set `docker_password` (Docker Hub password)
     - Adjust `linux_name` if needed
     - Update file paths if necessary

3. **Initialize Terraform**:
   ```bash
   terraform init
   ```

4. **Review the plan**:
   ```bash
   terraform plan
   ```

5. **Apply infrastructure**:
   ```bash
   terraform apply
   ```
   
   This will:
   - Create the EC2 instance
   - Automatically run the Ansible playbook via provisioner
   - Output the server IP address

### Manual Ansible Execution

If you want to run Ansible playbooks manually (without Terraform):

1. **Update inventory** (`hosts` file):
   ```ini
   [docker_server]
   <your-server-ip> ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=ec2-user
   ```

2. **Run a playbook**:
   ```bash
   ansible-playbook -i hosts deploy-docker.yaml
   ```

## 🔧 Configuration

### Terraform Variables

Edit `terraform.tfvars`:
- `vpc_cidr_block`: VPC network range
- `subnet_1_cidr_block`: Subnet network range
- `avail_zone`: AWS availability zone
- `env_prefix`: Environment name prefix
- `my_ip`: Your IP address for SSH access
- `instance_type`: EC2 instance type
- `public_key_location`: Path to SSH public key
- `private_key_location`: Path to SSH private key

### Ansible Variables

Edit `project-vars`:
- `linux_name`: Linux username for deployments
- `location`: Local path to application files
- `version`: Application version
- `user_home_dir`: Home directory path
- `docker_password`: Docker Hub password (keep secure!)

## 📝 Usage Examples

### Deploy Docker Containers
```bash
# Via Terraform (automatic)
terraform apply

# Or manually with Ansible
ansible-playbook -i hosts deploy-docker-newuser.yaml
```

### Deploy Node.js Application
```bash
ansible-playbook -i hosts deploy-node.yaml
```

### Install Nexus Repository Manager
```bash
ansible-playbook -i hosts deploy-nexus.yaml
```

## 🔒 Security Considerations

1. **SSH Keys**: Keep private keys secure, never commit them
2. **Passwords**: Store sensitive values (like `docker_password`) securely:
   - Use Ansible Vault for encryption
   - Or use environment variables
   - Never commit passwords to git
3. **Security Groups**: The current setup allows SSH only from your IP
4. **User Permissions**: Playbooks use `become` for privileged operations

## 🧹 Cleanup

To destroy all infrastructure:
```bash
terraform destroy
```

**Note**: This will terminate the EC2 instance and all associated resources.

## 📚 Learning Objectives

This project demonstrates:
- ✅ Infrastructure provisioning with Terraform
- ✅ Configuration management with Ansible
- ✅ Integration between Terraform and Ansible
- ✅ Docker container management
- ✅ Node.js application deployment
- ✅ Nexus Repository Manager setup
- ✅ User and permission management
- ✅ Service lifecycle management

## 🐛 Troubleshooting

### Ansible Connection Issues
- Verify SSH key permissions: `chmod 600 ~/.ssh/id_rsa`
- Check security group allows SSH from your IP
- Ensure instance is running and accessible

### Docker Issues
- Verify user is in docker group: `groups`
- May need to reconnect after adding user to group
- Check Docker daemon is running: `systemctl status docker`

### Terraform Provisioner Issues
- Provisioners run on local machine, ensure Ansible is installed
- Check Ansible can reach the instance (security groups, network)
- Review Terraform logs for provisioner output

## 📖 Additional Resources

- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Docker Collection](https://docs.ansible.com/ansible/latest/collections/community/docker/)
- [Nexus Repository Manager](https://help.sonatype.com/repomanager3)

## 📄 License

This project is for educational purposes.

