# Windows 11 Hardening: Encryption, Attack Surface & Telemetry

## Context
Laboratory for the Information Storage Management course at Technological University of Panama. The exercise consisted of taking a freshly installed ("out-of-the-box") Windows 11 virtual machine and applying a comprehensive hardening process, validating the results with network audits before and after the procedure.

## Objective
Reduce the attack surface of a Windows 11 workstation across three fronts: disk encryption at rest, closure of unnecessary services/ports, and mitigation of data leakage via telemetry — fully automating the entire process into a reusable PowerShell script.

## Technologies
- Windows 11 (PowerShell, Disk Management, Local Group Policy Editor)
- BitLocker + TPM 2.0
- Nmap (port auditing)
- PowerShell scripting (`.ps1`)

## Methodology

The laboratory was divided into 4 phases:

1. **Baseline Audit** — initial "out-of-the-box" state of the system.
2. **Hardening Execution** — application of security measures across 3 blocks (encryption, services, privacy).
3. **Automation** — a `.ps1` script that replicates the entire process in a repeatable manner.
4. **Final Verification** — re-audit to confirm the impact of the changes.

## Implementation

### Phase 1: Baseline Audit (The "Before")

Exposed services were identified by default using `netstat -an` alongside a targeted Nmap scan:

![Initial connection state with netstat -an](screenshots/01-netstat-baseline.png)
![Nmap scan on identified ports](screenshots/02-nmap-escaneo-inicial.png)

The following were flagged as critical:
- **Port 445 (SMB)** — exposed by default, risk of remote code execution.
- **Port 135 (RPC)** — used for Remote Registry, facilitates system enumeration.
- **Port 5985 (WinRM)** — remote management unnecessarily active on a workstation.

Additionally, a complete list of active services was exported for review:

![List of services exported to Excel](screenshots/03-lista-servicios-exportada.png)

From this review, **Print Spooler** (vulnerable to PrintNightmare, CVE-2021-34527), **Remote Registry**, and **Xbox services** (unnecessary in an enterprise environment based on the principle of least functionality) were identified as candidates for disabling.

Regarding storage, `Get-BitLockerVolume` confirmed that the main drive had no active encryption (`Protection Status: Off`), and the availability of the TPM chip was verified:

![TPM management on the host](screenshots/04-tpm-administracion.png)
![BitLocker disabled in the initial state](screenshots/05-bitlocker-desactivado-inicial.png)

Finally, the privacy panel was audited, showing telemetry set to **'Full'** and the advertising ID active:

![Privacy panel with telemetry and advertising active](screenshots/06-telemetria-privacidad-inicial.png)

### Phase 2: Hardening Execution

**Block 1 — Disk Encryption (BitLocker)**

The TPM chip was verified to be ready and active:

![TPM chip verification with Get-Tpm](screenshots/07-get-tpm-verificacion.png)

The **TPM+PIN** policy (additional startup authentication) was configured prior to activating BitLocker to avoid policy conflicts:

![TPM+PIN policy configuration](screenshots/08-directiva-tpm-pin.png)

`Enable-BitLocker` was executed with AES-256 encryption and TPM+PIN protection, adding a backup recovery key via `Add-BitLockerKeyProtector` before rebooting:

![Execution of Enable-BitLocker requesting the PIN](screenshots/09-enable-bitlocker-pin.png)

Upon reboot, the system requested the configured PIN to unlock the drive:

![BitLocker screen requesting PIN at boot](screenshots/10-bitlocker-pin-boot.png)

**Block 2 — Attack Surface Reduction: Windows Services**

Running services were listed, and the results were exported for review:

![Running services with Get-Service](screenshots/11-get-service-running.png)
![Service list exported to CSV](screenshots/12-export-csv-servicios.png)

Unnecessary services identified earlier were disabled: **Remote Registry**, **Print Spooler**, **WinRM**, and **Xbox services**, alongside disabling the **SMBv1** protocol:

![Disabling unnecessary services](screenshots/13-deshabilitar-servicios.png)

**Block 3 — Privacy & Telemetry**

Using registry keys, telemetry was configured to **Level 0 (Security)**, Cortana was disabled, diagnostic and tracking services were stopped (DiagTrack, dmwappushservice, SysMain), the Customer Experience Improvement Program scheduled task was turned off, and **LLMNR**, **NetBIOS**, and **WPAD** were disabled to reduce name resolution attack vectors (relevant against Responder-style attacks):

![Telemetry, LLMNR, and NetBIOS hardening script](screenshots/14-script-telemetria-llmnr-netbios.png)

System policies were updated to apply the changes:

![Updating policies with gpupdate /force](screenshots/15-gpupdate-force.png)

### Phase 3: Automation (`.ps1` script)

The entire process was consolidated into a PowerShell script (`Hardening.ps1`) automating:
1. Disabling dangerous services (`Spooler`, `RemoteRegistry`, `WinRM`, `SMB1`).
2. Setting telemetry to Level 0.
3. Turning off the advertising ID.
4. Blocking NetBIOS/LLMNR.
5. Verifying final BitLocker status.

This ensures the hardening process is **repeatable and free of human error** when deployed across multiple machines.

### Phase 4: Final Verification (The "After")

The Nmap scan was repeated on the same IP to confirm the reduction of the exposed surface:

![Final Nmap scan after hardening](screenshots/16-nmap-final-verificacion.png)

The status of each applied block was verified:

![BitLocker fully encrypted and active](screenshots/17-bitlocker-final-encriptado.png)

![Target services confirmed disabled](screenshots/18-servicios-deshabilitados-final.png)

## Results

| Attack Vector | Initial State | Final State | Applied Measure |
|---|---|---|---|
| Disk Encryption | Disabled | Active (AES-256) | BitLocker configuration with PIN |
| Port 445 (SMB) | Open | Filtered/Closed | Disabling SMBv1 and associated services |
| Telemetry | Full | Level 0 (Minimum) | Registry key modification |
| Name Resolution | Active (LLMNR) | Disabled | DNS Client and NetBIOS hardening |

The final Nmap scan demonstrated a drastic reduction in exposed ports compared to the initial baseline, confirming the effectiveness of the security controls applied.

## Key Takeaways
- Effective hardening combines three independent layers (encryption, network surface, privacy) that reinforce each other.
- The principle of least functionality (disabling all non-essential features, such as Xbox services on a workstation) reduces the attack surface without impacting operations.
- Automating the process into a script ensures consistency and enables reliable execution across multiple systems without manual mistakes.
- BitLocker protectors (TPM+PIN) and recovery keys must be managed with the same level of care as any critical credential — they should never be exposed in public documentation.