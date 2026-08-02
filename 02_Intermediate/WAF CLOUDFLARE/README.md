# Cloudflare WAF (Web Application Firewall) Laboratory

**Web Application Protection via Cloudflare: DNS, Tunnel, SSL/TLS, and Custom WAF Rules**

Technological University of Panama · Faculty of Computer Systems Engineering  
Department of Computer Architecture and Networks · Bachelor's Degree in Cybersecurity  
Course: **Cybersecurity IV**

---

## 📌 Introduction

In the current cybersecurity context, web applications are one of the primary targets for attacks. This laboratory documents the protection of an application using a **Web Application Firewall (WAF)** provided by **Cloudflare**, covering everything from domain registration to real-world attack blocking tests.

**Domain used:** [`nicocypher.me`](https://nicocypher.me)

---

## 🌐 1. Domain Registration

The domain `nicocypher.me` was registered on Namecheap.

<p align="center">
  <img src="screenshots/01_compra_dominio_namecheap.png" width="650" alt="Namecheap Dashboard with active nicocypher.me domain">
</p>
<p align="center"><em>Namecheap Dashboard — active <code>nicocypher.me</code> domain.</em></p>

---

## 🔌 2. Configuring Cloudflare — Tunnel

The initial Cloudflare configuration pointed to an AWS server; however, upon expiration of the trial period, a decision was made to establish a **tunnel** to a local Ubuntu Server (from previous laboratories).

**Steps:**
1. From the Cloudflare Dashboard (**Zero Trust → Networks → Tunnels**), create the tunnel.
2. Copy the generated command and execute it in the Ubuntu Server terminal (via PuTTY).

<table>
<tr>
<td width="50%">
<p align="center"><img src="screenshots/02_crear_tunnel_panel.png" width="380" alt="Create a tunnel in Cloudflare Zero Trust"></p>
<p align="center"><em>Tunnel creation in Zero Trust.</em></p>
</td>
<td width="50%">
<p align="center"><img src="screenshots/03_tunnel_terminal_putty.png" width="380" alt="cloudflared installation via PuTTY"></p>
<p align="center"><em>Installing <code>cloudflared</code> on the Ubuntu Server.</em></p>
</td>
</tr>
</table>

### Active Connector and Service Routing

Once installed, the connector becomes available and online. To ensure Cloudflare knows where to route incoming traffic, complete the **Service** field with the local URL (`localhost:80`) before saving.

<table>
<tr>
<td width="50%">
<p align="center"><img src="screenshots/04_connector_online.png" width="380" alt="Connector connected and online"></p>
<p align="center"><em>Connector <strong>Connected</strong>.</em></p>
</td>
<td width="50%">
<p align="center"><img src="screenshots/05_service_config.png" width="380" alt="Service configuration in the tunnel"></p>
<p align="center"><em><code>Service</code> Configuration (HTTP → localhost:80).</em></p>
</td>
</tr>
</table>

<p align="center">
  <img src="screenshots/06_tunnel_listo.png" width="600" alt="Cloudflare Tunnel ready and healthy">
</p>
<p align="center"><em><code>cloudflared</code> Tunnel in <strong>HEALTHY</strong> status.</em></p>

---

## 🗂️ 3. DNS Records and SSL/TLS

DNS records (proxied) for the domain are verified, and active **SSL/TLS** encryption between the browser, Cloudflare, and the origin server is confirmed.

<p align="center">
  <img src="screenshots/07_dns_records.png" width="650" alt="Proxied DNS records in Cloudflare">
</p>
<p align="center"><em>Domain DNS records, all with Cloudflare proxy active.</em></p>

<table>
<tr>
<td width="50%">
<p align="center"><img src="screenshots/08_ssl_tls_modo.png" width="380" alt="Full SSL/TLS encryption mode"></p>
<p align="center"><em>Encryption mode: <strong>Full</strong>.</em></p>
</td>
<td width="50%">
<p align="center"><img src="screenshots/09_hsts.png" width="380" alt="Always Use HTTPS and HSTS active"></p>
<p align="center"><em><strong>Always Use HTTPS</strong> + HSTS enabled.</em></p>
</td>
</tr>
</table>

**DNS Resolution Test:** upon navigating to `https://nicocypher.me`, the site responds successfully (serving the default Apache2 page on Ubuntu), confirming end-to-end operation of both the tunnel and DNS.

<p align="center">
  <img src="screenshots/10_apache_default_ok.png" width="650" alt="Apache2 Default Page served via nicocypher.me">
</p>
<p align="center"><em>Domain correctly resolves to the Ubuntu server's Apache2 instance.</em></p>

---

## 🛡️ 4. Configuring the WAF — Layer by Layer

Cloudflare's **Free** plan allows adding up to **5 custom WAF rules** for the domain.

<p align="center">
  <img src="screenshots/11_plan_free_waf.png" width="600" alt="Cloudflare Free Plan - limit of 5 WAF rules">
</p>
<p align="center"><em>Free Plan: 5 WAF Rules, 70 Cloudflare Rules, Universal SSL, Global CDN.</em></p>

### Layer 1 — Cloudflare Managed Ruleset

A ruleset maintained by Cloudflare covering the **OWASP Top 10**, known exploits, and malicious bots. Activated with a single click and **must always be ON**.

<p align="center">
  <img src="screenshots/12_capa1_managed_ruleset.png" width="600" alt="Cloudflare Managed Ruleset active">
</p>

### Layer 2 — Custom WAF Rules (5/5 used)

<p align="center">
  <img src="screenshots/13_capa2_reglas_custom.png" width="650" alt="Table of 5 active custom WAF rules">
</p>
<p align="center"><em>All 5 custom rules are <strong>Active</strong>.</em></p>

| # | Rule | Expression (Summary) | Function / Protection |
|---|---|---|---|
| 1 | **Custom-SQLi-Basic** | `http.request.uri.query contains "UNION SELECT"` | Blocks SQL Injection: detects the `UNION SELECT` pattern used to combine legitimate queries with malicious statements to extract hidden data. |
| 2 | **Custom-XSS-Basic** | contains `<script>`, `javascript:`, `onerror=`, `alert(` | Blocks Cross-Site Scripting: prevents injection of JavaScript capable of stealing session cookies or credentials. |
| 3 | **Protect-Login-Endpoint** | `http.request.uri.path equals ...` (login path) | Protects authentication endpoints against unauthorized access and brute-force attempts. |
| 4 | **Geo-blocking-LATAM** | `ip.geoip.country ne "PA"` | Geoblocking: restricts access to requests outside Panama, reducing malicious traffic from regions without a legitimate business need. |
| 5 | **Anti-scanner-Tools** | `http.user_agent contains "sqlmap"` (also `nikto`, `dirbuster`) | Detects and blocks automated security scanners via User-Agent strings, neutralizing noise prior to enumeration. |

### Layer 3 — Bot Fight Mode & Security Level

<table>
<tr>
<td width="50%">
<p align="center"><img src="screenshots/14_bot_fight_mode.png" width="380" alt="Bot Fight Mode active"></p>
<p align="center"><em><strong>Bot Fight Mode</strong> ON — behavioral fingerprinting (Free plan baseline version).</em></p>
</td>
<td width="50%">
<p align="center"><img src="screenshots/15_security_level.png" width="380" alt="Security Level - I'm Under Attack Mode disabled"></p>
<p align="center"><em><strong>"I'm Under Attack Mode"</strong> disabled — reserved for active DDoS attacks; enabling it pre-emptively introduces unnecessary friction for legitimate users.</em></p>
</td>
</tr>
</table>

The **Challenge Passage** is set to its default duration of 30 minutes.

<p align="center">
  <img src="screenshots/16_challenge_passage.png" width="600" alt="Challenge Passage set to 30 minutes">
</p>

### Layer 4 — DDoS Protection & Analytics

<table>
<tr>
<td width="50%">
<p align="center"><img src="screenshots/17_ddos_protection.png" width="380" alt="HTTP DDoS Attack Protection active"></p>
<p align="center"><em><strong>HTTP DDoS Attack Protection</strong> — active automatically, sensitivity adjusted to <em>High</em> for the lab environment.</em></p>
</td>
<td width="50%">
<p align="center"><img src="screenshots/18_analytics_events.png" width="380" alt="Analytics - Security Events logging blocks"></p>
<p align="center"><em><strong>Security → Events</strong>: real-time block events recorded (tested from an external network / hotspot).</em></p>
</td>
</tr>
</table>

---

## 🔥 5. Real-World Testing (WAF Validation)

The **XSS** rule was evaluated by submitting the following payload request:

```
https://nicocypher.me/?buscar=<script>alert(1)</script>
```
The WAF blocks the request instantly:

<p align="center">
  <img src="screenshots/19_xss_bloqueado.png" width="600" alt="Cloudflare 'Sorry, you have been blocked' page">
</p>
<p align="center"><em>Cloudflare blocks the XSS attempt: "Sorry, you have been blocked".</em></p>

When navigating normally (without malicious payloads), the application remains fully accessible:

<table>
<tr>
<td width="50%">
<p align="center"><img src="screenshots/20_verificacion_seguridad.png" width="380" alt="Cloudflare security check in progress"></p>
</td>
<td width="50%">
<p align="center"><img src="screenshots/21_sitio_protegido.png" width="380" alt="DEVILSEC ACADEMY site protected by Cloudflare"></p>
</td>
</tr>
</table>
<p align="center"><em>Legitimate traffic passes through without disruption — application secured by Cloudflare WAF.</em></p>

---

## ✅ Conclusion

Deploying Cloudflare's WAF offers an effective solution to harden web applications against increasingly sophisticated cyber threats. Through domain integration, Cloudflare's proxying capabilities, and custom rule design, common attacks such as code injections, automated reconnaissance, and unauthorized geographic traffic can be efficiently mitigated. This project demonstrates the critical value of pairing advanced technical tools with robust defensive strategies to maintain effective application security posture.