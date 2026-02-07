# RCEMiner

## Overview

***Correlate network traffic, RCE exploits, and C2 communications using Wireshark to reconstruct a multi-stage web server compromise, cryptomining, and lateral movement.***

***

*Category:* `Network Forensics`

*Tactics:* `Execution`, `Discovery`, `Lateral Movement`, `Command and Control`, `Impact`

*Tools:* `Wireshark`, `Brim`

*Difficulty:* `Medium`

***

**Scenario**

***Over the past 24 hours, the IT department has noticed a drastic increase in CPU and memory usage on several publicly accessible servers. Initial assessments indicate that the spike may be linked to unauthorized crypto-mining activities. Your team has been provided with a network capture (PCAP) file from the affected servers for analysis.
Analyze the provided PCAP file using the network analysis tools available to you. Your goal is to identify how the attacker gained access and what actions they took on the compromised server.***

***

**QUESTIONS**

1. ***To identify the entry point of the attack and prevent similar breaches in the future, it’s crucial to recognize the vulnerability that was exploited and the method used by the attacker to execute unauthorized commands. Which vulnerability was exploited to gain initial access to the public webserver?***

By using google we can go ahead and google the name of the lab *RCEMIner* and we receive a hit on a vulnerability regarding the exploit on a PHP server that leads to a remote code execution. More can be found in [Akamai's](https://www.akamai.com/blog/security-research/2024-php-exploit-cve-one-day-after-disclosure#com) article.

<img width="1083" height="450" alt="image" src="https://github.com/user-attachments/assets/69eadcd5-0021-4d40-b230-43d0cba5518c" />

*ANS: `CVE-2024-4577`

2. ***A specific Unicode character is used in the exploit to manipulate how the server interpretes command-line arguments, bypassing the standard input handling. What is the Unicode code point of this character?***







































































































