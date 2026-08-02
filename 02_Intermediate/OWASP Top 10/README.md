# Pentest Lab — Burp Suite · OWASP Juice Shop

**Web security assessment with Burp Suite against OWASP Juice Shop**

## 📌 Executive summary

A security assessment was performed against the **OWASP Juice Shop** web application for training and vulnerability-analysis practice purposes. During the process **10 security flaws** were identified, of which **5 were classified as critical severity** due to the risk they pose to the confidentiality, integrity, and availability of the system.

---

## 🛠️ Full setup

| | |
|---|---|
| **Burp Suite** — start screen, target scope definition | <img src="screenshots/00_setup_burpsuite.png" width="380"> |
| **Browser's own proxy** (FoxyProxy) pointed at Burp | <img src="screenshots/01_setup_proxy_navegador.png" width="380"> |
| **Docker installed** — `docker --version` | <img src="screenshots/02_setup_docker_instalado.png" width="380"> |
| **Scope** — Juice Shop container running, health check OK | <img src="screenshots/03_setup_scope.png" width="380"> |

---

## 📊 Findings summary

| Vulnerability | Severity | OWASP | Status |
|---|---|---|---|
| Admin Panel – Broken Access Control | **Critical** | A01 | ✅ Confirmed |
| SQL Injection – Login Bypass | **Critical** | A02 | ✅ Confirmed |
| DOM XSS | **Critical** | A05 | ✅ Confirmed |
| Bonus Payload (iframe injection) | **Critical** | A05 | ✅ Confirmed |
| Admin Registration | High | A01 | ✅ Confirmed |
| IDOR – Access to Another User's Profile | **Critical** | A01 | ✅ Confirmed |
| Directory Listing — Exposed /ftp | Medium | A01 | ✅ Confirmed |
| Error Handling | Medium | A02 | ✅ Confirmed |
| API Data Exposure — User Enumeration | Medium | A01 | ✅ Confirmed |
| Broken Auth — No Rate Limiting on Login | Medium | A07 | ✅ Confirmed |

---

## 🔍 Detailed findings

### Write-up 1 — Admin Panel: Broken Access Control
**Severity:** CWE-284: Improper Access Control

**Description:** The administration panel is accessible without authentication or privilege validation; any user can reach admin functionality simply by discovering the `/admin` route.

**Steps to reproduce:**
1. Access `http://localhost:3000`.
2. Review the Site Map or manually inspect loaded resources.
3. Analyze `main.js` to identify hidden routes.
4. Find the reference to `/admin`.
5. Navigate directly to `http://localhost:3000/admin`.
6. Observe that the panel is accessible without authentication.
7. Confirm the "Challenge completed" banner.

**Evidence:**

| Site map tree. Looking for hidden routes (`main.js`) | Finding the route to admin |
|---|---|
| <img src="screenshots/04_wu1_sitemap_arbol.png" width="380"> | <img src="screenshots/05_wu1_busqueda_ruta_admin.png" width="380"> |

| Admin panel | Challenge completed banner |
|---|---|
| <img src="screenshots/06_wu1_panel_administrador.png" width="380"> | <img src="screenshots/07_wu1_banner_reto_completado.png" width="380"> |

**Real impact:** unauthorized access to administrative functionality, possible manipulation of sensitive data, exposure of internal information, privilege escalation risk; in a real environment this could mean full system compromise.

**Remediation:** mandatory authentication on `/admin` and any sensitive route; RBAC on the backend; avoid exposing sensitive routes in public JS; backend middleware validating tokens/sessions; periodic security testing.

**OWASP/CWE reference:** [A01:2025 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/) · [CWE-284](https://cwe.mitre.org/data/definitions/284.html)

---

### Write-up 2 — SQL Injection: Login Bypass
**Severity:** CWE-89 – Improper Neutralization of Special Elements used in an SQL Command

**Description:** The `email` parameter on the login endpoint is not validated or sanitized, allowing an attacker to alter the authentication SQL logic and bypass the login.

**Steps to reproduce:**
1. Access the Juice Shop login page.
2. Intercept traffic with Burp Suite.
3. Attempt login with any credentials (`test@test.com` / `test`).
4. Intercept `POST /rest/user/login` and send it to Repeater.
5. Modify the `email` field with a payload that alters the SQL logic.
6. Send and observe the response contains a JWT token (login bypassed).
7. Use the token to confirm access with elevated privileges.

**Evidence:**

| Modifying response interception rules | Enabling intercept |
|---|---|
| <img src="screenshots/08_wu2_modificacion_reglas_intercepcion.png" width="380"> | <img src="screenshots/09_wu2_activar_intercept.png" width="380"> |

| User logging in | Intercepted POST request |
|---|---|
| <img src="screenshots/10_wu2_usuario_login.png" width="380"> | <img src="screenshots/11_wu2_post_interceptado.png" width="380"> |

| Before modifying the payload | Modified payload + response with token |
|---|---|
| <img src="screenshots/12_wu2_antes_modificacion_payload.png" width="380"> | <img src="screenshots/13_wu2_payload_modificado_token.png" width="380"> |

<p align="center"><img src="screenshots/14_wu2_acceso_con_token.png" width="500"></p>
<p align="center"><em>Application access using the obtained token.</em></p>

<p align="center"><img src="screenshots/15_wu2_banner_reto_completado.png" width="600"></p>
<p align="center"><em>"Login Admin" challenge completed.</em></p>

**Real impact:** unauthorized access to accounts (including admin), exposure/modification/deletion of data, full application compromise, privilege escalation, and potential pivoting.

**Remediation:** parameterized queries (prepared statements); strict input validation/sanitization; avoid string-concatenated SQL; WAF, anomaly monitoring, and rate limiting; code review; rotate compromised credentials/tokens.

**OWASP/CWE reference:** [A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/) · [CWE-89](https://cwe.mitre.org/data/definitions/89.html)

---

### Write-up 3 — DOM XSS
**Severity:** CWE-79 – Improper Neutralization of Input During Web Page Generation

**Description:** Client-side JavaScript processes untrusted URL/DOM data without sanitization, enabling DOM-based XSS.

**Steps to reproduce:**
1. In the Juice Shop search bar, enter: `<iframe src="javascript:alert('DOM_XSS')">`
2. Press Enter → the `alert` fires.
3. In Burp HTTP History, check `GET /rest/products/search?q=` and see the payload in the URL.

**Evidence:**

| Alert firing in the browser | URL with the payload |
|---|---|
| <img src="screenshots/16_wu3_alerta_dom_xss.png" width="380"> | <img src="screenshots/18_wu3_url_con_payload.png" width="380"> |

<p align="center"><img src="screenshots/17_wu3_banner_reto_completado.png" width="500"></p>
<p align="center"><em>"DOM XSS" challenge completed.</em></p>

**Real impact:** arbitrary JS execution, session token/cookie theft, identity spoofing, content manipulation, malicious redirects, escalation to CSRF/internal phishing.

**Remediation:** avoid `innerHTML`/`document.write`/`eval`; use `textContent`/`innerText`/`createElement`; strict sanitization of URL/DOM data; CSP; review dangerous sinks.

**OWASP/CWE reference:** [A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/) · [CWE-79](https://cwe.mitre.org/data/definitions/79.html)

---

### Write-up 4 — Bonus Payload
**Severity:** CWE-79 – Improper Neutralization of Input During Web Page Generation

**Description:** The vulnerable DOM XSS sink also allows injecting arbitrary HTML, including an `<iframe>` from an external domain (SoundCloud in the test).

**Steps to reproduce:**
1. Identify the vulnerable functionality reflecting URL/DOM data without sanitization.
2. Confirm the insecure sink (`innerHTML`).
3. Replace the value with an HTML payload containing an `<iframe>`.
4. Reload and observe the external iframe rendering inside the app.
5. Confirm the content comes from an external domain.

**Evidence:**

<p align="center"><img src="screenshots/19_wu4_banner_bonus_payload.png" width="600"></p>
<p align="center"><em>"Bonus Payload" challenge completed — SoundCloud iframe embedded inside Juice Shop.</em></p>

**Real impact:** visual phishing inside the application itself, fake forms/interfaces, manipulation of displayed content, potential social engineering, loss of user trust.

**Remediation:** avoid insecure sinks; strict sanitization (e.g., DOMPurify); validate/encode output; CSP blocking unauthorized external iframes; review client-side data flows.

**OWASP/CWE reference:** [A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/) · [CWE-79](https://cwe.mitre.org/data/definitions/79.html)

---

### Write-up 5 — Admin Registration
**Severity:** CWE-639 - Authorization Bypass Through User-Controlled Key

**Description:** The app trusts a client-supplied role parameter during registration without server-side validation, allowing creation of admin-privileged accounts.

**Steps to reproduce:**
1. Access the user registration page.
2. Intercept traffic with Burp Suite.
3. Fill out the form and submit.
4. Intercept `POST /api/Users` and send it to Repeater.
5. Modify/add the role parameter.
6. Send the modified request.
7. Confirm successful registration with admin privileges.

**Evidence:**

| Modified payload + server response | Challenge completed banner |
|---|---|
| <img src="screenshots/20_wu5_payload_modificado_respuesta.png" width="380"> | <img src="screenshots/21_wu5_banner_reto_completado.png" width="380"> |

**Real impact:** unauthorized creation of admin accounts, access to management panels, data modification/exfiltration, full compromise of integrity/availability.

**Remediation:** strip any role parameter sent from the client; assign roles server-side only; strict backend validation; authorization controls on all sensitive endpoints; monitor parameter tampering.

**OWASP/CWE reference:** [A01:2025 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/) · [CWE-639](https://cwe.mitre.org/data/definitions/639.html)

---

### Write-up 6 — IDOR: Access to Another User's Profile
**Severity:** CWE-639 - Authorization Bypass Through User-Controlled Key

**Description:** Internal resources can be accessed/manipulated by simply modifying an identifier in the URL or request body, without authorization checks.

**Steps to reproduce:**
1. Log in with a legitimate user.
2. Navigate to a resource retrieved by ID (`/api/orders/101`).
3. Intercept with Burp Suite → Repeater.
4. Modify the ID (101 → 102, 103...).
5. Send the modified request.
6. Observe the server returns another user's data.

**Evidence:**

<p align="center"><img src="screenshots/22_wu6_idor_request_response.png" width="650"></p>
<p align="center"><em><code>GET /api/Users/1</code> returns <code>admin@juice-sh.op</code>'s data without validating the requester is that user.</em></p>

**Real impact:** access to other users' personal information, reading/modifying orders/profiles, horizontal and vertical escalation, massive data exposure, compliance risk (GDPR/PCI/HIPAA).

**Remediation:** server-side per-resource authorization checks; validate authenticated user's permission; unpredictable UUIDs; RBAC/ABAC; monitor suspicious access; periodic Broken Access Control testing.

**OWASP/CWE reference:** [A01:2025 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/) · [CWE-639](https://cwe.mitre.org/data/definitions/639.html)

---

### Write-up 7 — Directory Listing: Exposed /ftp
**Severity:** CWE-548 – Exposure of Information Through Directory Listing

**Description:** The `/ftp` directory is publicly exposed and allows listing its contents without authentication (directory listing enabled).

**Steps to reproduce:**
1. Access `/ftp` from the browser.
2. Observe the full directory listing.
3. Confirm no authentication is required.
4. Browse subdirectories and files.
5. Download files to confirm full access.
6. Check for sensitive content.

**Evidence:**

| Access to the public /ftp endpoint | Site map tree — /ftp folder expanded |
|---|---|
| <img src="screenshots/23_wu7_acceso_endpoint_ftp.png" width="380"> | <img src="screenshots/24_wu7_sitemap_arbol_ftp.png" width="300"> |

<p align="center"><img src="screenshots/25_wu7_documento_confidencial.png" width="500"></p>
<p align="center"><em>File containing confidential information found inside <code>/ftp</code>.</em></p>

<p align="center"><img src="screenshots/26_wu7_banner_reto_completado.png" width="600"></p>
<p align="center"><em>"Confidential Document" challenge completed.</em></p>

**Real impact:** access to sensitive files (logs, backups, configs), leakage of credentials/tokens/API keys, internal structure reconnaissance, groundwork for more advanced attacks (LFI, RFI, path traversal, brute force).

**Remediation:** disable directory listing (Apache/Nginx/IIS); restrict access via authentication or firewall rules; move sensitive files outside publicly reachable paths; clean up unnecessary files; enforce least privilege; correct read/write permissions; analyze exposed files for real impact.

**OWASP/CWE reference:** [A01:2025 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/) · [CWE-548](https://cwe.mitre.org/data/definitions/548.html)

---

### Write-up 8 — Error Handling
**Severity:** CWE-209: Generation of Error Message Containing Sensitive Information

**Description:** The server responds with an HTTP 500 including the full stack trace when receiving malformed data (incomplete JSON/undefined parameters).

**Steps to reproduce:**
1. Intercept a request to `/api/Feedbacks` with Burp.
2. Send it to Repeater.
3. Modify the JSON to make it syntactically incorrect.
4. Send and observe the HTML/text response with the stack trace.

**Evidence:**

| Error response exposing technical info | Challenge completed banner |
|---|---|
| <img src="screenshots/27_wu8_response_error.png" width="380"> | <img src="screenshots/28_wu8_banner_reto_completado.png" width="380"> |

**Real impact:** reveals critical infrastructure information (absolute file paths, library versions, backend internal logic), aiding attacker reconnaissance.

**Remediation:** disable detailed error messages in production; implement a global error handler returning generic messages.

**OWASP/CWE reference:** [A02:2025 Security Misconfiguration](https://owasp.org/Top10/2025/A02_2025-Security_Misconfiguration/) · [CWE-209](https://cwe.mitre.org/data/definitions/209.html)

---

### Write-up 9 — API Data Exposure: User Enumeration
**Severity:** CWE-213: Exposure of Sensitive Information Due to Incompatible Policies

**Description:** The `/api/Users` endpoint allows any authenticated user to list every record in the user database, without role-based authorization filters.

**Steps to reproduce:**
1. Log in with a regular user and obtain its JWT.
2. In Repeater, issue `GET /api/Users`.
3. Include `Authorization: Bearer [token]`.
4. Analyze the JSON containing the full user list.

**Evidence:**

| Repeater — request | Response — full user list |
|---|---|
| <img src="screenshots/29_wu9_repeater_request.png" width="380"> | <img src="screenshots/30_wu9_response_lista_usuarios.png" width="380"> |

**Real impact:** PII leakage — emails, roles, last-login IPs, and special tokens for every user, enabling phishing or privilege escalation.

**Remediation:** object-level authorization controls (BOLA); the endpoint should only return the requester's own profile or be restricted to admin roles.

**OWASP/CWE reference:** [A01:2025 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/) · [CWE-213](https://cwe.mitre.org/data/definitions/213.html)

---

### Write-up 10 — Broken Auth: No Rate Limiting on Login
**Severity:** CWE-307: Improper Restriction of Excessive Authentication Attempts

**Description:** The login endpoint processes unlimited attempts without blocking the account or IP, enabling automated brute force.

**Steps to reproduce:**
1. Capture a failed login (`POST /rest/user/login`) and send it to Intruder.
2. Configure the payload position on the `password` field.
3. Load a common password dictionary in the Payloads tab.
4. Start the attack and observe 401 responses with no blocking or delay.

**Evidence:**

| Request to intercept | Request in Intruder |
|---|---|
| <img src="screenshots/31_wu10_request_interceptar.png" width="380"> | <img src="screenshots/32_wu10_request_intruder.png" width="380"> |

<p align="center"><img src="screenshots/33_wu10_intruder_resultados.png" width="600"></p>
<p align="center"><em>Multiple attempts executed with no blocking (all HTTP 401 responses, same length/timing).</em></p>

**Real impact:** account compromise via dictionary attacks or credential stuffing; account security depends solely on password complexity.

**Remediation:** account lockout after N failed attempts; progressive throttling; CAPTCHA upon suspicious activity.

**OWASP/CWE reference:** [A07:2025 Authentication Failures](https://owasp.org/Top10/2025/A07_2025-Authentication_Failures/) · [CWE-307](https://cwe.mitre.org/data/definitions/307.html)

---

## ✅ Conclusions and recommendations

Among the most concerning vulnerabilities are a **SQL injection** in the authentication mechanism (allowing an attacker to bypass login or access sensitive information directly) and **weak access control on the admin panel** (allowing unauthorized users to execute admin-only actions). Due to their potential impact, these flaws require immediate attention and priority mitigation.

**Remediation priorities are focused on critical severity findings.**