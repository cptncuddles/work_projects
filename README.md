# Dell SupportAssist CLI Silent Installer

A PowerShell script designed to automate the silent deployment of **Dell SupportAssist for Business PCs** across managed endpoints. The script handles runtime dependency checks, registry-based idempotency checks, download execution, and automatic cleanup.

---

## 📌 Overview

This script streamlines and automates the SupportAssist installation process:
1. **Dependency Verification:** Checks for .NET Desktop Runtime 8 and dynamically installs or upgrades required .NET components if missing.
2. **Idempotency Check:** Scans 32-bit and 64-bit Windows Registry uninstall keys to verify if SupportAssist is already present before running.
3. **Automated Download:** Fetches the official standalone business installer directly from Dell's mirror using optimized download parameters.
4. **Silent Installation:** Invokes the `.exe` bootstrapper with MSI engine flags (`/s /v"/qn /norestart ..."`), outputting detailed verbosity logs to the temporary directory.
5. **Post-Install Cleanup:** Automatically removes temporary installer files regardless of script outcome.

---

## 🚀 Script Workflow

```
[Start Script]
      │
      ▼
Check .NET Desktop Runtime 8 Availability
      │
      ├──> (Missing) ──> Download & Run Microsoft dotnet-install.ps1
      │
      ▼
Scan Registry for Existing SupportAssist Installation
      │
      ├──> (Already Installed) ──> Log Message & Exit
      │
      ▼
Download SupportAssist Business Installer to $env:TEMP
      │
      ▼
Execute Silent Install with Verbose Logging
      │
      ▼
Evaluate Exit Code
      │
      ├──> Exit Code 0 ──> Output Success
      └──> Non-Zero   ──> Log Error & Point to $env:TEMP Log File
      │
      ▼
[Finally Block] Clean Up Temporary .exe File
```

---

## ⚙️ Configuration Parameters

| Variable | Description | Default Value |
| :--- | :--- | :--- |
| `$DownloadUrl` | Official mirror URI for Dell SupportAssist Business standalone installer. | `https://downloads.dell.com/serviceability/catalog/SupportAssistBusinessInstaller.exe` |
| `$LocalPath` | Staging location for the downloaded executable. | `$env:TEMP\SupportAssistBusinessInstaller.exe` |
| `$LogPath` | Destination for detailed MSI installation logs. | `$env:TEMP\SupportAssistInstallLog.txt` |
| `$installArgs` | Silent execution and log switches passed to the installer package. | `/s /v"/qn /norestart /l*v "$LogPath""` |
| `$searchName` | Wildcard match string used during registry search. | `*SupportAssist*` |

---

## 🛠️ Prerequisites & System Requirements

* **Operating System:** Windows 10 / 11 (64-bit recommended) running on Dell enterprise/business hardware.
* **Privileges:** Administrator or `SYSTEM` execution rights.
* **Network Access:** Unrestricted outbound HTTPS access to `downloads.dell.com` and `dot.net`.
* **Dependencies:** .NET Desktop Runtime 8 (automatically handled by script if missing).

---

## 📂 File & Logging Locations

* **Staging Executable:** `%TEMP%\SupportAssistBusinessInstaller.exe` *(deleted post-install)*
* **Detailed Install Log:** `%TEMP%\SupportAssistInstallLog.txt`

---

## 🔧 Deployment Notes

* **RMM Integration:** Ready for execution via Datto RMM, Intune, or PowerShell script policies running under the `SYSTEM` context.
* **MSI Alternate:** If your environment uses custom TechDirect MSI packages, modify the `$DownloadUrl` and update `$installArgs` to standard `msiexec.exe /i` syntax (`/qn /norestart`).
