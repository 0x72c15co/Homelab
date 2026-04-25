# Project: Cyber Security Homelab
##### Date: April 2026

#### Objective: To build a private, isolated sandbox for environment to simulate real-world attacks and defence using VirtualBox.

## 1. Network Architecture
My lab environment consists of three virtual machines hosted in a localized, isolated network to ensure safe testing.

#### Management/Update Segment (NATNetwork): 
Provides controlled internet access to the Attacker and Targets for updates.

#### Target Segment A (labnet):
An internal network for modern infrastructure (Windows 10 and Ubuntu Server).

#### Target Segment B (msfnet): 
A dedicated internal network for legacy/vulnerable systems (Metasploitable 2).

### IP Schema

| Machine | Operating System | IP Address | Network Segment | Role |
| :--- | :--- | :--- | :--- | :--- |
| **Attacker** | Kali Linux 2026.1 | 192.168.100.10 | LabNet (Internal) | Primary attack platform |
| **Target 1** | Ubuntu Server 26.04 | 192.168.100.20 | LabNet (Internal) | General purpose server for configuration testing |
| **Target 2** | Windows 10 Pro | 192.168.100.30 | LabNet (Internal) | Modern workstation |
| **Target 3** | Metasploitable2 | 192.168.200.50 | MSFNet (Internal) | Purposefully vulnerable linux server | 

## 2. Connectivity testing & Troubleshooting
To verify the integrity of the lab, I performed ICMP (ping) tests between the attacker and targets.

### The Results:

| Machine | Operating System | Status |
| :--- | :--- | :--- |
| **Target 1** | Ubuntu Server 26.04 | Success |
| **Target 2** | Windows 10 pro | Failed (Request timed out) |
| **Target 3** | Metasploitable2 | Success |

#### Issue: ICMP Silent Host
Even after configuring static IPs, the Kali machine could not ping the Windows 10 VM, resulting in Destination Host Unreachable.

#### Diagnosis:
While the Linux targets responded immediately, windows 10 classifies internal networks without a gateway as "Unidentified Networks," automatically applying the most restrictive Public Firewall Profile. This profile silences the host by dropping all ICMP (ping) requests.

#### The Root Cause: 
Windows Firewall's default state allows all outbound traffic but blocks almost all unsolicited inbound traffic (Default-Deny). While the network 'plumbing' was correct, the host-based security policy was dropping the Attacker's ICMP Echo Requests.

#### The Fix:
I performed a two-step verification and fix:

##### 1. Isolation Test: 
Briefly disabled all firewall profiles using netsh advfirewall set allprofiles state off to confirm the VirtualBox internal wiring was correct and reconfigured the internal network adapter by manually assigning ip address and the subnet mask.

##### 2. Granular Hardening:
Re-enabled the firewall and applied a specific exception rule to allow ICMP traffic while maintaining the system's defensive posture:

      netsh advfirewall firewall add rule name="Allow ICMPv4" protocol=icmpv4:8,any dir=in action=allow

## 3. Lab Verification (Proof of Concept)
Once connectivity was stabilized, I verified the lab's functionality by performing a series of initial exploits against the legacy segment.

### Key exploitations performed:

##### 1. Service Enumeration: 
Used nmap to identify open ports on the 192.168.200.0/24 subnet.

##### 2. vsftpd 2.3.4 Backdoor: 
Triggered the smile face (:)) backdoor on port 21, successfully gaining a root shell.

##### 3. PostgreSQL: 
Confirmed unauthenticated login vulnerabilities, allowing for database credential harvesting.

##### 4. Samba usermap_script: 
Leveraged Metasploit to gain an initial foothold on the Metasploitable 2 machine.
