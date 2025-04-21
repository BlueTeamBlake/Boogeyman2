# Boogeyman2
# 🕵️ TryHackMe - Boogeyman2 (Tools: Volatility & Olevba)

## Overview

After having a severe attack from the Boogeyman, Quick Logistics LLC improved its security defences. However, the Boogeyman returns with new and improved tactics, techniques and procedures

Maxine, a Human Resource Specialist working for Quick Logistics LLC, received an application from one of the open positions in the company. Unbeknownst to her, the attached resume was malicious and compromised her workstation.

The security team was able to flag some suspicious commands executed on the workstation of Maxine, which prompted the investigation. Given this, you are tasked to analyse and assess the impact of the compromise.
---


### We can use this screenshot for questions 1-3

![email](email_screenshot.png)

### Question 1: What email was used to send the phishing email?

**Answer:** `westaylor23@outlook.com`

---

### Question 2: What is the email of the victim employee?

**Answer:** `maxine.beck@quicklogisticsorg.onmicrosoft.com`

---

### Question 3: What is the name of the attached malicious document?

**Answer:** `Resume_WesleyTaylor.doc`

---

### Question 4: What is the MD5 hash of the malicious attachment?
![md5sum](question4_answer.png)

**Explanation:** Save the file, open a new terminal and `md5sum <filename>`

**Answer:** `52c4384a0b9e248b95804352ebec6c5b`

---

### Question 5: What URL is used to download the stage 2 payload based on the document's macro?
![olevba](Olevba_screenshot.png)

**Explanation:**: In our tool `olevba <filename>`, under our IOC we'll see our filepath

**Answer:** `https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png`

---

### Question 6: What is the name of the process that executed the newly downloaded stage 2 payload?

![olevba](Olevba_screenshot.png)

**Explanation:**: Also under our IOC in olevba. Additionally we can confirm in volatility by checking windows.cmdline

**Answer:** `wscript.exe`

---

### Question 7: What is the full file path of the malicious stage 2 payload?

![pidpayload](question7_answer.png)

**Explanation:** In volatility, `vol -f WKSTN-2961.raw windows.cmdline | grep update`


**Answer:** `C:\ProgramData\update.js`


### Question 8: What is the PID of the process that executed the stage 2 payload?

![PID](question9_answer.png)

**Explanation:** `vol -f WKSTN-2961.raw windows.pslist & cmdline` shows the list of processes, as I went through this lab I noticed updater.js > updater.exe in the windows.cmdline

**Answer:** `PID: 4260`

### Question 9: What is the parent PID of the process that executed the stage 2 payload?

![pidpayload](question9_answer.png)


**Explanation:** Checking the parent pid of the same .exe. 


**Answer:** `PPID: 1124`

### Question 10: What URL is used to download the malicious binary executed by the stage 2 payload?

![url](Olevba_screenshot.png)

**Explanation:** There are a few ways to find this answer, simple one if the IOC on olevba report. 
**Answer:** `https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.exe`

### Question 11: What is the PID of the malicious process used to establish the C2 connection?

![question11](question11_answer.png)

**Explanation:** Back to tracking updater.exe, check outbound connections with windows.netstat

**Answer:** `PID: 6216`

### Question 11: What is the full file path of the malicious process used to establish the C2 connection?

![path](question10_answer.png)

**Explanation:** Checking the cmdline again we can see where the file was placed.

**Answer:** `C:\Windows\Tasks\updater.exe`

### Question 12: What is the IP address and port of the C2 connection initiated by the malicious binary? (Format: IP address:port)

![ip](question11_answer.png)

**Explanation:** Here check the windows.netstat and search for the updater.exe outbound connection

**Answer:** `128.199.95.189:8080`

### Question 13: What is the full file path of the malicious email attachment based on the memory dump?

![path2](question12_answer.png)

**Explanation:** With Volatility we can check the paths of file in mem with `windows.filescan`

**Answer:** `C:\Users\maxine.beck\AppData\Local\Microsoft\Windows\INetCache\Content.Outlook\WQHGZCFI\Resume_WesleyTaylor (002).doc`

### Question 14: The attacker implanted a scheduled task right after establishing the c2 callback. What is the full command used by the attacker to maintain persistent access?


![final](question14_answer.png)

**Explanation:** we can simply using strings on the .raw file and grep | schtasks for scheduled tasks and we'll find our answer

**Answer:** `schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c \"IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))\"'`







