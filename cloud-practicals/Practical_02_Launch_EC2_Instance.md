# Practical 02 — Launch Your First Amazon EC2 Instance

---

## 📌 Objective

Deploy a virtual machine on AWS using Amazon EC2, configure security groups for SSH access, and connect to the instance remotely via SSH.

---

## 🧠 Conceptual Background (Know-How)

### What is Amazon EC2?

**Amazon Elastic Compute Cloud (EC2)** provides scalable virtual computing capacity in the cloud. Each running EC2 virtual machine is called an **instance**. You can think of it as renting a computer in Amazon's data center.

### Core EC2 Concepts

```
┌─────────────────────────────────────────────────────────┐
│                     EC2 Ecosystem                       │
│                                                         │
│  ┌──────────┐    ┌──────────┐    ┌───────────────────┐  │
│  │   AMI    │───▶│ Instance │───▶│   Security Group  │  │
│  │(Template)│    │(Running  │    │  (Virtual         │  │
│  │          │    │  VM)     │    │   Firewall)        │  │
│  └──────────┘    └────┬─────┘    └───────────────────┘  │
│                       │                                 │
│                  ┌────▼─────┐    ┌───────────────────┐  │
│                  │   EBS    │    │    Key Pair       │  │
│                  │ (Storage │    │  (SSH Auth)       │  │
│                  │  Volume) │    │                   │  │
│                  └──────────┘    └───────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition |
|---|---|
| **AMI** | Amazon Machine Image — a pre-configured OS template used to launch instances |
| **Instance Type** | The hardware specification (CPU, RAM, network) — e.g., t2.micro |
| **Security Group** | Acts as a virtual firewall controlling inbound/outbound traffic |
| **Key Pair** | RSA key pair for SSH authentication (you keep the private key) |
| **EBS** | Elastic Block Store — persistent storage volume attached to your instance |
| **Elastic IP** | A static public IPv4 address you can associate with your instance |
| **Region** | Geographic location of the data center (e.g., us-east-1 = N. Virginia) |
| **Availability Zone** | Isolated data centers within a region (e.g., us-east-1a, us-east-1b) |

### EC2 Instance Type Naming Convention

```
  t  2  . micro
  │  │    └── Size (nano, micro, small, medium, large, xlarge, 2xlarge...)
  │  └─────── Generation (2, 3, 4...)
  └────────── Family:
              t = Burstable (general purpose, good for free tier)
              m = General purpose
              c = Compute optimized
              r = Memory optimized
              g = GPU instances
              i = Storage optimized
```

### What is an AMI?

An **AMI (Amazon Machine Image)** is a snapshot of an OS with optional pre-installed software. Think of it as a USB bootable drive image for your VM.

- **Amazon Linux 2** — Lightweight, AWS-optimized, based on RHEL/CentOS, great for servers
- **Ubuntu** — Popular Linux distro, large community support
- **Windows Server** — For Windows workloads (extra licensing cost)
- **Community/Marketplace AMIs** — Pre-configured stacks (WordPress, LAMP, etc.)

---

## 🛠️ Step-by-Step Guide

### Prerequisites
- AWS account (free tier)
- SSH client: Terminal (Linux/Mac) or PuTTY/Windows Terminal (Windows)
- Basic Linux command-line familiarity

---

### Step A — Launch an EC2 Instance from the AWS Console

1. Sign in to [AWS Management Console](https://console.aws.amazon.com)
2. In the search bar, type **EC2** and click the service
3. Ensure you are in the correct region (top-right corner) — use **us-east-1** for free tier
4. Click **Launch instance**

**Configure the instance as follows:**

#### Name and Tags
```
Name: my-first-ec2
```

#### Application and OS Images (AMI)
- Select **Amazon Linux 2023 AMI** (or Amazon Linux 2)
- Architecture: **64-bit (x86)**
- This is **free tier eligible**

#### Instance Type
```
t2.micro  →  1 vCPU, 1 GB RAM  →  Free tier eligible
```

> On newer regions, use **t3.micro** if t2.micro isn't available

#### Key Pair (Login)
- Click **Create new key pair**
- Key pair name: `my-ec2-keypair`
- Key pair type: **RSA**
- Private key format:
  - `.pem` → for Linux/Mac (OpenSSH)
  - `.ppk` → for Windows (PuTTY)
- Click **Create key pair** — the `.pem` file downloads automatically
- **Store this file safely — you cannot download it again!**

#### Network Settings
- VPC: default
- Subnet: No preference (any AZ)
- Auto-assign public IP: **Enable**
- Firewall (Security Groups): **Create security group**
  - Security group name: `my-ec2-sg`
  - Allow SSH traffic: ✅ from **My IP** (not 0.0.0.0/0 in production)

#### Configure Storage
```
8 GiB  gp3  Root volume  (Free tier: up to 30 GB)
```

#### Summary
- Number of instances: **1**
- Click **Launch instance**

---

### Step B — Understanding the AMI (Amazon Linux 2)

Amazon Linux 2 key characteristics:
```bash
# Package manager
sudo yum install <package>       # Amazon Linux 2
sudo dnf install <package>       # Amazon Linux 2023

# Default user
ec2-user    # Amazon Linux / RHEL-based
ubuntu      # Ubuntu AMIs
admin       # Debian AMIs

# System info
cat /etc/os-release
uname -a
```

---

### Step C — Configure Security Groups to Allow SSH Access

A **Security Group** is a stateful virtual firewall. Rules define what traffic is allowed.

**SSH Inbound Rule:**
```
Type:        SSH
Protocol:    TCP
Port Range:  22
Source:      My IP  (e.g., 203.0.113.25/32)
Description: Allow SSH from my workstation
```

> ⚠️ **Never set Source to 0.0.0.0/0 for SSH in production** — it opens your instance to the entire internet (brute-force attacks)

**Security Group Rule Logic:**
```
Inbound Rules:
┌──────────┬──────────┬────────┬─────────────────────────┐
│ Protocol │   Port   │ Source │       Description        │
├──────────┼──────────┼────────┼─────────────────────────┤
│   TCP    │    22    │ My IP  │ SSH Access               │
│   TCP    │   80     │ 0.0.0.0│ HTTP (if hosting a site) │
│   TCP    │  443     │ 0.0.0.0│ HTTPS                    │
└──────────┴──────────┴────────┴─────────────────────────┘
Outbound Rules:
│   All    │   All    │ 0.0.0.0│ Allow all outbound       │
```

**To modify security group after launch:**
```
EC2 Dashboard → Instances → Select instance →
Security tab → Security groups → Edit inbound rules
```

---

### Step D — Connect to the Instance Using SSH

#### 1. Get the Public IP Address
```
EC2 → Instances → Select your instance →
Public IPv4 address: e.g., 54.123.45.67
```

#### 2. Set Correct Permissions on the Key File (Linux/Mac)
```bash
chmod 400 my-ec2-keypair.pem
```
> Without this, SSH will reject the key with a "permissions too open" error

#### 3. SSH Command
```bash
ssh -i /path/to/my-ec2-keypair.pem ec2-user@54.123.45.67
```

**Generic syntax:**
```bash
ssh -i <path-to-key.pem> <username>@<public-ip-or-dns>
```

**Default usernames by AMI:**
| AMI | Username |
|---|---|
| Amazon Linux / Amazon Linux 2 | `ec2-user` |
| Ubuntu | `ubuntu` |
| CentOS | `centos` or `ec2-user` |
| Debian | `admin` |
| RHEL | `ec2-user` or `root` |

#### 4. Accept the Host Fingerprint
```
The authenticity of host '54.123.45.67 (54.123.45.67)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxx
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```
Type `yes` and press Enter.

#### 5. You Are Now Connected!
```bash
       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

[ec2-user@ip-172-31-xx-xx ~]$
```

#### 6. Basic Commands to Explore the Instance
```bash
# System info
whoami                    # Shows current user
hostname                  # Shows hostname
cat /etc/os-release       # OS details
uname -r                  # Kernel version

# Hardware resources
nproc                     # Number of CPUs
free -h                   # RAM usage
df -h                     # Disk usage
lscpu                     # CPU details

# Network info
curl ifconfig.me          # Shows public IP
ip addr show              # Shows network interfaces

# Update packages
sudo yum update -y        # Amazon Linux 2
sudo dnf update -y        # Amazon Linux 2023

# Install something
sudo yum install -y httpd  # Install Apache web server
sudo systemctl start httpd
sudo systemctl enable httpd
```

#### 7. Connecting from Windows (PuTTY)
```
1. Open PuTTY
2. Host Name: ec2-user@54.123.45.67
3. Connection → SSH → Auth → Credentials
   → Private key file: browse to .ppk file
4. Click Open
```

---

## 📊 EC2 Instance States

```
           ┌─────────┐
           │ pending │  ← Instance is starting up
           └────┬────┘
                │
           ┌────▼────┐
      ┌────│ running │────┐
      │    └─────────┘    │
      │                   │
 ┌────▼──────┐     ┌──────▼─────┐
 │ stopping  │     │  rebooting  │
 └────┬──────┘     └─────────────┘
      │
 ┌────▼──────┐
 │  stopped  │  ← You're not billed for compute (EBS still charged)
 └────┬──────┘
      │
 ┌────▼──────────┐
 │  terminated   │  ← Permanently deleted, EBS deleted (default)
 └───────────────┘
```

> 💡 **Stop vs Terminate:**
> - **Stop** → Like shutting down your PC. Can restart. Not billed for compute hours.
> - **Terminate** → Deletes the instance permanently (and EBS by default).

---

## 💰 Cost Awareness

```
Free Tier: 750 hours/month of t2.micro for 12 months
750 hrs ÷ 24 hrs = 31.25 days → You can run ONE t2.micro 24/7 for free

If you run TWO t2.micro instances simultaneously:
750 ÷ 2 = 375 hours free → ~15 days before charges begin

Always STOP or TERMINATE unused instances!
```

**Set a billing alert:**
```
AWS Console → Billing → Budgets → Create budget →
Cost budget → $5 threshold → Email notification
```

---

## ✅ Learning Outcomes

After completing this practical, you should be able to:

- [ ] Launch an EC2 instance from the AWS Management Console
- [ ] Understand AMI types and select the appropriate one
- [ ] Decode EC2 instance type naming (family, generation, size)
- [ ] Create a key pair and understand its role in authentication
- [ ] Configure security groups with principle of least privilege
- [ ] SSH into an EC2 instance from Linux/Mac/Windows
- [ ] Understand instance states (running, stopped, terminated)
- [ ] Estimate costs and set billing alerts

---

## 🔐 Security Best Practices

1. Use **My IP** as SSH source, never `0.0.0.0/0`
2. Rotate key pairs periodically
3. Use **IAM roles** for application access to AWS services (not hardcoded keys)
4. Enable **CloudTrail** to log all API actions on your account
5. Consider **Session Manager (SSM)** as a replacement for SSH — no open port 22 needed

---

## 📚 Further Reading

- [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/index.html)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [Amazon Machine Images](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
- [Security Groups Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
