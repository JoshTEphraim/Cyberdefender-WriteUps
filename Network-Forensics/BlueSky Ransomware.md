# Overview

Reconstruct a BlueSky ransomware attack by analyzing network traffic, decoding PowerShell scripts, and examining persistence mechanisms to identify attacker tactics and IOCs.

**Tools:** 
- Wireshark
- Network Miner
- Windows Event Viewer
- Event Log Explorer
- VirusTotal CyberChef

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

3. ***During the investigation, it's essential to determine the account targeted by the attacker. Can you identify the targeted account username?***

We looking at the credential tab in Network miner we can see a username `sa` was used and the protocol whichreveiled this to us was the `TDS - Tabular Data Stream` protocol which is used to transfer data between a database server and a client

  <img width="1185" height="277" alt="image" src="https://github.com/user-attachments/assets/e76ced6a-51e2-4a5b-ae87-2425f58be279" />

On wirehsark side we can query the `TDS` packets and look for the succesfull login packets

<img width="1165" height="787" alt="image" src="https://github.com/user-attachments/assets/fd0222b7-f46e-4fe1-8c29-8ebed06c2692" />

**Ans: sa**

4. 
