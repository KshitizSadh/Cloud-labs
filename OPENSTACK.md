Here is a **clean, polished, submission-ready rewrite** of your content.
I’ve **kept everything technically identical**, but improved:

* Language (formal, exam-friendly)
* Flow and clarity
* Consistency in headings
* Minor grammatical fixes
* Professional tone suitable for **GitHub + college practical submission**

You can replace your existing README with this.

---

# 🚀 Launching a Virtual Machine on Google Cloud Platform (GCP)

*(OpenStack-equivalent concepts implemented using GCP)*

---

## 🎯 Objective

To create and launch a **Virtual Machine (VM)** on **Google Cloud Platform (GCP)** using **Compute Engine**, while applying **cost-optimization best practices**.

---

## 🔁 Concept Mapping: OpenStack → GCP

| OpenStack Concept | Google Cloud Platform Equivalent |
| ----------------- | -------------------------------- |
| Project           | GCP Project                      |
| User & Roles      | IAM Users & Roles                |
| Glance Image      | Compute Engine Images            |
| Flavor            | Machine Type                     |
| Horizon / CLI     | GCP Console / `gcloud` CLI       |
| Instance          | VM Instance                      |

---

## STEP 1: Create a GCP Project

*(Equivalent to an OpenStack Project)*

### Steps (Using GCP Console):

1. Open **[https://console.cloud.google.com](https://console.cloud.google.com)**
2. Click **Project Selector → New Project**
3. Enter project name: `vm-lab-demo`
4. Organization: *None* (student/free account)
5. Click **Create**

📸 *Screenshot attached*

### Cost Minimization:

* ✔ Use **only one project**
* ✔ Delete unused projects after completion

---

## STEP 2: Enable Billing (Mandatory)

1. Navigate to **Billing**
2. Link your **Free Trial billing account**

   * GCP provides **$300 free credits**
3. Enable billing for the project

📸 *Screenshot attached*

### Cost Minimization:

* ✔ Free credits are sufficient for this lab
* ✔ Set **billing alerts**:

  * Billing → Budgets → Create Budget
  * Alert at **₹100 / $1 usage**

---

## STEP 3: Configure IAM – Users and Roles

*(Equivalent to assigning roles in OpenStack)*

### Steps:

1. Go to **IAM & Admin → IAM**
2. Click **Grant Access**
3. Add user email
4. Assign one of the following roles:

   * `Compute Admin`, or
   * `Compute Instance Admin (v1)` *(recommended for least privilege)*
5. Click **Save**

📸 *Screenshot attached*

### Cost Minimization:

* ✔ Avoid assigning **Owner** role
* ✔ Follow **least-privilege principle**

---

## STEP 4: Select and Configure VM Image

*(Equivalent to OpenStack Glance Image)*

### Recommended Low-Cost Images:

* **Ubuntu 20.04 LTS**
* **Ubuntu 22.04 LTS**

### Steps:

1. Navigate to **Compute Engine → VM Instances**
2. Click **Create Instance**
3. Under **Boot Disk**, click **Change**
4. Select:

   * OS: Ubuntu
   * Version: Ubuntu 20.04 LTS
5. Configure disk:

   * Disk type: **Standard Persistent Disk**
   * Disk size: **10 GB**

### Cost Minimization:

* ✔ Avoid custom images
* ✔ Use **standard disk** instead of SSD
* ✔ 10 GB disk is sufficient

---

## STEP 5: Choose Machine Type

*(Equivalent to OpenStack Flavor)*

### Recommended Low-Cost Configuration:

```
Machine type: e2-micro
vCPU: Shared (0.25–2)
RAM: 1 GB
```

> Eligible for **GCP Always Free Tier** in supported regions.

### Steps:

1. Machine family: **E2**
2. Series: **e2-micro**

### Cost Minimization:

* ✔ Use **e2-micro only**
* ✔ Avoid N2 / C2 series
* ✔ Do not attach GPUs

---

## STEP 6: Configure Networking

1. Network: **default**
2. External IP: **Ephemeral**
3. Firewall rules:

   * ❌ Allow HTTP
   * ❌ Allow HTTPS
   * ✔ Allow SSH

### Cost Minimization:

* ✔ No load balancer
* ✔ No static IP
* ✔ No additional VPCs

---

## STEP 7: Create the VM Instance

1. Review all configurations
2. Click **Create**
3. VM initializes within **30–60 seconds**

🎉 **Virtual Machine successfully launched**

📸 *Screenshot attached*

---

## STEP 8: Connect to the VM

### Method 1: Browser-Based SSH (Recommended)

1. Click **SSH** button
2. Terminal opens in browser

### Method 2: Command Line Interface

```bash
gcloud compute ssh vm-lab-demo
```

📸 *Screenshot attached*

### Cost Minimization:

* ✔ Browser SSH avoids data egress charges

---

## STEP 9: Verify VM Configuration

Run the following commands inside the VM:

```bash
lsb_release -a
free -h
df -h
```

---

## STEP 10: Stop or Delete the VM (IMPORTANT)

### Stop VM (Temporary)

* Compute Engine → VM Instances → **Stop**
* CPU billing stops, disk billing continues

### Delete VM (Recommended)

* Select instance → **Delete**

### Cost Minimization:

* ✔ Delete VM after lab completion
* ✔ Remove attached disks and snapshots
* ✔ Remove unused firewall rules

---

## CLI-Only Method (Optional – Exam Ready)

```bash
gcloud compute instances create vm-demo \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --image-family=ubuntu-2004-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=10GB
```

To delete:

```bash
gcloud compute instances delete vm-demo --zone=us-central1-a
```

---

## 💰 Cost Summary

| Resource          | Approximate Cost |
| ----------------- | ---------------- |
| e2-micro VM       | ₹0 (Free Tier)   |
| 10 GB Disk        | ~₹10/month       |
| SSH Access        | Free             |
| Total (1-day lab) | ₹0 – ₹1          |

---

## 🧾 Conclusion (Exam-Ready)

**Conclusion:**
A virtual machine was successfully created and deployed on **Google Cloud Platform** using **Compute Engine**. The instance was configured with an Ubuntu operating system and a low-cost **e2-micro** machine type, following best practices to ensure minimal operational cost.

---
