#===============================
hashtag#DNSSEC Hardening v2 
hashtag#Author: noname | Date: 2025-05-19
#===============================
$log = "$env:SystemDrive\dnssec_hardening.log"
function Log($msg) {
 $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
 Add-Content -Path $log -Value "[$timestamp] $msg"
 Write-Host $msg
}

Log "=== DNSSEC Hardening Started ==="

#=== Backup current DNS configuration ===,
$backupFile = "$env:SystemDrive\dnsbackup$((Get-Date).ToString("yyyyMMddHHmmss")).txt"
Get-DnsClientServerAddress | Out-File $backupFile
Log "Backed up current DNS config to $backupFile"

#=== Set secure DNS resolvers (Cloudflare, Quad9) ===,
$dnsPrimary = "1.1.1.1"
$dnsSecondary = "9.9.9.9"
$interfaces = Get-DnsClient | Where-Object { $_.InterfaceAlias -ne "Loopback Pseudo-Interface 1" }

foreach ($iface in $interfaces) {
 try {
 Set-DnsClientServerAddress -InterfaceAlias $iface.InterfaceAlias -ServerAddresses ($dnsPrimary, $dnsSecondary) -ErrorAction Stop
 Log "DNS set for interface '$($iface.InterfaceAlias)' to $dnsPrimary / $dnsSecondary"
 } catch {
 Log "Error setting DNS on interface $($iface.InterfaceAlias): $"
 }
}

#=== Force DNSSEC via registry ===
$regPath = "HKLM:\Software\Policies\Microsoft\Windows NT\DNSClient"
New-Item -Path $regPath -Force | Out-Null
New-ItemProperty -Path $regPath -Name "EnableDnsSec" -PropertyType DWord -Value 1 -Force | Out-Null
Log "Forced DNSSEC policy via registry"

#=== Disable LLMNR & NetBIOS ===
New-ItemProperty -Path $regPath -Name "EnableMulticast" -PropertyType DWord -Value 0 -Force | Out-Null
Get-WmiObject -Class Win32NetworkAdapterConfiguration -Filter "IPEnabled=TRUE" | ForEach-Object {
 $_.SetTcpipNetbios(2) | Out-Null
}
Log "Disabled LLMNR and NetBIOS"

#=== Optional: Disable insecure DoH/DoT (Windows 11+ only) ===,
$DoHKey = "HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters"
if (-not (Test-Path $DoHKey)) { New-Item -Path $DoHKey -Force | Out-Null }
New-ItemProperty -Path $DoHKey -Name "EnableAutoDoh" -PropertyType DWord -Value 0 -Force | Out-Null
Log "Disabled automatic DoH configuration (Windows 11+ only)"

#=== Flush DNS ===
Clear-DnsClientCache
Log "Flushed DNS client cache"

#=== Test DNSSEC validation ===
try {
 $test = Resolve-DnsName dnssec-failed.org -Server $dnsPrimary -ErrorAction Stop
 Log " DNSSEC NOT enforced (dnssec-failed.org resolved)"
} catch {
 Log " DNSSEC enforced correctly (invalid domain rejected)"
}

Log "=== DNSSEC Hardening Completed ==="

