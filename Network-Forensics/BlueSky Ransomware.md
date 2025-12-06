Q1. - Atttacker ip 87.96.21.84 
    - Activity: Heavy SYN scanning on ip 87.96.21.84
    - Filter used ```tcp.flags.syn == 1 && tcp.flags.ack == 0```
<img width="1492" height="422" alt="image" src="https://github.com/user-attachments/assets/485ece5b-ef99-44b9-ad61-6c0719d0cf30" />

Q2: - Finding targeted account -> Filtered on TDS (Tabular Data Streams) login attempts
    - Protocol: TDS
    - Looking for login packets
<img width="1538" height="810" alt="image" src="https://github.com/user-attachments/assets/f5ce881b-20af-4350-bab4-191b4e983f07" />

Q3:- Finding if attacker gained access and get the password used
    - Protocol: TDS
    - Look for login packets and locate the password
    <img width="1553" height="811" alt="image" src="https://github.com/user-attachments/assets/590150e2-f77d-4d23-8cfa-8f764647f9c7" />

Q4: - Checking for settings changed that may facilitate lateral movement
    - Protocol: TDS
    - Looked for SQL configuration changes
    - EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
    - sp_configure is used to display or change server-level settings
    - RECONFIGURE prevents conflicts and applies the changes made
    - xp_cmdshell allows attacker to run arbitrary code
    <img width="1547" height="735" alt="image" src="https://github.com/user-attachments/assets/4125fef4-3fc4-4b63-bbb1-ea8f6b26af93" />

Q%: - 
