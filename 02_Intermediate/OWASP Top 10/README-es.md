# Laboratorio Pentest: Burp Suite - OWASP Juice Shop

---

## 📌 Resumen Ejecutivo

Se llevó a cabo una evaluación de seguridad sobre la aplicación web **OWASP Juice Shop** con fines formativos y de práctica en análisis de vulnerabilidades. Durante el proceso se identificaron **10 fallos de seguridad**, de los cuales **5 fueron clasificados como de severidad crítica** debido al nivel de riesgo que representan para la confidencialidad, integridad y disponibilidad del sistema.

---

## ⚙️ Setup Completo

* **Burp Suite** + Proxy del navegador
* **Docker** instalado
* **Scope** delimitado en la aplicación de pruebas local (`http://localhost:3000`)

---

## 📊 Resumen de Hallazgos

| Vulnerabilidad | Severidad | OWASP | Estado |
| :--- | :---: | :---: | :---: |
| Admin Panel - Broken Access Control | Crítica | A01 | Confirmado |
| SQL Injection - Login Bypass | Crítica | A02 | Confirmado |
| DOM XSS | Crítica | A05 | Confirmado |
| Bonus Payload | Crítica | A05 | Confirmado |
| Admin Registration | Alta | A01 | Confirmado |
| IDOR - Acceso a Perfil de Otro Usuario | Crítica | A01 | Confirmado |
| Directory Listing — `/ftp` Expuesto | Media | A01 | Confirmado |
| Error Handling | Media | A02 | Confirmado |
| API Data Exposure — Enumerar Usuarios | Media | A01 | Confirmado |
| Broken Auth — Sin Rate Limiting en Login | Media | A07 | Confirmado |

---

## 📝 Hallazgos Detallados (Write-ups)

### Write-up 1: Admin Panel - Broken Access Control

* **Severidad:** CWE-284: Improper Access Control
* **Descripción:** Se identificó que la aplicación web expone un panel de administración accesible sin autenticación ni validación de privilegios. Esto permite que cualquier usuario no autenticado pueda acceder a funcionalidades administrativas simplemente conociendo o descubriendo la ruta.

#### Pasos a reproducir
1. Acceder a la aplicación en `http://localhost:3000`.
2. Revisar el Site Map generado o inspeccionar manualmente los recursos cargados.
3. Analizar el archivo `main.js` para identificar rutas ocultas.
4. Encontrar referencia a la ruta `/admin`.
5. Navegar directamente a `http://localhost:3000/admin`.
6. Confirmar el acceso al área restringida.

#### Impacto Real
* Acceso no autorizado a funcionalidades administrativas y posible manipulación de datos sensibles.
* Riesgo de escalamiento de privilegios y compromiso total de la aplicación.

#### Remediación
* Requerir autenticación obligatoria para acceder a `/admin` y aplicar controles RBAC.
* Evitar exponer rutas sensibles en archivos JavaScript públicos.

#### Referencias
* [OWASP Top 10 – A01:2025 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/)
* [CWE-284: Improper Access Control](https://cwe.mitre.org/data/definitions/284.html)

---

### Write-up 2: SQL Injection - Login Bypass

* **Severidad:** CWE-89 – Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection')
* **Descripción:** El parámetro `email` en el endpoint de autenticación no valida ni sanitiza la entrada del usuario antes de utilizarla en una consulta SQL, lo que permite manipular la lógica y eludir la autenticación.

#### Pasos a reproducir
1. Acceder a la página de Login de la aplicación.
2. Interceptar la solicitud `POST /rest/user/login` con Burp Suite.
3. Modificar el cuerpo JSON reemplazando el valor del campo `email` por un payload SQL (ej. `' OR 1=1--`).
4. Enviar la solicitud y verificar que se devuelve un token JWT con privilegios elevados.

#### Impacto Real
* Acceso no autorizado a cuentas de usuarios (incluyendo administradores).
* Modificación, eliminación o exfiltración masiva de datos.

#### Remediación
* Implementar consultas parametrizadas (*prepared statements*) en todas las operaciones SQL.
* Validar y sanitizar estrictamente las entradas del usuario.

#### Referencias
* [OWASP Top 10 - A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
* [CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)

---

### Write-up 3: DOM XSS

* **Severidad:** CWE-79 – Improper Neutralization of Input During Web Page Generation ('Cross-Site Scripting')
* **Descripción:** La aplicación procesa datos no confiables provenientes de la URL o del DOM directamente en el navegador mediante JavaScript sin la debida sanitización.

#### Pasos a reproducir
1. En la barra de búsqueda, ingresar el payload: `<iframe src="javascript:alert('DOM_XSS')">`.
2. Presionar Enter y observar la ejecución de la alerta de JavaScript.

#### Impacto Real
* Ejecución arbitraria de JavaScript, robo de tokens de sesión/JWT y suplantación de identidad.

#### Remediación
* Evitar funciones inseguras como `innerHTML` o `document.write`; utilizar `textContent` o `innerText`.
* Implementar Content Security Policy (CSP).

#### Referencias
* [OWASP Top 10 - A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
* [CWE-79: Cross-Site Scripting](https://cwe.mitre.org/data/definitions/79.html)

---

### Write-up 4: Bonus Payload (HTML Injection)

* **Severidad:** CWE-79 – Improper Neutralization of Input During Web Page Generation ('Cross-Site Scripting')
* **Descripción:** Se comprobó que el sink vulnerable en el cliente no solo ejecuta JavaScript, sino que permite la inyección de elementos HTML arbitrarios (como `<iframe>` con fuentes externas).

#### Pasos a reproducir
1. Sustituir el parámetro de búsqueda por un payload HTML con un iframe de un dominio externo.
2. Cargar la página y verificar que el recurso externo se renderiza dentro del DOM.

#### Impacto Real
* Phishing visual dentro de la aplicación y manipulación de la interfaz gráfica.

#### Remediación
* Utilizar librerías de sanitización como DOMPurify y configurar restricciones CSP para iframes.

---

### Write-up 5: Admin Registration

* **Severidad:** CWE-639 - Authorization Bypass Through User-Controlled Key
* **Descripción:** El endpoint de registro confía en parámetros enviados desde el cliente para asignar el rol de usuario, permitiendo registrar cuentas administrativas sin controles en el servidor.

#### Pasos a reproducir
1. Interceptar la solicitud `POST /api/Users` al registrar un usuario.
2. Modificar el JSON agregando o cambiando el parámetro que define el rol a administrador.
3. Enviar la petición y confirmar la creación de la cuenta privilegiada.

#### Impacto Real
* Creación no autorizada de cuentas administrativas y compromiso total del sistema.

#### Remediación
* Asignar roles únicamente desde la lógica interna del backend y eliminar cualquier parámetro de rol en el request del cliente.

---

### Write-up 6: IDOR — Acceso a Perfil de Otro Usuario

* **Severidad:** CWE-639 - Authorization Bypass Through User-Controlled Key
* **Descripción:** El sistema expone identificadores directos en solicitudes API que no validan si el usuario autenticado tiene permisos para acceder al recurso solicitado.

#### Pasos a reproducir
1. Iniciar sesión y realizar una petición a una ruta que recupere un recurso por ID (ej. `/api/orders/101`).
2. Interceptar la petición en Burp Repeater y modificar el ID (ej. pasar de `101` a `102`).
3. Verificar que el servidor responde con los datos del recurso ajeno.

#### Impacto Real
* Escalada horizontal de privilegios y fuga de información personal de otros usuarios.

#### Remediación
* Validar en backend que el objeto pertenezca al usuario autenticado e implementar UUIDs.

---

### Write-up 7: Directory Listing — `/ftp` Expuesto

* **Severidad:** CWE-548 – Exposure of Information Through Directory Listing
* **Descripción:** El directorio `/ftp` está expuesto públicamente, permitiendo navegar por la estructura de carpetas y descargar archivos internos sin autenticación.

#### Pasos a reproducir
1. Navegar a `http://localhost:3000/ftp`.
2. Observar el listado de archivos expuesto por el servidor web.
3. Descargar archivos para verificar contenido confidencial.

#### Impacto Real
* Filtración de información sensible, logs o credenciales almacenadas en archivos internos.

#### Remediación
* Deshabilitar el Directory Listing en el servidor web y mover archivos sensibles fuera de la raíz web.

---

### Write-up 8: Error Handling

* **Severidad:** CWE-209: Generation of Error Message Containing Sensitive Information
* **Descripción:** Al enviar solicitudes malformadas al backend, la aplicación responde con un error HTTP 500 que incluye el Stack Trace completo del entorno.

#### Pasos a reproducir
1. Interceptar una petición a `/api/Feedbacks`.
2. Omitir un carácter estructural en el JSON para generar un error sintáctico.
3. Enviar la petición y analizar la respuesta con detalles de código y rutas absolutas.

#### Impacto Real
* Fuga de información técnica (versiones de librerías, rutas en servidor, lógica interna).

#### Remediación
* Implementar un manejador global de excepciones para devolver mensajes de error genéricos en producción.

---

### Write-up 9: API Data Exposure — Enumerar Usuarios

* **Severidad:** CWE-213: Exposure of Sensitive Information Due to Incompatible Policies
* **Descripción:** El endpoint `/api/Users` permite a cualquier usuario autenticado consultar la lista completa de usuarios registrados sin requerir rol administrativo.

#### Pasos a reproducir
1. Iniciar sesión con un usuario común y obtener el token JWT.
2. Realizar una petición `GET` a `/api/Users` incluyendo la cabecera `Authorization: Bearer [token]`.
3. Analizar la respuesta JSON con la lista completa de registros.

#### Impacto Real
* Fuga masiva de PII (correos, roles, datos de acceso).

#### Remediación
* Restringir el acceso a la lista completa de usuarios únicamente a roles de administración.

---

### Write-up 10: Broken Auth — Sin Rate Limiting en Login

* **Severidad:** CWE-307: Improper Restriction of Excessive Authentication Attempts
* **Descripción:** La interfaz de inicio de sesión no restringe la cantidad de intentos fallidos consecutivas, permitiendo ataques de fuerza bruta automatizados.

#### Pasos a reproducir
1. Capturar la petición `POST /rest/user/login` y enviarla a Burp Intruder.
2. Configurar un ataque de diccionario sobre el campo `password`.
3. Ejecutar la ráfaga de peticiones y verificar que ninguna es bloqueada ni ralentizada.

#### Impacto Real
* Compromiso de cuentas mediante *credential stuffing* o ataques de diccionario.

#### Remediación
* Implementar políticas de bloqueo temporal, throttling (retardos) o uso de CAPTCHA tras varios intentos fallidos.

---

## 🎯 Conclusiones y Recomendaciones

El análisis demostró que las vulnerabilidades críticas más apremiantes residen en el mecanismo de autenticación (**Inyección SQL**) y en un control de acceso deficiente (**Admin Panel y Registro de Admins**). 

**Prioridades de remediación:**
1. **Fase 1 (Inmediata):** Sanitización de consultas SQL (prepared statements) y cierre de endpoints administrativos expuestos sin control de acceso.
2. **Fase 2:** Implementación de validaciones en el servidor para evitar IDORs y la asignación arbitraria de roles.
3. **Fase 3:** Hardening general del servidor web (desactivación de directory listing, manejo seguro de errores y adición de rate limiting).