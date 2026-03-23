# Practical 08 — Launch Your First Instance on OpenStack

---

## 📌 Objective

Create a virtual machine (VM) using OpenStack by creating a project, assigning roles to users, uploading a cloud image to Glance, defining a flavor, and launching an instance via the Horizon dashboard or CLI.

---

## 🧠 Conceptual Background (Know-How)

### OpenStack VM Launch Flow

```
 Before launching a VM, you need:
 
 1. Project (Tenant)     → Logical container for resources
         │
 2. User + Role         → Who can access what in the project
         │
 3. Image (Glance)      → OS disk image (like AWS AMI)
         │
 4. Flavor              → Hardware spec (CPU, RAM, disk) — like EC2 instance type
         │
 5. Network (Neutron)   → Which network the VM connects to
         │
 6. Security Group      → Firewall rules for the VM
         │
 7. Key Pair            → SSH key for authentication
         │
         ▼
 8. Launch Instance → Nova schedules on hypervisor → VM created
```

### OpenStack Projects and Users

In OpenStack, **projects** (also called **tenants**) are used to group and isolate resources. Each user can belong to multiple projects with different roles.

**Default Roles:**
| Role | Permissions |
|---|---|
| admin | Full access to all resources across all projects |
| member | Standard user — can manage own resources within a project |
| reader | Read-only access within a project |

### OpenStack Image Service (Glance)

**Glance** stores and provides disk images for instance creation. It supports:

| Format | Description |
|---|---|
| qcow2 | QEMU Copy-On-Write — most common for OpenStack |
| raw | Raw disk image |
| vmdk | VMware format |
| vhd | Hyper-V format |
| iso | CD/DVD ISO format |

### OpenStack Flavors

A **flavor** defines the virtual hardware profile for an instance:

```
Flavor: m1.small
  ├── vCPUs: 1
  ├── RAM:   2 GB
  └── Disk:  20 GB root disk

This is equivalent to EC2 instance types (t2.micro, m5.large, etc.)
```

---

## 🛠️ Step-by-Step Guide

### Prerequisites
- OpenStack installed (from Practical 07)
- Access to Horizon dashboard or CLI
- Admin credentials: `source /opt/stack/openrc admin admin`

---

### Step A — Create a Project and Assign Roles to Users

#### Via Horizon Dashboard

**Create a New Project:**
```
Horizon → Identity → Projects → Create Project
  Name:               CloudLab-Project
  Description:        Student cloud computing lab environment
  Enabled:            ✅
  
Quotas tab (optional):
  Instances:          10
  VCPUs:              20
  RAM (MB):           51200
  Floating IPs:       5
  
→ Create Project
```

**Create a New User:**
```
Horizon → Identity → Users → Create User
  Username:           student01
  Email:              student01@lab.local
  Password:           Student@123
  Primary Project:    CloudLab-Project
  Role:               member
→ Create User
```

**Assign Role to User in Project:**
```
Horizon → Identity → Projects → Find CloudLab-Project →
  Edit → Project Members tab →
  Add "student01" → Role: "member"
→ Save
```

#### Via CLI

```bash
# Source admin credentials
source /opt/stack/openrc admin admin

# Create a project
openstack project create \
  --description "Student cloud computing lab" \
  --enable \
  CloudLab-Project

# Create a user
openstack user create \
  --project CloudLab-Project \
  --password "Student@123" \
  --email student01@lab.local \
  --enable \
  student01

# Assign member role
openstack role add \
  --project CloudLab-Project \
  --user student01 \
  member

# Verify
openstack project list
openstack user list
openstack role assignment list --project CloudLab-Project
```

---

### Step B — Upload an Image to Glance

#### Option 1 — Use Pre-downloaded Cirros (Already in DevStack)

```bash
# Cirros is pre-loaded by DevStack
openstack image list
# Should show: cirros-0.6.2-x86_64-disk
```

#### Option 2 — Upload Ubuntu Cloud Image

**Download from Ubuntu Cloud Images:**
```bash
# Navigate to the images directory
cd /tmp

# Download Ubuntu 22.04 Cloud Image (minimal, ~600MB)
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img

# Verify the download
file jammy-server-cloudimg-amd64.img
# Should show: QEMU QCOW2 Image
```

**Upload via CLI:**
```bash
# Upload the image to Glance
openstack image create \
  --container-format bare \
  --disk-format qcow2 \
  --file /tmp/jammy-server-cloudimg-amd64.img \
  --public \
  --min-ram 512 \
  --min-disk 8 \
  "Ubuntu 22.04 LTS"

# Verify
openstack image list
openstack image show "Ubuntu 22.04 LTS"
```

**Upload via Horizon:**
```
Horizon → Project → Compute → Images → Create Image
  Image Name:          Ubuntu 22.04 LTS
  Image Description:   Ubuntu Jammy cloud image
  File:                Browse → select jammy-server-cloudimg-amd64.img
  Format:              QCOW2
  Architecture:        x86_64
  Minimum Disk (GB):   8
  Minimum RAM (MB):    512
  Visibility:          Public
→ Create Image
```

**Image upload output:**
```
+------------------+--------------------------------------+
| Field            | Value                                |
+------------------+--------------------------------------+
| container_format | bare                                 |
| disk_format      | qcow2                                |
| id               | a1b2c3d4-xxxx-xxxx-xxxx-xxxxxxxxxxxx |
| min_disk         | 8                                    |
| min_ram          | 512                                  |
| name             | Ubuntu 22.04 LTS                     |
| size             | 629145600                            |
| status           | active                               |
| visibility       | public                               |
+------------------+--------------------------------------+
```

---

### Step C — Define a Flavor

#### List Existing Flavors

```bash
openstack flavor list

# Default DevStack flavors:
# m1.tiny   → 1 vCPU, 512MB RAM, 1GB disk
# m1.small  → 1 vCPU, 2GB RAM, 20GB disk
# m1.medium → 2 vCPU, 4GB RAM, 40GB disk
# m1.large  → 4 vCPU, 8GB RAM, 80GB disk
# m1.xlarge → 8 vCPU, 16GB RAM, 160GB disk
```

#### Create a Custom Flavor

```bash
# Create a lab-specific flavor
openstack flavor create \
  --vcpus 1 \
  --ram 1024 \
  --disk 10 \
  --public \
  lab.small

# Create a minimal flavor for testing
openstack flavor create \
  --vcpus 1 \
  --ram 512 \
  --disk 5 \
  --public \
  lab.tiny

# Verify
openstack flavor show lab.small
```

**Via Horizon:**
```
Horizon → Admin → Compute → Flavors → Create Flavor
  Name:         lab.small
  vCPUs:        1
  RAM (MB):     1024
  Root Disk:    10
  Ephemeral:    0
  Swap:         0
  Public:       ✅
→ Create Flavor
```

---

### Step D — Launch an Instance

#### Prerequisites: Create Security Group and Key Pair

**Create a Security Group:**
```bash
# Create security group
openstack security group create \
  --description "Allow SSH and ICMP" \
  lab-sg

# Allow SSH
openstack security group rule create \
  --protocol tcp \
  --dst-port 22 \
  --remote-ip 0.0.0.0/0 \
  lab-sg

# Allow ICMP (ping)
openstack security group rule create \
  --protocol icmp \
  --remote-ip 0.0.0.0/0 \
  lab-sg

# Allow HTTP
openstack security group rule create \
  --protocol tcp \
  --dst-port 80 \
  --remote-ip 0.0.0.0/0 \
  lab-sg
```

**Create a Key Pair:**
```bash
# Generate a key pair and save locally
openstack keypair create lab-keypair > ~/.ssh/lab-keypair.pem
chmod 400 ~/.ssh/lab-keypair.pem

# Or import an existing public key
ssh-keygen -t rsa -b 2048 -f ~/.ssh/openstack-key
openstack keypair create \
  --public-key ~/.ssh/openstack-key.pub \
  lab-keypair
```

**Get the network ID:**
```bash
openstack network list
# Note the ID of the 'private' network
NETWORK_ID=$(openstack network show private -f value -c id)
```

#### Launch via CLI

```bash
# Launch the instance
openstack server create \
  --flavor lab.small \
  --image "Ubuntu 22.04 LTS" \
  --network $NETWORK_ID \
  --security-group lab-sg \
  --key-name lab-keypair \
  --wait \
  my-first-instance

# Monitor the launch
openstack server show my-first-instance
openstack server list
```

**Instance states during launch:**
```
BUILD → ACTIVE  (success)
BUILD → ERROR   (failure — check nova logs)
```

**View the console log:**
```bash
openstack console log show my-first-instance
```

**Get VNC console access:**
```bash
openstack console url show \
  --novnc my-first-instance
# Open the URL in a browser for graphical console access
```

#### Launch via Horizon Dashboard

```
Horizon → Project → Compute → Instances → Launch Instance

Step 1 — Details:
  Instance Name:    my-first-instance
  Description:      My first OpenStack VM
  Count:            1

Step 2 — Source:
  Select Boot Source:     Image
  Create New Volume:      No
  ✅ Ubuntu 22.04 LTS → Click ↑ to add it to the Allocated list

Step 3 — Flavor:
  ✅ lab.small → Click ↑ to allocate

Step 4 — Networks:
  ✅ private → Click ↑ to allocate

Step 5 — Security Groups:
  Remove 'default' (optional)
  ✅ lab-sg → Click ↑ to allocate

Step 6 — Key Pair:
  ✅ lab-keypair → Click ↑ to allocate

Step 7 — Launch Instance
```

---

## 🔍 Verification

```bash
# Check instance status
openstack server list

# Expected output:
+--------------------------------------+------------------+--------+...
| ID                                   | Name             | Status |...
+--------------------------------------+------------------+--------+...
| a1b2c3d4-xxxx-xxxx-xxxx-xxxxxxxxxxxx | my-first-instance| ACTIVE |...
+--------------------------------------+------------------+--------+...

# Get instance details
openstack server show my-first-instance

# Get console log
openstack console log show my-first-instance | tail -20
```

---

## ✅ Learning Outcomes

After completing this practical, you should be able to:

- [ ] Create and manage OpenStack projects
- [ ] Create users and assign roles using RBAC
- [ ] Upload OS images to the Glance image service
- [ ] Define custom flavors for specific workloads
- [ ] Create security groups and key pairs
- [ ] Launch instances using both Horizon and CLI
- [ ] Interpret instance status and console logs

---

## 📚 Further Reading

- [OpenStack Nova (Compute)](https://docs.openstack.org/nova/latest/)
- [OpenStack Glance (Image Service)](https://docs.openstack.org/glance/latest/)
- [Ubuntu Cloud Images](https://cloud-images.ubuntu.com/)
- [OpenStack CLI Reference](https://docs.openstack.org/python-openstackclient/latest/)
