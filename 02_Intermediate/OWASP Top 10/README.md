# Pentest Laboratory: Burp Suite - OWASP Juice Shop

---

## 📌 Executive Summary

A security assessment was conducted on the **OWASP Juice Shop** web application for educational and vulnerability analysis practice purposes. During the process, **10 security flaws** were identified, of which **5 were classified as critical severity** due to the level of risk they pose to the system's confidentiality, integrity, and availability.

---

## ⚙️ Complete Setup

* **Burp Suite** + Browser Proxy
* **Docker** installed
* **Scope** limited to the local testing application (`http://localhost:3000`)

---

## 📊 Summary of Findings

| Vulnerability | Severity | OWASP | Status |
| :--- | :---: | :---: | :---: |
| Admin Panel - Broken Access Control | Critical | A01 | Confirmed |
| SQL Injection - Login Bypass | Critical | A02 | Confirmed |
| DOM XSS | Critical | A05 | Confirmed |
| Bonus Payload | Critical | A05 | Confirmed |
| Admin Registration | High | A01 | Confirmed |
| IDOR - Access to Another User's Profile | Critical | A01 | Confirmed |
| Directory Listing — `/ftp` Exposed | Medium | A01 | Confirmed |
| Error Handling | Medium | A02 | Confirmed |
| API Data Exposure — Enumerate Users | Medium | A01 | Confirmed |
| Broken Auth — No Rate Limiting on Login | Medium | A07 | Confirmed |

---

## 📝 Detailed Findings (Write-ups)

### Write-up 1: Admin Panel - Broken Access Control

* **Severity:** CWE-284: Improper Access Control
* **Description:** The web application exposes an administration panel that is accessible without authentication or privilege validation. This allows any unauthenticated user to access administrative functionalities simply by knowing or discovering the path.

#### Steps to Reproduce
1. Access the application at `http://localhost:3000`.
2. Review the generated Site Map or manually inspect the loaded resources.
3. Analyze the `main.js` file to identify hidden routes.
4. Find a reference to the `/admin` path.
5. Navigate directly to `http://localhost:3000/admin`.
6. Confirm access to the restricted area.

#### Real Impact
* Unauthorized access to administrative functionalities and possible manipulation of sensitive data.
* Risk of privilege escalation and complete compromise of the application.

#### Remediation
* Require mandatory authentication to access `/admin` and apply RBAC controls.
* Avoid exposing sensitive routes in public JavaScript files.

#### References
* [OWASP Top 10 – A01:2025 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/)
* [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html)

---

### Write-up 2: SQL Injection - Login Bypass

* **Severity:** CWE-89 – Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection')
* **Description:** The `email` parameter in the authentication endpoint does not validate or sanitize user input before using it in an SQL query, allowing the logic to be manipulated to bypass authentication.

#### Steps to Reproduce
1. Access the application's Login page.
2. Intercept the `POST /rest/user/login` request using Burp Suite.
3. Modify the JSON body by replacing the value of the `email` field with an SQL payload (e.g., `' OR 1=1--`).
4. Send the request and verify that a JWT token with elevated privileges is returned.

#### Real Impact
* Unauthorized access to user accounts (including administrators).
* Massive data modification, deletion, or exfiltration.

#### Remediation
* Implement parameterized queries (*prepared statements*) for all SQL operations.
* Strictly validate and sanitize user inputs.

#### References
* [OWASP Top 10 - A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
* [CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)

---

### Write-up 3: DOM XSS

* **Severity:** CWE-79 – Improper Neutralization of Input During Web Page Generation ('Cross-Site Scripting')
* **Description:** The application processes untrusted data from the URL or DOM directly in the browser via JavaScript without proper sanitization.

#### Steps to Reproduce
1. In the search bar, enter the payload: `<iframe src="javascript:alert('DOM_XSS')">`.
2. Press Enter and observe the execution of the JavaScript alert.

#### Real Impact
* Arbitrary execution of JavaScript, session/JWT token theft, and identity spoofing.

#### Remediation
* Avoid unsafe functions such as `innerHTML` or `document.write`; use `textContent` or `innerText` instead.
* Implement a Content Security Policy (CSP).

#### References
* [OWASP Top 10 - A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
* [CWE-79: Cross-Site Scripting](https://cwe.mitre.org/data/definitions/79.html)

---

### Write-up 4: Bonus Payload (HTML Injection)

* **Severity:** CWE-79 – Improper Neutralization of Input During Web Page Generation ('Cross-Site Scripting')
* **Description:** It was confirmed that the vulnerable sink on the client side not only executes JavaScript but also allows the injection of arbitrary HTML elements (such as an `<iframe>` loading external sources).

#### Steps to Reproduce
1. Replace the search parameter with an HTML payload containing an iframe from an external domain.
2. Load the page and verify that the external resource is rendered within the DOM.

#### Real Impact
* Visual phishing within the application and manipulation of the graphical interface.

#### Remediation
* Use sanitization libraries like DOMPurify and configure CSP restrictions for iframes.

---

### Write-up 5: Admin Registration

* **Severity:** CWE-639 - Authorization Bypass Through User-Controlled Key
* **Description:** The registration endpoint trusts parameters sent from the client to assign the user role, allowing the registration of administrative accounts without server-side controls.

#### Steps to Reproduce
1. Intercept the `POST /api/Users` request when registering a new user.
2. Modify the JSON by adding or changing the parameter that defines the role to administrator.
3. Send the request and confirm the creation of the privileged account.

#### Real Impact
* Unauthorized creation of administrative accounts leading to total system compromise.

#### Remediation
* Assign roles solely from internal backend logic and eliminate any role parameters from client requests.

---

### Write-up 6: IDOR — Access to Another User's Profile

* **Severity:** CWE-639 - Authorization Bypass Through User-Controlled Key
* **Description:** The system exposes direct identifiers in API requests without validating whether the authenticated user has permissions to access the requested resource.

#### Steps to Reproduce
1. Log in and make a request to a route that retrieves a resource by ID (e.g., `/api/orders/101`).
2. Intercept the request in Burp Repeater and modify the ID (e.g., change `101` to `102`).
3. Verify that the server responds with the data of the other user's resource.

#### Real Impact
* Horizontal privilege escalation and leakage of other users' personal information.

#### Remediation
* Validate on the backend that the object belongs to the authenticated user and implement UUIDs instead of sequential IDs.

---

### Write-up 7: Directory Listing — `/ftp` Exposed

* **Severity:** CWE-548 – Exposure of Information Through Directory Listing
* **Description:** The `/ftp` directory is publicly exposed, allowing navigation through the folder structure and downloading of internal files without authentication.

#### Steps to Reproduce
1. Navigate to `http://localhost:3000/ftp`.
2. Observe the file listing exposed by the web server.
3. Download files to check for confidential content.

#### Real Impact
* Leakage of sensitive information, logs, or credentials stored in internal files.

#### Remediation
* Disable Directory Listing on the web server and move sensitive files outside the web root.

---

### Write-up 8: Error Handling

* **Severity:** CWE-209: Generation of Error Message Containing Sensitive Information
* **Description:** When sending malformed requests to the backend, the application responds with an HTTP 500 error that includes the environment's full Stack Trace.

#### Steps to Reproduce
1. Intercept a request to `/api/Feedbacks`.
2. Omit a structural character in the JSON body to generate a syntax error.
3. Send the request and analyze the response containing code details and absolute paths.

#### Real Impact
* Technical information leakage (library versions, server paths, internal logic).

#### Remediation
* Implement a global exception handler to return generic error messages in production.

---

### Write-up 9: API Data Exposure — Enumerate Users

* **Severity:** CWE-213: Exposure of Sensitive Information Due to Incompatible Policies
* **Description:** The `/api/Users` endpoint allows any authenticated user to query the complete list of registered users without requiring an administrative role.

#### Steps to Reproduce
1. Log in with a standard user account and obtain the JWT token.
2. Make a `GET` request to `/api/Users` including the `Authorization: Bearer [token]` header.
3. Analyze the JSON response containing the full list of records.

#### Real Impact
* Massive leakage of PII (emails, roles, access data).

#### Remediation
* Restrict access to the complete user list exclusively to administration roles.

---

### Write-up 10: Broken Auth — No Rate Limiting on Login

* **Severity:** CWE-307: Improper Restriction of Excessive Authentication Attempts
* **Description:** The login interface does not restrict the number of consecutive failed attempts, permitting automated brute-force attacks.

#### Steps to Reproduce
1. Capture the `POST /rest/user/login` request and send it to Burp Intruder.
2. Set up a dictionary attack on the `password` field.
3. Execute the burst of requests and verify that none are blocked or rate-limited.

#### Real Impact
* Account compromise via credential stuffing or dictionary attacks.

#### Remediation
* Implement temporary lockout policies, throttling (delays), or CAPTCHAs after several failed attempts.

---

## 🎯 Conclusions and Recommendations

The analysis demonstrated that the most pressing critical vulnerabilities reside in the authentication mechanism (**SQL Injection**) and poor access controls (**Admin Panel and Admin Registration**).

**Remediation Priorities:**
1. **Phase 1 (Immediate):** Sanitize SQL queries (prepared statements) and close unauthenticated administrative endpoints.
2. **Phase 2:** Implement server-side validations to prevent IDORs and arbitrary role assignment.
3. **Phase 3:** General web server hardening (disable directory listing, implement secure error handling, and add rate limiting).