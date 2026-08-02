# Laboratory Guide: Setting Up an OpenVPN Server

**Cryptography · Laboratory Guide**

Technological University of Panama · Faculty of Computer Systems Engineering

## 📌 Objective

To set up an **OpenVPN** server on Ubuntu Server, generating its own Public Key Infrastructure (PKI) with **EasyRSA**, configuring the service, issuing a client certificate, and validating the end-to-end VPN connection.

---

## 🖥️ 1. Server Preparation

### Root access / privileges

```bash
sudo whoami
# root

```

### Package update

```bash
sudo apt update && sudo apt upgrade -y

```

---

## 📦 2. Installing OpenVPN and Easy-RSA

```bash
sudo apt install openvpn easy-rsa -y

```

---

## 🔐 3. Creating the Public Key Infrastructure (PKI) with EasyRSA

### Working directory

```bash
make-cadir ~/openvpn-ca
cd ~/openvpn-ca

```

### Initialize PKI

```bash
sudo ./easyrsa init-pki

```

### Build the Certificate Authority (CA)

```bash
sudo ./easyrsa build-ca

```

Common Name used: `openvpn-ca`

### Server certificate

```bash
sudo ./easyrsa sign-req server server

```

### Diffie-Hellman parameters

```bash
sudo ./easyrsa gen-dh

```

### HMAC key (ta.key) — additional security layer

```bash
openvpn --genkey secret ta.key

```

### Copy necessary files to /etc/openvpn/

```bash
sudo cp pki/ca.crt pki/issued/server.crt pki/private/server.key pki/dh.pem ta.key /etc/openvpn/

```

### File verification

```bash
ls -l /etc/openvpn/

```

---

## ⚙️ 4. OpenVPN Server Configuration

### `/etc/openvpn/server.conf` file

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

### Enable IP forwarding

Uncomment `net.ipv4.ip_forward=1` in `/etc/sysctl.conf`:

```bash
sudo sysctl -p

```

### Start and enable the service

```bash
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server

```

### Verify service status

```bash
sudo systemctl status openvpn@server.service

```

---

## 👤 5. Client Certificate

### Generate certificate request (without passphrase)

```bash
sudo ./easyrsa gen-req client1 nopass

```

### Sign the client certificate

```bash
sudo ./easyrsa sign-req client client1

```

### Copy files to the client

```bash
mkdir client-files
sudo cp /etc/openvpn/ca.crt client-files/
sudo cp /etc/openvpn/ta.key client-files/
sudo cp openvpn-ca/pki/issued/client1.crt client-files/
sudo cp openvpn-ca/pki/private/client1.key client-files/

```

---

## 📄 6. Client Configuration File (`client1.ovpn`)

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

> 📝 **Laboratory Note:** *"I had to change the cipher part because it said it was obsolete."* — The simple cipher (`cipher AES-256-CBC`) was replaced by `data-ciphers` with a fallback, as recent OpenVPN clients flag `cipher` as a deprecated directive.

---

## ✅ 7. Connection Test

### OpenVPN GUI Client (Windows)

### Verification from the server (host machine)

### Connectivity test (ping through the tunnel)

```bash
ping 10.8.0.1

```

### Test on an alternative port (5000)

---

## 🧩 Flow Summary

```
Ubuntu Server (root)
   ↓ apt install openvpn easy-rsa
   ↓ easyrsa: init-pki → build-ca → sign-req server → gen-dh → genkey ta.key
   ↓ cp certs/keys → /etc/openvpn/
   ↓ server.conf (port 1194/UDP, subnet 10.8.0.0/24, AES-256)
   ↓ ip_forward=1 + systemctl start/enable openvpn@server
   ↓ easyrsa: gen-req client1 → sign-req client client1
   ↓ client1.ovpn (embedded certs, updated cipher)
   ↓ Connection from OpenVPN GUI → IP 10.8.0.2 assigned, ping OK

```