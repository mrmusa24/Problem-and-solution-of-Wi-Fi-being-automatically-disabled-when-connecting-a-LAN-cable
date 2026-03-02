# 🔧 Fix: LAN Connected হলে Wi-Fi Auto Disable Problem (Windows)

## 📌 Overview

অনেক সময় Windows কম্পিউটারে LAN (Ethernet) কেবল সংযোগ দিলে Wi-Fi অ্যাডাপ্টার স্বয়ংক্রিয়ভাবে Disable হয়ে যায়।  
এমনকি ম্যানুয়ালি Enable করলেও কাজ করে না।

এই গাইডে BIOS সেটিংস থেকে শুরু করে Registry Fix এবং One-Click PowerShell Script পর্যন্ত সম্পূর্ণ সমাধান দেওয়া হয়েছে।

---

# 🎯 Problem Description

- LAN সংযোগ দিলে Wi-Fi বন্ধ হয়ে যায়  
- Wi-Fi Enable অপশন গ্রে আউট থাকে  
- HP ডিভাইসে Switching Service এর কারণে সমস্যা  
- Power Management Conflict  
- Dual Network Interface Conflict  

---

# 🛠 Complete Solution Guide

---

## 1️⃣ BIOS / UEFI সেটিংস চেক করুন

1. কম্পিউটার Restart করুন  
2. Boot হওয়ার সময় `F2 / F10 / F12 / DEL` চাপুন  
3. **Advanced / Device Configuration** মেনুতে যান  
4. **LAN/WLAN Auto Switching** খুঁজুন  
5. যদি **Enabled** থাকে → **Disabled** করুন  
6. Save & Exit  

---

## 2️⃣ HP Switching Service Disable করুন (HP Users Only)

PowerShell → Run as Administrator

```powershell
Stop-Service "HPDSUWatcherService" -ErrorAction SilentlyContinue
Stop-Service "HPLanWlanWwanSwitchingService" -ErrorAction SilentlyContinue

Set-Service "HPDSUWatcherService" -StartupType Disabled -ErrorAction SilentlyContinue
Set-Service "HPLanWlanWwanSwitchingService" -StartupType Disabled -ErrorAction SilentlyContinue
```

---

## 3️⃣ Network Adapter Power Management Disable করুন

1. Press `Win + X` → Device Manager  
2. Expand **Network Adapters**  
3. Right-click Wi-Fi → Properties  
4. Power Management Tab  
5. Uncheck:  
   `Allow the computer to turn off this device to save power`  
6. একইভাবে Ethernet অ্যাডাপ্টারের জন্যও করুন  

---

## 4️⃣ Advanced Registry Fix (Optional)

⚠ Registry Backup নেওয়ার পর Apply করুন।

### ➜ File Name: `wifi-lan-fix.reg`

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\0001]
"PnPCapabilities"=dword:00000024

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\0002]
"PnPCapabilities"=dword:00000024
```

Double Click → Yes → Restart PC

---

## 5️⃣ Network Reset

1. Settings → Network & Internet  
2. Advanced Network Settings  
3. Network Reset  
4. Restart PC  

---

## 6️⃣ CMD Network Repair

Run CMD as Administrator:

```cmd
netsh int ip reset
netsh winsock reset
ipconfig /release
ipconfig /renew
ipconfig /flushdns
shutdown /r /t 0
```

---

# 🚀 One-Click Full Fix Script

### ➜ File Name: `network-full-fix.ps1`

```powershell
Write-Host "Starting Network Fix..." -ForegroundColor Green

# Stop HP Switching Service
Write-Host "Stopping HP Switching Services..." -ForegroundColor Yellow
Get-Service "HP*switch*" -ErrorAction SilentlyContinue | Stop-Service
Get-Service "HP*switch*" -ErrorAction SilentlyContinue | Set-Service -StartupType Disabled

# Disable Power Management
Write-Host "Disabling Power Management..." -ForegroundColor Yellow
$adapters = Get-NetAdapter | Where-Object {
    $_.InterfaceDescription -like "*wireless*" -or 
    $_.InterfaceDescription -like "*Ethernet*"
}

foreach ($adapter in $adapters) {
    Write-Host "Configuring: $($adapter.Name)"
    Disable-NetAdapterPowerManagement -Name $adapter.Name -ErrorAction SilentlyContinue
}

# Reset Network Stack
Write-Host "Resetting Network Stack..." -ForegroundColor Yellow
netsh int ip reset
netsh winsock reset
ipconfig /flushdns

Write-Host "Fix Completed! Please Restart Your PC." -ForegroundColor Green
```

Run PowerShell as Administrator →  
```
Set-ExecutionPolicy RemoteSigned -Scope Process
.\network-full-fix.ps1
```

---

# ✅ Troubleshooting Checklist

- [ ] BIOS Auto Switching Disabled  
- [ ] HP Switching Service Disabled  
- [ ] Power Management Disabled  
- [ ] Registry Fix Applied  
- [ ] Network Reset Completed  
- [ ] Drivers Updated  
- [ ] VPN Removed  

---

# 📋 Required Information for Support

If issue persists, provide:

- Computer Brand & Model  
- Windows Version (`winver`)  
- Network Adapter Model  
- BIOS Version  

---

# ⚠ Disclaimer

This guide is provided for educational and troubleshooting purposes only.  
Apply changes carefully and at your own risk.
