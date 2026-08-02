# Self-Signed TLS with OpenSSL

## Context
Academic laboratory focused on understanding the process of creating and configuring TLS certificates using OpenSSL.

## Objective
Implement encrypted communication on a web server using a self-signed certificate.

## Technologies
- OpenSSL
- Apache / Nginx
- TLS

## Implementation

### Generation of Private Key and Self-Signed Certificate
A self-signed certificate with SAN (Subject Alternative Name) was generated directly using OpenSSL, including `example.local` and `127.0.0.1` as alternative names.

![Generation of self-signed certificate with OpenSSL](screenshots/01-generar-certificado.png)

### Web Server Configuration for TLS

**Apache**

Installation and enablement of the SSL module:

![Apache installation and SSL module](screenshots/02-instalar-apache-ssl.png)
![SSL module enabled in Apache](screenshots/03-apache-ssl-habilitado.png)

Creation of the HTTPS VirtualHost pointing to the generated certificate:

![HTTPS VirtualHost in Apache](screenshots/04-virtualhost-apache-conf.png)

Site enablement and service reload:

![SSL site enabled in Apache](screenshots/05-habilitar-sitio-apache.png)

**Nginx**

Installation of Nginx:

![Nginx installation](screenshots/06-instalar-nginx.png)
![Nginx installed successfully](screenshots/07-nginx-instalado.png)

Creation of the HTTPS server block:

![HTTPS server block in Nginx](screenshots/08-server-block-nginx-conf.png)

Site activation and service reload:

![Site activated in Nginx](screenshots/09-activar-sitio-nginx.png)

## Testing
TLS encryption and browser behavior regarding untrusted certificates were verified.

Test with `curl` (ignoring certificate validation due to being self-signed):

![TLS connection test with curl](screenshots/10-curl-prueba-tls.png)

Verification of certificate details (subject, issuer, validity dates, SAN) with `openssl s_client`:

![Certificate details via openssl s_client](screenshots/11-openssl-s-client-prueba.png)

## Optional: Custom Certificate Authority (CA)

As an additional exercise, a local CA was created and the server certificate was signed with it instead of using a direct self-signed certificate, adding SAN extensions via a configuration file.

![Creation of local CA and server CSR](screenshots/12-crear-ca-y-csr.png)
![SAN extensions file](screenshots/13-extensiones-san.png)
![Signing server certificate with local CA](screenshots/14-firmar-certificado.png)

> **Note:** when creating the extensions file with `sudo cat > file`, a permission denied error was encountered (the `>` redirection executes with current shell permissions, not `sudo`). The solution was using `sudo tee file`, which writes the content with root privileges without failing redirection.

## Key Takeaways
Understanding certificate workflow, TLS trust models, and security warnings.