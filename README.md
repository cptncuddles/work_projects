# Dell Command | Update (DCU) Automated Installation Script

An enterprise-ready PowerShell script designed for Remote Monitoring and Management (RMM) platforms (this script was specifically deployed with Datto RMM) to automatically scan, download, and apply Dell driver and firmware updates on target client endpoints.

---

## 📌 Overview

This script automates the full lifecycle of Dell Command | Update execution on managed Windows endpoints:
1. **Directory Preparation:** Creates local logging paths if missing.
2. **CLI Detection:** Dynamically locates the `dcu-cli.exe` executable across standard and legacy installation directories.
3. **Service Verification:** Ensures the `DellClientManagementService` is configured to `delayed-auto` start and actively running prior to invoking the CLI.
4. **Automated Update Execution:** Triggers update installation with auto-suspended BitLocker and suppressed automatic reboots.
5. **Exit Code Mapping:** Evaluates DCU return codes to identify update status, reboot requirements, or failure conditions (e.g., running on battery power).
6. **User Notification & RMM Integration:** Prompts active user sessions via `msg.exe` if a reboot is needed and outputs environment flags compatible with RMM reboot policies.

---

## 🚀 Script Workflow

```
[Start Script]
      │
      ▼
Ensure C:\ProgramData\Dell Directory Exists
      │
      ▼
Locate dcu-cli.exe Path
      │
      ├──> (Not Found) ──> Log Error ──> Exit 1
      │
      ▼
Verify DellClientManagementService
      │
      ├──> (Disabled / Stopped) ──> Config & Start Service
      │
      ▼
Execute dcu-cli.exe (/applyUpdates -autoSuspendBitLocker=enable -reboot=disable -silent)
      │
      ▼
Evaluate Exit Codes (0, 1, 2, 4, 5, 6)
      │
      ├──> (Reboot Required) ──> Dispatch msg.exe alert to user & set SET_REBOOT_REQUIRED=True
      │
      ▼
Log Results & Exit 0
```

---

## ⚙️ Parameters & Configuration Details

### Execution Arguments Passed to DCU CLI

| Argument | Purpose |
| :--- | :--- |
| `/applyUpdates` | Scans, downloads, and applies all applicable updates. |
| `-autoSuspendBitLocker=enable` | Automatically suspends BitLocker drive encryption to prevent recovery prompts during BIOS/firmware flashes. |
| `-reboot=disable` | Prevents DCU from automatically forcing an immediate system reboot upon completion. |
| `-silent` | Runs headlessly without displaying the graphical user interface (GUI). |
| `-outputLog=$env:ProgramData\Dell\dcu_log.log` | Directs raw DCU command logs to a centralized local directory. |

---

## 📋 DCU Exit Code Reference

The script captures and logs specific exit codes returned by `dcu-cli.exe`:

| Code | Status | Action / Script Handling |
| :---: | :--- | :--- |
| **0** | Success | Updates applied successfully. No reboot required. |
| **1** | Reboot Required | Updates applied; sets reboot flag and notifies user. |
| **2** | No Updates Found | System is up to date; no changes made. |
| **4** | Reboot Required | Updates applied; sets reboot flag and notifies user. |
| **5** | Pending Reboot | Reboot is pending from a prior installation; flags retry status. |
| **6** | Ineligible / Battery | System is on DC (battery) power. AC power is required by DCU. |
| **Default**| Unexpected Code | Logs warning with specific exit code for analysis. |

---

## 🛠️ Prerequisites

* **Operating System:** Windows 10 / 11 (Dell Client Hardware)
* **Software:** Dell Command | Update (Universal or Classic version) pre-installed on the endpoint.
* **Privileges:** Administrator / `SYSTEM` privilege (required for service modification and driver installation).

---

## 📂 File Logging & Output Locations

* **DCU CLI Log:** `C:\ProgramData\Dell\dcu_log.log`
* **Script Exception Error Log:** `C:\ProgramData\Dell\dell_log.log`

---

## 🔧 RMM Deployment Instructions (Datto RMM)

1. Create a new Component in your RMM platform.
2. Set the **Script Type** to `PowerShell`.
3. Set the **Execution Context** to `System`.
4. Paste the script content into the code block.
5. (Optional) Configure an RMM Reboot Policy rule to monitor for `SET_REBOOT_REQUIRED=True` in the stdout log to trigger downstream reboot prompts.

## IMPORTANT NOTES ##

If the version of DCU installed is not the most current version this script will not pull the most up to date firmware updates.
