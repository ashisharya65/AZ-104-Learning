# Describe the options available to create and manage an Azure Virtual Machine

## 1. Overview: Beyond the Azure Portal
While the Azure Portal offers an intuitive GUI for individual deployments, scaling to dozens or hundreds of VMs requires programmatic, automated, and declarative tools to eliminate human error and achieve repeatable deployments.

---

## 2. Infrastructure as Code (IaC) & Declarative Templates

### A. Azure Resource Manager (ARM) Templates
* **Format:** JSON files defining infrastructure elements declaratively (what to deploy, not how).
* **Idempotency:** Redeploying an existing template updates resources to match the desired state without re-creating unchanged components.
* **Parameterization:** Isolates dynamic values (e.g., VM name, subnet, admin user) into parameter files to redeploy across Dev, Test, and Production environments seamlessly.
* **Export Template:** Accessible directly from any resource blade via **Automation > Export template** to reverse-engineer running infrastructure into code.

### B. Terraform (HashiCorp)
* **Configuration Syntax:** HashiCorp Configuration Language (HCL).
* **Multi-Cloud Focus:** Uses the **AzureRM provider** to define Azure infrastructure alongside third-party services.
* **Core Workflow:**
  1. `terraform plan`: Generates an execution plan to preview infrastructure changes before provisioning.
  2. `terraform apply`: Executes verified modifications against the Azure APIs.

---

## 3. Command-Line & Scripting Tools

| Tool | Engine & Mechanics | Primary Syntax / Cmdlets | Ideal Use Case |
| :--- | :--- | :--- | :--- |
| **Azure CLI** | Cross-platform, Python-based CLI (`az`). Output formatting supports JSON, TSV, and Table. Integrates easily with Bash/Python. | `az vm create \<br>  --resource-group TestRG \<br>  --name test-vm \<br>  --image Ubuntu2204 \<br>  --generate-ssh-keys` | Ad-hoc administration, Linux/macOS environments, CI/CD Bash pipelines |
| **Azure PowerShell** | Modular cmdlets running on PowerShell core. Outputs structured .NET objects instead of plain text streams. | `New-AzVm ` <br>  `-ResourceGroupName "TestRG" ` <br>  `-Name "test-vm" ` <br>  `-Location "East US" ` <br>  `-Image Debian11 ` <br>  `-OpenPorts 22` | Windows-centric shops, complex enterprise task orchestration, script pipelines passing object properties |

---

## 4. Programmatic Interfaces (Developer Integration)

Used when VM provisioning must be embedded directly inside bespoke software, workflow engines, or enterprise web portals.

### A. Azure REST API
* Provides HTTP URI endpoints mapped to standard operations (`GET`, `PUT`, `POST`, `DELETE`, `PATCH`).
* Standardized, platform-agnostic baseline upon which all Azure developer tooling sits.

### B. Azure Client SDKs
* Strongly-typed abstraction layers wrapping underlying REST endpoints.
* Available for **.NET (C#), Java, Node.js, Python, Go, PHP, Ruby**.
* Eliminates boilerplate HTTP client coding, token renewal logic, and payload serialization.

---

## 5. Post-Deployment Configuration & Extensions

### Azure VM Extensions
* **Definition:** Small applications/agents deployed post-boot that run tasks inside the guest OS automatically.
* **Primary Capabilities:**
  * Post-deployment configuration (e.g., executing scripts, installing IIS/Apache, Chef/Puppet hooks).
  * Antivirus/security monitoring agent injection (e.g., Microsoft Defender for Cloud).
  * Diagnostics and performance log pipeline routing.

---

## 6. Azure Automation & Operations Management

Centralized enterprise services designed to reduce administrative overhead and streamline lifecycle ops.

### A. Core Automation Pillars
* **Process Automation:** Employs runbooks (PowerShell / Python) and watcher tasks that automatically remediate events or run maintenance schedules across datacenter assets.
* **Configuration Management:** Tracks guest OS configurations, software changes, inventory shifts, and integrates with systems like Microsoft Endpoint Configuration Manager.
* **Update Management:** Schedules, assesses, and orchestrates OS security updates and patches for mixed fleets (both Windows and Linux) from a centralized console.

### B. Auto-Shutdown (Cost Governance Feature)
* **Function:** Automatically transitions an Azure VM into a `Stopped (Deallocated)` state on a predetermined schedule.
* **Configuration Location:** VM blade under the **Operations** section (`Operations > Auto-shutdown`).
* **Settings:** Specific time, target time zone, daily recurrence, and optional webhook/email notification 15 minutes before trigger.
* **Key Advantage:** Ensures non-production dev/test environments cease compute-hour consumption outside standard business hours.

---

## ⚡ Key Traps & Exam Reminders

* **Portal vs. CLI Parity:** While the Portal provides pre-flight parameter guidance, all operations ultimately call the Azure REST API; scripts and templates execute the same underlying ARM engine operations.
* **VM Extension Failure:** An extension execution failure (such as an invalid bash script in a Custom Script Extension) causes Azure Resource Manager to mark the entire VM deployment as failed, even if the VM OS itself is booted and healthy.
* **Auto-Shutdown Is One-Way:** The Auto-shutdown feature provides an automated shutdown mechanism, but does **not** include an automatic scheduled boot/start feature. To schedule auto-start, an Azure Automation Runbook or Azure Logic App is required.
