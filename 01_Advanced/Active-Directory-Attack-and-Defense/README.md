# Threat Hunting con Wazuh y Atomic Red Team (SIEM Lab)

## Contexto
Proyecto semestral de la asignatura **Ciberseguridad 2** en la Universidad Tecnológica de Panamá.  
El objetivo fue implementar y evaluar una infraestructura de seguridad basada en **Wazuh (SIEM/XDR)** para detectar y analizar eventos de seguridad generados por ataques simulados, alineados a la matriz **MITRE ATT&CK**.

 ## Créditos
Proyecto desarrollado en equipo como parte de la asignatura Ciberseguridad II.

## Rol personal
- Instalación y configuración de agentes en Windows Server y Windows 10  
- Ejecución de ataques desde Kali Linux (Responder, CrackMapExec, Kerberoasting, BloodHound, Mimikatz)  
- Documentación de hallazgos técnicos y evidencias  
- Propuesta de mitigaciones basadas en MITRE ATT&CK  

## Metodología
1. **Configuración del servidor y tailnet** con CentOS, VirtualBox y Tailscale.  
2. **Instalación de agentes** en Windows Server 2019 y Windows 10.  
3. **Acceso y configuración del dashboard** de Wazuh.  
4. **Simulaciones de ataque** en laboratorio controlado.  
5. **Pruebas avanzadas** con **Atomic Red Team** para validar detecciones.  

## Técnicas utilizadas
- Reconocimiento y envenenamiento LLMNR/NBT-NS (Responder)  
- Password Spraying con CrackMapExec/Hydra  
- Kerberoasting con Impacket y John the Ripper  
- Recolección de privilegios con BloodHound/SharpHound  
- Movimiento lateral (psexec, wmiexec, smbexec)  
- Credential Dumping con Mimikatz y secretsdump  
- Ejecución de pruebas atómicas con **Atomic Red Team** (TA0043, TA0001, TA0002, TA0003, TA0004, TA0005, TA0006, TA0007, TA0008, TA0010)  

## Detección
- Alertas generadas en Wazuh correlacionadas con eventos de Windows (4625, 4769, 4104, etc.)  
- Reglas personalizadas en Sysmon y Wazuh para tácticas críticas (TA0043, TA0010).  
- Monitoreo de procesos, conexiones LDAP/RPC y dumps de memoria.  

## Mitigación
- **Deshabilitar LLMNR y NetBIOS** vía GPO.  
- **Políticas de contraseñas fuertes y MFA**.  
- **Restricción de privilegios en cuentas de servicio**.  
- **Segmentación de red y filtrado de tráfico**.  
- **Auditoría de tareas programadas y claves de registro**.  
- **Protección de credenciales (Credential Guard, EDR)**.  
- **Hardening de políticas de AD y GPO**.  

## Aprendizaje
- Comprensión práctica del ciclo de ataque y defensa en entornos corporativos.  
- Validación de la capacidad de detección de un SIEM frente a técnicas MITRE ATT&CK.  
- Importancia del **tuning de reglas** para reducir falsos positivos.  
- Relevancia de combinar detección, mitigación y monitoreo continuo para fortalecer la postura de seguridad.  

## Referencias
- [Wazuh – Instalación oficial](https://wazuh.com/install/)  
- [Atomic Red Team – GitHub](https://github.com/redcanaryco/atomic-red-team)  
- [MITRE ATT&CK Framework](https://attack.mitre.org/)  
- [Invoke-AtomicRedTeam Docs](https://www.atomicredteam.io/invoke-atomicredteam/docs)  
