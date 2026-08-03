# Attack Surface Analysis & Identity Enumeration 🚀

![License](https://img.shields.io/badge/License-Educational_Purpose_Only-red)
![Focus](https://img.shields.io/badge/Focus-OSINT_%7C_Recon_%7C_Identity-blue)

This repository documents a series of **passive reconnaissance (OSINT)** exercises on real infrastructures. The goal is to demonstrate technical capability to identify attack vectors, metadata leaks, and misconfigurations in enterprise environments without performing intrusive actions.

---

## 🔍 Case Studies

### 1. Directory Enumeration in Collaboration Environments — Logistics Case
* **Vulnerability:** Permissive *External Access* configuration in Microsoft Teams.
* **Technique:** Leveraging global identity search for profile harvesting.
* **Finding:** Successful extraction of executive profiles (photos, full names, job titles) without interaction or prior authorization.
* **Risk Correlation:** A lack of **TLS/SSL** certificates was detected on the main web infrastructure, suggesting an immature security posture and a high likelihood of success for *Man-in-the-Middle* (MitM) or targeted *phishing* attacks.

Using DeHashed, the enumerated identities were correlated with historical data breaches, identifying that **X%** of executive profiles had credentials leaked in plaintext, increasing the risk of enterprise account compromise.

### 2. Bypass of Infrastructure Obfuscation — Healthcare/Insurance Case
* **Scenario:** Corporate domain protected by a Secure Email Gateway (SpamTitan) and SPF flattening techniques.
* **Technique:** Pivoting via Microsoft 365 **Login Redirect** flow for identity validation and origin resolution.
* **Discovery:** A **shared tenant** was resolved, revealing the hidden corporate structure and the organization’s real assets that were not visible from the initial contact domain.

### 3. Recruitment Channel Audit — Critical Infrastructure Case
* **Detected Issue:** The official recruitment contact (`miprimerempleo@...`) was non-functional, returning **SMTP 550 5.4.1 (Access Denied)**, indicating a misconfigured recipient filter.
* **Resolution/Validation:** By analyzing institutional naming patterns and enumeration techniques, the real contact vector was validated using the `[firstinitial][lastname]` pattern.
* **Impact:** Directory-Based Edge Blocking (DBEB) policies were successfully bypassed, confirming the possibility of user brute-force attacks or *Account Takeover* (ATO).

---

## 🛠 Tools and Methodology
These audits were conducted using **OSINT (Open Source Intelligence)** and **EASM (External Attack Surface Management)** techniques, leveraging the following platforms:

* **Identity and Footprint Discovery:** [SignalHire](https://www.signalhire.com/) & [RocketReach](https://rocketreach.co/): extraction and validation of corporate email structures and org charts.
    * [Epieos](https://epieos.com/): investigation of profiles linked to email addresses (Reverse Email Lookup) to detect leaks on external platforms.
* **Infrastructure and Email Analysis:**
    * [MXToolbox](https://mxtoolbox.com/): DNS record diagnostics (MX, SPF, DKIM) and network reputation analysis.
    * [VerifyEmailAddress](https://www.verifyemailaddress.org/): SMTP deliverability validation to confirm contact vectors without sending malicious traffic.
* **Credential and Data Leak Identification:**
    * [DeHashed](https://dehashed.com/): querying breach databases to assess the risk of *credential stuffing* from enumerated identities.

## ⚠️ Disclaimer
All information contained in this repository is for **educational and cybersecurity awareness** purposes only. No disruptive attacks were performed and system integrity was not compromised. All critical vulnerabilities were reported/detected under a professional ethical framework.
