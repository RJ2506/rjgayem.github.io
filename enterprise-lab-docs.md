# Technical Documentation: Enterprise Hybrid Systems & Infrastructure Lab
**Author:** Rodolf John Gayem  
**Target Environment:** Virtualized Corporate Deployment (`homelab.local`)  

---

## 1. Executive Project Overview
This deployment serves as a production-representative staging environment designed to demonstrate enterprise-grade identity management, directory services automation, network isolation, and security hardening. 

The architecture features a headless **Windows Server Core** deployment to mimic real-world infrastructure constraints, focusing on reducing attack surface area and resource footprint while utilizing automated administration tools.

---

## 2. Infrastructure & Network Architecture Diagram
The topology is built on an isolated virtual network utilizing **Oracle VM VirtualBox Manager**, configuring host-only networks with disabled internal DHCP to allow Windows Server to maintain authoritative network controls.

```text
                  [ ISOLATED PRIVATE NETWORK ]
                               |
       +-----------------------+-----------------------+
       | (Static IP: 10.0.2.10)                        | (DHCP Assigned)
+--------------+                                +--------------+
| Windows Core |                                |  Windows 11  |
|  Domain Ctrl |                                | Workstation  |
| (homelab.dc1)|                                | (homelab.ws1)|
+--------------+                                +--------------+
| AD DS / DNS  |                                | Domain User  |
| DHCP Server  |                                | Environment  |
+--------------+                                +--------------+
IP Addressing Schema
Active Directory Domain Controller (homelab.dc1): 10.0.2.10 /24 | Gateway: 10.0.2.1 | DNS Primary: 127.0.0.1

DHCP Scope Range: 10.0.2.50 to 10.0.2.150

Subnet Mask: 255.255.255.0

3. Deployment & Configuration Phase Log
Phase 1: Headless Server Baseline & Network Configuration
Operating completely from a minimal command interface on Windows Server Core, the network interfaces were statically defined using PowerShell:

PowerShell
# Rename the computer interface
Rename-Computer -NewName "homelab-dc1" -Restart

# Configure static IPv4 parameters
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.2.10 -PrefixLength 24 -DefaultGateway 10.0.2.1
Set-NetDNSServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1
Phase 2: Active Directory Domain Services (AD DS) Provisioning
The domain authority for homelab.local was promoted using automated script deployment:

PowerShell
# Install Active Directory binaries
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Provision the new Active Directory Forest
Install-ADDSForest -DomainName "homelab.local" -ForestMode WinThreshold -DomainMode WinThreshold -Force
Phase 3: Identity & Access Management Structure
To manage users efficiently and follow the Principle of Least Privilege, an Organizational Unit (OU) structure was engineered alongside tailored security groups:

PowerShell
# Create Corporate Organizational Units
New-ADOrganizationalUnit -Name "Corporate_Users" -Path "DC=homelab,DC=local"
New-ADOrganizationalUnit -Name "Corporate_Groups" -Path "DC=homelab,DC=local"

# Provision Departmental Security Groups
New-ADGroup -Name "SG_Accounting" -GroupScope Global -Category Security -Path "OU=Corporate_Groups,DC=homelab,DC=local"
New-ADGroup -Name "SG_ProjectManagement" -GroupScope Global -Category Security -Path "OU=Corporate_Groups,DC=homelab,DC=local"
4. Group Policy Object (GPO) Architecture
To automate standard workspace deployment and ensure operational security, specialized GPOs were linked directly to the root domain.

1. Drive Mapping Automation Policy (GPO_Corporate_Drives)
Objective: Automatically map corporate network shares when an authorized user authenticates.

Configuration: Targets User Configuration Preferences. Maps network storage paths to the H: drive utilizing security filtering to selectively target specific security groups (e.g., SG_Accounting maps to accounting file shares).

2. Default Domain Hardening Baseline (GPO_Security_Hardening)
To defend directory assets against automated brute-force attacks, password complexities and account lockout policies were tightened:

Account Lockout Threshold: 5 invalid authentication attempts.

Account Lockout Duration: 30 minutes.

Reset Lockout Counter After: 30 minutes.

Minimum Password Length: 12 characters requiring complex character sets.

5. Verification, Testing, and Troubleshooting Workflows
To validate that client machines could successfully register, communicate, and fetch policies from the headless core engine:

DNS Resolution Audit: Executed nslookup homelab.local from a Windows 11 client to ensure the Core Domain Controller was correctly resolving identity lookups.

Domain Integration Test: Joined a Windows 11 workstation to the homelab.local domain, verifying successful credential validation against the Active Directory schema.

Policy Verification Log: Executed gpresult /r on target machines to confirm that GPO_Corporate_Drives and security configurations were successfully applied during authentication.
