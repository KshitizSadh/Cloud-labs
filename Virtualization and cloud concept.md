33.7 HARDWARE                     │     │
│  │   CPU | Memory | Storage | Network            │     │
│  └───────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────┘

Characteristics:
• Runs directly on hardware
• Best performance
• Production environments
• Cloud providers use this


┌─────────────────────────────────────────────────────────┐
│              TYPE 2 HYPERVISOR (Hosted)                 │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  ┌────────┐  ┌────────┐  ┌────────┐                    │
│  │  VM 1  │  │  VM 2  │  │  VM 3  │                    │
│  │  Guest │  │  Guest │  │  Guest │                    │
│  │   OS   │  │   OS   │  │   OS   │                    │
│  └────┬───┘  └────┬───┘  └────┬───┘                    │
│       └───────────┴───────────┘                         │
│                   │                                      │
│  ┌────────────────▼──────────────────────────┐         │
│  │      HYPERVISOR (VirtualBox, VMware      │         │
│  │      Workstation, Parallels)              │         │
│  └────────────────┬──────────────────────────┘         │
│                   │                                      │
│  ┌────────────────▼──────────────────────────┐         │
│  │      HOST OPERATING SYSTEM                │         │
│  │      (Windows, macOS, Linux)              │         │
│  └────────────────┬──────────────────────────┘         │
│                   │                                      │
│  ┌────────────────▼──────────────────────────┐         │
│  │      PHYSICAL HARDWARE                    │         │
│  └───────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────┘

Characteristics:
• Runs on top of host OS
• Lower performance (additional layer)
• Development/testing
• Desktop use cases
```

### 3.7 Container Virtualization

```
┌─────────────────────────────────────────────────────────┐
│      CONTAINERS vs VIRTUAL MACHINES                     │
└─────────────────────────────────────────────────────────┘

VIRTUAL MACHINES:
┌──────────────────────────────────────────────────────────┐
│                   Physical Server                        │
│  ┌────────────────────────────────────────────────────┐ │
│  │              HYPERVISOR                            │ │
│  └──┬──────────┬──────────┬──────────┬───────────────┘ │
│     │          │          │          │                  │
│  ┌──▼───┐   ┌──▼───┐   ┌──▼───┐   ┌──▼───┐           │
│  │ VM 1 │   │ VM 2 │   │ VM 3 │   │ VM 4 │           │
│  ├──────┤   ├──────┤   ├──────┤   ├──────┤           │
│  │ App A│   │ App B│   │ App C│   │ App D│           │
│  │ Bins │   │ Bins │   │ Bins │   │ Bins │           │
│  │ Libs │   │ Libs │   │ Libs │   │ Libs │           │
│  ├──────┤   ├──────┤   ├──────┤   ├──────┤           │
│  │ Guest│   │ Guest│   │ Guest│   │ Guest│           │
│  │  OS  │   │  OS  │   │  OS  │   │  OS  │           │
│  │(1 GB)│   │(1 GB)│   │(1 GB)│   │(1 GB)│           │
│  └──────┘   └──────┘   └──────┘   └──────┘           │
│                                                          │
│  Total: 4+ GB just for OS overhead                      │
└──────────────────────────────────────────────────────────┘

CONTAINERS:
┌──────────────────────────────────────────────────────────┐
│                   Physical Server                        │
│  ┌──────────────────────────────────────────────────┐   │
│  │         HOST OPERATING SYSTEM (Linux)            │   │
│  └──────────────────┬───────────────────────────────┘   │
│                     │                                    │
│  ┌──────────────────▼───────────────────────────────┐   │
│  │    CONTAINER RUNTIME (Docker Engine)             │   │
│  └──┬──────┬──────┬──────┬──────┬──────┬──────┬────┘   │
│     │      │      │      │      │      │      │         │
│  ┌──▼──┐┌──▼──┐┌──▼──┐┌──▼──┐┌──▼──┐┌──▼──┐┌──▼──┐   │
│  │Cont││Cont││Cont││Cont││Cont││Cont││Cont││   │
│  │  1 ││  2 ││  3 ││  4 ││  5 ││  6 ││  7 ││   │
│  ├────┤├────┤├────┤├────┤├────┤├────┤├────┤   │
│  │AppA││AppB││AppC││AppD││AppE││AppF││AppG││   │
│  │Bins││Bins││Bins││Bins││Bins││Bins││Bins││   │
│  │Libs││Libs││Libs││Libs││Libs││Libs││Libs││   │
│  │    ││    ││    ││    ││    ││    ││    ││   │
│  │    ││    ││    ││    ││    ││    ││    ││   │
│  │    ││    ││    ││    ││    ││    ││    ││   │
│  │50MB││60MB││45MB││70MB││55MB││48MB││52MB││   │
│  └────┘└────┘└────┘└────┘└────┘└────┘└────┘   │
│                                                          │
│  Total: ~380 MB for all applications                     │
│  No OS duplication - all share host kernel              │
└──────────────────────────────────────────────────────────┘
```

### 3.8 Container Isolation Mechanisms

```
┌─────────────────────────────────────────────────────────┐
│        LINUX CONTAINER ISOLATION TECHNOLOGIES           │
└─────────────────────────────────────────────────────────┘

1. NAMESPACES (Isolation):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  PID Namespace                                          │
│  ├─ Container sees only its own processes               │
│  ├─ Process ID 1 is container's init                    │
│  └─ Cannot see host or other container processes        │
│                                                          │
│  Network Namespace                                      │
│  ├─ Own network stack (IP, routing table, firewall)    │
│  ├─ Virtual ethernet pairs connect to host              │
│  └─ Isolated from other containers                      │
│                                                          │
│  Mount Namespace                                        │
│  ├─ Own root filesystem (/)                            │
│  ├─ Cannot access host filesystem                       │
│  └─ Volume mounts for persistent data                   │
│                                                          │
│  UTS Namespace                                          │
│  ├─ Own hostname                                        │
│  └─ Container has unique identity                       │
│                                                          │
│  User Namespace                                         │
│  ├─ Root in container ≠ root on host                   │
│  └─ UID/GID mapping for security                       │
│                                                          │
│  IPC Namespace                                          │
│  ├─ Inter-process communication isolation               │
│  └─ Shared memory, semaphores, message queues           │
│                                                          │
└─────────────────────────────────────────────────────────┘

2. CGROUPS (Resource Limits):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  CPU Control                                            │
│  ├─ Limit: container gets max 2 CPU cores               │
│  └─ Shares: proportional allocation                     │
│                                                          │
│  Memory Control                                         │
│  ├─ Limit: 512 MB maximum                              │
│  ├─ OOM killer if exceeded                             │
│  └─ Swap limits                                         │
│                                                          │
│  Disk I/O Control                                       │
│  ├─ Read/write IOPS limits                             │
│  └─ Bandwidth throttling                                │
│                                                          │
│  Network Control                                        │
│  └─ Bandwidth limits per container                      │
│                                                          │
└─────────────────────────────────────────────────────────┘

3. SECURITY:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Capabilities                                           │
│  ├─ Drop dangerous Linux capabilities                   │
│  ├─ Example: CAP_SYS_ADMIN, CAP_NET_RAW               │
│  └─ Least privilege principle                          │
│                                                          │
│  Seccomp (Secure Computing)                            │
│  ├─ Filter system calls                                │
│  └─ Block 300+ syscalls by default                     │
│                                                          │
│  AppArmor / SELinux                                     │
│  ├─ Mandatory Access Control                           │
│  └─ Profile-based restrictions                          │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 3.9 Performance Comparison

```
┌─────────────────────────────────────────────────────────┐
│         VIRTUALIZATION PERFORMANCE METRICS              │
├───────────────┬──────────────┬────────────┬────────────┤
│ Metric        │ Bare Metal   │ VM         │ Container  │
├───────────────┼──────────────┼────────────┼────────────┤
│ Boot Time     │ 30-60s       │ 1-2 min    │ <1 second  │
│ Memory        │ Baseline     │ +1-2 GB    │ +10-50 MB  │
│ CPU Overhead  │ 0%           │ 2-10%      │ <2%        │
│ Disk Size     │ Baseline     │ +1-5 GB    │ +50-500 MB │
│ Density       │ 1 app        │ 10-20 VMs  │ 100s cont. │
│ Isolation     │ Strong       │ Strong     │ Good       │
│ Portability   │ None         │ Medium     │ High       │
│ Startup       │ Slow         │ Slow       │ Instant    │
└───────────────┴──────────────┴────────────┴────────────┘
```

---

## 4. Service Models

### 4.1 Service Model Stack

```
┌─────────────────────────────────────────────────────────┐
│            CLOUD SERVICE MODELS                         │
└─────────────────────────────────────────────────────────┘

┌────────────────┬────────┬────────┬────────┬────────────┐
│                │ On-Prem│  IaaS  │  PaaS  │   SaaS     │
├────────────────┼────────┼────────┼────────┼────────────┤
│ Applications   │  YOU   │  YOU   │  YOU   │  PROVIDER  │
│ Data           │  YOU   │  YOU   │  YOU   │  YOU       │
│ Runtime        │  YOU   │  YOU   │PROVIDER│  PROVIDER  │
│ Middleware     │  YOU   │  YOU   │PROVIDER│  PROVIDER  │
│ OS             │  YOU   │  YOU   │PROVIDER│  PROVIDER  │
│ Virtualization │  YOU   │PROVIDER│PROVIDER│  PROVIDER  │
│ Servers        │  YOU   │PROVIDER│PROVIDER│  PROVIDER  │
│ Storage        │  YOU   │PROVIDER│PROVIDER│  PROVIDER  │
│ Network        │  YOU   │PROVIDER│PROVIDER│  PROVIDER  │
└────────────────┴────────┴────────┴────────┴────────────┘

          ↑               ↑          ↑            ↑
     Most Control    High Control  Low Control  No Control
     Most Work       Medium Work   Low Work     Minimal Work
```

### 4.2 IaaS (Infrastructure as a Service)

```
┌─────────────────────────────────────────────────────────┐
│                    IaaS MODEL                           │
└─────────────────────────────────────────────────────────┘

What You Get:
┌─────────────────────────────────────────────────────────┐
│  • Virtual Machines (compute instances)                 │
│  • Virtual Networks (VPC, subnets)                      │
│  • Storage (block, object, file)                        │
│  • Load Balancers                                       │
│  • IP Addresses                                         │
│  • Firewalls                                            │
└─────────────────────────────────────────────────────────┘

What You Manage:
┌─────────────────────────────────────────────────────────┐
│  • Operating System installation & patches              │
│  • Application installation                             │
│  • Security configuration                               │
│  • Backup strategies                                    │
│  • Scaling decisions                                    │
│  • Monitoring setup                                     │
└─────────────────────────────────────────────────────────┘

Examples:
┌─────────────────────────────────────────────────────────┐
│  AWS:    EC2, VPC, EBS, S3                             │
│  Azure:  Virtual Machines, VNet, Managed Disks         │
│  GCP:    Compute Engine, VPC, Persistent Disks         │
└─────────────────────────────────────────────────────────┘

Use Cases:
┌─────────────────────────────────────────────────────────┐
│  ✓ Migrating existing applications ("lift and shift")  │
│  ✓ Testing and development environments                │
│  ✓ High-performance computing                          │
│  ✓ Big data analysis                                   │
│  ✓ Custom OS/software requirements                     │
│  ✓ Maximum control needed                              │
└─────────────────────────────────────────────────────────┘
```

### 4.3 PaaS (Platform as a Service)

```
┌─────────────────────────────────────────────────────────┐
│                    PaaS MODEL                           │
└─────────────────────────────────────────────────────────┘

What You Get:
┌─────────────────────────────────────────────────────────┐
│  • Runtime environment (Node.js, Python, Java, etc.)   │
│  • Development frameworks                               │
│  • Database services                                    │
│  • Build & deployment tools                            │
│  • Auto-scaling                                         │
│  • Load balancing                                       │
│  • Monitoring & logging                                │
└─────────────────────────────────────────────────────────┘

What You Manage:
┌─────────────────────────────────────────────────────────┐
│  • Application code                                     │
│  • Application configuration                            │
│  • Data & content                                       │
└─────────────────────────────────────────────────────────┘

Examples:
┌─────────────────────────────────────────────────────────┐
│  AWS:    Elastic Beanstalk, App Runner                 │
│  Azure:  App Service, Azure Functions                  │
│  GCP:    App Engine, Cloud Run                         │
│  Other:  Heroku, Cloud Foundry, OpenShift              │
└─────────────────────────────────────────────────────────┘

Typical Workflow:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  1. Write Code                                          │
│     └─ git push origin main                            │
│                                                          │
│  2. Platform Detects                                    │
│     ├─ Language/framework (package.json, requirements) │
│     └─ Builds application automatically                 │
│                                                          │
│  3. Platform Deploys                                    │
│     ├─ Creates containers                              │
│     ├─ Configures load balancer                        │
│     └─ Provides HTTPS endpoint                         │
│                                                          │
│  4. Platform Manages                                    │
│     ├─ Scaling                                         │
│     ├─ Health checks                                   │
│     └─ SSL certificates                                │
│                                                          │
└─────────────────────────────────────────────────────────┘

Use Cases:
┌─────────────────────────────────────────────────────────┐
│  ✓ Web applications                                     │
│  ✓ API backends                                         │
│  ✓ Mobile backends                                      │
│  ✓ Microservices                                        │
│  ✓ Rapid development & deployment                       │
│  ✓ Focus on code, not infrastructure                   │
└─────────────────────────────────────────────────────────┘
```

### 4.4 SaaS (Software as a Service)

```
┌─────────────────────────────────────────────────────────┐
│                    SaaS MODEL                           │
└─────────────────────────────────────────────────────────┘

What You Get:
┌─────────────────────────────────────────────────────────┐
│  • Complete, ready-to-use application                   │
│  • Accessible via web browser or app                    │
│  • Multi-tenant architecture                            │
│  • Automatic updates                                    │
│  • Built-in security                                    │
│  • Managed availability                                 │
└─────────────────────────────────────────────────────────┘

What You Manage:
┌─────────────────────────────────────────────────────────┐
│  • User accounts & permissions                          │
│  • Your data & content                                  │
│  • Configuration/customization                          │
└─────────────────────────────────────────────────────────┘

Examples:
┌─────────────────────────────────────────────────────────┐
│  Email:          Gmail, Outlook 365                     │
│  Productivity:   Google Workspace, Microsoft 365        │
│  CRM:            Salesforce, HubSpot                    │
│  Collaboration:  Slack, Microsoft Teams, Zoom           │
│  Storage:        Dropbox, Box                           │
│  Accounting:     QuickBooks Online, Xero                │
│  HR:             Workday, BambooHR                      │
└─────────────────────────────────────────────────────────┘

Characteristics:
┌─────────────────────────────────────────────────────────┐
│  • Subscription pricing (per user/month)                │
│  • No installation required                             │
│  • Accessible from anywhere                             │
│  • Provider handles all technical aspects               │
│  • Standardized features                                │
│  • Limited customization                                │
└─────────────────────────────────────────────────────────┘
```

### 4.5 Service Model Decision Tree

```
┌─────────────────────────────────────────────────────────┐
│           CHOOSING THE RIGHT SERVICE MODEL              │
└─────────────────────────────────────────────────────────┘

START: What do you need?
    │
    ├─ Ready-made application?
    │  └─→ Use SaaS (e.g., Gmail, Salesforce)
    │
    ├─ Just run your code?
    │  │
    │  ├─ Event-driven/functions?
    │  │  └─→ Use FaaS/Serverless (Lambda, Cloud Functions)
    │  │
    │  └─ Web app/API?
    │     └─→ Use PaaS (Heroku, App Engine)
    │
    ├─ Containers/microservices?
    │  └─→ Use CaaS (EKS, AKS, GKE)
    │
    └─ Need full OS control?
       ├─ Custom kernel/drivers?
       │  └─→ Use IaaS (EC2, Azure VMs)
       │
       └─ Complex networking?
          └─→ Use IaaS
```

---

## 5. Deployment Models

### 5.1 Public Cloud

```
┌─────────────────────────────────────────────────────────┐
│                   PUBLIC CLOUD                          │
└─────────────────────────────────────────────────────────┘

Architecture:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│            CLOUD PROVIDER (AWS/Azure/GCP)               │
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │        Shared Infrastructure                   │    │
│  │                                                 │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐    │    │
│  │  │Customer A│  │Customer B│  │Customer C│    │    │
│  │  │Resources │  │Resources │  │Resources │    │    │
│  │  │          │  │          │  │          │    │    │
│  │  │ • VMs    │  │ • VMs    │  │ • VMs    │    │    │
│  │  │ • Storage│  │ • Storage│  │ • Storage│    │    │
│  │  │ • Network│  │ • Network│  │ • Network│    │    │
│  │  └──────────┘  └──────────┘  └──────────┘    │    │
│  │                                                 │    │
│  │  (Isolated but sharing physical hardware)      │    │
│  └────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘

Characteristics:
✓ Owned & operated by third party
✓ Resources shared across customers
✓ Internet connectivity required
✓ Pay-per-use pricing
✓ Highly scalable
✓ No capital expenditure
✓ Provider manages infrastructure

Examples: AWS, Microsoft Azure, Google Cloud, IBM Cloud

Best For:
• Startups and SMBs
• Variable workloads
• Development/testing
• Cost-sensitive projects
• Global reach needed
```

### 5.2 Private Cloud

```
┌─────────────────────────────────────────────────────────┐
│                   PRIVATE CLOUD                         │
└─────────────────────────────────────────────────────────┘

On-Premises Private Cloud:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│         YOUR COMPANY'S DATA CENTER                      │
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │    Dedicated Infrastructure                    │    │
│  │                                                 │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐    │    │
│  │  │  Dept 1  │  │  Dept 2  │  │  Dept 3  │    │    │
│  │  │Resources │  │Resources │  │Resources │    │    │
│  │  └──────────┘  └──────────┘  └──────────┘    │    │
│  │                                                 │    │
│  │  ┌──────────────────────────────────────┐     │    │
│  │  │  Private Cloud Platform              │     │    │
│  │  │  (VMware, OpenStack, Azure Stack)    │     │    │
│  │  └──────────────────────────────────────┘     │    │
│  │                                                 │    │
│  │  ┌──────────────────────────────────────┐     │    │
│  │  │  Physical Servers & Storage          │     │    │
│  │  └──────────────────────────────────────┘     │    │
│  └────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘

Hosted Private Cloud:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│    THIRD-PARTY DATA CENTER (Dedicated to You)          │
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │  Your Dedicated Hardware & Infrastructure      │    │
│  │  (Managed by provider or yourself)             │    │
│  └────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘

Characteristics:
✓ Dedicated resources (single tenant)
✓ Full control over infrastructure
✓ Enhanced security & compliance
✓ Customizable architecture
✓ Higher capital cost
✓ Company manages or contracts management
✓ Private network connectivity

Best For:
• Financial institutions
• Healthcare (HIPAA compliance)
• Government agencies
• Large enterprises
• Regulatory compliance requirements
• Sensitive data handling
```

### 5.3 Hybrid Cloud

```
┌─────────────────────────────────────────────────────────┐
│                   HYBRID CLOUD                          │
└─────────────────────────────────────────────────────────┘

Architecture:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  ┌──────────────────────────────────────────────┐      │
│  │         PRIVATE CLOUD                        │      │
│  │  (On-premises / Hosted)                      │      │
│  │                                               │      │
│  │  ┌────────────────────────────────┐          │      │
│  │  │ • Sensitive data                │          │      │
│  │  │ • Core databases                │          │      │
│  │  │ • Legacy applications           │          │      │
│  │  │ • Compliance workloads          │          │      │
│  │  └────────────────────────────────┘          │      │
│  └──────────────────┬───────────────────────────┘      │
│                     │                                   │
│         ┌───────────┴──────────┐                        │
│         │   SECURE CONNECTION  │                        │
│         │ • VPN                │                        │
│         │ • Direct Connect     │                        │
│         │ • ExpressRoute       │                        │
│         └───────────┬──────────┘                        │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────┐      │
│  │         PUBLIC CLOUD                         │      │
│  │  (AWS / Azure / GCP)                         │      │
│  │                                               │      │
│  │  ┌────────────────────────────────┐          │      │
│  │  │ • Web applications              │          │      │
│  │  │ • Development/testing           │          │      │
│  │  │ • Burst capacity                │          │      │
│  │  │ • Non-sensitive workloads       │          │      │
│  │  │ • Global content delivery       │          │      │
│  │  └────────────────────────────────┘          │      │
│  └──────────────────────────────────────────────┘      │
│                                                          │
└─────────────────────────────────────────────────────────┘

Example Use Case - E-commerce:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  PRIVATE CLOUD:                                         │
│  • Customer database (PCI compliance)                   │
│  • Payment processing                                   │
│  • Order management system                              │
│  • Inventory database                                   │
│                                                          │
│  PUBLIC CLOUD:                                          │
│  • Website frontend (auto-scales for Black Friday)     │
│  • Image/video storage (CDN delivery)                  │
│  • Analytics & reporting                                │
│  • Development environments                             │
│                                                          │
└─────────────────────────────────────────────────────────┘

Benefits:
✓ Flexibility - right workload, right cloud
✓ Cost optimization - expensive workloads private, burst public
✓ Compliance - sensitive data stays private
✓ Disaster recovery - backup to public cloud
✓ Cloud migration - gradual transition

Challenges:
✗ Complex management
✗ Network latency between clouds
✗ Security configuration complexity
✗ Data synchronization
```

### 5.4 Multi-Cloud

```
┌─────────────────────────────────────────────────────────┐
│                   MULTI-CLOUD                           │
└─────────────────────────────────────────────────────────┘

Architecture:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│              YOUR ORGANIZATION                           │
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │     Multi-Cloud Management Layer               │    │
│  │  (Terraform, CloudHealth, Morpheus)            │    │
│  └─────┬──────────────┬──────────────┬────────────┘    │
│        │              │              │                  │
│  ┌─────▼──────┐ ┌─────▼──────┐ ┌─────▼──────┐         │
│  │    AWS     │ │   Azure    │ │    GCP     │         │
│  │            │ │
