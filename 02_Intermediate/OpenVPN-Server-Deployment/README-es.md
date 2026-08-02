# Guía de Laboratorio: Montaje de un Servidor OpenVPN

**Criptografía · Guía de Laboratorio**

Universidad Tecnológica de Panamá · Facultad de Ingeniería de Sistemas Computacionales


## 📌 Objetivo

Montar un servidor **OpenVPN** sobre Ubuntu Server, generando su propia infraestructura de llaves (PKI) con **EasyRSA**, configurando el servicio, emitiendo un certificado de cliente y validando la conexión VPN de extremo a extremo.

---

## 🖥️ 1. Preparación del servidor

### Acceso root / privilegios

```bash
sudo whoami
# root
```

<p align="center">
  <img src="screenshots/01_acceso_root_whoami.png" width="600" alt="Verificación de privilegios root con sudo whoami">
</p>

### Actualización de paquetes

```bash
sudo apt update && sudo apt upgrade -y
```

<p align="center">
  <img src="screenshots/02_actualizando_paquetes.png" width="650" alt="Actualización de paquetes del sistema">
</p>
<p align="center"><em>Actualización en progreso...</em></p>

<p align="center">
  <img src="screenshots/03_paquetes_actualizados.png" width="650" alt="Paquetes actualizados correctamente">
</p>
<p align="center"><em>Paquetes actualizados, sin servicios pendientes de reinicio crítico.</em></p>

---

## 📦 2. Instalación de OpenVPN y Easy-RSA

```bash
sudo apt install openvpn easy-rsa -y
```

<p align="center">
  <img src="screenshots/04_instalando_openvpn.png" width="650" alt="Instalación de openvpn y easy-rsa">
</p>

<p align="center">
  <img src="screenshots/05_instalando_openvpn_unpack.png" width="650" alt="Desempaquetado de openvpn y easy-rsa">
</p>
<p align="center"><em>Paquetes <code>openvpn</code>, <code>easy-rsa</code> y dependencias instalados correctamente.</em></p>

---

## 🔐 3. Creación de la infraestructura de llaves (PKI) con EasyRSA

### Directorio de trabajo

```bash
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
```

<p align="center">
  <img src="screenshots/06_creando_directorio_easyrsa.png" width="600" alt="Creación del directorio openvpn-ca">
</p>

### Inicializar PKI

```bash
sudo ./easyrsa init-pki
```

<p align="center">
  <img src="screenshots/07_inicializando_pki.png" width="600" alt="Inicialización de la PKI">
</p>

### Construir la Autoridad Certificadora (CA)

```bash
sudo ./easyrsa build-ca
```

Common Name usado: `openvpn-ca`

<p align="center">
  <img src="screenshots/08_construyendo_ca.png" width="600" alt="Construcción de la CA con easyrsa build-ca">
</p>

### Certificado del servidor

```bash
sudo ./easyrsa sign-req server server
```

<p align="center">
  <img src="screenshots/09_certificado_servidor.png" width="650" alt="Firma del certificado del servidor, válido por 825 días">
</p>
<p align="center"><em>Certificado del servidor firmado y válido por 825 días.</em></p>

### Parámetros Diffie-Hellman

```bash
sudo ./easyrsa gen-dh
```

<p align="center">
  <img src="screenshots/10_diffie_hellman_1.png" width="650" alt="Generación de parámetros Diffie-Hellman - progreso">
</p>
<p align="center"><img src="screenshots/11_diffie_hellman_2.png" width="650" alt="Generación de parámetros Diffie-Hellman - resultado"></p>
<p align="center"><em>Parámetros DH de 2048 bits generados en <code>pki/dh.pem</code>.</em></p>

### Clave HMAC (ta.key) — capa adicional de seguridad

```bash
openvpn --genkey secret ta.key
```

<p align="center">
  <img src="screenshots/12_generando_hmac.png" width="600" alt="Generación de la clave HMAC ta.key">
</p>

### Copiar archivos necesarios a /etc/openvpn/

```bash
sudo cp pki/ca.crt pki/issued/server.crt pki/private/server.key pki/dh.pem ta.key /etc/openvpn/
```

<p align="center">
  <img src="screenshots/13_copiando_archivos.png" width="600" alt="Copia de certificados y llaves a /etc/openvpn">
</p>

### Verificación de archivos

```bash
ls -l /etc/openvpn/
```

<p align="center">
  <img src="screenshots/14_verificacion_archivos.png" width="500" alt="Verificación de archivos copiados en /etc/openvpn">
</p>
<p align="center"><em><code>ca.crt</code>, <code>dh.pem</code>, <code>server.crt</code>, <code>server.key</code>, <code>ta.key</code> presentes.</em></p>

---

## ⚙️ 4. Configuración del servidor OpenVPN

### Archivo `/etc/openvpn/server.conf`

<p align="center">
  <img src="screenshots/15_server_conf.png" width="450" alt="Contenido del archivo server.conf">
</p>

```
port 1194
proto udp
dev tun
topology subnet
ca ca.crt
cert server.crt
key server.key
dh dh.pem
tls-auth ta.key 0
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
keepalive 10 120
cipher AES-256-CBC
persist-key
persist-tun
status openvpn-status.log
verb 3
user nobody
group nogroup
```

### Habilitar reenvío de IP (IP forwarding)

Se descomenta `net.ipv4.ip_forward=1` en `/etc/sysctl.conf`:

<p align="center">
  <img src="screenshots/16_habilitando_reenvio_ip.png" width="650" alt="Descomentando net.ipv4.ip_forward en sysctl.conf">
</p>

```bash
sudo sysctl -p
```

<p align="center">
  <img src="screenshots/17_aplicando_cambios_sysctl.png" width="600" alt="Aplicando cambios de sysctl">
</p>

### Iniciar y habilitar el servicio

```bash
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server
```

<p align="center">
  <img src="screenshots/18_iniciar_servicio_start.png" width="600" alt="Inicio del servicio openvpn@server">
</p>
<p align="center">
  <img src="screenshots/19_iniciar_servicio_enable.png" width="600" alt="Habilitación del servicio openvpn@server al arranque">
</p>

### Verificar estado del servicio

```bash
sudo systemctl status openvpn@server.service
```

<p align="center">
  <img src="screenshots/20_verificando_status.png" width="650" alt="Estado active (running) del servicio OpenVPN">
</p>
<p align="center"><em>Servicio <strong>active (running)</strong>: túnel <code>tun0</code> con IP 10.8.0.1, "Initialization Sequence Completed".</em></p>

---

## 👤 5. Certificado de cliente

### Generar solicitud de certificado (sin passphrase)

```bash
sudo ./easyrsa gen-req client1 nopass
```

<p align="center">
  <img src="screenshots/21_creando_cert_cliente.png" width="650" alt="Generación de la solicitud de certificado para client1">
</p>

### Firmar el certificado del cliente

```bash
sudo ./easyrsa sign-req client client1
```

<p align="center">
  <img src="screenshots/22_sign_req_client.png" width="650" alt="Firma del certificado de cliente client1, válido 825 días">
</p>

### Copiar archivos al cliente

```bash
mkdir client-files
sudo cp /etc/openvpn/ca.crt client-files/
sudo cp /etc/openvpn/ta.key client-files/
sudo cp openvpn-ca/pki/issued/client1.crt client-files/
sudo cp openvpn-ca/pki/private/client1.key client-files/
```

<p align="center">
  <img src="screenshots/23_copiando_al_cliente.png" width="650" alt="Copiando archivos de certificado al cliente">
</p>

---

## 📄 6. Archivo de configuración del cliente (`client1.ovpn`)

<p align="center">
  <img src="screenshots/24_archivo_cliente_ovpn.png" width="550" alt="Contenido del archivo .ovpn del cliente con certificados embebidos">
</p>

```
client
dev tun
proto udp
remote 127.0.0.1 1194
resolv-retry infinite
nobind
persist-key
persist-tun
data-ciphers AES-256-CBC:AES-256-GCM:AES-128-GCM
data-ciphers-fallback AES-256-CBC
remote-cert-tls server
key-direction 1
<ca> ... </ca>
<cert> ... </cert>
<key> ... </key>
<tls-auth> ... </tls-auth>
```

> 📝 **Nota del laboratorio:** *"Le tuve que cambiar la parte del cipher porque me decía que era obsoleto."* — Se reemplazó el cifrador simple (`cipher AES-256-CBC`) por `data-ciphers` con fallback, ya que los clientes OpenVPN recientes marcan `cipher` como una directiva en desuso.

---

## ✅ 7. Prueba de conexión

### Cliente OpenVPN GUI (Windows)

<p align="center">
  <img src="screenshots/25_prueba_conexion_gui.png" width="500" alt="Cliente OpenVPN GUI conectado, IP asignada 10.8.0.2">
</p>
<p align="center"><em>Estado: <strong>Conectado</strong> · IP asignada: <code>10.8.0.2</code>.</em></p>

### Verificación desde el servidor (máquina host)

<p align="center">
  <img src="screenshots/26_acceso_root_cliente_host.png" width="650" alt="Acceso y estado del sistema en la máquina host/servidor">
</p>

### Prueba de conectividad (ping por el túnel)

```bash
ping 10.8.0.1
```

<p align="center">
  <img src="screenshots/27_conectividad_ping.png" width="650" alt="Ping exitoso a 10.8.0.1 a través del túnel tun0">
</p>
<p align="center"><em>Interfaz <code>tun0</code> activa (10.8.0.1/24) — respuesta ICMP exitosa en ~0.03–0.05 ms.</em></p>

### Prueba en un puerto alternativo (5000)

<p align="center">
  <img src="screenshots/28_otro_puerto_5000.png" width="500" alt="Conexión OpenVPN exitosa usando el puerto 5000">
</p>
<p align="center"><em>Conexión exitosa reconfigurando el servidor/cliente al puerto <code>5000</code>, IP asignada nuevamente <code>10.8.0.2</code>.</em></p>

---

## 🧩 Resumen del flujo

```
Ubuntu Server (root)
   ↓ apt install openvpn easy-rsa
   ↓ easyrsa: init-pki → build-ca → sign-req server → gen-dh → genkey ta.key
   ↓ cp certs/keys → /etc/openvpn/
   ↓ server.conf (puerto 1194/UDP, subred 10.8.0.0/24, AES-256)
   ↓ ip_forward=1 + systemctl start/enable openvpn@server
   ↓ easyrsa: gen-req client1 → sign-req client client1
   ↓ client1.ovpn (certs embebidos, cipher actualizado)
   ↓ Conexión desde OpenVPN GUI → IP 10.8.0.2 asignada, ping OK
```

```
