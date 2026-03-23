# Practical 07 — Install OpenStack

---

## 📌 Objective

Set up a local OpenStack environment for practicing private cloud infrastructure deployment and management.

---

## 🧠 Conceptual Background (Know-How)

### What is OpenStack?

**OpenStack** is an open-source cloud operating system that controls large pools of compute, storage, and networking resources throughout a data center. It is the most widely used platform for building private clouds — essentially giving you your own AWS-like environment on your own hardware.

### OpenStack vs AWS

```
┌──────────────────────────────────────────────────────┐
│         OpenStack ↔ AWS Service Mapping              │
├─────────────────────┬────────────────────────────────┤
│    OpenStack        │           AWS                  │
├─────────────────────┼────────────────────────────────┤
│ Nova                │ EC2 (Compute)                  │
│ Neutron             │ VPC (Networking)               │
│ Cinder              │ EBS (Block Storage)            │
│ Swift               │ S3 (Object Storage)            │
│ Glance              │ AMI / EC2 Image Service        │
│ Keystone            │ IAM (Identity)                 │
│ Horizon             │ AWS Management Console         │
│ Heat                │ CloudFormation                 │
│ Ceilometer          │ CloudWatch                     │
│ Barbican            │ Secrets Manager                │
└─────────────────────┴────────────────────────────────┘
```

### OpenStack Architecture

```
                        ┌──────────────────┐
                        │     Horizon      │  ← Web Dashboard (UI)
                        │  (Dashboard)     │
                        └────────┬─────────┘
                                 │
                        ┌────────▼─────────┐
                        │    Keystone      │  ← Identity & Auth
                        │  (Identity)      │
                        └────────┬─────────┘
                                 │
           ┌─────────────────────┼─────────────────────┐
           │                     │                     │
   ┌───────▼──────┐    ┌─────────▼────┐    ┌───────────▼──┐
   │    Nova      │    │   Neutron    │    │    Glance    │
   │  (Compute)   │    │ (Networking) │    │   (Images)   │
   └───────┬──────┘    └─────────┬────┘    └──────────────┘
           │                     │
   ┌───────▼──────┐    ┌─────────▼────┐    ┌──────────────┐
   │   Cinder     │    │    Swift     │    │     Heat     │
   │   (Block     │    │   (Object    │    │(Orchestration│
   │   Storage)   │    │   Storage)   │    │              │
   └──────────────┘    └──────────────┘    └──────────────┘
```

### Installation Options

```
1. DevStack (Recommended for learning)
   → All-in-one script-based installation
   → Runs on a single VM or bare metal server
   → Resets on reboot (development only)
   → Best for learning and testing

2. Packstack (RHEL/CentOS)
   → Puppet-based installer
   → More persistent than DevStack
   → Good for single-node and multi-node

3. MicroStack (Ubuntu Snap)
   → Simplest installation
   → sudo snap install microstack
   → Good for quick demos

4. Kolla-Ansible (Production)
   → Docker-container based deployment
   → Production-grade, multi-node
   → Complex but scalable

5. Manual Installation
   → Component by component
   → Best for deep understanding
   → Time-consuming
```

---

## 🛠️ Installation Guide

### System Requirements

**Minimum for DevStack (All-in-One):**
```
CPU:     4 cores (8 recommended)
RAM:     8 GB minimum (16 GB recommended)
Disk:    50 GB free
OS:      Ubuntu 22.04 LTS (recommended)
Network: Internet access for downloading packages
```

**Hypervisor Check (Nested Virtualization):**
```bash
# Check if KVM is available (on your host machine)
egrep -c '(vmx|svm)' /proc/cpuinfo
# If output is > 0, KVM is supported

# Enable nested virtualization (Intel)
sudo modprobe -r kvm_intel
sudo modprobe kvm_intel nested=1
echo "options kvm-intel nested=1" | sudo tee /etc/modprobe.d/kvm-intel.conf
```

---

### Method 1 — MicroStack (Quickest, Ubuntu only)

```bash
# Install MicroStack via snap
sudo snap install microstack --beta

# Initialize MicroStack (all-in-one)
sudo microstack init --auto --control

# Check installation status
sudo microstack.openstack service list

# Get admin password
sudo snap get microstack config.credentials.keystone-password

# Access Horizon dashboard
# URL: http://10.20.20.1
# User: admin
# Password: (from above command)
```

---

### Method 2 — DevStack (Recommended for Learning)

#### 1. Prepare a Fresh Ubuntu 22.04 System/VM

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y git python3-pip
```

#### 2. Create the Stack User (Required by DevStack)

```bash
# Create dedicated user
sudo useradd -s /bin/bash -d /opt/stack -m stack

# Give sudo privileges without password (required for DevStack)
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack

# Switch to stack user
sudo su - stack
```

#### 3. Clone DevStack

```bash
# Clone DevStack from the stable branch
git clone https://opendev.org/openstack/devstack -b stable/2024.1
cd devstack
```

#### 4. Create the Configuration File

```bash
cat > local.conf <<'EOF'
[[local|localrc]]
# ─── Admin Passwords ──────────────────────────────
ADMIN_PASSWORD=secret123
DATABASE_PASSWORD=secret123
RABBIT_PASSWORD=secret123
SERVICE_PASSWORD=secret123

# ─── Network Settings ─────────────────────────────
HOST_IP=$(hostname -I | awk '{print $1}')  # Auto-detect your IP
FLOATING_RANGE=192.168.1.224/27
FIXED_RANGE=10.11.12.0/24
FIXED_NETWORK_SIZE=256
FLAT_INTERFACE=eth0   # Change to your network interface (check with: ip addr)

# ─── Services to Enable ───────────────────────────
# Core services (enabled by default)
# ENABLED_SERVICES=key,rabbit,mysql,horizon
# ENABLED_SERVICES+=,n-api,n-crt,n-obj,n-cpu,n-cond,n-sch
# ENABLED_SERVICES+=,placement-api
# ENABLED_SERVICES+=,q-svc,q-agt,q-dhcp,q-l3,q-meta
# ENABLED_SERVICES+=,c-api,c-vol,c-sch,c-bak
# ENABLED_SERVICES+=,g-api,g-reg

# ─── Log Settings ─────────────────────────────────
LOGFILE=/opt/stack/logs/stack.sh.log
VERBOSE=True
LOG_COLOR=True

# ─── Image Settings ───────────────────────────────
# Download and use Cirros (tiny test image, ~12MB)
IMAGE_URLS="http://download.cirros-cloud.net/0.6.2/cirros-0.6.2-x86_64-disk.img"
EOF
```

> 💡 **Check your network interface name:**
> ```bash
> ip addr show | grep -E "^[0-9]+:" | awk '{print $2}' | sed 's/://'
> # Common names: eth0, ens33, enp0s3, ens3
> # Update FLAT_INTERFACE in local.conf accordingly
> ```

#### 5. Run the DevStack Installer

```bash
# This takes 20-45 minutes
./stack.sh

# Watch the logs in another terminal:
tail -f /opt/stack/logs/stack.sh.log
```

**What to expect:**
```
[Step 1/X] Installing prerequisite packages...
[Step 2/X] Setting up MySQL database...
[Step 3/X] Installing Keystone...
[Step 4/X] Installing Glance...
[Step 5/X] Installing Nova...
[Step 6/X] Installing Neutron...
[Step 7/X] Installing Cinder...
[Step 8/X] Installing Horizon...
...
DevStack finished at [time]
Keystone is serving at http://10.0.2.15/identity/
Horizon is now available at http://10.0.2.15/dashboard
The default users are: admin and demo
The password: secret123
```

#### 6. Access Horizon Dashboard

```
URL:      http://<your-vm-ip>/dashboard
Username: admin
Password: secret123  (or whatever you set in local.conf)
```

---

### Method 3 — Packstack (CentOS/Rocky Linux)

```bash
# Install RDO repository
sudo dnf install -y centos-release-openstack-yoga

# Update packages
sudo dnf update -y

# Install Packstack
sudo dnf install -y openstack-packstack

# Generate the answer file
packstack --gen-answer-file /root/answers.cfg

# Edit key settings in answers.cfg:
# CONFIG_KEYSTONE_ADMIN_PW=admin123
# CONFIG_DEFAULT_PASSWORD=admin123
# CONFIG_HORIZON_SSL=n
# CONFIG_NEUTRON_L2_AGENT=linuxbridge

# Run the installation
packstack --answer-file /root/answers.cfg
# Takes 15-30 minutes
```

---

## 🔍 Post-Installation Verification

```bash
# Source the admin credentials
source /opt/stack/openrc admin admin
# or
source /etc/openstack/admin-openrc.sh

# Verify services are running
openstack service list
openstack endpoint list

# Check compute nodes
openstack compute service list

# Check network agents
openstack network agent list

# List available images
openstack image list

# List available flavors (instance sizes)
openstack flavor list
```

**Expected service list output:**
```
+----+----------+----------+
| ID | Name     | Type     |
+----+----------+----------+
|  1 | keystone | identity |
|  2 | glance   | image    |
|  3 | nova     | compute  |
|  4 | neutron  | network  |
|  5 | cinder   | volume   |
|  6 | heat     | ..       |
+----+----------+----------+
```

---

## 🚨 Common Issues and Fixes

```bash
# Issue: stack.sh fails partway through
# Fix: Re-run it — DevStack is mostly idempotent
./stack.sh

# Issue: Services not starting after reboot
# Fix: DevStack doesn't persist across reboots
cd /opt/stack/devstack
./rejoin-stack.sh   # Re-attach to existing screen sessions

# Issue: Horizon not loading
sudo systemctl restart apache2

# Issue: "Connection refused" on dashboard
# Check if services are running
sudo systemctl status devstack@*

# Issue: Nova can't launch VMs (KVM not available)
# Check virtualization type
virt-host-validate
# If no KVM, VMs will run as QEMU (much slower but functional)
```

---

## ✅ Learning Outcomes

After completing this practical, you should be able to:

- [ ] Explain what OpenStack is and how it compares to AWS
- [ ] Map OpenStack components to AWS equivalents
- [ ] Install OpenStack using DevStack or MicroStack
- [ ] Access the Horizon dashboard
- [ ] Use the OpenStack CLI with sourced credentials
- [ ] Verify that all core OpenStack services are running
- [ ] Understand the role of each core OpenStack component

---

## 📚 Further Reading

- [OpenStack Documentation](https://docs.openstack.org/)
- [DevStack Documentation](https://docs.openstack.org/devstack/)
- [OpenStack Components Overview](https://www.openstack.org/software/)
- [OpenStack vs AWS Deep Dive](https://docs.openstack.org/keystone/latest/)
