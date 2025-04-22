# Task Scheduler UAC Bypass
By [Ruben Enkaoua](https://x.com/rubenlabs) and [Cymulate](https://cymulate.com/)
<br>
<br>
[Original Blog: Task Scheduler New Vulnerabilities](https://cymulate.com/blog/task-scheduler-new-vulnerabilities-for-schtasks-exe/)
<br>
<br>

#### Description
<br>
A UAC Bypass vulnerability has been found in Microsoft Windows, enabling attackers to bypass the User Account Control prompt, allowing them to execute high-privilege (SYSTEM) commands without user approval. By exploiting this weakness, attackers can elevate their privileges and run malicious payloads with Administrators’ rights, leading to unauthorized access, data theft, or further system compromise. 

It occurs when an attacker creates a task using Batch Logon and not with an Interactive Token. The Task Scheduler service, impersonating the user, is granting the running process the maximum allowed rights, elevating it from any integrity to the highest available. 

Notice that it elevates to the highest privileges not only for Administrators users but also for Backup Operators (allowing to get SeBackup privileges) and for Performance Log Users (Read more in the Cymulate blog).
<br>
<br>

#### Requirements
<br>

- Batch Logon rights for the task principal user (impersonated user)<br>
- Credentials of the task principal
<br>

#### Command
<br>

```bash
# Elevate to High Integrity
schtasks /create /tn poc /tr command.exe /sc minute /mo 1 /ru <username> /rp <password> /f
schtasks /run /tn poc

# Elevate to SYSTEM
schtasks /create /tn tmp /sc minute /mo 1 /ru <username> /rp <password> /tr "schtasks /create /tn poc /tr command.exe /sc minute /mo 1 /RL HIGHEST"
schtasks /run /tn tmp
schtasks /delete /tn tmp /f
```
<br>

#### Explanations
<br>

The taskhostw.exe process, hosted by svchost.exe, is running as SYSTEM. When the service is impersonating the user, a high Integrity context is initiated for the task to run.
The difference between a password registration (Batch Logon), and a default  registration (InteractiveToken) is the authentication method, as mentioned in MSDN. It will determine the integrity of the process to run. 

Along the same lines, after elevating from a medium to a high integrity, it is not enough for our task to run as SYSTEM. Impersonating an admin user with batch logon doesn’t guarantee the ability to run as SYSTEM since it wasn’t intended to run elevated. If we add to our method the /RL HIGHEST flag, we get Access Denied. 

But to remediate it, we can elevate our privileges to high with the previously mentioned UAC bypass by creating a temporary task, which then creates our SYSTEM main task from inside (see before, Elevate to SYSTEM).

<br>

#### Notes
<br>
This code is for educational and research purposes only.<br>
The author takes no responsibility for any misuse of this code.
