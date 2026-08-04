# AD-CS Pentesting & Hardening — Proyecto Semestral

## Resumen

Este proyecto evalúa la seguridad de un entorno de **Active Directory Certificate Services (AD-CS)** configurado manualmente sobre **Windows Server 2019**. Inicialmente se planeó usar un laboratorio virtual preconfigurado (OVA), pero por dificultades técnicas el equipo replicó las mismas vulnerabilidades intencionales desde cero en un Domain Controller nuevo: plantillas de certificado vulnerables, permisos débiles y configuraciones inseguras en la CA.

El trabajo se dividió en tres fases:

1. **Configuración del entorno** — despliegue de un Domain Controller con AD-CS e importación de plantillas vulnerables, usuarios de prueba y reglas de firewall.
2. **Evaluación de seguridad** — enumeración de la PKI y explotación de **ESC1, ESC2, ESC3, ESC4, ESC7** y un ataque **PKINIT "Golden Certificate"** para obtener credenciales de Domain Admin.
3. **Mitigación** — hardening de plantillas y CA, deshabilitación de Web Enrollment, restricción de NTLM, y validación de que todas las rutas de ataque quedaron cerradas.

---

## Topología del laboratorio

- **Dominio:** `CYBERLAB.LOCAL`
- **CA:** `cyberlab-DC01-CYBER-CA` (alojada en el Domain Controller WS19)
- **Máquina atacante:** Kali Linux (`certipy`, `netexec`, `evil-winrm`, `impacket`)
- **Usuario de bajo privilegio:** `s.jackson`
- **Usuario secundario de prueba (ESC7):** `m.roland`

![Ping entre atacante y DC](screenshots/003_pagina3.png)
*Prueba de conectividad entre la máquina atacante Kali y el Domain Controller WS19.*

![Configuración del rol ADCS en el WS19 DC](screenshots/001_pagina2.png)
*Configuración del rol AD-CS durante el despliegue del entorno.*

![Scripts para importar plantillas vulnerables, usuarios y reglas de firewall](screenshots/004_pagina4.png)
*Scripts de PowerShell usados para descargar las plantillas vulnerables, crear a `s.jackson` y `m.roland`, y abrir los puertos necesarios para la enumeración con Certipy.*

---

## Enumeración inicial

Con la cuenta de bajo privilegio `s.jackson`, el equipo ejecutó `certipy find` contra la CA para mapear todas las plantillas de certificado, sus permisos y sus Usos Extendidos de Clave (EKU) habilitados.

![Salida de certipy find](screenshots/007_pagina5.png)
*Certipy descubre 39 plantillas, 1 CA y 17 plantillas habilitadas en `cyberlab-DC01-CYBER-CA`.*

Certipy marcó la plantilla inyectada `Vuln-ESC1` como **Vulnerable: Yes**, porque combina el flag `ENROLLEE_SUPPLIES_SUBJECT` con el permiso `Enroll` otorgado a `Domain Users`.

![Vuln-ESC1 vulnerable a ESC1](screenshots/009_pagina6.png)
*Salida de Certipy confirmando la vulnerabilidad ESC1 en `Vuln-ESC1`.*

---

## Cadena de ataques

### ESC1 — Plantilla de certificado mal configurada

**Causa raíz:** la plantilla permite al solicitante especificar un nombre alternativo del sujeto (SAN) arbitrario, mientras habilita Autenticación de Cliente e inscripción para Usuarios del Dominio.

`s.jackson` solicitó un certificado con la plantilla `Vuln-ESC1`, especificando el UPN del Administrador en el campo SAN, y usó el `.pfx` resultante para autenticarse vía PKINIT y obtener el hash NTLM del Domain Administrator.

![certipy req emitiendo un certificado de Administrator](screenshots/010_pagina7.png)
*Certipy solicita un certificado en nombre de `Administrator@CYBERLAB.LOCAL`, explotando `Vuln-ESC1`.*

![certipy auth recuperando el hash NTLM del Administrator](screenshots/011_pagina7.png)
*La autenticación PKINIT con el certificado robado devuelve el hash NTLM del Domain Administrator.*

![Sesión evil-winrm como CYBERLAB\Administrator](screenshots/012_pagina8.png)
*Inicio de sesión Pass-the-Hash vía `evil-winrm`, confirmando acceso total de Domain Admin.*

### ESC2 — Plantilla con EKU "Any Purpose"/Agente de Inscripción

`Vuln-ESC2` otorga `Enroll` a Domain Users y tiene un EKU que permite actuar como **agente de inscripción**, es decir, solicitar certificados *en nombre de* otros usuarios.

![Pestaña Extensions de Vuln-ESC2 mostrando "Any Purpose"](screenshots/016_pagina10.png)
*El EKU de la plantilla está configurado como Any Purpose, habilitando el abuso de agente de inscripción.*

`s.jackson` obtuvo primero un certificado de agente, luego lo usó para solicitar un certificado `User` con `-on-behalf-of 'CYBERLAB\Administrator'`, y finalmente se autenticó vía PKINIT para volcar el hash del Administrator.

![certipy req on-behalf-of Administrator](screenshots/019_pagina12.png)
*Certificado del Administrator emitido a través de la identidad de agente de inscripción robada.*

### ESC3 — Plantillas de agente de inscripción mal configuradas

Dos plantillas encadenadas (`Vuln-ESC3-1`, `Vuln-ESC3-2`) otorgan ambas el permiso `Enroll` a Domain Users y tienen el EKU **Certificate Request Agent**, permitiendo que un usuario de bajo privilegio solicite un certificado de agente y luego suplante al Administrator a través de la segunda plantilla.

![Pestaña Security de Vuln-ESC3-1 con la política Certificate Request Agent](screenshots/023_pagina14.png)
*Domain Users tiene derechos de Enroll/Autoenroll, y la política de aplicación Certificate Request Agent está presente.*

La cadena termina igual que ESC1/ESC2: un certificado de Administrator falsificado, autenticación PKINIT, extracción del hash NTLM y una shell `evil-winrm` como `CYBERLAB\Administrator`.

### ESC4 — ACL débil en la plantilla (permiso Write)

La DACL de `Vuln-ESC4` originalmente no tenía el permiso `Write` que el laboratorio requería, por lo que el equipo otorgó manualmente acceso de escritura a `Authenticated Users` para reproducir el escenario previsto. Con ese permiso, `s.jackson` reconfiguró la propia plantilla — habilitando Autenticación de Cliente y `ENROLLEE_SUPPLIES_SUBJECT` — convirtiendo una plantilla previamente segura en un vector tipo ESC1.

![ACL de Vuln-ESC4 mostrando Write para Authenticated Users](screenshots/032_pagina20.png)
*Causa raíz: Authenticated Users tiene el permiso Write en la DACL de la plantilla.*

![certipy-ad template modificando Vuln-ESC4](screenshots/044_pagina28.png)
*Certipy abusa del permiso Write para añadir las banderas peligrosas a la configuración de la plantilla.*

El atacante solicitó luego un certificado con `-upn administrator@CYBERLAB.LOCAL -sid <SID del Domain Admin>`, se autenticó vía PKINIT, volcó el hash y confirmó el acceso con `impacket-secretsdump` y `evil-winrm`.

### ESC7 — Control de acceso vulnerable en la CA (Manage CA)

A la cuenta `m.roland` se le otorgó el permiso **Manage CA** directamente sobre el objeto de la Autoridad de Certificación — una falla crítica de control de acceso.

![Pestaña Security de la CA mostrando Manage CA otorgado a m.roland](screenshots/044_pagina28.png)
*`m.roland` tiene derechos de Read y Manage CA sobre `cyberlab-DC01-CYBER-CA`.*

Usando `certipy ca -add-officer`, el atacante escaló de Manage CA a **Manage Certificates**, habilitó la peligrosa plantilla `SubCA`, envió una solicitud con el UPN del Administrator (que la CA rechazó inicialmente) y luego forzó la emisión de la solicitud pendiente con `-issue-request`. El certificado resultante fue recuperado, usado para PKINIT, y convertido en acceso total de Domain Admin.

![certipy ca -issue-request forzando la emisión del certificado](screenshots/049_pagina30.png)
*Los derechos de Manage Certificates se usan para cambiar la solicitud pendiente a "Issued."*

![Autenticación PKINIT devolviendo el hash del Administrator](screenshots/052_pagina32.png)
*Certipy recupera el hash NTLM tras autenticarse con el certificado emitido forzosamente.*

### PKINIT — "Golden Certificate"

El ataque más severo: una vez que una cuenta con derechos de Domain Admin (`m.roland`, escalada arriba) puede llegar al servidor de la CA, su **clave privada** puede respaldarse y robarse con `certipy ca -backup`. Esa clave permite a un atacante **falsificar (forge)** certificados para cualquier entidad — incluida la cuenta de máquina del DC — sin volver a interactuar nunca más con el flujo de emisión de la CA, de forma similar en espíritu a un Golden Ticket.

![certipy ca -backup obteniendo la clave privada de la CA](screenshots/054_pagina33.png)
*El PFX de la CA (certificado + clave privada) se exporta desde el Domain Controller.*

![certipy forge creando un certificado falsificado del Administrator](screenshots/055_pagina34.png)
*Se falsifica sin conexión un certificado nuevo para `administrator@cyberlab.local` usando la clave robada de la CA — sin necesidad de interactuar con el servidor de la CA.*

---

## Mitigaciones aplicadas

| Vulnerabilidad | Medida de hardening | Ataques mitigados |
|---|---|---|
| ACL débil en plantillas (Write/Enroll) | Principio de mínimo privilegio: eliminar Write/Enroll de grupos de bajo privilegio (Authenticated Users, Domain Users) | ESC1, ESC2, ESC3-1, ESC4 |
| Banderas de suplantación | Deshabilitar `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` salvo que sea estrictamente necesario | ESC1, ESC3-1, ESC4 |
| EKUs peligrosos | Restringir los EKU de Autenticación de Cliente / Any Purpose detrás de aprobación de gerente o restricciones de agente | ESC1, ESC2, ESC3-1, ESC3-2, ESC4 |
| ACL débil en la CA | Eliminar Manage CA / Manage Certificates de cuentas que no sean administradores de PKI designados | ESC7 |
| Abuso de agente de inscripción | Aplicar la función de restricción de Agente de Inscripción | ESC2 |
| Superficie de ataque de Web Enrollment | Desinstalar el rol de Web Enrollment, o restringir los puertos 80/443 a administradores de PKI | Relay NTLM vía IIS |
| NTLM como credencial | Denegar NTLM entrante/saliente vía GPO en el DC/CA, forzar solo NTLMv2 | PKINIT/Golden Certificate, Pass-the-Hash |

Cada ataque fue vuelto a probar después de la remediación y **falló como se esperaba** (`CERTSRV_E_TEMPLATE_DENIED`, `Access denied`, `RPC_E_CALL_COMPLETE`, etc.).

![Ataque ESC1 fallando después del hardening](screenshots/059_pagina37.png)
*Validación post-hardening: la solicitud ESC1 es rechazada por la CA.*

![Ataque ESC7 fallando después del hardening](screenshots/071_pagina41.png)
*Validación post-hardening: `add-officer` y la activación de la plantilla son denegados por permisos insuficientes.*

![Ataque Golden Certificate fallando después del hardening](screenshots/074_pagina42.png)
*Validación post-hardening: la operación de respaldo de la CA es denegada (`rpc_s_access_denied`).*

![Web Enrollment deshabilitado y confirmado con certipy find](screenshots/075_pagina43.png)
*`certipy find` ahora reporta Web Enrollment HTTP/HTTPS como deshabilitado y `User Specified SAN: Disabled`.*

![GPOs de restricción NTLM aplicadas en el DC](screenshots/083_pagina45.png)
*Directiva de grupo configurada para denegar el tráfico NTLM entrante y saliente, forzando autenticación exclusiva por Kerberos.*

![Autenticación Pass-the-Hash/NTLM fallando después de las restricciones](screenshots/084_pagina46.png)
*El hash NTLM previamente válido ya no puede usarse para abrir una shell `evil-winrm`.*

---

## Conclusiones principales

- La **fortaleza criptográfica teórica de los certificados es irrelevante** si las plantillas, los permisos o los controles de acceso de la CA están mal configurados — el abuso de ESC1 a ESC7 y PKINIT son fallos administrativos, no rupturas criptográficas.
- El **principio de mínimo privilegio en las plantillas de certificado** (Enroll, Write, Autoenroll) es el control de mayor impacto; cierra ESC1, ESC2, ESC3 y ESC4 simultáneamente.
- El **control de acceso a nivel de CA** (Manage CA / Manage Certificates) merece el mismo escrutinio que la membresía del grupo Domain Admins — ESC7 demuestra que un solo permiso mal asignado puede comprometer todo el dominio.
- La **restricción de NTLM y la eliminación de Web Enrollment** son capas de defensa en profundidad: incluso si una plantilla es comprometida, un certificado robado pierde gran parte de su utilidad sin una forma de convertirlo en hash NTLM o de retransmitirlo vía HTTP.
- La **auditoría continua** (evento 5136, logs de autenticación PKINIT, barridos periódicos de `certipy find`) es necesaria porque los permisos de plantillas y de la CA cambian con el tiempo.

---

## Referencias

- Arth0sz. (s.f.). *Practice-AD-CS-Domain-Escalation* [Código fuente]. GitHub. https://github.com/arth0sz/Practice-AD-CS-Domain-Escalation
- Mayfly277. (s.f.). *ADCS Parte 14: Certificate Templates, Versioning, and Certipy Exploit Techniques*. https://mayfly277.github.io/posts/ADCS-part14/
- The Weekly Purple Team. (25 de agosto de 2025). *Análisis profundo de Certipy: Escalado mediante AD CS con ESC4–ESC7* [Video]. YouTube.
- Microsoft Defender for Identity Team. (16 de mayo de 2023). *Securing AD CS: Microsoft Defender for Identity's Sensor Unveiled*. Microsoft Tech Community.
- Microsoft. (s.f.). *Recomendaciones de directivas de auditoría del sistema*. Microsoft Learn.
