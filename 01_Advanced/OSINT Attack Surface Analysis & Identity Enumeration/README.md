# Attack Surface Analysis & Identity Enumeration 🚀

![License](https://img.shields.io/badge/License-Educational_Purpose_Only-red)
![Focus](https://img.shields.io/badge/Focus-OSINT_%7C_Recon_%7C_Identity-blue)

Este repositorio documenta una serie de ejercicios de **reconocimiento pasivo (OSINT)** sobre infraestructuras reales. El objetivo es demostrar la capacidad técnica para identificar vectores de ataque, fugas de metadatos y configuraciones erróneas en entornos empresariales sin realizar acciones intrusivas.

---

## 🔍 Casos de Estudio

### 1. Enumeración de Directorio en Entornos de Colaboración (Caso: Logística)
* **Vulnerabilidad:** Configuración permisiva de *External Access* en Microsoft Teams.
* **Técnica:** Aprovechamiento de la búsqueda global de identidades para el *harvesting* de perfiles internos.
* **Hallazgo:** Extracción exitosa de perfiles directivos (Fotos, Nombres completos, Cargos) sin interacción ni autorización previa.
* **Correlación de Riesgo:** Se detectó la falta de certificados **TLS/SSL** en la infraestructura web principal, lo que sugiere una postura de seguridad inmadura y una alta probabilidad de éxito para ataques de *Man-in-the-Middle* (MitM) o *Phishing* dirigido.



### 2. Bypass de Ofuscación de Infraestructura (Caso: Salud/Seguros)
* **Escenario:** Dominio empresarial protegido por un Secure Email Gateway (SpamTitan) y técnicas de *SPF Flattening*.
* **Técnica:** Pivotaje mediante el flujo de **Login Redirect** en Microsoft 365 para validación de identidad y resolución de origen.
* **Descubrimiento:** Se logró resolver un **Tenant compartido**, revelando la estructura corporativa oculta y los activos reales de la organización que no eran visibles tras el dominio de contacto inicial.

### 3. Auditoría de Canales de Reclutamiento (Caso: Infraestructura Crítica)
* **Fallo Detectado:** El canal oficial de captación de talento (`miprimerempleo@...`) se encontraba inoperante, devolviendo un error **SMTP 550 5.4.1 (Access Denied)**, lo que indica una mala configuración del filtro de destinatarios.
* **Resolución/Validación:** Mediante el análisis de patrones de nomenclatura institucional y técnicas de enumeración, se logró validar el vector de contacto real utilizando el patrón `[inicial][apellido]`.
* **Impacto:** Se superaron con éxito las políticas de **Directory-Based Edge Blocking (DBEB)**, confirmando la posibilidad de realizar ataques de fuerza bruta de usuarios o *Account Takeover* (ATO).

---

## 🛠 Herramientas y Metodología
Para estos ejercicios se utilizaron técnicas puramente pasivas y herramientas de consulta pública:
* **Reconocimiento de DNS:** `dig`, `nslookup`, `DNSDumpster`.
* **Identity Validation:** Microsoft 365 Tenant Discovery & Teams Global Search.
* **SMTP Analysis:** Telnet/Netcat para análisis de cabeceras y códigos de error.
* **Frameworks:** OSINT Framework y metodologías de reconocimiento de activos externos (EASM).

## ⚠️ Disclaimer
Toda la información contenida en este repositorio tiene fines exclusivamente **educativos y de concienciación en ciberseguridad**. No se realizaron ataques disruptivos ni se comprometió la integridad de los sistemas analizados. Todas las vulnerabilidades críticas han sido reportadas/detectadas bajo un marco de ética profesional.
