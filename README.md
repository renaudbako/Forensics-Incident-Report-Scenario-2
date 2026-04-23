# Forensics Incident Report Scenario-2
Unauthorized access and privilege escalation on the Debian server

# Incident Summary
A security analysis was performed on a compromised Debian server. The investigation identified a multi-stage attack involving a successful SSH brute-force entry, privilege escalation via SUID binaries, and multiple layers of persistence, including an unencrypted VNC backdoor and a scheduled malicious binary execution.

# 1. Initial Access: SSH Brute Force/Credential Stuffing
The attack originated from the internal IP address 10.0.0.14. The logs indicate a rapid succession of failed login attempts for the user saturn, suggesting a brute-force or credential-stuffing attack.

Target User: saturn (UID 1000)
Source IP: 10.0.0.14
Service: SSH (Port 22)
Outcome: Successful login at 07:58:43

Evidence:
![init](./images/init.jpg)

# 2. Privilege Escalation Analysis
Once the attacker gained access as saturn, they immediately attempted to gain administrative control.

Failed Sudo Attempts
The attacker first attempted to use sudo to switch to the root user. These attempts failed as the user saturn is not in the sudoers file, triggering an incident report in the logs.

Exploitation of SUID Binaries
The logs show activity surrounding the /usr/bin/passwd binary. While passwd is a legitimate SUID binary, the attacker attempted to use it to modify the root account’s password.

At 08:07:12, the log shows a successful SSH login for the root user from the same attacker IP (10.0.0.14). This indicates the privilege escalation was successful, either through a specific SUID exploit or by successfully resetting the root password during the previous session.

![esc](./images/escalate.jpg)

# 3. Persistence Mechanisms
The attacker established three distinct methods to maintain access to the system:

A. Insecure VNC Gateway
The attacker created a script at /usr/local/bin/vncstart.sh. This script initializes a VNC server that bypasses standard security protocols, allowing unencrypted, remote desktop access.

![pers](./images/persistence2.jpg)

B. Scheduled Task (Cron Job)
A root-level cron job was identified that ensures a malicious payload is executed every time the system reboots.

Entry: @reboot /etc/system/payload

C. Malicious Binary: /etc/system/payload
Analysis of the file /etc/system/payload reveals a 32-bit ELF executable designed for stealth.

File Type: ELF 32-bit LSB executable, Intel 80386

Characteristics: Statically linked, no section headers.

Behavior: The absence of section headers and the raw hex output (containing syscall-like patterns) suggest this is a compiled shellcode dropper or a reverse shell designed to evade standard forensic analysis and library dependencies.

![pers](./images/cronjob.jpg)

# 4. Technical Artifacts

Attacker IP|10.0.0.14|Source of SSH and VNC traffic.

Backdoor Script|/usr/local/bin/vncstart.sh|Insecure TigerVNC configuration.

Persistence Entry|@reboot /etc/system/payload|Cron job for boot-time execution.

Malicious Binary|/etc/system/payload|32-bit ELF executable (No section headers).

# 5. Remediation Actions

System Isolation: Disconnect the server from the network to prevent further data exfiltration via the VNC backdoor or the ELF payload.

Payload Removal: * Delete /etc/system/payload.

Remove the @reboot entry from the root crontab.

Delete /usr/local/bin/vncstart.sh.

Credential Management: Rotate all passwords, specifically for the root and saturn accounts.

Security Hardening: * Disable SSH password authentication (use SSH keys only).

Implement Fail2Ban to mitigate brute-force attempts.

Audit all SUID/SGID binaries to ensure no others have been tampered with.
