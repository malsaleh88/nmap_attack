# 📌 Windows Security Lab: Nmap Attack & Detection

## 🛠️ **Setup Overview**

This lab tests **network attack and defense** by using **Nmap** from **Kali Linux** to scan a **Windows VM** and then detecting the attack using **Windows Event Viewer & Wireshark**.

---

## 🔹 **Steps Performed**

### **1️⃣ Set Up VirtualBox Network**

- **Windows VM (Target)**: Set network adapter to **Bridged Adapter** (or Host-Only if testing locally).
- **Kali Linux (Attacker)**: Set network adapter to **Bridged Adapter**.
- Found Windows IP address using:
  ```cmd
  ipconfig
  ```

### **2️⃣ Open Ports on Windows VM**

Enabled and verified **SSH (22), HTTP (80), and RDP (3389)**:

#### **✅ Enable RDP (3389)**

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
netsh advfirewall firewall set rule group="Remote Desktop" new enable=Yes
net start termservice
```

#### **✅ Enable HTTP (80)**

```cmd
dism /online /enable-feature /featurename:IIS-WebServerRole /all
net start W3SVC
```

#### **✅ Enable SSH (22)**

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
New-NetFirewallRule -DisplayName "Allow SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow
```

#### **✅ Verify Open Ports**

```cmd
netstat -an | findstr "22 80 3389"
```

---

### **3️⃣ Run Nmap Scan from Kali**

Scanned the Windows target to detect open ports:

```bash
nmap -sS -Pn 192.168.1.53
```

**Results:**

![1](https://github.com/user-attachments/assets/f51d5e15-374b-478c-9d6e-ee61ebc1418c)


---

### **4️⃣ Detect the Attack on Windows**

#### **✅ Using Windows Event Viewer (**``**)**

- Checked **Windows Logs → Security** for:
  - **Event ID 4624** (Successful Logon)
  - **Event ID 4798** (Enumeration of User Accounts)
  - **Event ID 4799** (Security Group Enumeration)


![Screenshot 2025-02-14 220721](https://github.com/user-attachments/assets/5691c619-0419-4903-b57d-646fd0d085ae)


#### **🚨 Analysis of Event Findings**

- **Event ID 4624 (Successful Logon)**

  - This means a user successfully logged in.
  - Key details to check:
    - **Logon Type:**
      - `2` → Local login (physical access)
      - `3` → Network login (remote access, SMB, RDP)
      - `10` → Remote Interactive login (Remote Desktop)
      - `11` → Cached credentials login
    - **Account Name & Domain:** Who logged in?
    - **Source IP Address:** Where did the login come from?
  - ⚠️ If the Source IP is unfamiliar, this could be an attack!

- **Event ID 4798 (Enumeration of User Accounts)**

  - This means someone queried the system for user accounts.
  - Often used in **privilege escalation attacks** or **reconnaissance**.
  - Key details to check:
    - **Account Name:** Who initiated the query?
    - **Logon ID:** Compare with other logs to find the process/user.

- **Event ID 4799 (Security Group Enumeration)**

  - Indicates someone is checking which users belong to a security group.
  - Attackers use this to find **admin or privileged accounts**.
  - Key details to check:
    - Which groups were enumerated? (Admins, Remote Desktop Users, etc.)
    - Which process or user initiated this event?

#### **🚨 What This Means**

✅ **4624 (Login Success)** → Remote or Local User Accessed the System ✅ **4798 (User Enumeration)** → Someone is Gathering Usernames ✅ **4799 (Security Group Enumeration)** → Possible Attack Reconnaissance

These logs strongly suggest **attacker activity**, especially if the logon came from an **unknown IP**.

---

## 🔥 **Next Steps: Attack & Defense**

1. **Try SSH access from Kali to Windows:**
   ```bash
   ssh windows-user@192.168.1.53
   ```
2. **Perform brute-force attacks with Hydra.**
3. **Enable logging in Splunk or Graylog for deeper analysis.**
4. **Implement Windows security hardening to prevent future attacks.**

---

✅ **Lab Completed!** Let me know if you need more security challenges! 🚀

