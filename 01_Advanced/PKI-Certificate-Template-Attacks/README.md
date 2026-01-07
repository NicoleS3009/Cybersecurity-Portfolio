# AD-CS Pentesting & Hardening Lab

## Contexto
Proyecto semestral de la asignatura **Criptografía** en la Universidad Tecnológica de Panamá.  
El trabajo consistió en la evaluación de seguridad de un entorno **Active Directory Certificate Services (AD-CS)** configurado manualmente en **Windows Server 2019**, replicando vulnerabilidades comunes en infraestructuras PKI.

## Integrantes
- Hillary Brouwer  
- Yahna Chee  
- Camilo Rodríguez  
- Nicole Staff  

## Rol personal
- Ejecución de ataques desde Kali Linux utilizando **Certipy** y **Evil-WinRM**  
- Documentación de evidencias y resultados  
- Propuesta de medidas de mitigación y hardening  

## Técnicas utilizadas
- Enumeración de plantillas vulnerables (ESC1–ESC7, PKINIT Golden Certificate)  
- Credential Access (obtención de hashes NTLM)  
- Lateral Movement mediante Pass-the-Hash y autenticación PKINIT  
- Suplantación de identidad con certificados maliciosos  

## Procedimiento
1. **Configuración del entorno**: despliegue de Domain Controller con AD-CS y plantillas vulnerables.  
2. **Evaluación de seguridad**: explotación de vulnerabilidades ESC1–ESC7 y PKINIT.  
3. **Mitigación**: aplicación de hardening en plantillas, restricciones de permisos y deshabilitación de roles inseguros.  

## Detección
- Uso de logs nativos del DC (evento 5136)  
- Validación de ataques mediante correlación de eventos  
- Recomendación de uso de **XDR** para detección avanzada de modificaciones en plantillas y abuso de certificados  

## Mitigación
- Principio de mínimo privilegio en permisos de inscripción  
- Deshabilitar banderas inseguras como **CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT**  
- Restricción de EKUs peligrosos (Any Purpose, Client Authentication)  
- Eliminación de permisos **Manage CA** y **Manage Certificates** en usuarios no autorizados  
- Deshabilitación del rol **Web Enrollment**  
- Restricciones de **NTLM** vía GPO para forzar autenticación Kerberos  

## Aprendizaje
- Comprensión práctica de vulnerabilidades en AD-CS y su impacto en la seguridad de PKI  
- Importancia de la configuración segura de plantillas y permisos  
- Aplicación de defensa en profundidad: criptografía robusta + configuración segura + monitoreo continuo  
- Reforzamiento de habilidades en explotación, detección y mitigación de ataques en entornos corporativos  

## Referencias
- Arth0sz – [Practice-AD-CS-Domain-Escalation](https://github.com/arth0sz/Practice-AD-CS-Domain-Escalation)  
- Mayfly277 – [ADCS Parte 14: Certipy Exploit Techniques](https://mayfly277.github.io/posts/ADCS-part14/)  
- Microsoft Tech Community – [Securing AD CS](https://techcommunity.microsoft.com/blog/microsoftthreatprotectionblog/securing-ad-cs-microsoft-defender-for-identitys-sensor-unveiled/3980265)  
- Microsoft Learn – [Recomendaciones de auditoría de seguridad](https://learn.microsoft.com/es-es/windowsserver/identity/ad-ds/plan/security-best-practices/audit-policy-recommendations)  
