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

**ANS: `checking.ps1`***

7. ***Understanding which group Security Identifier (SID) the malicious script checks to verify the current user's privileges can provide insights into the attacker's intentions. Can you provide the specific Group SID that is being checked?***















