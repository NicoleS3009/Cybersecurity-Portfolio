# Laboratorio de Criptografía – Implementación de Dominio, DNS y TLS

⚠️ Laboratorio realizado en un entorno académico controlado con fines educativos.  
La información sensible (dominios, direcciones IP) ha sido anonimizada por motivos de seguridad.

## Contexto
Proyecto académico de la asignatura **Criptografía** en la Universidad Tecnológica de Panamá.  
El objetivo fue implementar criptografía aplicada mediante la obtención de un certificado **TLS válido**, utilizando un dominio propio, **Cloudflare** como proveedor DNS y un servidor público en la nube para desplegar un servicio seguro vía **HTTPS**.

## Rol personal
- Registro y configuración de un dominio propio para laboratorio.  
- Integración del dominio en **Cloudflare** (plan gratuito).  
- Creación y gestión de una instancia pública en la nube (Debian).  
- Configuración de registros DNS y validación del certificado TLS.  
- Documentación de evidencias del dominio, servidor y certificado.  

## Implementación (visión general)
El procedimiento se describe a nivel conceptual, sin incluir configuraciones específicas ni comandos sensibles.

1. **Registro del dominio**
   - Adquisición de un dominio destinado a laboratorio.
   - Verificación del control del dominio mediante el proveedor.

2. **Configuración DNS con Cloudflare**
   - Asociación del dominio al panel de Cloudflare.
   - Actualización de nameservers.
   - Creación de registros DNS requeridos.
   - Verificación del estado activo del dominio.

3. **Despliegue del servidor**
   - Creación de una instancia Linux (Debian/Ubuntu) en la nube.
   - Configuración básica de red y acceso.
   - Exposición controlada de servicios web (HTTP/HTTPS).

4. **Validación de TLS**
   - Acceso al servicio web mediante HTTPS.
   - Verificación del certificado TLS desde el navegador y herramientas de inspección.
   - Confirmación de comunicación cifrada en la capa de transporte.

## Resultados
- Servicio web accesible mediante **HTTPS**.
- Certificado **TLS válido** correctamente emitido y gestionado por Cloudflare.
- Infraestructura funcional con seguridad aplicada en la comunicación web.

## Aprendizaje
- Comprensión práctica de la relación entre **dominio, DNS y certificados TLS**.
- Uso de **Cloudflare** como proveedor de DNS y capa adicional de seguridad.
- Despliegue de servicios públicos seguros en infraestructura cloud.
- Importancia de la criptografía aplicada en la protección de comunicaciones.

## Referencias
- [Cloudflare Dashboard](https://dash.cloudflare.com/)  
- [Debian Official Documentation](https://www.debian.org/doc/)  
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)  
- [YouTube – Configuración de DNS en Cloudflare](https://www.youtube.com/watch?v=XKDqH0Ndp0Q)  
