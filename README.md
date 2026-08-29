# Active Directory Security Monitoring Lab

**Author:** Nwobodo Chukwuemeka Victor
**Major:** Security Operation Centre Analyst
**Date:** August 29, 2026

<img width="644" height="665" alt="Active Directory drawio" src="https://github.com/user-attachments/assets/99d9cbe0-23ec-4f29-a253-0a2a25891852" />

## Project Overview

This project is an isolated Active Directory home lab built to practise Windows domain administration, endpoint telemetry collection, and security monitoring with Splunk.

The environment simulates a small Windows domain, collects logs from endpoints and the domain controller, and uses controlled attack simulations to validate detection capabilities.

> All testing was performed only on virtual machines that I owned and controlled.

## Objectives

- Deploy a Windows Server 2022 Active Directory domain controller.
- Join a Windows 10 endpoint to the domain.
- Collect Windows and Sysmon telemetry centrally with Splunk.
- Simulate authorized RDP password-guessing activity and investigate the logs.
- Run Atomic Red Team tests mapped to the MITRE ATT&CK framework.

## Lab Architecture

| System | Purpose |
| --- | --- |
| Windows Server 2022 | Active Directory Domain Services, DNS, and Domain Controller |
| Windows 10 | Domain-joined target endpoint |
| Ubuntu Server 22.04 | Splunk Enterprise server |
| Kali Linux | Authorized attack-simulation workstation |
| VirtualBox | Virtualization platform and private NAT network |
| Draw.io | Network diagram and project planning |

The Windows endpoint and domain controller were configured with Sysmon and Splunk Universal Forwarder. Both systems forwarded Windows and Sysmon logs to a dedicated Splunk index named `Endpoint`.

## Technologies Used

- Oracle VirtualBox
- Windows Server 2022
- Windows 10
- Ubuntu Server 22.04
- Kali Linux
- Active Directory Domain Services (AD DS)
- Sysmon
- Splunk Enterprise
- Splunk Universal Forwarder
- Atomic Red Team
- MITRE ATT&CK
- Draw.io

## Project Implementation

### 1. Virtual Network Setup
<img width="1919" height="1031" alt="Screenshot 2026-08-28 225357" src="https://github.com/user-attachments/assets/032e9c0f-1a86-43bd-b872-bdad78e13f98" />

I created a dedicated NAT network in VirtualBox and connected all virtual machines to it. This enabled the systems to communicate with each other in an isolated lab environment.

Static IP addresses were assigned to the servers and endpoints to ensure reliable communication.

### 2. Splunk Server Configuration
<img width="2000" height="1125" alt="1" src="https://github.com/user-attachments/assets/6445bc41-3159-4077-b62a-2571a94a4d51" />

Splunk Enterprise was installed on an Ubuntu Server virtual machine.

The Splunk server was configured to:

- Start automatically after reboot.
- Receive logs from Splunk Universal Forwarders.
- Listen on port `9997`.
- Store incoming Windows logs in an index named `Endpoint`.

I verified that logs from the Windows endpoint were successfully received and searchable in Splunk.

### 3. Sysmon and Splunk Universal Forwarder

Sysmon and Splunk Universal Forwarder were installed on both the Windows 10 endpoint and Windows Server 2022 domain controller.

Sysmon was configured using the [Olaf Hartong Sysmon Modular configuration](https://github.com/olafhartong/sysmon-modular).
<img width="2000" height="1125" alt="1 (1)" src="https://github.com/user-attachments/assets/dc9f2752-091b-44c5-9d33-9181a7b8f856" />

The Splunk Universal Forwarder collected and forwarded:

- Windows Security logs
- Windows System logs
- Windows Application logs
- Sysmon operational logs
<img width="1280" height="592" alt="WhatsApp Image 2026-08-29 at 00 07 31" src="https://github.com/user-attachments/assets/fe654012-a178-4626-bf7d-48b930d2450b" />

Custom forwarder settings were placed in `etc/system/local/inputs.conf` rather than modifying the default configuration.

### 4. Active Directory Deployment
<img width="2000" height="1125" alt="1 (2)" src="https://github.com/user-attachments/assets/67bdfe4f-7907-42fd-a1fa-3507bd4c6f38" />

On Windows Server 2022, I installed the Active Directory Domain Services role and promoted the server to a domain controller.

A new forest and domain named `VICTORAN.local` were created.

Using Active Directory Users and Computers, I created:

- Organizational Unit for **SOC**
- Test user account
- Domain object for lab testing

### 5. Domain Joining the Windows Endpoint

Before joining the Windows 10 endpoint to the domain, I configured its preferred DNS server to point to the domain controller.

This was necessary because Active Directory relies on DNS for domain discovery and authentication.

After verifying DNS configuration, I joined the Windows 10 machine to `VICTORAN.local` and successfully logged in with a domain user account.

## Detection Validation

### Authorized RDP Password-Guessing Simulation
<img width="1919" height="1031" alt="Screenshot 2026-08-29 001724" src="https://github.com/user-attachments/assets/c92399a5-d784-4a3e-8fd3-9e6553e8881c" />

Using Kali Linux, I performed a controlled password-guessing simulation against Remote Desktop Protocol on the Windows endpoint.

The activity was performed only against the lab environment using a test account and a small test password list.

<img width="1280" height="591" alt="WhatsApp Image 2026-08-29 at 00 32 36" src="https://github.com/user-attachments/assets/b2149520-9ff7-46d6-ae1d-5779723c19ab" />

Splunk captured the related Windows authentication events:

| Event ID | Description |
| --- | --- |
| `4625` | Failed logon attempt |
| `4624` | Successful logon |

The failed logon events appeared within a short time window, which is a useful indicator of possible password-guessing activity.

The successful logon event included details such as the source workstation and source IP address, making it possible to trace the activity back to the Kali Linux machine.

### Atomic Red Team Testing
<img width="2000" height="1125" alt="1 (3)" src="https://github.com/user-attachments/assets/31d31e3c-db64-46c6-a477-626d710adad4" />

Atomic Red Team was installed on the Windows 10 endpoint to generate controlled telemetry mapped to the MITRE ATT&CK framework.

Tests included:

- `T1136.001` — Create Account: Local Account
- `T1059.001` — Command and Scripting Interpreter: PowerShell
<img width="2000" height="1125" alt="1 (4)" src="https://github.com/user-attachments/assets/edbd24e8-9b0f-49b4-b2ac-cb07753e9ade" />

The generated events were then reviewed in Splunk to confirm whether the logging pipeline captured the expected activity.

This exercise demonstrated the value of attack simulation for identifying visibility gaps and improving detection coverage.

## Key Findings

1. **DNS is essential for Active Directory.**  
   Domain-joined systems must use the domain controller for DNS to locate domain services.

2. **Centralized logging improves investigations.**  
   Splunk made it possible to review failed logons, successful logons, targeted accounts, and source systems in one place.

3. **Sysmon improves endpoint visibility.**  
   It provides more detailed security telemetry than standard Windows logging alone.

4. **Attack simulations validate detections.**  
   Atomic Red Team helps confirm that monitoring controls are working and can reveal gaps in visibility.

5. **Virtual machine snapshots are important.**  
   Snapshots make it easy to restore a known-good lab state after testing.


## References

- [Microsoft Sysmon](https://learn.microsoft.com/sysinternals/downloads/sysmon)
- [Olaf Hartong Sysmon Modular Configuration](https://github.com/olafhartong/sysmon-modular)
- [Splunk Documentation](https://docs.splunk.com/)
- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)
- [MITRE ATT&CK](https://attack.mitre.org/)

---

*This project was completed in an isolated lab for educational and defensive-security purposes only.*
