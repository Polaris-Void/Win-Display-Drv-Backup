<div align="center">

# Display Driver Backup & Restore

**One-click Windows batch scripts to back up and restore third-party display (GPU) drivers using PnPUtil.**

![Platform](https://img.shields.io/badge/platform-Windows%2010%20%7C%2011-0078D6?logo=windows&logoColor=white)
![Language](https://img.shields.io/badge/language-Batch%20%2B%20PowerShell-4EAA25?logo=powershell&logoColor=white)

English | [فارسی](README.FA.md)

</div>

---

## Overview

Graphics driver updates sometimes go wrong: black screens, lower performance, broken control panels, or a clean Windows install with no working GPU driver. **Display Driver Backup & Restore** gives you a simple safety net.

- `Backup_Display_Driver.bat` exports your currently installed third-party **Display class** drivers (NVIDIA, AMD, Intel, etc.) into a local folder.
- `Restore_Display_Driver.bat` reinstalls those saved drivers whenever you need them.

No installation, no extra tools, no dependencies. Everything uses components already built into Windows.

## Features

- One-click operation: just double-click the script
- Automatic Administrator elevation (UAC prompt), no need to "Run as administrator" manually
- Exports **only** third-party Display class drivers (`oem*.inf`), not your whole driver store
- Restore installs every driver package found in the backup folder, including subfolders
- Clear status messages (`[INFO]`, `[WARN]`, `[ERROR]`, `[SUCCESS]`) and meaningful exit codes
- Works offline
- Plain text scripts you can read and audit in a minute

## Requirements

| Requirement | Details |
|---|---|
| Operating system | Windows 10 or Windows 11 |
| Permissions | Administrator (requested automatically) |
| Tools | `pnputil` and Windows PowerShell 5.1 (both included with Windows) |

## Quick Start

### 1. Back up your current driver

1. Download or clone this repository.
2. Double-click **`Backup_Display_Driver.bat`**.
3. Accept the UAC prompt.
4. Wait for `[SUCCESS] Display drivers backed up successfully.`

### 2. Restore it later

1. Double-click **`Restore_Display_Driver.bat`**.
2. Accept the UAC prompt.
3. Wait for `[SUCCESS] Display drivers restored successfully.`
4. **Restart your computer** to apply the changes.

## Backup Location

Both scripts use the same fixed folder on your system drive:

```
%SystemDrive%\Backup_Display_Driver
```

On most PCs this is `C:\Backup_Display_Driver`.

> **Reinstalling Windows?** Copy this folder to a USB drive or another disk *before* formatting, then copy it back to the same path on the new installation and run the restore script.

To use a different location, edit this line at the top of **both** scripts:

```bat
set "BACKUP_DIR=%SystemDrive%\Backup_Display_Driver"
```

## How It Works

**Backup**

1. Checks for Administrator rights and re-launches itself elevated if needed.
2. Creates the backup folder if it does not exist.
3. Uses PowerShell's `Get-WindowsDriver -Online` to list installed third-party drivers whose class is *Display* (GUID `{4d36e968-e325-11ce-bfc1-08002be10318}`).
4. Exports each one with `pnputil /export-driver`.

**Restore**

1. Checks for Administrator rights and re-launches itself elevated if needed.
2. Verifies that the backup folder exists and contains `.inf` files.
3. Installs everything with:

```bat
pnputil /add-driver "%BACKUP_DIR%\*.inf" /subdirs /install
```

## Exit Codes and Messages

The backup script's PowerShell step returns these codes:

| Code | Meaning |
|---|---|
| `0` | All display drivers were exported successfully |
| `1` | Reading the driver list failed, or at least one export failed |
| `2` | No third-party Display class drivers were found |

## Notes

- Only **third-party** display drivers are backed up. The built-in *Microsoft Basic Display Adapter* driver is part of Windows and is not exported.
- The restore script installs **every** driver package inside the backup folder. Keep only the drivers you actually want to restore in it.
- If the backup folder is missing or contains no `.inf` files, the restore script stops with a clear message and changes nothing.
- A restart is recommended after restoring.

## Troubleshooting

| Problem | What to try |
|---|---|
| `Get-WindowsDriver failed` | Make sure the script runs elevated and that Windows is healthy (`sfc /scannow`, `DISM /Online /Cleanup-Image /RestoreHealth`). |
| `No third-party Display class drivers found` | Your PC is probably using the generic Microsoft driver. Install the GPU vendor's driver first, then run the backup again. |
| `Display backup directory not found` | Run the backup first, or copy your saved folder back to the expected path. |
| The window closes too quickly | The scripts wait 5 seconds before closing. Run them from an open Command Prompt to keep the output on screen. |

## Disclaimer

These scripts install driver packages on your system. Use them at your own risk, and only restore drivers that you trust and that match your hardware. The authors are not responsible for any damage or data loss.
