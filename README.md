# -Forensics-Incident-Report-Scenario-2
Unauthorized access and privilege escalation on the Debian server

# Incident Summary
On February 6, 2026, a security audit of the auth.log files revealed a successful compromise of a low-privileged user account, followed by successful privilege escalation to root and the installation of a persistence mechanism via an insecure VNC configuration.

# 1. Initial Access: SSH Brute Force/Credential Stuffing
The attack originated from the internal IP address 10.0.0.14. The logs indicate a rapid succession of failed login attempts for the user saturn, suggesting a brute-force or credential-stuffing attack.

Target User: saturn (UID 1000)
Source IP: 10.0.0.14
Service: SSH (Port 22)
Outcome: Successful login at 07:58:43

Evidence:
