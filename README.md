# Ardamax-Keylogger_Analysis
This repository provides a step-by-step guide on how to analyze a Windows-based keylogger (Ardamax) using static and dynamic malware analysis techniques.

### Introduction

This lab was conducted in a fully isolated environment and tested for:
- Persistence mechanisms (Registry modifications)
- Log file storage (File system changes)
- Network exfiltration (Wireshark analysis)

### Key Findings
- No persistence mechanisms were identified.
- No keystroke logs were found.
- No network traffic was detected (No C2 communication).
- Possible reasons: The sample may be in trial mode, require user configuration, or be incomplete.

### Lab Setup
## Prerequisites
1. VirtualBox (or any hypervisor)
2. REMnux (Linux-based Malware Analysis VM)
3. Windows 10 VM (Malware Execution Environment)
4. Tools Installed:
- Ghidra (for static analysis)
- Process Monitor (ProcMon) (for registry and file system tracking)
- Wireshark (for network analysis)
  
### Step 1: Static Analysis
## 1.1 Load the Malware into Ghidra
bash: ghidra
  - Import ArdamaxKeylogger.exe
  - Analyze defined strings for registry paths, IPs, and suspicious function calls.

## 1.2 Extract Strings Using the strings Command
Run the following command on REMnux:

bash: strings ArdamaxKeylogger.exe | less

Look for:
  - Hardcoded file paths, URLs, registry keys
  - Command-line arguments

### Step 2: Dynamic Analysis
## 2.1 Monitoring Registry Changes (ProcMon)
  - Run procmon.exe on the Windows VM
  - Set filters to track registry modifications:
      - Process Name → is → ArdamaxKeylogger.exe → Include
      - Operation → contains → RegSetValue → Include
        
Expected Result:
plaintext: HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run

Findings: No registry modifications were made.

## 2.2 File System Monitoring (ProcMon)
  - Filter file system writes:
    - Operation → contains → WriteFile → Include
    - Path → contains → C:\Users → Include
  - Look for .log, .dat, .txt files.

Findings: No keylogging logs were created.

### Step 3: Network Traffic Analysis
## 3.1 Capturing Network Traffic with Wireshark
  - Start Wireshark on the Windows VM
  - Apply this filter:
    plaintext: http || dns
  - Run Ardamax and observe for external connections.
    
Findings: No outbound network activity detected.

## 3.2 Checking for Non-Standard Ports
  - Open Wireshark → Statistics → Conversations
  - Check for unknown external IP addresses
  - Apply a new filter for external traffic:
    plaintext: !(ip.src == 192.168.56.0/24 || ip.dst == 192.168.56.0/24)

Findings: No communication with a C2 (Command & Control) server was observed.

### Final Findings
- No registry modifications → No persistence mechanisms were found.
- No log files were created → No local keystroke storage detected.
- No network traffic was observed → No data exfiltration detected.

## Possible Explanations:
- User configuration required → May need manual setup (FTP, email).
- Trial-mode restriction → Keylogging disabled until activation.
- Modified or incomplete sample → Some public malware samples are broken or stripped of key functionality.

### How to Recreate This Lab
## 1. Set Up the Virtual Machines
1. Install VirtualBox and set up:
    - REMnux (Linux for static analysis)
    - Windows 10 (Malware execution environment)
      
2. Configure Windows VM Networking:
  - Host-Only Adapter (For isolation)
  - Bridged Adapter (Temporarily for network capture)

## 2. Download and Install the Required Tools
  - REMnux Tools:
    bash: sudo apt update && sudo apt install ghidra wireshark
- Windows Tools:
  - Process Monitor (ProcMon)
  - Wireshark
