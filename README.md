# Active Directory Project

## Overview
This project involves building a comprehensive virtual cybersecurity lab using Oracle VM VirtualBox. The lab includes Windows 10, Kali Linux, Windows Server 2022, and Ubuntu Server VMs, each configured on a shared NAT network. The goal is to simulate a secure enterprise environment with log monitoring, endpoint detection, domain management, and security testing.

The lab integrates advanced tools such as Splunk for log analysis, Sysmon for deep system monitoring, and Universal Forwarder for log forwarding. Attack simulations are conducted using Crowbar and Atomic Red Team (ART), while PowerShell is used for automation. Remote Desktop is enabled on Windows machines, and all systems are joined to an Active Directory domain.

![AD drawio](https://github.com/user-attachments/assets/796b3216-aa96-47fa-9ec6-cfa62e0234c8)
  
*Figure 1: Active Directory Lab Architecture*

---

## Objectives
- Set up and configure a multi-VM cybersecurity lab environment.
- Practice system administration, log collection, and endpoint monitoring.
- Deploy and analyze results from security testing tools.
- Automate administrative tasks using PowerShell scripting.

This hands-on experience enhances foundational skills in cybersecurity, virtualization, system configuration, network administration, and security operations.

---

## Skills Acquired
- VM setup in Oracle VM VirtualBox (Windows, Linux distributions)
- NAT networking configuration and IP addressing
- Splunk Enterprise deployment and log collection via Universal Forwarder
- Sysmon installation for detailed endpoint event monitoring
- Security testing using Crowbar and Atomic Red Team
- Active Directory domain creation and management
- Remote Desktop configuration and troubleshooting
- PowerShell automation and scripting

---

## Tools and Technologies
- **Oracle VM VirtualBox**: Virtualization platform
- **Windows 10 & Server 2025**: Target and domain controller
- **Kali Linux**: Attacker machine
- **Ubuntu Server 24.04.2 LTS**: Splunk Server host
- **Splunk Enterprise**: Centralized log analysis platform
- **Splunk Universal Forwarder**: Forwards logs from endpoints
- **Sysmon (System Monitor)**: Monitors endpoint-level events
- **Crowbar**: Brute-force attack tool
- **Atomic Red Team (ART)**: Adversary simulation framework
- **PowerShell**: Used for configuration scripting

---

## Phase 1: Virtual Machine Setup

### 1. Install Oracle VM VirtualBox
Download from https://www.virtualbox.org/. Install the software with all necessary extensions.

### 2. Install Guest Operating Systems
#### a. **Windows 10**
- Download Windows 10 ISO from Microsoft's official site https://www.microsoft.com/en-ca/software-download/windows10.
- Allocate: 3072MB RAM, 1 CPU, 40GB disk
- Boot from ISO and follow standard installation steps.

#### b. **Kali Linux**
- Download the Kali VM image from https://www.kali.org/
- Extract using 7-Zip and import into VirtualBox.

#### c. **Windows Server 2025**
- Download ISO from Microsoft Evaluation Center https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025.
- Allocate: 3072MB RAM, 1 CPU, 40GB disk
- Install Desktop Experience version.

#### d. **Ubuntu Server**
- Download from https://ubuntu.com/server 24.04.2 LTS
- Allocate: 4096MB RAM, 2 CPUs, 50GB disk
- Install and update system:  
  `sudo apt-get update && sudo apt-get upgrade -y`

---

## Phase 2: Networking Configuration

### 1. Create NAT Network in VirtualBox
- Tools > Network > NAT Networks > New
- Configure IP range (e.g., 192.168.10.0/24)
- Attach each VM to this NAT Network

### 2. Assign Static IPs
**Ubuntu (Splunk Server)**:
Edit the file -> sudo nano /etc/netplan/00-installer-config.yaml:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.10.10/24]
      nameservers:
        addresses: [8.8.8.8]
      routes:
        - to: default
          via: 192.168.10.1
```
Run: sudo netplan apply

## Phase 3: Components Setup and Configuration

### Install Splunk on Ubuntu Server

```
1. Download the `.deb` package from Splunk's website or use `wget` to grab the link directly.
2. Transfer it to your Ubuntu server (via shared folder or `wget`) and install:

sudo dpkg -i splunk*.deb
cd /opt/splunk/bin
sudo ./splunk start
sudo ./splunk enable boot-start -user splunk

Make sure the user splunk exists. If not, create it with:

sudo adduser splunk

```

**Windows 10 (Target-PC)**:
---

## 1. Rename the Computer

- Open the **Start Menu**, search for **"About"**, and open the system settings.
- Click on **"Rename this PC"**.
- Rename it to `Target-PC`.
- Restart the system to apply the changes.

---

## 2. Assign Static IP Address

- Press `Win + R`, type `ncpa.cpl`, and hit Enter to open network adapters.
- Right-click the active adapter > **Properties**.
- Select **Internet Protocol Version 4 (TCP/IPv4)** > **Properties**.
- Choose **Use the following IP address** and enter:

  ```
  IP Address:      192.168.10.100
  Subnet Mask:     255.255.255.0
  Default Gateway: 192.168.10.1
  Preferred DNS:   8.8.8.8
  ```

- Apply the settings and confirm changes.
- Open Command Prompt and run:

  ```bash
  ipconfig
  ```

  to verify the new IP address.

---

## 3. Verify Access to Splunk Server

- Open a browser and go to:

  ```
  http://192.168.10.10:8000
  ```

- You should see the Splunk Web login page if the server is running.

---

## 4. Install Splunk Universal Forwarder

- Go to [splunk.com](https://www.splunk.com/).
- Navigate to:

  ```
  Products > Free Trials & Downloads > Universal Forwarder
  ```

- Download the installer that matches your system architecture (32/64-bit).
- Run the installer:
  - Skip setting a password.
  - Do not enable deployment server.
  - For **Receiving Indexer**, use:

    ```
    Host: 192.168.10.10
    Port: 9997
    ```

- Complete the installation.

---

## 5. Install and Configure Sysmon

### Step A: Download Tools

- Download Sysmon:  
  [https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

- Go to:  
  [https://github.com/olafhartong/sysmon-modular](https://github.com/olafhartong/sysmon-modular)

- Scroll to `sysmonconfig.xml`, click **Raw**, then right-click > **Save As**.

### Step B: Install Sysmon

- Extract the Sysmon archive.
- Open **PowerShell as Administrator**.
- Navigate to the extracted Sysmon folder:

  ```powershell
  cd "C:\Path\To\Sysmon"
  ```

- Run the following command to install with the config:

  ```powershell
  .\Sysmon64.exe -i ..\sysmonconfig.xml
  ```

- Accept the license agreement when prompted.

---

## 6. Configure Splunk Forwarder Inputs

- Open **Notepad as Administrator**.
- Create a new file named `inputs.conf` and paste the following:

```
[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```
```
  you should put the inputs.conf in this folder C:\Program Files\SplunkUniversalForwarder\etc\system\local
```
---

#  Windows Server 2025 (Win Server) Setup Checklist

>  Follow the same steps as the Windows 10 (target-PC) setup. Below are the actions to perform on this server.

---

##  Configuration Steps

1. **Rename the Computer**
   - Set the hostname to `Win Server`
   - Restart the system to apply changes

2. **Assign a Static IP Address**
   - Use the following configuration:
     ```
     IP Address:      192.168.10.7
     Subnet Mask:     255.255.255.0
     Default Gateway: 192.168.10.1
     Preferred DNS:   8.8.8.8
     ```

3. **Verify Access to Splunk Server**
   - Open browser and go to:
     ```
     http://192.168.10.10:8000
     ```
   - Ensure the Splunk Web UI is accessible

4. **Install Splunk Universal Forwarder**
   - Download from [Splunk Official Site](https://www.splunk.com/en_us/download/universal-forwarder.html)
   - During installation:
     - Skip deployment server
     - Set receiving indexer: `192.168.10.10:9997`

5. **Install and Configure Sysmon**
   - Download Sysmon from Microsoft Docs
   - Use [sysmon-modular config](https://github.com/olafhartong/sysmon-modular)
   - Install using:
     ```powershell
     .\Sysmon64.exe -i sysmonconfig.xml
     ```

6. **Configure Splunk Forwarder Inputs**
   - Create and save `inputs.conf` with required event logs
   - Place the file in:
     ```
     C:\Program Files\SplunkUniversalForwarder\etc\system\local
     ```

---

 Once completed, logs from `Win Server` will be forwarded to your Splunk server successfully.

---

## Phase 4: Active Directory and Control Domain
---

## 🛠️ 1. Setup Active Directory Domain Services on Windows Server (Win Server)

### 🪟 Add the AD DS Role

1. Open **Server Manager**.
2. Navigate to:  
   `Manage` → `Add Roles and Features`.
3. Proceed through the wizard:
   - Click **Next** until you reach the "Server Roles" section.
   - Check **Active Directory Domain Services**.
   - When prompted, click **Add Features**.
   - Continue clicking **Next** until the **Install** button becomes available.
   - Click **Install**.

> 📌 Once installation completes, you’ll see the message:  
> `"Configuration required. Installation succeeded on Win Server."`

---

### 👑 Promote to Domain Controller

1. Click the **flag icon** at the top of Server Manager.
2. Select **"Promote this server to a domain controller"**.
3. Choose:
   - **Add a new forest**
   - **Root domain name**: `JK.local`
4. On the Domain Controller Options screen:
   - Leave all defaults checked
   - Create a **DSRM password** (Directory Services Restore Mode)
5. Proceed through the wizard:
   - Let the system validate prerequisites
   - Click **Install**

> 🔄 The server will automatically reboot.  
> 🧑‍💻 After reboot, the login screen should show:  
> `JK\Administrator`

---

### 🧍 Create OUs and Users in Active Directory

1. In **Server Manager**, go to:  
   `Tools` → `Active Directory Users and Computers`
2. Right-click on `JK.local` → `New` → `Organizational Unit`
   - Name: `IT`
3. Right-click the **IT** OU → `New` → `User`
   - First Name: `D`
   - Last Name: `K`
   - Full Name: `D K`
   - User Logon Name: `DK`
   - Set a password
4. Repeat the process:
   - Create another OU called **HR**
   - Add a new user under HR with different credentials

---

## 💻 2. Join Windows 10 Machine to the Domain

1. On the **Windows 10 (Target-PC)**:
   - Open **Start Menu** → Search: `Advanced System Properties`
   - Click **Change Settings** under **Computer Name**
   - Click **Change...** and check **Domain:**
     - Enter: `JK.LOCAL`

2. Configure DNS:
   - Right-click the **Network icon** in system tray → Open **Network & Internet Settings**
   - Click: `Change adapter options`
   - Right-click active adapter → `Properties`
   - Select: **Internet Protocol Version 4 (TCP/IPv4)** → `Properties`
     - Set **Preferred DNS server** to: `192.168.10.7`

3. Verify DNS Settings:
   ```cmd
   ipconfig /all
4. Login to the Domain:
   - Reboot the machine
   -  When you go to login, select "Other User", and verify the domain the login is pointing to is "JK". Login using the credentials of a user you created 
   earlier under the IT or HR organizational unit.
   
## Phase 5: Kali Linux Brute Force Attack on the Target Machine
### *1. Configure Network*
Power up the Kali Linux VM and log in. From the desktop, right-click the connections icon at the top right of the screen > Edit Connections > Wired connection 1 > Edit selected connection > IPv4 Settings > set Method to "Manual" > Add > Address: `192.168.10.250` > Netmask: `24` > Gateway: `192.168.10.1` > DNS servers: `8.8.8.8` > Save. Disconnect from the network, then rejoin. Open the terminal and run `ip a` and you should see the changes be reflected here. You can verify connectivity by running `ping` `google.com` and/or `192.168.10.10`. Now run `sudo apt-get update && sudo apt-get upgrade -y`. Continue once finished installing.
### *2. Setting up the Attack*
Run `mkdir ad-project` and `sudo apt-get install -y crowbar`. Now navigate with `cd /usr/share/wordlists`. Verify "rockyou.txt.gz" is present by running `ls`, and unzip it using `sudo gunzip rockyou.txt.gz`. Now copy this file into the directory we made earlier using `cp rockyou.txt ~/Desktop/ad-project`. Now run `cd ~/Desktop/ad-project`. Now we will take the first 20 lines of this file and output it into a new file called passwords.txt using `head -n 20 rockyou.txt > passwords.txt`
### *3. Enable Remote Desktop*
On our target machine, search "This PC" > Properties > Advanced System Settings > Remote > check Allow remote connections to this computer > Select users > Add > Enter the usernames we created earlier such as "DK" and "RK" > Apply.
### *4. Attack*
Navigate back to the Linux machine. `crowbar -h` to get more information about the Crowbar tool. Run `crowbar -b rdp -u DK -C passwords.txt -s 192.168.10.100/32`. If a password listed in `passwords.txt` matches a password assigned to the user-specified, you will get a success message with the password attributed to the specified user.
### *5. Analyze Attack Using Splunk*
We can view all activity of this endpoint using `index=endpoint`, and if we know something happened to Terry's account we can also add `DK`. We will notice when we scroll down and select `EventCode` there are 3 events, and one stands out. Value `4625` stands out because it occurred 20 times. A quick Google search (https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4625) verifies that this is 60 counts of an account failing to log on. When investigating further, it can be noted that all of the counts are occurring at approximately the same time which is an indication of brute force activity. We also notice an event code `4624` which when is looked into (https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4624) we know is an indication of a successful logon. We can expand this event by clicking "Show all x lines", and we can then see the soruce of this logon.
### *6. Run Tests on the Target Machine using Atomic Red Team (ART)*
Open Powershell as administrator. Run `Set-ExecutionPolicy Bypass CurrentUser` > `Y`. Click the up arrow located at the bottom right of your window. Windows Security > Virus & threat protection > Manage Settings > Add or Remove Exclusions > Add an exclusion > Folder > This PC > Local Disk (C:). Now navigate back to PowerShell and run `IEX ( IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);` followed by `Install-AtomicRedTeam -getAtomics` and `Y`.<br><br>Now navigate to the (C:) drive > AtomicRedTeam > Atomics. We can also navigate to https://attack.mitre.org/ in order to view adversary attacks and techniques, as well as use it as a key for the "T" values. Run `Invoke-AtomicTest T1136.001` to T1136 is a persistence, create local account test. After running this test and checking Splunk, we can see that no events popped up with the `NewLocalUser` that was created. This is excellent because we have just identified a gap in our protection. We can continue this with as many tests as we please. This makes Atomic Red Team very valuable for an organization.
### *Summary*
You have reached the end of the walk-through! With the conclusion of the last part of this step-by-step walk-through, you should have a successful setup and communication between 4 Virtual Machines: Windows 10 Target Machine, Kali Linux Attacker Machine, Splunk, and Windows Servers. You should be able to view events with Splunk, test Splunk functionality and defense using Crowbar brute force password cracker, and more. <br><br>
### A sincere thank you to Steven Sir (MyDFIR) for the exceptional project. It provided an excellent opportunity to translate theoretical knowledge into practical experience, greatly enhancing my skills and understanding throughout the process

*Build. Monitor. Defend.*
