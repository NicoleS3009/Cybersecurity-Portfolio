# 🔒 SSH Hardening and Security Audit

Offensive/Defensive cybersecurity laboratory: an SSH server (Ubuntu Server) was hardened following industry best practices and subsequently validated by launching controlled brute-force attacks from a Kali Linux machine, comparing the server's behavior **before** and **after** hardening.

> Project executed as part of the Bachelor's Degree in Cybersecurity, course *Cybersecurity IV* (Laboratory No. 5). All work was conducted within an isolated virtual laboratory environment (private NAT network) without targeting third-party systems.

---

## 🎯 Objective

Demonstrate, in a practical and measurable manner, how a set of hardening controls in `sshd_config` combined with Fail2Ban transforms a brute-force vulnerable SSH server into a target virtually immune to this attack vector, while documenting log evidence.

## 🗺️ Laboratory Topology

Private NAT network `192.168.159.0/24`, virtual bus/star topology.

| Node | Operating System | IP | Role |
|---|---|---|---|
| Attacker | Kali Linux | `192.168.159.134` | Source of scanning (`nc`) and brute-forcing (`hydra`, `medusa`) |
| Victim | Ubuntu Server | `192.168.159.139` | Hardened Host: SSH + Ed25519 Keys + Fail2Ban |
| Channel | Virtual Switch | N/A | Virtual interface (VMware/VirtualBox) connecting nodes |

---

## 🛡️ Phase 1 — SSH Server Hardening

**Installation and Verification of OpenSSH Server**

The package was installed during the initial Ubuntu Server setup, and service activity was confirmed via `systemctl status ssh`.

![OpenSSH Server Installation](screenshots/01_pagina2.png)
![Active SSH Service](screenshots/02_pagina2.png)

**Security Directives Applied in `/etc/ssh/sshd_config`**

A backup (`sshd_config.bak`) was generated prior to editing. The key hardened directives include:

| Directive | Value | Purpose |
|---|---|---|
| `Port` | `2222` | Reduces automated scanning targeting default port 22 |
| `PermitRootLogin` | `no` | Prevents direct root logins |
| `MaxAuthTries` | `3` | Limits authentication attempts per connection |
| `PubkeyAuthentication` | `yes` | Enables public key authentication |
| `PasswordAuthentication` | `no` | Completely disables password-based login |
| `ClientAliveInterval` / `ClientAliveCountMax` | `300` / `0` | Terminates inactive sessions |
| `Banner` | `/etc/ssh/banner.txt` | Displays a legal warning prior to authentication |
| `AllowUsers` | `user` | Restricts which users are permitted to connect via SSH |

![Editing sshd_config - Block 1](screenshots/03_pagina3.png)
![Editing sshd_config - Block 2](screenshots/04_pagina3.png)
![Editing sshd_config - Block 3](screenshots/05_pagina4.png)
![SSH Service Restart](screenshots/06_pagina4.png)

**Public Key Authentication (Ed25519)**

An Ed25519 key pair was generated on the attacker/client host (`ssh-keygen -t ed25519`) and the public key was transferred to the server via `ssh-copy-id` over port 2222.

![Generating Ed25519 Keys](screenshots/07_pagina5.png)
![Transferring Public Key to Server](screenshots/08_pagina5.png)
![Successful Public Key Connection](screenshots/09_pagina5.png)
![Correct Permissions on ~/.ssh](screenshots/10_pagina6.png)

**Fail2Ban** — automated IP blocking based on brute-force behavior:

```ini
[sshd]
enabled  = true
port     = ssh
maxretry = 3
bantime  = 3600
findtime = 600

```

**Legal Warning Banner** (`/etc/ssh/banner.txt`), displayed prior to any authentication attempt:

---

## ⚔️ Phase 2 — Controlled Attack (Before vs. After)

### Before Hardening (Port 22, Password Authentication)

**Hydra** was executed against the target server. Because processing the full `rockyou.txt` dictionary was time-prohibitive, a custom wordlist containing common passwords was leveraged to efficiently execute the attack, and findings were cross-verified using **Medusa**.

Real-time attack monitoring over `/var/log/auth.log`:

**Result:** The server accepted a valid password following a sequence of failed attempts — the dictionary attack succeeded.

### After Hardening (Port 2222, Public Key Only, Fail2Ban Enabled)

The exact same Hydra attack sequence was repeated against the hardened server.

**Result:** `0 valid passwords found`. The server rejects all incoming requests because `PasswordAuthentication` is disabled, and Fail2Ban automatically bans the attacking IP upon exceeding `maxretry`.

---

## 📊 Comparative Analysis

| Feature | Vulnerable Server | Hardened Server |
| --- | --- | --- |
| Attack Surface | Standard Port 22 (easily scanned) | Non-standard Port 2222 |
| Authentication Method | Passwords (susceptible to dictionary attacks) | Ed25519 Keys (immune to brute force) |
| Resilience | None; unlimited attempts permitted | High; automated IP banning after 3 failures |
| Privilege Management | Direct root login possible | Root disabled; specific non-root user required |

**Key Log Differences:** In the vulnerable scenario, `auth.log` records a sequence of `Failed password` messages followed by `Accepted password` — processing each request without resistance. In the hardened scenario, logs show protocol negotiation failures (no password mechanism exposed to test), while `fail2ban.log` records detection and automated IP banning, enforced at the network layer via `iptables`.

---

## ✅ Conclusions

* **Disabling `PasswordAuthentication` was the single most effective control.** Without password authentication exposed, tools such as Hydra or Medusa lose all utility regardless of wordlist size.
* **Security requires defense-in-depth:** changing default ports reduces noise, asymmetric keys eliminate password attack vectors, and Fail2Ban actively mitigates residual attempts.
* **System logs serve as the ultimate source of truth:** without inspecting `auth.log` and `fail2ban.log`, auditing or demonstrating activities during simulated security incidents would be impossible.

## 🚀 Additional Recommendations

* **Port Knocking** — keep the SSH port closed until a valid sequence of packet "knocks" is received.
* **MFA / 2FA (Google Authenticator or Duo via PAM)** — add an extra authentication factor in case private keys are compromised.
* **Auditing with Lynis** — execute automated security scans for kernel hardening and file system permissions.
* **Management VPN** — avoid exposing SSH directly to the Internet; require VPN access prior to reaching management endpoints.

---

## 🧰 Tools Used

`OpenSSH` · `Fail2Ban` · `ssh-keygen` (Ed25519) · `Hydra` · `Medusa` · `nc` · `iptables` · `Kali Linux` · `Ubuntu Server`

## ⚖️ Ethics & Disclaimer

All simulated attacks were executed inside an isolated, self-contained laboratory environment (private NAT network) targeting a virtual machine built explicitly for academic research. No third-party infrastructure was scanned or targeted. This project was developed strictly for educational and defensive research purposes.
