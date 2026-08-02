# Self-Signed-TLS-with-OpenSSL

## Contexto
Laboratorio académico enfocado en la comprensión del proceso de creación y configuración de certificados TLS mediante OpenSSL.

## Objetivo
Implementar comunicación cifrada en un servidor web utilizando un certificado autofirmado.

## Tecnologías
- OpenSSL
- Apache / Nginx
- TLS

## Implementación

### Generación de clave privada y certificado autofirmado
Se generó un certificado autofirmado con SAN (Subject Alternative Name) usando OpenSSL directamente, incluyendo `example.local` y `127.0.0.1` como nombres alternativos.

![Generación del certificado autofirmado con OpenSSL](screenshots/01-generar-certificado.png)

### Configuración del servidor web para TLS

**Apache**

Instalación del módulo SSL y habilitación:

![Instalación de Apache y módulo SSL](screenshots/02-instalar-apache-ssl.png)
![Módulo SSL habilitado en Apache](screenshots/03-apache-ssl-habilitado.png)

Creación del VirtualHost HTTPS apuntando al certificado generado:

![VirtualHost HTTPS en Apache](screenshots/04-virtualhost-apache-conf.png)

Habilitación del sitio y recarga del servicio:

![Sitio SSL habilitado en Apache](screenshots/05-habilitar-sitio-apache.png)

**Nginx**

Instalación de Nginx:

![Instalación de Nginx](screenshots/06-instalar-nginx.png)
![Nginx instalado correctamente](screenshots/07-nginx-instalado.png)

Creación del bloque de servidor HTTPS:

![Server block HTTPS en Nginx](screenshots/08-server-block-nginx-conf.png)

Activación del sitio y recarga del servicio:

![Sitio activado en Nginx](screenshots/09-activar-sitio-nginx.png)

## Pruebas
Se verificó el cifrado TLS y el comportamiento del navegador ante certificados no confiables.

Prueba con `curl` (ignorando la validación del certificado por ser autofirmado):

![Prueba de conexión TLS con curl](screenshots/10-curl-prueba-tls.png)

Verificación de los detalles del certificado (subject, issuer, fechas de validez, SAN) con `openssl s_client`:

![Detalles del certificado vía openssl s_client](screenshots/11-openssl-s-client-prueba.png)

## Opcional: Autoridad Certificadora (CA) propia

Como ejercicio adicional, se creó una CA local y se firmó el certificado del servidor con ella en lugar de usar un certificado autofirmado directo, agregando extensiones SAN mediante un archivo de configuración.

![Creación de la CA local y el CSR del servidor](screenshots/12-crear-ca-y-csr.png)
![Archivo de extensiones SAN](screenshots/13-extensiones-san.png)
![Firma del certificado del servidor con la CA local](screenshots/14-firmar-certificado.png)

> **Nota:** al crear el archivo de extensiones con `sudo cat > archivo`, se obtuvo un error de permiso denegado (el redireccionamiento `>` se ejecuta con los permisos del shell actual, no con los de `sudo`). La solución fue usar `sudo tee archivo`, que sí escribe el contenido con privilegios de root sin que falle el redireccionamiento.

## Aprendizaje
Comprensión del flujo de certificados, confianza TLS y advertencias de seguridad.
