# Creating an Azure Virtual Machine via Portal

## 1. Overview & Management Tools
Azure provides multiple management planes to provision and administer IaaS Virtual Machines:
* **Azure Portal:** Browser-based graphical interface (GUI) designed for interactive deployment, learning wizards, and visual resource monitoring.
* **Command-Line Tools:** Cross-platform orchestration via **Azure CLI** and **Azure PowerShell** (ideal for scripting and automation across Windows, macOS, and Linux).
* **Resource Group Prerequisite:** Every Azure resource must reside in exactly one Resource Group. Using a dedicated resource group for labs simplifies teardown, as deleting the group deletes all associated resources (VM, disks, NIC, IP, NSG) at once.

---

## 2. Core VM Configuration Parameters (Basics Tab)

| Parameter Section | Setting Name | Purpose & Best Practice |
| :--- | :--- | :--- |
| **Project Details** | **Subscription** | Determines billing destination and cost allocation boundaries. |
| | **Resource Group** | Logical container for lifecycle management (e.g., `myResourceGroupName`). |
| **Instance Details** | **Virtual Machine Name** | Defines both the Azure resource name and internal OS hostname (e.g., `test-ubuntu-cus-vm`). |
| | **Region** | Select physically close to target traffic to minimize network latency. |
| | **Availability Options** | Configures fault tolerance (e.g., *No infrastructure redundancy required* for dev/test vs. Availability Sets / Availability Zones for HA). |
| | **Security Type** | Standard vs. Trusted Launch (shielding against boot kits/rootkits). |
| | **Image** | Selected OS base image (e.g., `Ubuntu Server 24.04 LTS - Gen2`). Gen2 brings UEFI boot and larger disk support. |
| | **VM Architecture** | Instruction set: `x64` or `Arm64`. |
| | **Azure Spot Discount** | Unused Azure compute capacity at steep discounts; subject to eviction if capacity is reclaimed (unsuitable for production). |
| | **Size** | Balance of vCPU, RAM, and attached disk limits (e.g., `Standard_D2s_v3`: 2 vCPUs, 8 GiB RAM). |
| **Administrator Account**| **Authentication Type** | **SSH Public Key** (recommended for Linux) or **Password**. |
| | **SSH Key Source** | Options include: *Generate new key pair*, *Use existing public key*, or *Use stored Azure key*. |
| | **Key Pair Name** | Auto-generates an Azure SSH key resource (e.g., `test-ubuntu-cus-vm_key`). |
| **Inbound Port Rules** | **Public Inbound Ports** | Restricts external ingress traffic at initial deployment. |
| | **Select Inbound Ports** | Opens specific perimeter ports on the auto-generated NSG (e.g., Port `22` for SSH). |

---

## 3. Validation, Deployment & Key Management Lifecycle

1. **Pre-flight Validation (`Review + create`):**
   * Azure validates parameters, quota limits, and required fields across all tabs prior to submitting the Azure Resource Manager (ARM) template.
   * If a required field is invalid or missing, Azure flags the specific tab with a red badge.
2. **Private Key Generation Prompt:**
   * When selecting *Generate new key pair*, clicking **Create** triggers the modal: `Download private key and create resource`.
   * **Critical Guardrail:** The private key (`.pem` file) is generated and exposed for download **only once**. If not downloaded immediately, SSH access to the machine using that key identity cannot be recovered.
3. **Deployment Tracking:**
   * Real-time deployment status is observable via the top **Notifications** bell icon or the ARM **Deployment details** pane.
4. **Post-Deployment Verification:**
   * Navigating to the VM **Overview** blade exposes the dynamically allocated **Public IP address**, private IP address, OS disk status, and current state (`Running`).

---

## 4. Connectivity & Access Architecture

* **SSH Connectivity (Port 22):**
  * Inbound Port Rule adds a high-priority rule to the VM's Network Security Group (NSG) allowing TCP traffic on port `22`.
  * Connection command syntax using the downloaded `.pem` key:
    ```bash
    chmod 400 test-ubuntu-cus-vm_key.pem
    ssh -i test-ubuntu-cus-vm_key.pem <username>@<Public-IP-Address>
    ```
* **Security Consideration:** Exposing port 22 directly to the open Internet (`0.0.0.0/0`) presents brute-force risks. Enterprise production deployments leverage **Azure Bastion** or Just-In-Time (JIT) VM access instead of public IP endpoints.

---

## ⚡ Key Traps & Exam Reminders

* **Ephemeral Key Download:** Azure does not store the private key for recovery. If you close the download prompt without saving the `.pem` file, you must reset the SSH credentials via the VM's *Support + troubleshooting* blade.
* **Auto-Created Resources:** Deploying a VM via the portal automatically provisions an OS disk, a VNet, a Subnet, a Public IP, a NIC, and an NSG if existing ones are not explicitly supplied.
* **Cost Cleanup:** Deleting the VM alone does **not** delete the OS disk, Public IP, or VNet. Deleting the entire **Resource Group** guarantees full cost cessation.
