# Active Directory Home Lab

A small Windows domain built in VirtualBox to practice the day-to-day work of an IT support role: user and group management, Group Policy, and troubleshooting. Built October 2026.

## Environment
| Machine  | OS                              | Role                | IP        |
|----------|---------------------------------|---------------------|-----------|
| DC01     | Windows Server 2022 (eval)      | Domain controller, DNS | 10.0.3.10 |
| CLIENT01 | Windows 11 Pro                  | Domain-joined workstation | 10.0.3.20 |

- Domain: `corp.lab`
- Network: VirtualBox NAT Network (`10.0.3.0/24`) so both VMs can reach each other and the internet

## What I built
1. Installed Windows Server, set a static IP, and promoted it to a domain controller in a new forest (AD DS + DNS).
2. Joined a Windows 11 Pro client to the domain and moved it into a `Workstations` OU.
3. Organized the directory into OUs (Sales, IT, HR, Disabled Users, Workstations) with user accounts and an `IT-Helpdesk` security group.
4. Practiced help desk tasks: password resets, account unlock/disable, moving users between OUs, group membership changes.
5. Configured Group Policy (below).

## Group Policy
- **IT - Restrict Control Panel**: linked to the IT OU; blocks access to Control Panel and PC settings.
  - Test: signed in as `CORP\bwilson` on CLIENT01 and was blocked with "This operation has been cancelled due to restrictions in effect on this computer."
  - Verification: `gpresult /r` listed the GPO under Applied Group Policy Objects, applied from DC01.corp.lab.
- **Account lockout policy** (Default Domain Policy): 3 invalid attempts, 30-minute lockout duration, 30-minute counter reset.

## Troubleshooting scenario: locked-out user (simulated)
**Report:** "I can't sign in to my computer."
**Environment:** CLIENT01 (Windows 11 Pro), domain corp.lab, DC01
**Symptoms:** Sign-in screen showed "The referenced account is currently locked out and may not be logged on to." Entering the correct password didn't help.
**Diagnosis:**
1. Opened Active Directory Users and Computers on DC01 and found the user (`bwilson`).
2. Account tab of the user's Properties: "This account is currently locked out."
**Root cause:** Three failed sign-in attempts met the lockout threshold I set in the Default Domain Policy.
**Resolution:** Ticked "Unlock account" on the Account tab.
**Verification:** User signed in successfully on CLIENT01.
**Notes:** In a real environment I'd also check for stale saved passwords (phones, mapped drives) that cause repeated lockouts, and confirm the user's identity before unlocking.

## Troubleshooting log (lab build)
| Problem | Cause | Fix |
|---|---|---|
| VirtualBox error "VERR_PATH_NOT_FOUND" starting the server VM | Unattended-install files pointed to a folder from the VM's old name | Removed the stale disc from Storage settings, recreated the VM with unattended install off |
| Windows Server setup restarted after finishing the first stage | Pressed a key at "Press any key to boot from CD or DVD" and booted the installer again | Removed the ISO from the virtual drive and booted from the hard disk |
| Windows 11 VM stuck on a black screen, green turtle icon | VirtualBox was running on top of Windows' own virtualization layer (Hyper-V features) | Turned off Hyper-V, Virtual Machine Platform, Windows Hypervisor Platform and Memory integrity, restarted, recreated the VM |
| Client couldn't find the domain | Client DNS wasn't pointed to the domain controller | Set DNS to 10.0.3.10 |

## What I'd add next
- Shared folder with group-based permissions
- Bulk user creation with PowerShell
- A second GPO tied to the Workstations OU

## Screenshots
