# 🔧 DNSSEC Hardening v2 – Lockdownsurface Patch

PowerShell script to harden DNS client configurations under Windows, enforce DNSSEC, and log all changes made.  
Includes rollback, log tracing, DNSSEC test, and control over DoH fallback (Windows 11+).

## 📦 Features

- ✅ Backup of current DNS configuration
- ✅ Set hardened resolvers (Cloudflare `1.1.1.1`, Quad9 `9.9.9.9`)
- ✅ Enforce DNSSEC via registry
- ✅ Disable LLMNR and NetBIOS
- ✅ Disable automatic DoH (Windows 11+)
- ✅ Log actions to `C:\dnssec_hardening.log`
- ✅ Test DNSSEC enforcement using `dnssec-failed.org`

## 💡 Usage

```powershell
powershell -ExecutionPolicy Bypass -File .\DNSSEC_Hardening_v2.ps1
