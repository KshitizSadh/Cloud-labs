
# Launch Your First Instance on **Google Cloud Platform (GCP)**

*(OpenStack equivalent on GCP)*

## Objective

To create and launch a **Virtual Machine (VM)** on **Google Cloud Platform (GCP)** with minimum cost.

---

## Mapping: OpenStack → GCP

| OpenStack Concept | GCP Equivalent           |
| ----------------- | ------------------------ |
| Project           | GCP Project              |
| User & Roles      | IAM Users & Roles        |
| Glance Image      | Compute Engine Images    |
| Flavor            | Machine Type             |
| Horizon / CLI     | GCP Console / gcloud CLI |
| Instance          | VM Instance              |

---

## STEP 1: Create a GCP Project (Equivalent to OpenStack Project)

### Steps (Console):

1. Go to **[https://console.cloud.google.com](https://console.cloud.google.com)**
2. Click **Project Selector → New Project**
3. Project name: `vm-lab-demo`
4. Organization: *None* (student account)
5. Click **Create**
<img width="805" height="363" alt="image" src="https://github.com/user-attachments/assets/cfe7b82b-0261-409b-bddd-5df6030bc77b" />


### Cost Minimization:

* ✔ Only **one project**
* ✔ Delete unused projects later

---

## STEP 2: Enable Billing (Mandatory)

1. Go to **Billing**
2. Link your **Free Trial account**

   * GCP gives **$300 free credits**
3. Enable billing for the project

### Cost Minimization:

* ✔ Free credits cover everything
* ✔ Set **billing alerts**:

  * Billing → Budgets → Create budget
  * Alert at **₹100 / $1 usage**

---
<img width="1647" height="402" alt="image" src="https://github.com/user-attachments/assets/0859aa8c-4fb3-4f01-81b9-8dcee5f3e820" />


## STEP 3: IAM – Create User & Assign Roles

*(OpenStack: assign roles to users)*

### Steps:

1. Go to **IAM & Admin → IAM**
2. Click **Grant Access**
3. Add user email
4. Assign role:

   * `Compute Admin` (for VM)
   * OR safer: `Compute Instance Admin (v1)`
5. Save

### Cost Minimization:

* ✔ Do **NOT** give Owner role
* ✔ Use least-privilege roles

---
<img width="1642" height="308" alt="image" src="https://github.com/user-attachments/assets/0e59a932-165a-4461-a257-7ee1edfd8197" />

## STEP 4: Choose & Prepare Image

*(OpenStack Glance equivalent)*

### Best Low-Cost Image:

* **Ubuntu 20.04 LTS**
* **Ubuntu 22.04 LTS**

### Steps:

1. Go to **Compute Engine → VM Instances**
2. Click **Create Instance**
3. In **Boot Disk → Change**
4. Select:

   * OS: Ubuntu
   * Version: Ubuntu 20.04 LTS
5. Boot disk type:

   * **Standard Persistent Disk**
6. Disk size:

   * **10 GB**

### Cost Minimization:

* ✔ Avoid custom images
* ✔ Use **standard disk**, not SSD
* ✔ 10 GB is enough

---


## STEP 5: Select Machine Type (Flavor Equivalent)

### Recommended **FREE / CHEAP** Options:

#### Best Choice (Almost Free):

```
Machine type: e2-micro
vCPU: 0.25–2 shared
RAM: 1 GB
```

> This qualifies for **Always Free Tier** (in some regions).

### Steps:

1. Machine family: **E2**
2. Series: **e2-micro**

### Cost Minimization:

* ✔ Use **e2-micro only**
* ✔ Never use n2 or c2 (expensive)
* ✔ No GPU

---

## STEP 6: Networking (Keep It Simple)

1. Network: **default**
2. External IP:

   * ✔ **Ephemeral**
3. Firewall:

   * ✔ Allow HTTP ❌
   * ✔ Allow HTTPS ❌
   * ✔ Allow SSH ✔

### Cost Minimization:

* ✔ No load balancer
* ✔ No static IP
* ✔ No extra VPC

---

## STEP 7: Create the VM Instance

1. Review configuration
2. Click **Create**
3. VM will start in **30–60 seconds**

🎉 VM successfully launched!

---
<img width="1312" height="166" alt="image" src="https://github.com/user-attachments/assets/602bb331-1149-422b-9a33-5777e8955713" />
## STEP 8: Connect to VM (No Cost)

### Method 1: Browser SSH (Best)

1. Click **SSH** button
2. Opens terminal in browser

### Method 2: CLI

```bash
gcloud compute ssh vm-lab-demo
```
<img width="929" height="575" alt="image" src="https://github.com/user-attachments/assets/09b2551f-5f38-4ed0-a1ae-479968c7e630" /> 

### Cost Minimization:

* ✔ Browser SSH (no data egress cost)

---

## STEP 9: Verify VM

Inside VM:

```bash
lsb_release -a
free -h
df -h
```

---

## STEP 10: Stop / Delete VM (VERY IMPORTANT)

### Stop VM (Temporary):

```bash
Compute Engine → VM Instances → Stop
```

* CPU billing stops
* Disk still charged (small)

### Delete VM (Best):

```bash
Delete instance
```

### Cost Minimization:

* ✔ Delete VM after lab
* ✔ Delete disks & snapshots
* ✔ Delete firewall rules if created

---

## CLI Version (Optional – Exam Ready)

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

## COST SUMMARY (Realistic)

| Resource          | Cost           |
| ----------------- | -------------- |
| e2-micro VM       | ₹0 (Free Tier) |
| Disk (10 GB)      | ~₹10/month     |
| SSH               | Free           |
| Total (1 day lab) | ₹0–₹1          |

---

## EXAM / PRACTICAL CONCLUSION

**Conclusion:**
A virtual machine was successfully created and launched on **Google Cloud Platform** using **Compute Engine**. The instance was configured using an Ubuntu image and a low-cost e2-micro machine type while following best practices to minimize cost.

---

If you want, I can:

* Convert this into **PDF / Word lab record**
* Write **viva questions & answers**
* Give **OpenStack vs GCP comparison table**
* Help you do the **same lab on AWS / Azure**

Just tell me 👍
EXPERIMN
