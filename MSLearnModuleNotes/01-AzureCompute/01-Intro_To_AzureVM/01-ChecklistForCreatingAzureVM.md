# Compile a checklist for creating an Azure Virtual Machine

## 1. Core Architecture & Supporting Resources
An Azure IaaS VM is never deployed in isolation; it consists of an interconnected resource ecosystem:
* **Compute (VM):** The virtualized hardware slice (vCPU and RAM allocation).
* **Virtual Network (VNet) & Subnets:** The private communication isolation boundary.
* **Network Interface Card (NIC):** Bridges the VM to the subnet (NIC count is constrained by the chosen VM size; NICs carry no additional direct cost).
* **IP Addresses:** 
  * **Private IP:** Default internal addressing within the VNet.
  * **Public IP:** Optional routable address for inbound/outbound internet connectivity.
* **Network Security Group (NSG):** Stateful firewall evaluating inbound and outbound traffic rules. **Zero licensing cost.**
* **Disks:** OS disk, temporary scratch disk, and optional data disks.

---

## 2. Pre-Deployment Checklist

### A. Network Architecture (Top Priority)
* **Private IP Address Space:** Utilizes RFC 1918 non-routable CIDR blocks (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`).
* **Non-Overlapping Requirement:** VNet address ranges cannot overlap with peered networks or on-premises networks (re-addressing post-deployment is disruptive).
* **Reserved IP Rule:** Azure reserves **5 IP addresses** per subnet:
  * First 4 addresses: Network address, Default Gateway, and two Azure DNS/mapping IPs.
  * Last 1 address: Subnet broadcast address.
  * *Calculation:* Usable IPs = `2^(32 - CIDR) - 5`.
* **Subnet Routing:** By default, all subnets inside a VNet communicate freely with each other. Network Security Groups (NSGs) are required to restrict inter-subnet traffic.

### B. VM Naming Constraints
* **Character Limits:** 
  * **Windows OS:** Up to **15 characters** (NetBIOS constraint).
  * **Linux OS:** Up to **64 characters**.
* **Standard Naming Convention:** `[Environment]-[Location]-[App/Role]-[Instance]` (e.g., `deveus-webvm01`).

### C. Region & Location Considerations
* **Latency:** Position resources geographically closest to the user base.
* **Compliance & Sovereignty:** Align with regional data protection and legal mandates.
* **SKU Availability:** Not all hardware families or disk types exist in all regions.
* **Price Variance:** Compute and storage unit prices fluctuate significantly across regions.

---

## 3. VM Sizing & Workload Categories

| Category | CPU:RAM Ratio | Typical Workload Profile |
| :--- | :--- | :--- |
| **General Purpose** | Balanced (1:4) | Dev/test environments, low-to-medium traffic web servers, small databases |
| **Compute Optimized** | High CPU : Low RAM | Batch processing, web application servers, network virtual appliances |
| **Memory Optimized** | Low CPU : High RAM | Relational databases (SQL/Oracle), large in-memory caches, analytics |
| **Storage Optimized** | High IOPS / Throughput | Big Data nodes, NoSQL databases, transactional data warehouses |
| **GPU** | Specialized Video/Compute | Machine learning model training, AI inferencing, 3D graphics rendering |
| **High Performance (HPC)** | High-clock CPUs + InfiniBand | Scientific simulations, financial modeling, weather simulations |

### Resizing Rules
* **Running VM Resize:** Constrained strictly to sizes supported on the physical hardware cluster currently hosting the VM. **Triggers an automatic reboot.**
* **Deallocated VM Resize:** Putting the VM into **Stopped (Deallocated)** status releases cluster-level hardware reservations, unlocking any available size across the entire region.

---

## 4. Azure Managed Disks Comparison

* **OS Disk:** Defaults to 127 GiB for most general images (contains the boot sector and OS).
* **Temporary Disk:** Ephemeral scratch disk (typically labeled `D:` on Windows or `/dev/sdb1` on Linux). **Free of charge**, but all data is wiped on VM relocation or host failure.
* **Data Disks:** Dedicated volumes attached to isolate application data and logs from the operating system. Maximum attachment limits scale proportionally with VM vCPU capacity (generally 2 disks per vCPU).

| Disk Type | Media | Max Capacity | Max IOPS | Max Throughput | OS Disk Capable? | Target Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Ultra Disk** | NVMe SSD | 65,536 GiB | 160,000 | 4,000 MB/s | **No** | SAP HANA, mission-critical extreme transaction DBs |
| **Premium SSD v2** | SSD | 65,536 GiB | 80,000 | 1,200 MB/s | **No** | Sub-millisecond latency production workloads |
| **Premium SSD** | SSD | 32,767 GiB | 20,000 | 900 MB/s | **Yes** | Standard production enterprise environments |
| **Standard SSD** | SSD | 32,767 GiB | 6,000 | 750 MB/s | **Yes** | Dev/test, entry-level web tiers |
| **Standard HDD** | Magnetic | 32,767 GiB | 2,000 | 500 MB/s | **Yes** | Non-critical backups, cold data, audit archives |

---

## 5. Billing & Cost Architecture

The total cost of an Azure Virtual Machine separates into two decoupled meters:

```
Total VM Cost = Compute Billing (Per-Second) + Storage Allocation (Continuous)
```

* **Compute Charges:**
  * Billed per second of uptime (reflected per minute on billing reports).
  * **Stopped (Deallocated)** status drops the compute charge to **$0.00**.
  * Linux instances have lower base rates as they carry no Windows Server licensing fees.
  * **Azure Hybrid Benefit (AHB):** Allows existing on-premises Windows Server or Linux (RHEL/SLES) licenses with Software Assurance to offset VM base pricing.
  * **Billing Models:**
    * **Pay-As-You-Go:** No commitments, dynamic scale, ideal for unpredictable or temporary runs.
    * **Reserved Instances (RI):** 1-year or 3-year term commitments yielding up to **72% savings**.

* **Storage Charges:**
  * Managed disks (OS + data disks) continue to incur provisioned storage costs regardless of whether the VM is running or deallocated.

---

## 6. Operating System & Image Options

* **Base Marketplace Images:** Standard vendor OS templates (Canonical Ubuntu, Red Hat, Windows Server).
* **Packaged Solutions:** Turnkey stack images available directly from Azure Marketplace (e.g., WordPress, pre-configured SQL Server instances).
* **Azure Compute Gallery:** Central enterprise management tool used to store, version, control access to, and globally replicate generalized custom images.

---

## Key Traps & Exam Reminders

* **`Stopped` vs. `Stopped (Deallocated)`:** Shutting down an Azure VM from within the guest OS sets the state to `Stopped`, leaving compute resources allocated and **still billing you**. You must issue the shutdown via Azure Portal/CLI to enter `Stopped (Deallocated)`.
* **Subnet Sizing Trap:** A `/29` subnet has 8 total binary addresses, but because Azure reserves 5, only **3 usable host IPs** remain (`8 - 5 = 3`).
* **Ephemeral Data Disk:** Never host databases or stateful persistence on the local/temporary drive; host migrations or hypervisor updates permanently clear this disk.
* **NSG Cost:** Network Security Groups are standard software-defined constructs within the SDN fabric and incur zero additional licensing fees.
