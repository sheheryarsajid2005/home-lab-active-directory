# IT Infrastructure & Active Directory Home Lab
A documented home lab simulating an enterprise Active Directory environment using VirtualBox

# Project Overview
This home lab was built to simulate a standard enterprise IT environment from the ground up. The primary objective was to develop hands on practical skills in identity management, network configuration, Group Policy enforcement and endpoint administration using Microsoft technologies. This lab aims to mirror the day to day responsibilities of a Service Desk Analyst or Junior Systems Administrator.

# Network Topology & Architecture
Built using Oracle VirtualBox 7 with an isolated NAT Network (10.0.2.0/24), allowing VM to VM communication on a local subnet whilst retaining internet access from the host machine.

## Infrastructure Overview
Hostname | Role | OS | IP Address | DNS |
|----------|------|----|------------|-----|
| **DC01** | Domain Controller / DNS Server | Windows Server 2022 | `10.0.2.10` (Static) | `127.0.0.1` (Loopback) |
| **CLIENT01** | Standard User Workstation | Windows 10 Enterprise | DHCP Assigned | `10.0.2.10` |

## Core Configurations

### 1. Active Directory Domain Services (AD DS)
- Installed the AD DS role via Server Manager and promoted DC01 to a Domain Controller
- Provisioned a new Active Directory forest with root domain: 'sheheryarlab.local'
- NTDS database and SYSVOL shares generated and verified

**Organisational Unit Structure**
Designed a structured directory using Role-Based Access Control (RBAC) principles:
- Created a dedicated 'Staff' OU to isolate standard corporate users from built-in administrative containers
- Ensures GPOs can be scoped precisely to the correct user group without affecting domain admins

**User Lifecycle Management - ADUC**
| Account | Type | Purpose |
|---|---|---|
| sheheryar.admin | Domain Administrator | Administrative and elevated tasks |
| john.smith | Standard User | Simulated end user / helpdesk target |

### Service Desk operations practiced:
- Created new user accounts with correct OU placements and group membership
- Enforced password change on next logon for new hire simulation
- Simulated and resolved account lockouts from repeated failed login attempts
- Disabled terminated employee accounts, revoking their access whilst preserving the SID and profile data

<img width="543" height="298" alt="image" src="https://github.com/user-attachments/assets/3d71a52e-9d2a-49a7-9ff7-6ff78d54e523" />
Staff OU in ADUC



---

### 2. DNS Configuration & Domain Join
- Assigned DC01 a static IP ('10.0.2.10') to ensure consistent DNS resolution across the network
- Configured the DC01 to use loopback ('127.0.0.1') as its own DNS server, a requirement for Active Directory
- Pointed CLIENT01's TCP/IPv4 adapter to '10.0.2.10' as primary DNS, allowing it to locate the domain
- Successfully authenticated and joined CLIENT01 to 'sheheryarlab.local'
- Veriifed domain membership via System Properties

<img width="410" height="469" alt="image" src="https://github.com/user-attachments/assets/2f21aafa-4110-4061-982c-5beeb046ee3f" />

---

### 3. Group Policy (GPO) Enforcement:
- Created GPO: 'Block_Control_Panel'
- Linked exclusively to the 'Staff' OU (scoped to standard users for testing)
- Forced immediate client-side policy application via 'gpupdate /force'
- Verified enforcement: standard user (john.smith) blocked from Control Panel
- Verified admin exemption: sheheryar.admin retains full access by creating an OU for IT Department.
- Ran 'gpresult /r' to confirm correct GPO application and inheritance

<img width="591" height="155" alt="image" src="https://github.com/user-attachments/assets/00916d11-e4e4-458b-8422-6442e402ae52" />


---

### 4. Remote Desktop Protocol (RDP)
- Enabled Remote Desktop on DC01 via System Settings
- Connected from CLIENT01 to DC01 using MSTSC ('mstsc' -> '10.0.2.10')
- Authenticated using domain admin credentials across the domain-joined environment
- Simulates real-world service desk remote support workflow

---

### 5. GPO Lab - Additional Policies (From Roadmap)
**Password Policy (Default Common Policy)**
- Minimum password length: 10 characters
- Maximum password age: 90 days
- Enforce password history: 10 passwords
- Complexity requirements: Enabled
- Account lockout threshold: 5 attempts
- Lockout duration: 30 minutes

**Restrict_CMD GPO**
- Linked to Sheheryars_Staff OU
- Prevents standard users from accessing Command Prompt
- Verified on CLIENT01 logged in as test user.
<img width="980" height="529" alt="image" src="https://github.com/user-attachments/assets/12c5fef8-5176-46b0-a4f4-5b871c814590" />

**Map_Staff_Drive GPO**
- Linked to Sheheryars_Staff OU
- Automatically maps S:drive (Staff Share) on user login
- Share hosted on DC01 at \\DC01\StaffShare
- Verified, S:drive visible under Network Locations on CLIENT01
<img width="808" height="577" alt="image" src="https://github.com/user-attachments/assets/eccd3f3a-8e09-43f4-9b6a-f3c6da5f2461" />


## Troubleshooting & Problem-solving
Real issues encountered and resolved during the build. Documenting these because problem-solving under ambiguity is a core service desk skill.

**Issue 1 - Windows 11 ISO Boot Failure**
Attempted to provision Windows 11 Enterprise as the client endpoint, VirtualBox's UEFI/Secure Boot emulation caused a black-screen bug hang during the ISO boot sequence. Rather than spending time fighting the hypervisor, pivoted to Windows 10 Enterprise which bypassed the compatability issue entirely and allowed the lab to proceed.

**Issue 2 - Domain Trust Relationship Awareness**
Through research during the build, gained an understanding of domain trust relationship failures and how to recover from them. Used local machine credentials to bypass the broken trust and run 'Test-ComputerSecureChannel' in PowerShell to diagnose and rpeair the secure channel between the workstation and domain controller.

---

## Key Commands Reference
| Command | Purpose |
|---|---|
| `gpupdate /force` | Forces immediate application of Group Policy on the client |
| `gpresult /r` | Generates a diagnostic report of applied GPOs for current user/machine |
| `ipconfig /release` | Releases current DHCP lease |
| `ipconfig /renew` | Requests a new DHCP lease. Resolves APIPA (169.254.x.x) issues |
| `ipconfig /flushdns` | Clears the local DNS resolver cache |
| `ncpa.cpl` | Quick-launch Network Connections panel |
| `mstsc` | Launches Remote Desktop Connection |
| `Test-ComputerSecureChannel` | PowerShell. Diagnoses domain trust relationship health |
 
---

## Skills Demonstrated
 
- Windows Server 2022 administration and role deployment
- Active Directory forest creation and domain controller promotion
- OU design and Role-Based Access Control (RBAC) principles
- Full user lifecycle management — creation, password policy, lockout, disable
- Group Policy creation, scoping, linking, and enforcement verification
- Static IP and DNS configuration for domain environments
- Client domain join and authentication
- Remote Desktop Protocol (RDP) for remote administration
- Real troubleshooting under ambiguous conditions

---

## Purpose

Built independently to develop practical IT infrastructure skills ahead of junior roles, and as a long-term foundation for a career progression into cybersecurity. All configuration decisions made and documented personally. Lab will be expanded upon in the future with a below roadmap.

---

## Roadmap
- Install RSAT on CLIENT01 for remote domain management without logging into the server.
- [X] Additional GPUs: Password complexity policy, CMD restriction, mapped network drives, wallpaper enforcement
- Wireshark traffic capture. Analyse DNS queries and RDP packets between VMs.
- Shared folder with NTFS permission scoped to security groups.
- Microsoft Entra ID and Exchange Online. Cloud identity management theory and administration.
