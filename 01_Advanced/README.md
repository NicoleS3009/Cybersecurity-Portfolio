# Threat Hunting con Wazuh y Atomic Red Team (SIEM Lab)

## Contexto
Proyecto semestral de la asignatura Ciberseguridad 2 en la Universidad Tecnológica de Panamá.
El objetivo fue implementar y evaluar una infraestructura de seguridad basada en Wazuh (SIEM/XDR) para detectar y analizar eventos de seguridad generados por ataques simulados, alineados a la matriz MITRE ATT&CK.

## Créditos
Proyecto desarrollado en equipo como parte de la asignatura Ciberseguridad II.

**Rol personal**
- Instalación y configuración de agentes en Windows Server y Windows 10
- Ejecución de ataques desde Kali Linux (Responder, CrackMapExec, Kerberoasting, BloodHound, Mimikatz)
- Documentación de hallazgos técnicos y evidencias
- Propuesta de mitigaciones basadas en MITRE ATT&CK

## Metodología

1. **Configuración del servidor y tailnet** con CentOS, VirtualBox y Tailscale.

   ![Importación del OVA de Wazuh en VirtualBox](screenshots/wazuh-import.png)
   ![Login inicial al servidor Wazuh](screenshots/wazuh-login.png)

2. **Instalación de agentes** en Windows Server 2019 y Windows 10.

   ![Instalación de Tailscale para la red privada virtual](screenshots/tailscale-install.png)

3. **Acceso y configuración del dashboard de Wazuh.**

   ![Endpoints y agentes registrados en el dashboard](screenshots/dashboard-endpoints.png)
   ![Vista general del módulo MITRE ATT&CK](screenshots/mitre-overview.png)

   Tabla de direcciones IPv4 de las máquinas conectadas a la red privada virtual (Tailscale):

   ![Tabla de la tailnet](screenshots/tailnet-table.png)
   ![Estado de conexión de las máquinas en la tailnet](screenshots/tailnet-status.png)

4. **Simulaciones de ataque** en laboratorio controlado.
5. **Pruebas avanzadas** con Atomic Red Team para validar detecciones.

## Técnicas utilizadas

### Reconocimiento y envenenamiento LLMNR/NBT-NS (Responder)
Escaneo inicial con `nmap` y captura de hashes NTLMv2 mediante Responder sobre la interfaz de la tailnet.

![Ejecución de Responder capturando hashes NTLMv2](screenshots/responder-attack1.png)
![Hashes NTLMv2 capturados](screenshots/responder-hash.png)

### Password Spraying con CrackMapExec/Hydra
Validación de credenciales contra el DC utilizando CrackMapExec, incluyendo enumeración de grupos de dominio.

![CrackMapExec autenticando contra el DC](screenshots/crackmapexec.png)

### Kerberoasting con Impacket y John the Ripper
Creación de una cuenta de servicio con SPN, solicitud de tickets TGS con `GetUserSPNs.py` y cracking offline con John.

![Creación de cuenta de servicio con SPN](screenshots/kerberoasting-setup.png)
![Solicitud de tickets Kerberos (TGS) del SPN](screenshots/kerberoasting-hash.png)
![Cracking del hash Kerberos con John the Ripper](screenshots/kerberoasting-crack.png)

### Recolección de privilegios con BloodHound/SharpHound
Recolección de datos del dominio con SharpHound e importación a BloodHound para mapear rutas de escalamiento de privilegios.

![Ingesta de datos de SharpHound en BloodHound](screenshots/bloodhound-ingest.png)
![Grafo de relaciones de privilegios en BloodHound](screenshots/bloodhound-graph.png)

### Movimiento lateral (psexec, wmiexec, smbexec)
Uso de credenciales comprometidas para movimiento lateral y escalamiento de privilegios locales.

![Movimiento lateral agregando usuario al grupo de Administradores](screenshots/lateral-movement.png)

### Credential Dumping con Mimikatz y secretsdump
Extracción de hashes SAM y secretos LSA del controlador de dominio.

![Volcado de credenciales con secretsdump](screenshots/secretsdump.png)

### Ejecución de pruebas atómicas con Atomic Red Team
Pruebas atómicas mapeadas a 10 tácticas de MITRE ATT&CK (TA0043, TA0001, TA0002, TA0003, TA0004, TA0005, TA0006, TA0007, TA0008, TA0010), ejecutadas con el módulo `Invoke-AtomicRedTeam` en PowerShell.

## Detección

- Alertas generadas en Wazuh correlacionadas con eventos de Windows (4625, 4769, 4104, etc.)
- Reglas personalizadas en Sysmon y Wazuh para tácticas críticas (TA0043, TA0010).
- Monitoreo de procesos, conexiones LDAP/RPC y dumps de memoria.

![Detección de técnicas MITRE en el dashboard de Wazuh](screenshots/wazuh-mitre-detection.png)

## Mitigación

- Deshabilitar LLMNR y NetBIOS vía GPO.
- Políticas de contraseñas fuertes y MFA.
- Restricción de privilegios en cuentas de servicio.
- Segmentación de red y filtrado de tráfico.
- Auditoría de tareas programadas y claves de registro.
- Protección de credenciales (Credential Guard, EDR).
- Hardening de políticas de AD y GPO.

## Aprendizaje

- Comprensión práctica del ciclo de ataque y defensa en entornos corporativos.
- Validación de la capacidad de detección de un SIEM frente a técnicas MITRE ATT&CK.
- Importancia del tuning de reglas para reducir falsos positivos.
- Relevancia de combinar detección, mitigación y monitoreo continuo para fortalecer la postura de seguridad.

## Referencias

- [Wazuh – Instalación oficial](https://wazuh.com/install/)
- [Atomic Red Team – GitHub](https://github.com/redcanaryco/atomic-red-team)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Invoke-AtomicRedTeam Docs](https://www.atomicredteam.io/invoke-atomicredteam/docs)
