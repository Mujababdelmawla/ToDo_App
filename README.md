# Todo App - Automated AWS Provisioning & Deployment with Ansible

A complete **Infrastructure-as-Code** solution that automates EC2 instance provisioning, security group configuration, and full-stack Todo application deployment using **GitHub Actions** and **Ansible**.


## 🎯 Project Overview

This project demonstrates enterprise-grade **CI/CD and IaC practices** by automating the complete deployment pipeline:

- **Infrastructure Provisioning**: Dynamic EC2 instance launch with security groups
- **Configuration Management**: Automated server setup and application deployment
- **Process Management**: PM2 for backend process orchestration
- **Frontend Build Pipeline**: Automated npm build and optimization

The entire deployment is triggered by a simple `git push` to the `main` branch, fully hands-free.

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Repository                         │
│                   (Push to main branch)                      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              GitHub Actions Workflow Runner                  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  • Checkout code                                     │  │
│  │  • Setup Python & Ansible                            │  │
│  │  • Configure AWS credentials                         │  │
│  │  • Create SSH key pair                               │  │
│  │  • Execute Ansible playbook                          │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│           Ansible Playbook Execution (Localhost)            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  ec2_launch role:                                    │  │
│  │  ├─ Create security group (SSH, HTTP, app ports)    │  │
│  │  ├─ Launch t3.micro EC2 instance (Ubuntu)           │  │
│  │  ├─ Wait for instance readiness                     │  │
│  │  └─ Dynamically add to inventory                    │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│      Ansible Configuration on EC2 Instance (SSH)            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  common role:                                        │  │
│  │  ├─ Update apt cache                                │  │
│  │  ├─ Install Apache2, Node.js, npm                   │  │
│  │  ├─ Start Apache2 service                           │  │
│  │  └─ Copy source code to /opt/todo-app               │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  backend role:                                       │  │
│  │  ├─ Install npm dependencies                        │  │
│  │  ├─ Install PM2 process manager                     │  │
│  │  ├─ Start backend with PM2                          │  │
│  │  └─ Configure PM2 startup (systemd)                 │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  frontend role:                                      │  │
│  │  ├─ Install npm dependencies                        │  │
│  │  └─ Build optimized frontend bundle                 │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              Running Todo Application                       │
│  ├─ Frontend: Built React/Vue app (port 3000)             │
│  ├─ Backend: Node.js API via PM2 (port 3001)              │
│  ├─ Web Server: Apache2 reverse proxy (port 80)            │
│  └─ Status: Accessible via EC2 public IP                   │
└─────────────────────────────────────────────────────────────┘
```

## 📋 Prerequisites

### Required Accounts & Credentials
- **AWS Account** with permissions to:
  - Create EC2 instances
  - Create security groups
  - Manage key pairs
- **GitHub Account** with repository access

### Local Setup (Optional - for testing)
- Python 3.8+
- Ansible 2.9+
- AWS CLI configured with credentials
- SSH key pair created in AWS (`mujab-temp`)

## 🚀 Quick Start

### 1. Set Up GitHub Secrets

Add these secrets to your GitHub repository (`Settings > Secrets and variables > Actions`):

| Secret Name | Value |
|---|---|
| `AWS_ACCESS_KEY_ID` | Your AWS access key |
| `AWS_SECRET_ACCESS_KEY` | Your AWS secret access key |
| `EC2_SSH_PRIVATE_KEY` | Content of your `mujab-temp.pem` file |

**Getting your EC2 SSH private key:**
```bash
# In AWS EC2 console, download the key pair file
cat ~/Downloads/mujab-temp.pem
# Copy the entire content (including -----BEGIN PRIVATE KEY----- and -----END PRIVATE KEY-----)
# Paste into GitHub secret as EC2_SSH_PRIVATE_KEY
```

### 2. Push to Main Branch

```bash
git push origin main
```

The GitHub Actions workflow will automatically:
1. ✅ Provision an EC2 instance
2. ✅ Configure security groups
3. ✅ Deploy the Todo application
4. ✅ Start all services

### 3. Access Your Application

After the workflow completes (2-3 minutes):

1. Go to **AWS EC2 Dashboard**
2. Find the newly created instance
3. Copy the **Public IPv4 address**
4. Open in browser: `http://<public-ip>`

The application will be live with:
- **Frontend**: Available at the root path
- **Backend API**: Running on port 3001
- **Process Manager**: PM2 managing the Node.js backend

## 📁 Project Structure

```
.
├── .github/
│   └── workflows/
│       └── provision_of_ec2.yml          # GitHub Actions workflow
├── ansible/
│   ├── playbooks/
│   │   └── provision_and_configure_server.yml  # Main playbook
│   └── roles/
│       ├── ec2_launch/
│       │   ├── tasks/
│       │   │   └── main.yml              # Security group & EC2 setup
│       │   └── vars/
│       │       └── main.yml              # AWS configuration variables
│       ├── common/
│       │   └── tasks/
│       │       └── main.yml              # OS-level setup, dependencies
│       ├── backend/
│       │   └── tasks/
│       │       └── main.yml              # Backend app deployment
│       └── frontend/
│           └── tasks/
│               └── main.yml              # Frontend build & setup
├── todo-backend/                         # Node.js backend application
├── todo-frontend/                        # React/Vue frontend application
└── README.md                             # This file
```

## 🔧 Roles Explained

### `ec2_launch` Role
**Purpose**: Infrastructure provisioning on AWS

**Tasks**:
1. Creates a security group (`mujab-security`) with inbound rules for:
   - Port 22 (SSH) - Remote access
   - Port 80 (HTTP) - Web traffic
   - Port 3000 (Frontend) - Development/testing
   - Port 3001 (Backend API) - Application server
   - Port 3002 - Additional service port
2. Launches a t3.micro EC2 instance with Ubuntu image
3. Waits for SSH to be ready (port 22 accessible)
4. Dynamically adds the instance to Ansible inventory

**Variables** (`vars/main.yml`):
```yaml
aws_region: us-east-1
ec2_instance_type: t3.micro
ec2_image_id: ami-0b6d9d3d33ba97d99  # Ubuntu 22.04 LTS
ec2_key_name: mujab-temp
security_group_name: mujab-security
```

### `common` Role
**Purpose**: Base server setup and dependencies

**Tasks**:
1. Updates apt package cache
2. Installs system packages:
   - Apache2 (web server)
   - Node.js (runtime)
   - npm (package manager)
3. Starts and enables Apache2 service
4. Syncs project source code to `/opt/todo-app`

### `backend` Role
**Purpose**: Deploy Node.js backend application

**Tasks**:
1. Installs npm dependencies from `todo-backend/package.json`
2. Installs PM2 globally for process management
3. Validates that `start.js` exists
4. Starts backend with PM2 in watch mode:
   ```bash
   pm2 start /opt/todo-app/todo-backend/start.js --name todo-backend --watch
   ```
5. Configures PM2 to restart on system boot

**Why PM2?**
- Auto-restart on crash
- Process monitoring and logging
- Watch mode for development (auto-reload on file changes)
- System integration via systemd

### `frontend` Role
**Purpose**: Build and prepare frontend application

**Tasks**:
1. Installs npm dependencies from `todo-frontend/package.json`
2. Runs build process: `npm run build`
3. Creates optimized production bundle

## 🔐 Security Considerations

### Current Setup
- ✅ Security group restricts access to necessary ports only
- ✅ SSH key pair authentication (no password)
- ✅ Secrets stored securely in GitHub (encrypted at rest)
- ✅ EC2 instance only accessible via security group rules

### Recommendations for Production
1. **Restrict HTTP access** - Use HTTPS with SSL/TLS certificates
2. **Security group refinement** - Limit SSH to specific IPs instead of `0.0.0.0/0`
3. **AWS IAM roles** - Use instance roles instead of access keys
4. **Secrets management** - Use AWS Secrets Manager or GitHub encrypted secrets
5. **Network isolation** - Deploy inside a VPC with private subnets
6. **Monitoring** - Add CloudWatch alarms and log aggregation
7. **Auto-scaling** - Use Auto Scaling Groups for multiple instances

## 🐛 Troubleshooting

### GitHub Actions Workflow Fails at "Run Ansible Playbook"

**Error**: `Host key verification failed`
- **Cause**: `ANSIBLE_HOST_KEY_CHECKING` not disabled
- **Fix**: Already handled via environment variable in workflow

**Error**: `Permission denied (publickey)`
- **Cause**: SSH key not properly loaded or wrong path
- **Check**: 
  ```yaml
  - name: debug ssh key file
    run: ls -l ~/.ssh/ec2_key.pem
  ```

### EC2 Instance Provisioned but Application Not Running

**Check PM2 status:**
```bash
# SSH into the instance
ssh -i mujab-temp.pem ubuntu@<public-ip>

# Check PM2 processes
pm2 list

# View backend logs
pm2 logs todo-backend

# Restart if needed
pm2 restart todo-backend
```

### Cannot Connect to Application

1. **Verify instance is running**:
   - AWS EC2 Dashboard → Check instance state
   
2. **Check security group rules**:
   - Go to Security Groups → `mujab-security`
   - Verify inbound rules include port 80, 3000, 3001

3. **Check if services are listening**:
   ```bash
   ssh -i mujab-temp.pem ubuntu@<public-ip>
   netstat -tlnp | grep LISTEN
   ```

4. **Review application logs**:
   ```bash
   pm2 logs todo-backend
   sudo journalctl -u apache2
   ```

## 🔄 Workflow Execution Details

### Step-by-step execution:

1. **Checkout repository** (2s)
   - Clones the latest code from main branch

2. **Set up Python** (0s)
   - Installs Python 3.10 environment

3. **Install dependencies** (30s)
   - pip: ansible, boto3, botocore
   - Ansible collection: amazon.aws

4. **Configure AWS credentials** (0s)
   - Uses GitHub secrets for authentication

5. **Create SSH key file** (0s)
   - Writes EC2 private key to `~/.ssh/ec2_key.pem`

6. **Run Ansible Playbook** (2m 3s)
   - `ec2_launch` role: 1m 30s (instance startup time)
   - `common` role: 15s (packages + sync)
   - `backend` role: 10s (npm install + PM2 setup)
   - `frontend` role: 8s (npm build)

**Total execution time**: ~2 minutes 38 seconds

## 📈 Next Steps & Enhancements

- [ ] **Database integration** - Add RDS (MySQL/PostgreSQL) support
- [ ] **Load balancing** - Implement Application Load Balancer (ALB)
- [ ] **HTTPS support** - Add ACM certificate and HTTPS configuration
- [ ] **Monitoring** - Integrate CloudWatch, Prometheus, or DataDog
- [ ] **Logging** - Centralize logs with CloudWatch Logs or ELK stack
- [ ] **Auto-scaling** - Use ASG with dynamic scaling policies
- [ ] **Blue-Green deployment** - Implement zero-downtime deployments
- [ ] **Terraform migration** - Convert Ansible to Terraform for IaC comparison
- [ ] **Testing** - Add infrastructure tests (Terraform, Ansible)
- [ ] **Cost optimization** - Add cost tracking and budget alerts

## 🎓 Learning Value

This project demonstrates:

✅ **Infrastructure as Code (IaC)** - Infrastructure defined in version control  
✅ **CI/CD Automation** - Automated deployment pipeline  
✅ **Configuration Management** - Ansible roles and playbooks  
✅ **Cloud Provisioning** - Dynamic AWS resource creation  
✅ **Inventory Management** - Dynamic inventory with Ansible  
✅ **Process Management** - PM2 for Node.js applications  
✅ **Security Best Practices** - Key management, security groups, secrets  
✅ **DevOps Workflow** - Full automation from code to running application  


## 👤 Author

**Mujab Yousef**
- GitHub: [@Mujababdelmawla](https://github.com/Mujababdelmawla)

## 📧 Support

For issues, questions, or suggestions:
1. Check the [Troubleshooting](#-troubleshooting) section
2. Review GitHub Actions logs for detailed error messages
3. Open a GitHub issue with:
   - Workflow logs
   - Error message
   - Steps to reproduce

---