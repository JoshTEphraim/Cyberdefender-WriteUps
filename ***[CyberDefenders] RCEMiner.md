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

*ANS: `CVE-2024-4577`*

2. ***A specific Unicode character is used in the exploit to manipulate how the server interpretes command-line arguments, bypassing the standard input handling. What is the Unicode code point of this character?***

we use the query `http` we can see that there is a *POST* requests to the server on the IP `36.96.48.3` at port `80`

<img width="1683" height="776" alt="image" src="https://github.com/user-attachments/assets/030086af-dc0e-44fa-9e70-10162bdaa9bb" />

Lets follow the stream in order to make sense of what was being posted on the server

<img width="1261" height="729" alt="image" src="https://github.com/user-attachments/assets/6d998599-536b-4b02-a494-c1d42910a3dd" />

The request made to the server is susicious as we can see that the attacker has found a vulnerability in the PHP's version similar to the one  of *CVE-2012-1823* see the article on [Github](https://github.com/php/php-src/security/advisories/GHSA-3qgc-jrrr-25jv)

The attacker injects a command `-d allow_url_include=1 -d auto_prepend_file=php://input` where `-d allow_url_include=1` is used to manipulates PHP configuration by enablinbg it to allow the inclusion of remote files and the other input makes it worse which is `auto_append_file=php://input` which which allows the inclusion of PHP code directly from the HTTP body. This now allows the attacker to perform RCE on every request. We can see the code on the request  which uses the `system()` function to execute a command on the server by calling a URL `http://1.80.23.4:8000/`which outputs the string `1337` and later kills it using the `die` statement. To conclude the attacker must have found the vulneraility on the version of PHP used as we can see its `PHP/8.1.25`

Using google we find an article by [watchtowr](https://labs.watchtowr.com/no-way-php-strikes-again-cve-2024-4577/) that explains that the attacker used invocations of `php.exe` if which one was them was malicious where the dash before the `s` differ, when looked from a Hex editor one is indieed a normal dash (0x2D) whole the other is not, but its a soft hyphen (0xAD).

Why is this important, well here lies the vulnerability as where PHP sees the actual hyphen and it escspes or sanitizes it thinking its a dangerous option, but when the attacker used a soft hyphen (OxAD) apache thinks its safe, while PHP thinks its a command line option and executes it.

*ANS: `(0xAD)`*

3. ***The attacker executed commands to gather detailed system information, including CPU specifications, after gaining access. What is the exact model of the CPU identified by the attacker’s script?***

We know he attacker downloaded a script `1.ps1` now we need to check for the supplied data form input sent by looking for the `application/x-www-form-urlencoded`. Frome the statistics -> protocol hierachy we can see the presence of the form data input.

<img width="1439" height="388" alt="image" src="https://github.com/user-attachments/assets/17baea73-ef67-49e9-a96a-9fb04d5838f6" />

From the search query we can use `urlencoded-form` and follow the first stream that shows up

<img width="1271" height="729" alt="image" src="https://github.com/user-attachments/assets/e79cf2dd-efcb-4d10-aa6a-00483c12c503" />

We now can see the output of the command the attacker used to find about the servers exact model

*ANS: `Intel(R) Core(TM) i7-6700HQ`

4. ***Understanding how malware initiates the execution of downloaded files is crucial for stopping its spread and execution. After downloading the file, the malware executed it with elevated privileges to ensure its operation. What command was used to start the process with elevated permissions?***

In order to start a process with elevated privileges in powershell we use the cmdlet `Start-Process`, for which attacker likes using, lets query it in wireshark using the query string `frame matches "Start-Process"`

<img width="1494" height="215" alt="image" src="https://github.com/user-attachments/assets/914e069d-9cae-40f2-9074-04e5864a2d31" />

Following the stream we see the attacker started a process `2.exe` with elevated privileges

<img width="1003" height="525" alt="image" src="https://github.com/user-attachments/assets/f6275b2c-ec90-4470-92e7-e570112f0502" />

*ANS: `Start-Process C:\Windows\Temp\2.exe -Verb RunAs`*

5. ***After compromising the server, the malware used it to launch a massive number of HTTP requests containing malicious payloads, attempting to exploit vulnerabilities on additional websites. What vulnerable PHP framework was initially targeted by these outbound attacks from the compromised server?***

Using the query `http.request.method==GET` to find what request was being requested, we see alot of of GET request to `index/\think/` 

<img width="1550" height="470" alt="image" src="https://github.com/user-attachments/assets/1a731344-7ca6-4561-a8f0-96168c33f702" />

Doing a quick google search can tell us what was being targeted

<img width="1157" height="425" alt="image" src="https://github.com/user-attachments/assets/a348a16f-5b0c-466e-bd14-03753556609c" />

We find that the attacker was quering the `ThinkPHP` framework

*ANS: `ThinkPHP`*

6. ***The malware leveraged a common network protocol to facilitate its communication with external servers, blending malicious activities with legitimate traffic. This technique is documented in the MITRE ATT&CK framework. What is the specific sub-technique ID that involves the use of DNS queries for command-and-control purposes?***

Prorocols such as DNS, HTTPS are used by attackers to blend malicious traffic as most of this protocols are usually trusted, and by using them the attacker can avoid detection. We will go over to [Mitre ATT&CK](https://attack.mitre.org/) and go over the `Command and Control` technique

<img width="1643" height="777" alt="image" src="https://github.com/user-attachments/assets/e94ae767-f4cb-47a4-9f69-f13c11b03328" />

The question asks for the sub-technique and by scolling down we find one under *DNS*

<img width="1605" height="915" alt="image" src="https://github.com/user-attachments/assets/2e0b723d-c1e8-471f-891c-97eeac801f66" />

*ANS: `T1071.004`*

7. ***Identifying where the malware could be stored on a compromised system is crucial for ensuring the complete removal of the infection and preventing the malware from being executed again. The compromised server was used to host a malicious file, which was then delivered to other vulnerable websites. What is the full path where this malware was stored after being downloaded from the compromised server?***

we need to focus on the POST request here and the query I used was `http.request.method==POST and http contains "exe"` where we see alot of form data, following the stream of one of them we can see a file was uploaded but its encoded and this is where CyberChef comes in

<img width="1000" height="573" alt="image" src="https://github.com/user-attachments/assets/1bbff303-4010-402f-836d-c32a681e2ab2" />

Copying the encoded url and posting it on the swiss army kniife (CyberChef) we find the path of the stored malware

<img width="1534" height="631" alt="image" src="https://github.com/user-attachments/assets/ce23ffd3-c8d9-4fd3-88c9-6e46a637e099" />

*ANS: `C:\ProgramData\spread.exe`*

8. ***Knowing the destination of the data being exfiltrated or reported by the malware helps in tracing the attacker and blocking further communications to malicious servers. The compromised server was used to report system performance metrics back to the attacker. What is the IP address and port number to which this data was sent?***










































































