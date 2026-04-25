# Project: Cyber Security Homelab
Date: April 2026

## Objective: To build a private, isolated sandbox for penetration testing and network security research using VirtualBox.

### 1. Network Architecture
The lab is designed with three distinct network segments to simulate an enterprise environment with isolated trust zones.

##### Management/Update Segment (NATNetwork): 
Provides controlled internet access to the Attacker and Targets for updates.

#### Target Segment A (labnet):
An internal network for modern infrastructure (Windows 10 and Ubuntu Server).

#### Target Segment B (msfnet): 
A dedicated internal network for legacy/vulnerable systems (Metasploitable 2).

### IP Schema

| Device Name | Operating System | IP Address | Network Segment |
| :--- | :--- | :--- | :--- |
| **Kali Attacker** | Kali Linux 2024.1 | 192.168.100.10 | LabNet (Internal) |
| **Windows Target** | Windows 10 Pro | 192.168.100.30 | LabNet (Internal) |
| **Metasploitable** | Ubuntu (Vulnerable) | 192.168.200.50 | MSFNet (Internal) |

### 2. Technical Challenges & Troubleshooting
The primary hurdle during the deployment phase was establishing connectivity with the Windows 10 target.

### Issue: ICMP Silent Host
Even after configuring static IPs, the Kali machine could not ping the Windows 10 VM, resulting in Destination Host Unreachable.

### Diagnosis
Windows 10 classifies internal networks without a gateway as "Unidentified Networks," automatically applying the most restrictive Public Firewall Profile. This profile silences the host by dropping all ICMP (ping) requests.

### Resolution
I performed a two-step verification and fix:

#### 1. Isolation Test: Briefly disabled all firewall profiles using netsh advfirewall set allprofiles state off to confirm the VirtualBox internal wiring was correct.

#### 2. Granular Hardening: Re-enabled the firewall and applied a specific exception rule to allow ICMP traffic while maintaining the system's defensive posture:
      netsh advfirewall firewall add rule name="Allow ICMPv4" protocol=icmpv4:8,any dir=in action=allow



