# Project: Virtual Homelab
##### Date: April 2026

#### Objective: To build a private, isolated sandbox environment to simulate real-world attacks and defence using VirtualBox.


## 1. Network Architecture
My lab environment consists of three virtual machines hosted in a localized, isolated network to ensure safe testing.

* **Management/Update Network (NATNetwork):** 
Provides controlled internet access to the Attacker and Targets for updates.

* **Target Network A (labnet):**
An internal network for modern infrastructure (Windows 10 and Ubuntu Server).

* **Target Network B (msfnet):** 
A dedicated internal network for legacy/vulnerable systems (Metasploitable 2).

### Network Connectivity & Machine roles
A brief information about the network connectivity and roles of VMs using in the homelab

| Machine | Operating System | IP Address | Network connection | Role |
| :--- | :--- | :--- | :--- | :--- |
| **Attacker** | Kali Linux 2026.1 | 192.168.100.10 | NATNetwork, LabNet, MSFNet | Primary attack platform |
| **Target 1** | Ubuntu Server 26.04 | 192.168.100.20 | NATNetwork, LabNet | General purpose server for configuration testing |
| **Target 2** | Windows 10 Pro | 192.168.100.30 | NATNetwork, LabNet | Modern workstation |
| **Target 3** | Metasploitable2 | 192.168.200.50 | MSFNet | Purposefully vulnerable linux server | 



## 2. Connectivity testing & Troubleshooting
To verify the integrity of the lab, I performed ICMP (ping) tests between the attacker and targets.

### The Results:

| Machine | Operating System | Status |
| :--- | :--- | :--- |
| **Target 1** | Ubuntu Server 26.04 | Success |
| **Target 2** | Windows 10 pro | Failed (Request timed out) |
| **Target 3** | Metasploitable2 | Success |

#### Issue: ICMP Silent Host
* Even after configuring static IPs, the Kali machine could not ping the Windows 10 VM, resulting in Destination Host Unreachable.

#### Diagnosis:
* While the Linux targets responded immediately, windows 10 classifies internal networks without a gateway as "Unidentified Networks," automatically applying the most restrictive Public Firewall Profile. This profile silences the host by dropping all ICMP (ping) requests.

#### The Root Cause: 
* Windows Firewall's default state allows all outbound traffic but blocks almost all unsolicited inbound traffic (Default-Deny). While the network 'plumbing' was correct, the host-based security policy was dropping the Attacker's ICMP Echo Requests.

#### Solutions:

* **1. Isolation Test:**
Briefly disabled all firewall profiles using netsh advfirewall set allprofiles state off to confirm the VirtualBox internal wiring was correct and reconfigured the internal             network adapter by manually assigning ip address and the subnet mask.

* **2. Granular Hardening:**
Re-enabled the firewall and applied a specific exception rule to allow ICMP traffic while maintaining the system's defensive posture:

      netsh advfirewall firewall add rule name="Allow ICMPv4" protocol=icmpv4:8,any dir=in action=allow

A comparision table between before applying the solution and after applying the solution:

<table>
  <tr>
    <td><b>Before (Firewall On)</b></td>
    <td><b>After (Firewall Off)</b></td>
  </tr>
  <tr>
    <td><img src="pow/windows ping failed.png" width="400"></td>
    <td><img src="pow/windows ping success.png" width="400"></td>
  </tr>
</table>



## 3. Lab Verification (Proof of Concept)
Once connectivity was stabilized, I verified the lab's functionality by performing a series of initial exploits against the legacy segment.

### Key exploitations performed:

*  **1. Service Enumeration:**
        Used nmap to identify open ports on the 192.168.200.0/24 subnet.

* **2. vsftpd 2.3.4 Backdoor:**
        Triggered the smile face (:)) backdoor on port 21, successfully gaining a root shell.

* **3. PostgreSQL:** 
      Confirmed unauthenticated login vulnerabilities, allowing for database credential harvesting.
  
* **4. Samba usermap_script:** 
      Leveraged Metasploit to gain an initial foothold on the Metasploitable 2 machine.


### Exploitation Evidence: 
[vsftpd backdoor](./pow/vsftpd.png) | [postgre login](./pow/postgre_login.png) | [Samba usermap](./pow/samba_usermap.png)


## Network Reconnaissance & Vulnerability Assessment

### 1. Target: Metasploitable 2 (192.168.200.50)
This machine represents an intentionally insecure legacy Linux server. The scan revealed a massive attack surface with 23 open ports.

* **Status: Highly Vulnerable**

* Key findings: Port 21 (FTP) - vsftpd 2.3.4, Port 445 (SMB) - Samba 3.0.20
* weak configs: Port 23 (Telnet), Port 3306 (MySQL), Port 2121 (ProFTPD 1.3.1)
* Web vulnerabilities: Port 80 (HTTP) / Port 8180 (Apache Tomcat)
   

### 2. Target: Ubuntu Server (192.168.100.20)
This host demonstrates a modern "Default-Deny" security posture.

* **Status: Secured**
  
* **Observation:** Only Port 22 (SSH) is open. It is running OpenSSH 10.2p1, a current version patched against recent vulnerabilities.
* **Security Analysis:** The attack surface is minimal. Initial entry would likely require a brute-force attack or finding a vulnerability in a web application if one were deployed in the future.
* **Nmap Note:** OS detection was unable to find an exact match due to the lack of open ports, which is a defensive success.

### 3. Target: Windows 10 (192.168.100.30)
This scan highlights the effectiveness of the Windows Defender Firewall in an enterprise-style configuration.

* **Status: Filtered**
  
* **Observation:** Almost all 65,535 ports are "Filtered," meaning the firewall is silently dropping packets.
* **Finding:** Port 7680 is the only open port, used by Windows "Delivery Optimization."
* **Analysis:** Even with the ICMP (ping) exception I previously configured, the host remains largely invisible to service scanning, requiring more advanced enumeration techniques or local access.


### Technical Appendix (Nmap Reports)
* **Metasploit report:** [Metasploit](./Reports/metasploit_scan.txt)
* **Ubuntu Server report:** [Ubuntu server](./Reports/server_scan.txt)
* **windows report:** [Windows](./Reports/win_scan.txt)
