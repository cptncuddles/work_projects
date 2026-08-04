# USB Portable Network Printer Installer

An interactive PowerShell deployment tool designed for desktop technicians to rapidly install network printers on Windows endpoints directly from a portable USB drive without hardcoded drive letters or manual driver selection.

---

## 📌 Overview

This script streamlines manual on-site printer deployment by automating driver staging, port provisioning, and printer mapping:
1. **Dynamic USB Path Resolution:** Uses `$PSScriptRoot` to dynamically locate local driver repository directories on removable media regardless of assigned drive letters.
2. **Pre-Configured Driver Hashtable:** Maps common enterprise printer brands (HP, Kyocera, Konica Minolta, Ricoh) directly to their respective INF directory subfolders and exact Windows Print Spooler driver strings.
3. **Interactive Validation Loops:** Guides technicians through input prompts with built-in regex pattern validation for IPv4 addresses and null-string protection.
4. **Targeted Driver Staging:** Uses `PnPUtil.exe` to stage only the required driver INF package into the Windows Driver Store, followed by Print Spooler registration (`Add-PrinterDriver`).
5. **Standard TCP/IP Port & Object Creation:** Provisions Standard TCP/IP ports (Port 9100) and maps the printer object (`Add-Printer`) with real-time color-coded terminal progress.

---

## 🚀 Script Workflow

```
[Start Script]
      │
      ▼
Resolve USB Driver Path via $PSScriptRoot (\Drivers)
      │
      ├──> (Missing) ──> Throw Critical Error & Prompt Exit
      │
      ▼
Prompt Technician Inputs & Validate
  ├── Brand Lookup (Match against $driverMap hashtable)
  ├── IPv4 Address (Regex: '^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$')
  └── Display Name (Non-empty check)
      │
      ▼
Check If Printer Object ($printerName) Exists
      │
      ├──> (Exists) ──> Display Warning & Abort Execution
      │
      ▼
Check Print Spooler Driver Registration ($targetDriverName)
      │
      ├──> (Not Registered) ──> Stage INF via PnPUtil & Run Add-PrinterDriver
      └──> (Registered)     ──> Skip Driver Import
      │
      ▼
Check TCP/IP Port ($portName = "IP_$printerIP")
      │
      ├──> (Missing) ──> Create Port via Add-PrinterPort (Port 9100)
      └──> (Exists)  ──> Use Existing Port
      │
      ▼
Create Printer Object via Add-Printer
      │
      ├──> (Success) ──> Display Success Banner
      └──> (Failure) ──> Output Catch Error Details
      │
      ▼
[Finally Block] Pause for Technician Review ("Press Enter to finish...")
```

---

## 📁 USB Directory Structure

To ensure relative paths resolve correctly, structure the USB drive as follows:

```text
USB_ROOT:│
├── Install-Printer.ps1          <-- Main PowerShell Script
└── Drivers\                     <-- Driver Repository Directory
    ├── HP_Universal\            <-- HP INF package files
    ├── Kyocera_KX\              <-- Kyocera INF package files
    ├── KM_PCL\                  <-- Konica Minolta INF package files
    └── Ricoh_PCL\               <-- Ricoh INF package files
```

---

## ⚙️ Mapped Driver Configuration ($driverMap)

The script includes built-in mappings for standard universal print drivers:

| Brand Key | Mapped Spooler Driver Name | USB Target Subfolder |
| :--- | :--- | :--- |
| **HP** | `HP Universal Printing PCL 6` | `\Drivers\HP_Universal` |
| **Kyocera** | `Kyocera Classic Universaldriver PCL6 (A4)` | `\Drivers\Kyocera_KX` |
| **Konica Minolta** | `KONICA MINOLTA Universal PCL` | `\Drivers\KM_PCL` |
| **Ricoh** | `PCL6 V4 Driver for Universal Print` | `\Drivers\Ricoh_PCL` |

---

## 🛠️ Prerequisites & Operating Requirements

* **Operating System:** Windows 10, Windows 11, or Windows Server.
* **Privileges:** Run as Administrator (required for `PnPUtil` driver staging and TCP/IP port creation).
* **Execution Context:** Executed directly from the root of the USB drive.

---

## 🔧 Technician Usage Instructions

1. Insert the USB drive into the target Windows endpoint.
2. Open PowerShell as an **Administrator**.
3. Navigate to the USB drive and run the script:
   ```powershell
   .\Install-Printer.ps1
   ```
4. Follow the interactive prompts:
   * Select a brand from the displayed pre-configured list.
   * Input the target printer's IPv4 address.
   * Input the designated local display name (e.g., `COR-KYO4052ci`).
5. Review completion status and press **Enter** to close the window.

