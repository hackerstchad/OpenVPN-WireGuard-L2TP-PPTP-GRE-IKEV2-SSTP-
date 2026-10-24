# Guide Ultra-Avançé des Protocoles VPN et de Tunnelisation

<img width="1886" height="1061" alt="3a85df01-aa87-464c-af89-a9590a6dcefe_LE_upscale_prime" src="https://github.com/user-attachments/assets/347ab47e-4b17-4b7c-9e75-25406adeb9c6" />


**Auteur :** hackers_tchad  
**Date :** 2026  
**Version :** 2.0  
**Licence :** Usage éducatif, professionnel et recherche. Mention de l'auteur obligatoire pour redistribution.  


---

## Table des matières

1. [Introduction générale](#introduction-générale)
2. [OpenVPN](#openvpn)
3. [WireGuard](#wireguard)
4. [L2TP / L2TP/IPsec](#l2tp--l2tpipsec)
5. [PPTP](#pptp)
6. [GRE](#gre)
7. [IKEv2 / IPsec Mobility](#ikev2--ipsec-mobility)
8. [SSTP](#sstp)
9. [Comparatif technique global](#comparatif-technique-global)
10. [Architectures et topologies](#architectures-et-topologies)
11. [Commandes de vérification et dépannage](#commandes-de-vérification-et-dépannage)
12. [Applications, clients et implémentations](#applications-clients-et-implémentations)
13. [Sécurité, attaques et contre-mesures](#sécurité-attaques-et-contre-mesures)
14. [Ressources, bibliographie et liens](#ressources-bibliographie-et-liens)
15. [Fiches de synthèse rapide](#fiches-de-synthèse-rapide)
16. [Annexes](#annexes)

---

## Introduction générale

Les réseaux privés virtuels (VPN, *Virtual Private Network*) et les protocoles de tunnelisation constituent l'épine dorsale de la connectivité sécurisée moderne. Ils permettent d'étendre un réseau privé à travers un réseau public, généralement Internet, en chiffrant les données, en authentifiant les pairs et en assurant l'intégrité des flux. Ce guide traite en profondeur de **OpenVPN**, **WireGuard**, **L2TP** (souvent couplé à IPsec), **PPTP**, **GRE**, **IKEv2** et **SSTP**.

Ces protocoles répondent à des besoins variés :

- **Accès distant sécurisé** : télétravail, support IT, accès aux ressources internes.
- **Interconnexion de sites** : VPN site-à-site entre filiales, data centers et clouds.
- **Contournement de la censure** : accès à Internet libre dans des zones restreintes.
- **Protection de la vie privée** : masquage de l'adresse IP publique, chiffrement du trafic.
- **Segmentation réseau** : Zero Trust, micro-segmentation, cloisonnement des flux.
- **Haute disponibilité et mobilité** : basculement, roaming, MOBIKE.

Ce document s'adresse aux administrateurs réseau, architectes sécurité, ingénieurs DevOps/SRE, pentesters, étudiants en cybersécurité et toute personne souhaitant maîtriser les technologies de tunnelisation. Il couvre pour chaque protocole :

- Définition formelle et informelle.
- Historique, créateurs et standardisation.
- Architecture technique détaillée.
- Mécanismes cryptographiques.
- Modes de fonctionnement.
- Procédures de configuration complètes (serveur, client, routeurs, pare-feu, cloud).
- Commandes de vérification, diagnostic et dépannage.
- Applications et cas d'usage.
- Implémentations logicielles, matérielles et cloud.
- Outils d'administration, de test et de surveillance.
- Bibliographie, RFCs, livres, liens et ressources communautaires.

> **Ligne de conduite :** Aucun protocole n'est universellement supérieur. Le choix dépend du niveau de sécurité requis, des performances souhaitées, de la complexité d'administration, de la compatibilité client et du contexte réglementaire.

---

## TL;DR - Résumé exécutif

| Protocole | Couche OSI | Chiffrement | Authentification | Cas d'usage principal | Statut sécurité |
|-----------|-----------|-------------|------------------|----------------------|-----------------|
| **OpenVPN** | 4 (UDP/TCP 1194) | SSL/TLS, AES-GCM, ChaCha20-Poly1305 | Certificats X.509, TLS-Auth, user/pass | VPN accès distant, site-à-site, contournement | Très bon |
| **WireGuard** | 3 (UDP) | Curve25519, ChaCha20-Poly1305, Blake2s | Clés publiques (Noise protocol) | VPN haute performance, IoT, containers, mesh | Excellent |
| **L2TP/IPsec** | 2 (L2TP) + 3 (IPsec/ESP) | IPsec/ESP (AES, 3DES) | PSK, certificats, EAP | VPN intégré Windows/macOS/iOS/Android | Correct (IPsec) |
| **PPTP** | 2 (GRE + TCP 1723) | MPPE (RC4 obsolète) | MS-CHAPv2, EAP-TLS | Héritage, rarement recommandé | Obsolète / Dangereux |
| **GRE** | 3 (IP protocol 47) | Aucun natif (IPsec recommandé) | Aucun natif | Tunnelisation générique, MPLS, overlay | Neutre (tunnel brut) |
| **IKEv2/IPsec** | 3 (UDP 500/4500) | IPsec/ESP (AES-GCM, ChaCha20-Poly1305) | Certificats, EAP-MSCHAPv2, EAP-TLS | Mobilité (MOBIKE), smartphones, entreprise | Excellent |
| **SSTP** | 4 (TCP 443 / HTTPS) | TLS (SSL) | Certificats, Active Directory | Environnements Windows, pare-feu restrictifs | Très bon |

---

# 1. OpenVPN

## 1.1 Définition

**OpenVPN** est une solution VPN open source qui utilise la bibliothèque OpenSSL pour fournir un chiffrement SSL/TLS, ainsi que des mécanismes d'authentification et d'intégrité. Contrairement à IPsec qui opère à la couche 3, OpenVPN fonctionne principalement à la couche transport (couche 4 du modèle OSI), encapsulant le trafic IP dans des paquets UDP ou TCP. Il peut transporter à la fois du trafic IPv4 et IPv6, et créer des tunnels de couche 2 (TAP) ou de couche 3 (TUN).

OpenVPN est devenu un standard de facto pour les VPN auto-hébergés grâce à sa flexibilité, sa portabilité et sa robustesse cryptographique.

## 1.2 Historique et créateurs

- **Créateur :** James Yonan.
- **Première publication :** 2001.
- **Licence :** GNU GPL v2.
- **Organisation :** OpenVPN Inc. (anciennement OpenVPN Technologies, Inc.), qui gère également la version commerciale **OpenVPN Access Server**.
- **Version actuelle :** OpenVPN 2.6.x (2024-2025), supportant notamment les suites **ChaCha20-Poly1305** et **AES-GCM** en mode données.
- **Standardisation :** Bien qu'OpenVPN ne soit pas une RFC à proprement parler, il est documenté par le projet et des implémentations tierces. Le protocole est largement audité et déployé.

## 1.3 Architecture d'OpenVPN

```
┌────────────────────────────────────────────────────────────────────┐
│                         Applications                               │
├────────────────────────────────────────────────────────────────────┤
│                         TUN/TAP (couche 3/2)                       │
├────────────────────────────────────────────────────────────────────┤
│                         OpenVPN (SSL/TLS)                          │
├────────────────────────────────────────────────────────────────────┤
│                         UDP / TCP (port 1194)                      │
├────────────────────────────────────────────────────────────────────┤
│                         IP / Internet                              │
└────────────────────────────────────────────────────────────────────┘
```

OpenVPN se positionne au-dessus de UDP ou TCP. Il crée une interface virtuelle côté client et serveur, puis route ou bridge le trafic à travers le tunnel chiffré.

### Modes de fonctionnement

| Mode | Interface | Description | Cas d'usage |
|------|-----------|-------------|-------------|
| **TUN** | Couche 3 (routage IP) | Transporte des paquets IP routés | Accès distant, site-à-site, VPN classique |
| **TAP** | Couche 2 (bridge Ethernet) | Transporte des trames Ethernet | Jeux en réseau, protocoles non-IP, broadcast, VLAN étendu |
| **UDP** | Transport non fiable | Par défaut, meilleures performances | VPN généraliste, site-à-site |
| **TCP** | Transport fiable | Contournement des pare-feu stricts, faux-ami TCP | Réseaux restrictifs, proxy SOCKS/HTTP |

## 1.4 Mécanismes cryptographiques

### OpenSSL et TLS

OpenVPN s'appuie sur OpenSSL (ou mbed TLS dans certaines versions alternatives) pour :

- Le handshake TLS entre client et serveur.
- L'authentification via certificats X.509.
- La négociation des clés de session.
- L'établissement d'un canal de contrôle sécurisé.

### Chiffrement des données

Le canal de données utilise des suites cryptographiques définies par la directive `cipher` et `auth`. À partir d'OpenVPN 2.5, les modes AEAD (Authenticated Encryption with Associated Data) sont recommandés :

- `AES-256-GCM`
- `AES-128-GCM`
- `ChaCha20-Poly1305`

Anciennes options (à éviter) :

- `BF-CBC` (Blowfish)
- `AES-256-CBC` sans HMAC séparé
- `SHA1` pour l'authentification

### Authentification HMAC

La directive `auth` définit l'algorithme de HMAC pour le canal de données. En mode AEAD, ce paramètre est intégré au chiffrement. Recommandations :

- `SHA256`
- `SHA384`
- `SHA512`

### TLS-Auth / TLS-Crypt / TLS-Crypt-V2

- **tls-auth** : signe et vérifie les paquets de contrôle TLS avec une clé partagée, atténuant les attaques par déni de service et les scans de vulnérabilité.
- **tls-crypt** : chiffre et authentifie tous les paquets de contrôle (améliore la confidentialité et la résistance au DoS).
- **tls-crypt-v2** : permet de générer des clés uniques par client à partir d'une clé serveur maîtresse, facilitant la révocation individuelle.

### Échange de clés

OpenVPN supporte plusieurs méthodes d'échange de clés pour le canal de données :

- **TLS** (par défaut) : négociation via le handshake TLS.
- **Static Key** (pré-partagé, obsolète) : simple clé symétrique partagée, sans forward secrecy.
- **TLS + ECDHE** : forward secrecy assurée par les échanges éphémères Diffie-Hellman.

### PKI et certificats

OpenVPN utilise une infrastructure à clés publiques (PKI) pour émettre des certificats clients et serveurs. L'outil historique est **easy-rsa**, mais on peut aussi utiliser **cfssl**, **step-ca** ou une PKI d'entreprise.

```
CA racine
   │
   ├── Certificat serveur (server.crt + server.key)
   ├── Certificat client 1 (client1.crt + client1.key)
   ├── Certificat client 2 (client2.crt + client2.key)
   └── Certificat révoqué (CRL)
```

## 1.5 Installation

### Serveur Debian/Ubuntu

```bash
sudo apt update
sudo apt install openvpn easy-rsa iptables-persistent
```

### Serveur CentOS/RHEL/Rocky

```bash
sudo dnf install epel-release -y
sudo dnf install openvpn easy-rsa
```

### Client Windows

Télécharger l'installateur officiel sur https://openvpn.net/community-downloads/ ou utiliser le client GUI intégré.

### Client macOS

```bash
brew install openvpn
```

Ou utiliser **Tunnelblick** (GUI open source).

### Client Android/iOS

Applications recommandées :

- **OpenVPN Connect** (officielle)
- **Passepartout** (iOS)
- **OpenVPN for Android** (Android open source)

## 1.6 Configuration complète du serveur

### Étape 1 : Initialiser easy-rsa

```bash
cp -r /usr/share/easy-rsa /etc/openvpn/
cd /etc/openvpn/easy-rsa
./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa gen-dh
./easyrsa build-server-full server nopass
./easyrsa gen-crl
openvpn --genkey secret /etc/openvpn/ta.key
```

### Étape 2 : Fichier serveur `/etc/openvpn/server.conf`

```ini
# Port et protocole
port 1194
proto udp

# Interface virtuelle
dev tun

# Certificats et clés
ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key
dh /etc/openvpn/easy-rsa/pki/dh.pem
crl-verify /etc/openvpn/easy-rsa/pki/crl.pem

# Réseau VPN
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt

# Routes poussées vers les clients
push "route 192.168.10.0 255.255.255.0"
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 1.1.1.1"
push "dhcp-option DNS 8.8.8.8"
push "block-outside-dns"

# Cryptographie moderne
cipher AES-256-GCM
auth SHA256
tls-version-min 1.2
tls-crypt /etc/openvpn/ta.key

# Authentification supplémentaire (optionnel)
# plugin /usr/lib/openvpn/openvpn-plugin-auth-pam.so login

# Sécurité et performances
user nobody
group nogroup
persist-key
persist-tun
keepalive 10 120
max-clients 100

# Logs
status /var/log/openvpn/openvpn-status.log
verb 3
explicit-exit-notify 1
```

### Étape 3 : Activer le forwarding IP et le NAT

```bash
# Activer le forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

# NAT pour le sous-réseau VPN
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

### Étape 4 : Démarrer le service

```bash
sudo systemctl enable --now openvpn@server
sudo systemctl status openvpn@server
```

## 1.7 Configuration du client

### Fichier client.ovpn

```ini
client
dev tun
proto udp
remote vpn.example.com 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert client.crt
key client.key
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
tls-crypt ta.key 1
verb 3
```

### Génération d'un profil inline

Pour simplifier le déploiement, on peut inclure les certificats directement dans le fichier `.ovpn` :

```ini
<ca>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</ca>

<cert>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
</key>

<tls-crypt>
...
</tls-crypt>
```

### Connexion en ligne de commande

```bash
sudo openvpn --config client.ovpn
```

## 1.8 Configuration site-à-site avec OpenVPN

### Serveur Site A

```ini
dev tun
proto udp
local 203.0.113.1
port 1194

ifconfig 10.10.10.1 10.10.10.2
route 192.168.2.0 255.255.255.0

ca ca.crt
cert siteA.crt
key siteA.key
dh dh.pem

cipher AES-256-GCM
auth SHA256
persist-key
persist-tun
keepalive 10 60
verb 3
```

### Client Site B

```ini
dev tun
proto udp
remote 203.0.113.1 1194

ifconfig 10.10.10.2 10.10.10.1
route 192.168.1.0 255.255.255.0

ca ca.crt
cert siteB.crt
key siteB.key

remote-cert-tls server
cipher AES-256-GCM
auth SHA256
persist-key
persist-tun
keepalive 10 60
verb 3
```

## 1.9 OpenVPN en mode bridge (TAP)

Le mode TAP est utile pour étendre un segment Ethernet à travers Internet, notamment pour les protocoles nécessitant du broadcast ou des trames de couche 2.

### Serveur

```ini
dev tap0
server-bridge 192.168.1.4 255.255.255.0 192.168.1.100 192.168.1.200
```

### Configuration du bridge Linux

```bash
sudo ip link add br0 type bridge
sudo ip link set eth0 master br0
sudo ip link set tap0 master br0
sudo ip link set br0 up
```

> **Attention :** Le mode TAP est moins performant que TUN car il transporte les en-têtes Ethernet complets.

## 1.10 OpenVPN avec authentification double facteur

OpenVPN peut être couplé à des modules d'authentification (PAM, LDAP, RADIUS) ou à des solutions MFA comme :

- **Duo Security**
- **YubiKey** (via PIV ou challenge-response)
- **Google Authenticator** (TOTP)
- **FreeRADIUS**

Exemple avec PAM :

```ini
plugin /usr/lib/openvpn/openvpn-plugin-auth-pam.so login
reneg-sec 0
```

## 1.11 Surveillance et logging

### Fichiers de log importants

- `/var/log/openvpn/openvpn-status.log` : état des connexions.
- `/var/log/openvpn.log` : logs détaillés.
- `/var/log/syslog` ou `journalctl -u openvpn@server` : logs système.

### Métriques utiles

```bash
# Nombre de clients connectés
cat /var/log/openvpn/openvpn-status.log | grep -c "^CLIENT_LIST"

# Bande passante par client
cat /var/log/openvpn/openvpn-status.log | grep "^CLIENT_LIST"

# Journal en temps réel
sudo journalctl -u openvpn@server -f
```

### Intégration monitoring

- **Prometheus + Grafana** via l'exporter OpenVPN.
- **Zabbix** avec template OpenVPN.
- **Nagios/Icinga** via scripts de vérification.

## 1.12 Commandes de vérification OpenVPN

```bash
# Vérifier le statut du serveur
sudo systemctl status openvpn@server

# Lister les clients connectés
sudo cat /var/log/openvpn/openvpn-status.log

# Tester la connectivité du tunnel
ping 10.8.0.1

# Vérifier les routes
ip route | grep tun0

# Capturer le trafic OpenVPN
tshark -i any -f "udp port 1194"

# Vérifier la version et les options supportées
openvpn --version

# Tester un fichier de configuration
sudo openvpn --config /etc/openvpn/server.conf --verb 9
```

## 1.13 Où utiliser OpenVPN

- VPN d'accès distant pour télétravailleurs.
- VPN site-à-site entre filiales.
- VPN pour routeurs domestiques (pfSense, OPNsense, OpenWrt).
- VPN dans le cloud (AWS, Azure, GCP) pour interconnecter des VPC.
- VPN pour contournement de la censure (obfuscation possible avec Stunnel ou Cloak).
- VPN IoT et réseaux industriels.
- Accès sécurisé aux environnements de développement et de test.

## 1.14 Applications recommandées pour OpenVPN

- **Serveur :** OpenVPN Community, OpenVPN Access Server, pfSense, OPNsense, MikroTik, ASUSWRT, OpenWrt.
- **Client :** OpenVPN Connect, Tunnelblick, Passepartout, OpenVPN for Android, Viscosity.
- **Gestion PKI :** easy-rsa, step-ca, cfssl, Smallstep.

---

# 2. WireGuard

## 2.1 Définition

**WireGuard** est un protocole VPN moderne, simple et performant, conçu pour être plus facile à configurer et plus rapide que IPsec et OpenVPN. Il repose sur une cryptographie de pointe et s'intègre directement dans le noyau Linux. WireGuard fonctionne à la couche 3 (IP) et utilise exclusivement UDP.

Son slogan : *"VPN simple, rapide et moderne"*.

## 2.2 Historique et créateurs

- **Créateur :** Jason A. Donenfeld.
- **Première publication :** 2015.
- **Standardisation :** RFC 7421 (informations générales), RFC 8238, et spécification formelle dans le document du projet.
- **Intégration noyau :** Inclus dans le noyau Linux depuis la version 5.6 (2020).
- **Licence :** GPLv2.
- **Implémentations :** Linux (noyau), Windows, macOS, BSD, Android, iOS.

## 2.3 Architecture de WireGuard

```
┌────────────────────────────────────────────────────────────┐
│                      Applications                          │
├────────────────────────────────────────────────────────────┤
│                      IP (couche 3)                         │
├────────────────────────────────────────────────────────────┤
│                      WireGuard                             │
├────────────────────────────────────────────────────────────┤
│                      UDP (port 51820)                      │
├────────────────────────────────────────────────────────────┤
│                      Internet                              │
└────────────────────────────────────────────────────────────┘
```

WireGuard est un VPN de couche 3 routé. Chaque pair possède une clé publique et une liste d'adresses IP autorisées.

## 2.4 Mécanismes cryptographiques

WireGuard utilise une combinaison soigneusement choisie d'algorithmes modernes :

| Fonction | Algorithme | Justification |
|----------|-----------|---------------|
| Échange de clés | Curve25519 | Courbe elliptique rapide et sûre |
| Chiffrement symétrique | ChaCha20-Poly1305 | AEAD, rapide sans accélération matérielle AES |
| Hachage | BLAKE2s | Fonction de hachage rapide et sûre |
| Dérivation de clés | HKDF | Dérivation robuste selon RFC 5869 |
| Handshake | Noise Protocol Framework (Noise_IK) | Protocole éprouvé pour l'échange de clés |

### Avantages cryptographiques

- **Simplicité :** environ 4 000 lignes de code contre des centaines de milliers pour IPsec/OpenSSL.
- **Auditabilité :** surface d'attaque réduite.
- **Forward secrecy :** clés éphémères pour chaque session.
- **Résistance au replay :** compteurs internes.
- **Pas de négociation d'algorithmes :** configuration fixe, élimine les attaques par downgrade.

### Clés

- Chaque pair a une **clé privée** et une **clé publique**.
- Une **clé pré-partagée optionnelle** peut être ajoutée pour une couche de sécurité supplémentaire (utile contre les ordinateurs quantiques à long terme).
- Les adresses IP autorisées définissent quel trafic est routé vers quel pair.

## 2.5 Installation

### Linux (Debian/Ubuntu)

```bash
sudo apt update
sudo apt install wireguard wireguard-tools
```

### Linux (CentOS/RHEL/Rocky)

```bash
sudo dnf install epel-release -y
sudo dnf install wireguard-tools
```

### Windows / macOS / mobile

- Télécharger sur https://www.wireguard.com/install/.
- Applications officielles disponibles pour iOS et Android.

## 2.6 Configuration du serveur WireGuard

### Génération des clés

```bash
wg genkey | tee privatekey | wg pubkey > publickey
sudo mkdir -p /etc/wireguard
sudo cp privatekey /etc/wireguard/
sudo chmod 600 /etc/wireguard/privatekey
```

### Fichier `/etc/wireguard/wg0.conf`

```ini
[Interface]
PrivateKey = <cle_privee_serveur>
Address = 10.200.200.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
DNS = 1.1.1.1, 8.8.8.8

[Peer]
# Client 1
PublicKey = <cle_publique_client1>
AllowedIPs = 10.200.200.2/32

[Peer]
# Client 2
PublicKey = <cle_publique_client2>
AllowedIPs = 10.200.200.3/32
```

### Démarrage

```bash
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
sudo wg show
```

## 2.7 Configuration du client

### Fichier client `wg0.conf`

```ini
[Interface]
PrivateKey = <cle_privee_client>
Address = 10.200.200.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <cle_publique_serveur>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = vpn.example.com:51820
PersistentKeepalive = 25
```

### Explication des paramètres

- **AllowedIPs** : définit les destinations routées à travers le tunnel. `0.0.0.0/0` signifie tout le trafic.
- **PersistentKeepalive** : envoie des paquets de keepalive toutes les 25 secondes pour maintenir le NAT/firewall ouvert.
- **Endpoint** : adresse publique et port du serveur WireGuard.

## 2.8 Configuration site-à-site avec WireGuard

### Site A

```ini
[Interface]
PrivateKey = <cle_privee_siteA>
Address = 10.10.10.1/30
ListenPort = 51820

[Peer]
PublicKey = <cle_publique_siteB>
AllowedIPs = 192.168.2.0/24
Endpoint = 203.0.113.2:51820
PersistentKeepalive = 25
```

### Site B

```ini
[Interface]
PrivateKey = <cle_privee_siteB>
Address = 10.10.10.2/30
ListenPort = 51820

[Peer]
PublicKey = <cle_publique_siteA>
AllowedIPs = 192.168.1.0/24
Endpoint = 203.0.113.1:51820
PersistentKeepalive = 25
```

### Routage interne

Sur chaque site, ajouter une route statique pointant vers l'interface WireGuard pour le réseau distant, ou activer le forwarding et le NAT si nécessaire.

## 2.9 WireGuard en mode mesh

WireGuard est particulièrement adapté aux architectures **mesh** où chaque nœud est connecté à plusieurs autres. Des outils comme **Tailscale**, **Headscale**, **NetMaker** ou **wg-meshconf** automatisent la gestion des pairs.

### Exemple Headscale (self-hosted Tailscale)

```bash
# Serveur Headscale
headscale namespaces create prod
headscale preauthkeys create -u prod -e 24h

# Client
sudo tailscale up --login-server https://headscale.example.com --authkey <key>
```

## 2.10 Commandes de vérification WireGuard

```bash
# Afficher les interfaces WireGuard
sudo wg show

# Détails d'une interface
sudo wg show wg0

# Statistiques détaillées
sudo wg show wg0 dump

# Journal du noyau
sudo dmesg | grep wireguard

# Test de connectivité
ping 10.200.200.1

# Vérifier les routes
ip route show dev wg0

# Capturer le trafic WireGuard
tshark -i any -f "udp port 51820"
```

## 2.11 Où utiliser WireGuard

- VPN personnel haute performance.
- VPN d'accès distant d'entreprise.
- VPN site-à-site léger.
- Réseaux mesh pour infrastructure cloud et IoT.
- Conteneurs et Kubernetes (Cilium utilise eBPF et peut encapsuler avec WireGuard).
- VPN mobile (batterie préservée grâce à la simplicité).
- Accès sécurisé aux environnements DevOps (CI/CD, bastions).

## 2.12 Applications recommandées pour WireGuard

- **Serveur :** WireGuard natif, PiVPN, Algo VPN, Streisand, Headscale, NetMaker.
- **Client :** applications officielles WireGuard, Tailscale, Mullvad, AzireVPN.
- **Intégration :** Cilium (Kubernetes), OPNsense, pfSense, MikroTik, OpenWrt.

---

# 3. L2TP / L2TP/IPsec

## 3.1 Définition

**L2TP (Layer 2 Tunneling Protocol)** est un protocole de tunnelisation de couche 2 qui combine les fonctionnalités de **PPTP** (Microsoft) et de **L2F** (Cisco). À lui seul, L2TP ne fournit **aucun chiffrement** ; il est donc systématiquement couplé à **IPsec** pour assurer confidentialité, intégrité et authentification. L'ensemble est souvent appelé **L2TP/IPsec**.

L2TP/IPsec est largement supporté nativement par Windows, macOS, iOS, Android et la plupart des routeurs, ce qui en fait un choix pratique pour les environnements hétérogènes.

## 3.2 Historique et créateurs

- **L2F (Layer 2 Forwarding)** : Cisco, 1998, RFC 2341.
- **PPTP (Point-to-Point Tunneling Protocol)** : Microsoft et consortium, RFC 2637.
- **L2TP** : Fusion de L2F et PPTP, RFC 2661 (1999), puis RFC 3931 pour L2TPv3.
- **IPsec** : IETF, RFC 2401 et suivants (puis RFC 4301+).
- **L2TP/IPsec** : De facto standard pour les VPN natifs des systèmes d'exploitation.

## 3.3 Architecture L2TP/IPsec

```
┌────────────────────────────────────────────────────────────────────┐
│                         Applications                               │
├────────────────────────────────────────────────────────────────────┤
│                         IP (couche 3)                              │
├────────────────────────────────────────────────────────────────────┤
│                         PPP (couche 2)                             │
├────────────────────────────────────────────────────────────────────┤
│                         L2TP (UDP 1701)                            │
├────────────────────────────────────────────────────────────────────┤
│                         IPsec / ESP (UDP 500/4500)                 │
├────────────────────────────────────────────────────────────────────┤
│                         UDP / IP                                   │
└────────────────────────────────────────────────────────────────────┘
```

L2TP/IPsec encapsule d'abord les trames PPP dans L2TP, puis chiffre l'ensemble avec IPsec en mode transport (souvent) ou tunnel.

## 3.4 Mécanismes cryptographiques

### IPsec

- **IKEv1** : négociation des associations de sécurité (SA).
- **ESP (Encapsulating Security Payload)** : chiffrement et intégrité.
- **AH (Authentication Header)** : authentification et intégrité sans chiffrement (rarement utilisé seul).

### Algorithmes courants

- **Chiffrement :** AES-256-CBC, AES-128-CBC, 3DES-CBC (obsolète).
- **Intégrité :** HMAC-SHA1, HMAC-SHA256.
- **Groupe DH :** MODP2048 (group 14), MODP1024 (obsolète).

### Authentification L2TP

- **MS-CHAPv2** (le plus courant).
- **CHAP, PAP** (obsolètes).
- **EAP-TLS, EAP-MSCHAPv2** (plus sécurisés, mais moins supportés nativement).

### Ports utilisés

- **UDP 500** : IKE (négociation IPsec).
- **UDP 4500** : NAT-T (NAT Traversal) pour IPsec.
- **UDP 1701** : L2TP encapsulé dans IPsec.

## 3.5 Installation d'un serveur L2TP/IPsec

### Solution classique : strongSwan + xl2tpd

```bash
sudo apt update
sudo apt install strongswan strongswan-pki libcharon-extra-plugins xl2tpd ppp
```

### Configuration strongSwan `/etc/ipsec.conf`

```ini
config setup
    charondebug="ike 2, knl 2, cfg 2"

conn L2TP-PSK
    type=transport
    auto=add
    keyexchange=ikev1
    authby=secret
    left=%defaultroute
    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any
    ike=aes256-sha256-modp2048,3des-sha1-modp1024
    esp=aes256-sha256,3des-sha1
```

### Clé pré-partagée `/etc/ipsec.secrets`

```text
: PSK "votre_cle_partagee_tres_longue"
```

### Configuration xl2tpd `/etc/xl2tpd/xl2tpd.conf`

```ini
[global]
port = 1701
access control = no

[lns default]
ip range = 10.10.10.2-10.10.10.100
local ip = 10.10.10.1
require chap = yes
refuse pap = yes
require authentication = yes
name = l2tpd-server
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
```

### Options PPP `/etc/ppp/options.xl2tpd`

```text
ipcp-accept-local
ipcp-accept-remote
ms-dns 1.1.1.1
ms-dns 8.8.8.8
noccp
noauth
idle 1800
mtu 1280
mru 1280
nodefaultroute
debug
lock
connect-delay 5000
```

### Authentification `/etc/ppp/chap-secrets`

```text
# client  server  secret  IP addresses
alice l2tpd-server "motdepasse_alice" *
bob   l2tpd-server "motdepasse_bob"   *
```

### Démarrage

```bash
sudo systemctl restart strongswan
sudo systemctl restart xl2tpd
sudo systemctl enable strongswan xl2tpd
```

## 3.6 Configuration client L2TP/IPsec

### Windows

1. Paramètres > Réseau et Internet > VPN > Ajouter une connexion VPN.
2. Type de VPN : L2TP/IPsec avec clé pré-partagée.
3. Saisir l'adresse du serveur, le nom d'utilisateur et le mot de passe.
4. Dans les propriétés avancées, définir la clé pré-partagée.

### macOS / iOS

1. Préférences Système > Réseau > Ajouter > VPN L2TP sur IPSec.
2. Configurer le compte, le mot de passe et la clé partagée.

### Android

1. Paramètres > Réseau et Internet > VPN > Ajouter.
2. Type : L2TP/IPsec PSK.

### Linux (NetworkManager)

```bash
nmcli connection add type vpn ifname -- vpn-type l2tp con-name "Mon VPN L2TP"
nmcli connection modify "Mon VPN L2TP" vpn.data "gateway=vpn.example.com, user=alice, password-flags=0, ipsec-psk=votre_cle"
```

## 3.7 Commandes de vérification L2TP/IPsec

```bash
# Vérifier le statut IPsec
sudo ipsec statusall

# Vérifier les associations de sécurité
sudo ip xfrm state
sudo ip xfrm policy

# Logs xl2tpd
sudo tail -f /var/log/syslog | grep xl2tpd

# Vérifier les interfaces PPP
ip link show ppp0

# Tester la connectivité
ping 10.10.10.1
```

## 3.8 Où utiliser L2TP/IPsec

- VPN d'accès distant dans des environnements avec clients natifs variés.
- Réseaux hérités nécessitant la compatibilité Windows/macOS/iOS/Android sans installation d'application tierce.
- Petites entreprises avec des routeurs intégrés (NAS Synology, pfSense, etc.).
- Scénarios où l'installation de clients OpenVPN/WireGuard n'est pas possible.

## 3.9 Applications recommandées pour L2TP/IPsec

- **Serveur :** strongSwan + xl2tpd, libreswan, Windows Server RRAS, MikroTik, Cisco ASA, FortiGate.
- **Client :** clients natifs des OS, Shimo (macOS), VPNC (Linux).

---

# 4. PPTP

## 4.1 Définition

**PPTP (Point-to-Point Tunneling Protocol)** est un protocole VPN de couche 2 développé par un consortium incluant Microsoft, Ascend Communications, 3Com et d'autres. Il encapsule des trames PPP dans des paquets GRE (Generic Routing Encapsulation) et utilise TCP pour la gestion du tunnel.

PPTP est aujourd'hui considéré comme **obsolète et peu sûr** en raison de ses failles cryptographiques.

## 4.2 Historique et créateurs

- **Créateurs :** Microsoft et le PPTP Forum (Ascend, 3Com, etc.).
- **Publication :** 1999, RFC 2637.
- **Intégration :** support natif dans Windows depuis Windows 95.
- **Statut actuel :** obsolète, déconseillé, vulnérable.

## 4.3 Architecture PPTP

```
Client PPTP                          Serveur PPTP
     │                                     │
     │------ TCP 1723 (contrôle) --------->│
     │<----- GRE (données) --------------->│
     │                                     │
```

- **TCP 1723** : canal de contrôle pour établir, maintenir et terminer le tunnel.
- **GRE (IP protocol 47)** : transport des données encapsulées PPP.

## 4.4 Mécanismes cryptographiques

### MPPE (Microsoft Point-to-Point Encryption)

- Algorithme : **RC4**.
- Tailles de clé : 40, 56 ou 128 bits.
- Problèmes : RC4 est cassé, clés dérivées faiblement à partir du mot de passe.

### Authentification

- **MS-CHAPv2** : vulnérable aux attaques par dictionnaire et rainbow tables.
- **EAP-TLS** : plus sûr, mais rarement utilisé avec PPTP.

### Vulnérabilités majeures

- **Déchiffrement de MS-CHAPv2** : CloudCracker et autres services peuvent casser les mots de passe.
- **RC4 cassé** : attaques statistiques et biases.
- **Attaques Man-in-the-Middle** : pas de protection intégrée forte.
- **Absence de forward secrecy**.

## 4.5 Pourquoi éviter PPTP

En 2012, des démonstrations ont montré que les clés MPPE de 128 bits pouvaient être cassées en quelques heures. La NSA est soupçonnée d'exploiter ces faiblesses. PPTP ne doit être utilisé que dans des environnements isolés, temporaires ou à des fins pédagogiques.

## 4.6 Configuration PPTP (à titre informatif)

### Serveur Linux avec pptpd

```bash
sudo apt install pptpd
```

`/etc/pptpd.conf` :

```text
option /etc/ppp/pptpd-options
localip 192.168.100.1
remoteip 192.168.100.100-110
```

`/etc/ppp/pptpd-options` :

```text
name pptpd
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128
ms-dns 8.8.8.8
ms-dns 1.1.1.1
proxyarp
nodefaultroute
lock
nobsdcomp
novj
novjccomp
nologfd
```

`/etc/ppp/chap-secrets` :

```text
alice pptpd motdepasse_alice *
```

Démarrage :

```bash
sudo systemctl restart pptpd
```

## 4.7 Commandes de vérification PPTP

```bash
# Vérifier le service
sudo systemctl status pptpd

# Vérifier les connexions
sudo cat /var/log/syslog | grep pptpd

# Capturer le trafic PPTP
tshark -i any -f "tcp port 1723 or gre"
```

## 4.8 Où utiliser PPTP

- **Jamais en production.**
- Tests pédagogiques.
- Environnements hérités sans alternative et isolés d'Internet.
- Scénarios de rétrocompatibilité temporaires en réseau fermé.

## 4.9 Migration depuis PPTP

1. Identifier les clients et serveurs PPTP.
2. Choisir un remplaçant : OpenVPN, WireGuard, IKEv2/IPsec ou SSTP.
3. Déployer le nouveau serveur en parallèle.
4. Migrer les utilisateurs progressivement.
5. Désactiver PPTP et bloquer GRE/TCP 1723 au pare-feu.
6. Auditer le réseau pour détecter toute réapparition.

---

# 5. GRE

## 5.1 Définition

**GRE (Generic Routing Encapsulation)** est un protocole de tunnelisation défini par la RFC 2784. Il encapsule une grande variété de protocoles réseau à l'intérieur de paquets IP. Contrairement à IPsec ou OpenVPN, GRE **ne fournit ni chiffrement ni authentification natifs**. Il est souvent couplé à IPsec pour sécuriser le tunnel.

GRE est très utilisé dans les réseaux d'opérateurs, les architectures MPLS, les tunnels DMVPN de Cisco, et les overlays SD-WAN.

## 5.2 Historique et créateurs

- **Créateur :** Cisco Systems.
- **Première spécification :** RFC 1701 (1994), puis RFC 2784 (2000) et RFC 2890.
- **Utilisation :** tunnels point-à-point, DMVPN, réseaux privés virtuels d'opérateurs.

## 5.3 Architecture GRE

```
┌────────────────────────────────────────────────────────────────────┐
│  Payload original (IP, MPLS, CLNP, IPX, AppleTalk, etc.)           │
├────────────────────────────────────────────────────────────────────┤
│  En-tête GRE (4+ octets, flags, protocol type)                     │
├────────────────────────────────────────────────────────────────────┤
│  En-tête IP externe (IP protocol 47)                               │
└────────────────────────────────────────────────────────────────────┘
```

GRE encapsule le paquet d'origine dans un nouveau paquet IP utilisant le protocole IP numéro 47.

## 5.4 Mécanismes et options GRE

### En-tête GRE

- **Flags C, K, S** : checksum, key, sequence number.
- **Protocol Type** : identifie le protocole encapsulé (IPv4 = 0x0800, IPv6 = 0x86DD, MPLS = 0x8847).
- **Key** : champ optionnel pour identifier le tunnel (pas de sécurité).
- **Sequence Number** : pour ordonner les paquets et détecter les pertes.

### GRE + IPsec

Pour un tunnel sécurisé, GRE est généralement protégé par IPsec :

```
[Payload] -> [GRE] -> [IPsec ESP] -> [IP externe]
```

Cette combinaison est très utilisée dans les routeurs Cisco (GRE over IPsec).

## 5.5 Configuration GRE sous Linux

### Tunnel point-à-point

```bash
# Sur le routeur A
sudo ip tunnel add gre1 mode gre remote 203.0.113.2 local 203.0.113.1 ttl 255
sudo ip addr add 10.10.10.1/30 dev gre1
sudo ip link set gre1 up

# Sur le routeur B
sudo ip tunnel add gre1 mode gre remote 203.0.113.1 local 203.0.113.2 ttl 255
sudo ip addr add 10.10.10.2/30 dev gre1
sudo ip link set gre1 up
```

### Vérification

```bash
ip tunnel show
grep 10.10.10 /proc/net/dev
ping 10.10.10.2
```

## 5.6 Configuration GRE + IPsec sous Linux

### GRE over IPsec avec strongSwan

```ini
conn gre-tunnel
    type=transport
    left=203.0.113.1
    right=203.0.113.2
    authby=secret
    ike=aes256-sha256-modp2048!
    esp=aes256-sha256!
    auto=start
```

`/etc/ipsec.secrets` :

```text
203.0.113.1 203.0.113.2 : PSK "cle_partagee"
```

## 5.7 Configuration GRE sur Cisco IOS

```cisco
interface Tunnel0
 ip address 10.10.10.1 255.255.255.252
 tunnel source 203.0.113.1
 tunnel destination 203.0.113.2
 tunnel mode gre ip

! GRE over IPsec
crypto isakmp policy 10
 encr aes 256
 hash sha256
 authentication pre-share
crypto isakmp key cle_partagee address 203.0.113.2

crypto ipsec transform-set GRE_TS esp-aes 256 esp-sha-hmac
 mode transport

crypto map GRE_MAP 10 ipsec-isakmp
 set peer 203.0.113.2
 set transform-set GRE_TS
 match address GRE_ACL

access-list GRE_ACL permit gre host 203.0.113.1 host 203.0.113.2
```

## 5.8 Commandes de vérification GRE

```bash
# Afficher les tunnels GRE
ip tunnel show

# Statistiques
ip -s tunnel show gre1

# Capturer le trafic GRE
tshark -i any -f "ip proto 47"

# Vérifier les routes
ip route get 10.10.10.2

# Vérifier MTU
ip link show gre1
```

## 5.9 Où utiliser GRE

- Interconnexion de sites avec protocoles non-IP.
- Tunnels MPLS et overlays d'opérateurs.
- DMVPN Cisco (Dynamic Multipoint VPN).
- Réseaux SD-WAN nécessitant plusieurs overlays.
- Transport de multicast à travers un réseau unicast (PIM over GRE).
- Extension de VLAN ou de réseaux de couche 2.

## 5.10 Applications recommandées pour GRE

- **Routeurs :** Cisco IOS/IOS-XE, Juniper JunOS, MikroTik RouterOS, Huawei VRP.
- **Firewalls :** pfSense, OPNsense (via GRE ou IPsec).
- **Linux :** utilitaires `ip tunnel`, `gre` module.

---

# 6. IKEv2 / IPsec Mobility

## 6.1 Définition

**IKEv2 (Internet Key Exchange version 2)** est le protocole de négociation d'associations de sécurité (SA) pour IPsec, défini dans la RFC 7296. Il est conçu pour être plus simple, plus robuste et plus performant que son prédécesseur IKEv1. IKEv2 est particulièrement adapté à la mobilité grâce à l'extension **MOBIKE** (RFC 4555), qui permet à un client de changer d'adresse IP sans recréer le tunnel.

IKEv2/IPsec est le protocole de VPN natif recommandé sur iOS, macOS et Windows, et est largement supporté par les routeurs et pare-feu d'entreprise.

## 6.2 Historique et créateurs

- **IKEv1 :** RFC 2409, défini dans les années 1990.
- **IKEv2 :** RFC 4306 (2005), puis RFC 7296 (2014).
- **MOBIKE :** RFC 4555 (2006), extension pour la mobilité et le multihoming.
- **Créateurs :** IETF IPsec Working Group.

## 6.3 Architecture IKEv2/IPsec

```
┌────────────────────────────────────────────────────────────────────┐
│                         Applications                               │
├────────────────────────────────────────────────────────────────────┤
│                         IP (couche 3)                              │
├────────────────────────────────────────────────────────────────────┤
│                         IPsec / ESP / AH                           │
├────────────────────────────────────────────────────────────────────┤
│                         IKEv2 (UDP 500/4500)                       │
├────────────────────────────────────────────────────────────────────┤
│                         UDP / IP                                   │
└────────────────────────────────────────────────────────────────────┘
```

### Phases IKEv2

| Phase | But | Messages |
|-------|-----|----------|
| **IKE_SA_INIT** | Négocier les algorithmes, échanger les clés Diffie-Hellman | 2 messages |
| **IKE_AUTH** | Authentifier les pairs et établir la première CHILD_SA | 2 messages |
| **CREATE_CHILD_SA** | Créer des SA supplémentaires ou renégocier | 2 messages |
| **INFORMATIONAL** | Gestion du vivant, suppression de SA, notification | Variable |

## 6.4 Mécanismes cryptographiques

### Algorithmes IKEv2

- **Chiffrement :** AES-256-GCM, AES-128-GCM, AES-256-CBC, ChaCha20-Poly1305.
- **Intégrité :** SHA-256, SHA-384, SHA-512 (HMAC ou intégrité intégrée en mode AEAD).
- **Groupe DH :** ECP256, ECP384, ECP521, Curve25519, MODP2048, MODP4096.
- **PRF :** HMAC-SHA256, HMAC-SHA384.

### Méthodes d'authentification

- **Certificats X.509** (EAP-TLS, RSA signatures).
- **Clé pré-partagée (PSK)**.
- **EAP-MSCHAPv2** (très courant pour l'accès distant).
- **EAP-TLS, EAP-PEAP, EAP-AKA**.

### MOBIKE

MOBIKE permet à un client IKEv2 de basculer entre différentes interfaces réseau (Wi-Fi -> 4G/5G) sans interruption du tunnel. Le serveur conserve l'IKE_SA et met à jour les adresses IP.

## 6.5 Installation d'un serveur IKEv2/IPsec

### strongSwan

```bash
sudo apt update
sudo apt install strongswan strongswan-pki libcharon-extra-plugins
```

### Configuration `/etc/ipsec.conf`

```ini
config setup
    charondebug="ike 2, knl 2, cfg 2, net 2, esp 2, chd 2, dmn 2, mgr 2"

conn ikev2-vpn
    auto=add
    compress=no
    type=tunnel
    keyexchange=ikev2
    fragmentation=yes
    forceencaps=yes
    dpdaction=clear
    dpddelay=300s
    rekey=no

    ike=aes256gcm16-prfsha384-ecp384,aes256-sha256-modp2048!
    esp=aes256gcm16-ecp384!

    left=%any
    leftid=@vpn.example.com
    leftcert=serverCert.pem
    leftsendcert=always
    leftsubnet=0.0.0.0/0,::/0

    right=%any
    rightid=%any
    rightauth=eap-mschapv2
    rightsourceip=10.11.12.0/24
    rightdns=1.1.1.1,8.8.8.8
    rightsendcert=never
    eap_identity=%identity
```

### Clés et certificats

```bash
ipsec pki --gen --type ed25519 --outform pem > caKey.pem
ipsec pki --self --ca --lifetime 3650 --in caKey.pem --dn "C=FR, O=MonOrg, CN=VPN CA" --outform pem > caCert.pem

ipsec pki --gen --type ed25519 --outform pem > serverKey.pem
ipsec pki --pub --in serverKey.pem | ipsec pki --issue --lifetime 730 --cacert caCert.pem --cakey caKey.pem --dn "C=FR, O=MonOrg, CN=vpn.example.com" --san vpn.example.com --flag serverAuth --flag ikeIntermediate --outform pem > serverCert.pem

sudo cp caCert.pem /etc/ipsec.d/cacerts/
sudo cp serverCert.pem /etc/ipsec.d/certs/
sudo cp serverKey.pem /etc/ipsec.d/private/
```

### Secrets `/etc/ipsec.secrets`

```text
: RSA serverKey.pem

alice : EAP "motdepasse_alice"
bob   : EAP "motdepasse_bob"
```

### Démarrage

```bash
sudo systemctl restart strongswan-starter
sudo systemctl enable strongswan-starter
```

## 6.6 Configuration client IKEv2

### Windows 10/11

```powershell
# Ajouter une connexion VPN IKEv2
Add-VpnConnection -Name "IKEv2 VPN" -ServerAddress "vpn.example.com" -TunnelType "Ikev2" -AuthenticationMethod "EAP" -EncryptionLevel "Maximum"

# Activer MOBIKE (par défaut)
Set-VpnConnectionIPsecConfiguration -ConnectionName "IKEv2 VPN" -AuthenticationTransformConstants GCMAES256 -CipherTransformConstants GCMAES256 -EncryptionMethod GCMAES256 -IntegrityCheckMethod SHA384 -DHGroup ECP384 -PfsGroup ECP384 -Force
```

### macOS / iOS

Configurer un profil VPN IKEv2 avec :

- Serveur : `vpn.example.com`
- Authentification distante : certificat serveur
- Authentification utilisateur : nom d'utilisateur + mot de passe
- Groupe local : aucun

### Android

Utiliser l'application **strongSwan VPN Client** disponible sur Google Play.

### Linux

```bash
# Configuration NetworkManager
nmcli connection add type vpn ifname -- vpn-type strongswan con-name "IKEv2 VPN"
nmcli connection modify "IKEv2 VPN" vpn.data "address=vpn.example.com, certificate=/path/to/caCert.pem, usercert=/path/to/clientCert.pem, userkey=/path/to/clientKey.pem, propose-ipsec=enabled"
```

## 6.7 Commandes de vérification IKEv2/IPsec

```bash
# Statut complet
sudo ipsec statusall

# Associations de sécurité
sudo ip xfrm state
sudo ip xfrm policy

# Logs
sudo journalctl -u strongswan-starter -f

# MOBIKE events
grep -i mobike /var/log/syslog

# Ping via le tunnel
ping 10.11.12.1

# Capturer IKEv2
tshark -i any -f "udp port 500 or udp port 4500"
```

## 6.8 Où utiliser IKEv2/IPsec

- VPN d'accès distant pour smartphones et laptops (mobilité MOBIKE).
- VPN d'entreprise avec authentification Active Directory / RADIUS.
- VPN site-à-site haute sécurité.
- Environnements nécessitant une compatibilité native Windows/macOS/iOS/Android.
- Réseaux SD-WAN avec basculement automatique.

## 6.9 Applications recommandées pour IKEv2/IPsec

- **Serveur :** strongSwan, libreswan, Windows Server RRAS, Cisco ASA/IOS, Juniper SRX, FortiGate, Palo Alto, pfSense, OPNsense.
- **Client :** clients natifs Windows/macOS/iOS, strongSwan Android, VPNC, Viscosity.

---

# 7. SSTP

## 7.1 Définition

**SSTP (Secure Socket Tunneling Protocol)** est un protocole VPN développé par Microsoft. Il encapsule le trafic PPP dans un tunnel HTTPS (TLS/SSL) sur le port TCP 443. Cette approche permet à SSTP de traverser presque tous les pare-feu et proxies web, car le trafic ressemble à du HTTPS standard.

SSTP est principalement utilisé dans les environnements Windows et avec les routeurs/pare-feu compatibles.

## 7.2 Historique et créateurs

- **Créateur :** Microsoft Corporation.
- **Publication :** 2007, avec Windows Vista SP1 et Windows Server 2008.
- **Standardisation :** Protocole propriétaire Microsoft, spécification publiée via Microsoft Open Specifications.
- **Support :** Windows (natif), Linux (via third-party), routeurs MikroTik, ASUS, etc.

## 7.3 Architecture SSTP

```
┌────────────────────────────────────────────────────────────────────┐
│                         PPP (IP, IPCP, etc.)                       │
├────────────────────────────────────────────────────────────────────┤
│                         SSTP (encapsulation)                       │
├────────────────────────────────────────────────────────────────────┤
│                         TLS/SSL (TCP 443)                          │
├────────────────────────────────────────────────────────────────────┤
│                         TCP / IP / Internet                        │
└────────────────────────────────────────────────────────────────────┘
```

SSTP utilise une connexion TLS sur TCP 443 pour transporter des trames PPP. Cela le rend très discret face aux pare-feu.

## 7.4 Mécanismes cryptographiques

### TLS/SSL

- SSTP s'appuie sur TLS 1.0 minimum, TLS 1.2/1.3 recommandés.
- Le serveur doit disposer d'un certificat X.509 valide.
- L'authentification du client peut se faire via certificat ou credentials.

### PPP et authentification

- **EAP-TLS** : authentification par certificat client.
- **MS-CHAPv2** : authentification par nom d'utilisateur/mot de passe.
- **PEAP** : tunnel EAP sécurisé par TLS.

### Sécurité

- Contournement facile des pare-feu (TCP 443).
- Chiffrement et authentification assurés par TLS.
- Pas de forward secrecy si TLS utilise RSA key exchange (utiliser ECDHE).

## 7.5 Configuration serveur SSTP

### Windows Server (RRAS)

1. Installer le rôle **Remote Access**.
2. Activer **DirectAccess and VPN (RAS)**.
3. Configurer le serveur VPN avec SSTP.
4. Lier un certificat SSL au serveur.
5. Configurer l'authentification (Active Directory, RADIUS, certificats).
6. Définir les pools d'adresses IP et les routes.

### Linux avec SoftEther VPN

SoftEther VPN est une solution multi-protocole qui supporte SSTP, OpenVPN, L2TP/IPsec et autres.

```bash
# Télécharger et installer SoftEther
wget https://www.softether-download.com/.../softether-vpnserver-v4...tar.gz
tar xzvf softether-vpnserver-*.tar.gz
cd vpnserver
make
sudo ./vpnserver start
```

Configuration via le gestionnaire graphique **VPN Server Manager** ou le CLI **vpncmd**.

### MikroTik RouterOS

```rsc
/interface sstp-server server
set enabled=yes port=443 default-profile=sstp-profile

/ppp profile
add name=sstp-profile local-address=10.10.10.1 remote-address=pool-sstp

/ppp secret
add name=alice password=motdepasse_alice profile=sstp-profile service=sstp
```

## 7.6 Configuration client SSTP

### Windows

1. Paramètres > Réseau et Internet > VPN > Ajouter une connexion VPN.
2. Type de VPN : SSTP (Sélection automatique fonctionne souvent).
3. Saisir l'adresse, le nom d'utilisateur et le mot de passe.

### Linux

Utiliser **NetworkManager-sstp** ou **sstp-client** :

```bash
sudo apt install sstp-client
sudo sstpc --cert-warn --user alice --password motdepasse_alice vpn.example.com usepeerdns require-mschap-v2 noauth noipdefault defaultroute refuse-eap
```

### macOS

macOS ne supporte pas nativement SSTP. Utiliser des clients tiers comme **SSTP Connect** ou **VPN Tracker**.

### Android/iOS

Applications tierces disponibles, mais moins courantes.

## 7.7 Commandes de vérification SSTP

```bash
# Vérifier la connexion TLS
openssl s_client -connect vpn.example.com:443 -servername vpn.example.com

# Logs Windows Server
Get-WinEvent -FilterHashtable @{LogName='System'; ID='20255','20256'}

# Capturer le trafic SSTP
tshark -i any -f "tcp port 443"

# Vérifier les interfaces PPP
ip link show ppp0
```

## 7.8 Où utiliser SSTP

- Environnements Windows dominants.
- Réseaux avec pare-feu très restrictifs n'autorisant que HTTPS (TCP 443).
- Accès distant nécessitant un contournement des proxies web.
- Entreprises utilisant Active Directory et PKI Microsoft.
- Scénarios où UDP est bloqué (alternative à OpenVPN/WireGuard).

## 7.9 Applications recommandées pour SSTP

- **Serveur :** Windows Server RRAS, SoftEther VPN, MikroTik RouterOS.
- **Client :** Windows natif, sstp-client (Linux), SSTP Connect (macOS).

---

# 8. Comparatif technique global

| Critère | OpenVPN | WireGuard | L2TP/IPsec | PPTP | GRE | IKEv2/IPsec | SSTP |
|---------|---------|-----------|------------|------|-----|-------------|------|
| Couche OSI | 4 (UDP/TCP) | 3 (UDP) | 2+3 | 2 | 3 | 3 | 4 (TCP 443) |
| Chiffrement | SSL/TLS, AES-GCM, ChaCha20-Poly1305 | Curve25519, ChaCha20-Poly1305 | IPsec ESP | MPPE-RC4 (cassé) | Aucun | IPsec ESP, AES-GCM | TLS/SSL |
| Authentification | Certificats, TLS-Auth, user/pass | Clés publiques, PSK optionnel | PSK, certificats, EAP | MS-CHAPv2 (cassé) | Aucun | Certificats, EAP-MSCHAPv2, EAP-TLS | TLS, MS-CHAPv2, EAP-TLS |
| Forward secrecy | Oui (ECDHE) | Oui | Oui (DH éphémère) | Non | N/A | Oui | Oui (si ECDHE) |
| Mobilité | Moyenne | Bonne | Moyenne | Faible | N/A | Excellente (MOBIKE) | Bonne |
| Facilité de config | Moyenne | Très facile | Moyenne | Facile | Moyenne | Moyenne à complexe | Moyenne |
| Performance | Bonne | Excellente | Moyenne | Faible à moyenne | Excellente | Bonne à très bonne | Bonne |
| Traversée NAT/firewall | Bonne (UDP/TCP) | Bonne (UDP) | Moyenne (NAT-T) | Mauvaise (GRE bloqué) | Mauvaise sans IPsec | Bonne (NAT-T) | Excellente (TCP 443) |
| Support natif OS | Installation client | Application ou noyau récent | Natif Windows/macOS/iOS/Android | Natif mais obsolète | Routeurs | Natif Windows/macOS/iOS/Android | Windows, routeurs |
| Statut sécurité | Très bon | Excellent | Correct | Obsolète/dangereux | Neutre (tunnel brut) | Excellent | Très bon |

---

# 9. Architectures et topologies

## 9.1 VPN d'accès distant (Remote Access VPN)

```
                    Internet
                       │
        ┌──────────────┼──────────────┐
        │              │              │
    [Client A]    [Client B]    [Client C]
   OpenVPN/       WireGuard/     IKEv2/
   WireGuard      Mobile         SSTP
        │              │              │
        └──────────────┴──────────────┘
                       │
              [Serveur VPN / Firewall]
                       │
              [Réseau interne d'entreprise]
```

## 9.2 VPN site-à-site

```
Site A (192.168.1.0/24)                          Site B (192.168.2.0/24)
        │                                                │
   [Routeur VPN A]  <======== Tunnel VPN ========>  [Routeur VPN B]
   OpenVPN/WireGuard/                               OpenVPN/WireGuard/
   IPsec/GRE+IPsec                                  IPsec/GRE+IPsec
        │                                                │
   [LAN A]                                        [LAN B]
```

## 9.3 Architecture Zero Trust avec VPN

Dans une architecture Zero Trust, le VPN n'est plus un simple périmètre mais un point d'accès dynamique :

- **WireGuard/OpenVPN** : accès segmenté par micro-réseaux.
- **IKEv2/IPsec** : accès mobilité pour utilisateurs nomades.
- **SSTP** : accès de secours derrière des pare-feu restrictifs.
- **Authentification MFA** : couplage avec OIDC, RADIUS, LDAP.
- **Surveillance continue** : SIEM, logs, détection d'anomalies.

## 9.4 Architecture hybride cloud

```
On-Premises                       Internet / WAN                       Cloud (AWS/Azure/GCP)
┌──────────────┐                                              ┌──────────────────────┐
│   LAN        │────[OpenVPN/IPsec/GRE]──────────────────────▶│   VPC / VNet         │
│   WireGuard  │                                              │   WireGuard          │
└──────────────┘                                              └──────────────────────┘
       │                                                              │
       └────────────────────── TLS / mTLS ────────────────────────────┘
```

## 9.5 Architecture mesh

```
        ┌─────────┐
        │  Node A │◄──────────►┌─────────┐
        └────▲────┘            │  Node B │
             │                 └────▲────┘
             │       ┌─────────┐    │
             └──────►│  Node C │◄───┘
                     └─────────┘
```

Chaque nœud possède une clé publique WireGuard et communique directement avec les autres via des tunnels point-à-point.

---

# 10. Commandes de vérification et dépannage

## 10.1 Commandes générales réseau

```bash
# Vérifier les interfaces
ip addr show
ip link show

# Vérifier les routes
ip route show
ip -6 route show

# Vérifier les règles de pare-feu
sudo iptables -L -v -n
sudo iptables -t nat -L -v -n
sudo ip6tables -L -v -n

# Test de connectivité
ping <gateway>
traceroute <destination>
mtr <destination>

# Résolution DNS
nslookup example.com
dig example.com
systemd-resolve --status
```

## 10.2 Commandes OpenVPN

```bash
# Statut du service
sudo systemctl status openvpn@server

# Logs en direct
sudo journalctl -u openvpn@server -f

# Clients connectés
sudo cat /var/log/openvpn/openvpn-status.log

# Vérifier les routes OpenVPN
ip route show dev tun0

# Capture réseau
tshark -i any -f "udp port 1194"

# Test TLS
openssl s_client -connect vpn.example.com:1194 -tls1_3
```

## 10.3 Commandes WireGuard

```bash
# Afficher l'interface
sudo wg show

# Détails complets
sudo wg show wg0 dump

# Logs noyau
sudo dmesg | grep wireguard

# Routes
ip route show dev wg0

# Capture
sudo tcpdump -i any -n -v udp port 51820
```

## 10.4 Commandes L2TP/IPsec

```bash
sudo ipsec statusall
sudo ip xfrm state
sudo ip xfrm policy
sudo tail -f /var/log/syslog | grep -E "xl2tpd|ipsec|ppp"
ip link show ppp0
```

## 10.5 Commandes PPTP

```bash
sudo systemctl status pptpd
sudo cat /var/log/syslog | grep pptpd
tshark -i any -f "tcp port 1723 or gre"
```

## 10.6 Commandes GRE

```bash
ip tunnel show
ip -s tunnel show gre1
tshark -i any -f "ip proto 47"
```

## 10.7 Commandes IKEv2/IPsec

```bash
sudo ipsec statusall
sudo ip xfrm state
sudo ip xfrm policy
sudo journalctl -u strongswan-starter -f
tshark -i any -f "udp port 500 or udp port 4500"
```

## 10.8 Commandes SSTP

```bash
openssl s_client -connect vpn.example.com:443 -servername vpn.example.com
tshark -i any -f "tcp port 443"
Get-WinEvent -FilterHashtable @{LogName='System'; ID='20255','20256'}
```

## 10.9 Dépannage courant

| Problème | Cause probable | Solution |
|----------|---------------|----------|
| Impossible de se connecter | Pare-feu bloquant le port | Ouvrir UDP/TCP correspondant, vérifier iptables/nftables |
| Pas d'accès Internet via le VPN | NAT non configuré | Ajouter MASQUERADE ou SNAT |
| DNS non résolu | DNS non poussés | Configurer `push "dhcp-option DNS"` ou DNS client |
| Tunnel établi mais pas de ping | Routes manquantes | Vérifier `AllowedIPs`, `push route`, routes statiques |
| Déconnexions fréquentes | NAT timeout, keepalive | Activer `PersistentKeepalive`, `keepalive` |
| Lenteurs | MTU mal configuré | Ajuster MTU/MSS, utiliser `mssfix`, `mtu-disc` |
| Authentification échoue | Certificat invalide, PSK erroné | Vérifier dates, CN, clés partagées |
| MOBIKE ne fonctionne pas | Extension non activée | Vérifier support client/serveur IKEv2 |

---

# 11. Applications, clients et implémentations

## 11.1 Logiciels serveur

| Protocole | Logiciel | Plateforme |
|-----------|----------|------------|
| OpenVPN | OpenVPN Community, Access Server | Linux, Windows, BSD |
| WireGuard | WireGuard natif, PiVPN, Algo, Headscale | Linux, cloud, routeurs |
| L2TP/IPsec | strongSwan + xl2tpd, libreswan, Windows RRAS | Linux, Windows |
| PPTP | pptpd, Windows RRAS | Linux, Windows |
| GRE | Linux kernel, Cisco IOS, MikroTik | Routeurs, Linux |
| IKEv2/IPsec | strongSwan, libreswan, Windows RRAS, Cisco ASA | Multi-plateforme |
| SSTP | Windows Server RRAS, SoftEther, MikroTik | Windows, Linux, routeurs |

## 11.2 Clients VPN natifs

| OS | OpenVPN | WireGuard | L2TP/IPsec | PPTP | IKEv2/IPsec | SSTP |
|----|---------|-----------|------------|------|-------------|------|
| Windows | App tierce / officielle | App officielle | Natif | Natif | Natif | Natif |
| macOS | Tunnelblick / Viscosity | App officielle | Natif | Natif | Natif | App tierce |
| Linux | NetworkManager / CLI | CLI / NetworkManager | NetworkManager | pptp-linux | strongSwan | sstp-client |
| iOS | OpenVPN Connect | App officielle | Natif | Natif | Natif | App tierce |
| Android | OpenVPN / officielle | App officielle | Natif | Natif | strongSwan | App tierce |

## 11.3 Routeurs et pare-feu

- **pfSense / OPNsense** : OpenVPN, WireGuard, IPsec, L2TP, PPTP, GRE.
- **MikroTik RouterOS** : OpenVPN, WireGuard, L2TP, PPTP, GRE, SSTP, IPsec/IKEv2.
- **Cisco ASA/IOS/IOS-XE** : IPsec, GRE, DMVPN, IKEv2, AnyConnect (OpenVPN-like).
- **Juniper SRX** : IPsec, GRE, IKEv2.
- **FortiGate / Palo Alto** : IPsec, SSL-VPN, GRE, IKEv2.
- **OpenWrt / ASUSWRT** : OpenVPN, WireGuard, PPTP, L2TP.

## 11.4 Solutions cloud managées

- **AWS Site-to-Site VPN** : IPsec/IKEv2.
- **Azure VPN Gateway** : IPsec/IKEv2, SSTP, OpenVPN (Azure P2S).
- **Google Cloud VPN** : IPsec/IKEv2.
- **Cloudflare One** : WireGuard-based.
- **Tailscale / ZeroTier** : WireGuard/overlay mesh.
- **NordLayer / Perimeter 81** : IKEv2, WireGuard, OpenVPN.

## 11.5 Outils de test et d'audit

- **testssl.sh** : audit TLS/SSL.
- **nmap** : scan de ports et scripts VPN.
- **Wireshark / tshark** : capture et analyse de paquets.
- **iperf3** : test de performance.
- **mtr** : diagnostic réseau avancé.
- **OpenSSL** : vérification de certificats et handshakes.
- **ipsec statusall** : diagnostic IPsec.
- **wg show** : diagnostic WireGuard.

---

# 12. Sécurité, attaques et contre-mesures

## 12.1 Menaces communes aux VPN

- **Attaques Man-in-the-Middle (MITM)** : interception et altération du trafic.
- **Downgrade attacks** : forçage vers des algorithmes faibles.
- **Attaques par déni de service (DoS)** : saturation du serveur VPN.
- **Vol d'identifiants** : phishing, keyloggers, fuites de mots de passe.
- **Exploitation de vulnérabilités** : failles dans les implémentations logicielles.
- **Fuite DNS / WebRTC / IPv6** : exposition de l'adresse IP réelle.
- **Attaques quantiques** : menace future pour les courbes elliptiques classiques.

## 12.2 Attaques spécifiques par protocole

### OpenVPN

- **Attaques TLS downgrade** : utiliser `tls-version-min 1.2`.
- **Vulnérabilités OpenSSL** : maintenir OpenSSL à jour.
- **DoS sur le port 1194** : utiliser `tls-crypt` ou `tls-crypt-v2`.

### WireGuard

- **Silence du protocole** : WireGuard ne répond pas aux paquets non authentifiés, ce qui le rend très résistant au scan.
- **Clés statiques** : rotation régulière des clés.
- **Adresses IP autorisées** : configurer `AllowedIPs` de manière restrictive.

### L2TP/IPsec

- **Attaques sur IKEv1** : préférer IKEv2 lorsque possible.
- **Clés PSK faibles** : utiliser des PSK longs et aléatoires.
- **MS-CHAPv2** : utiliser EAP-TLS si possible.

### PPTP

- **MS-CHAPv2 cassable** : ne plus utiliser PPTP.
- **RC4 faible** : migration obligatoire.

### GRE

- **Pas de chiffrement** : toujours coupler GRE avec IPsec.
- **Spoofing** : filtrer les sources via ACLs.

### IKEv2/IPsec

- **Attaques sur IKE** : utiliser des groupes DH forts (ECP384, Curve25519).
- **EAP-MSCHAPv2** : coupler avec des mots de passe forts ou utiliser EAP-TLS.
- **MOBIKE** : vérifier la cohérence des adresses IP.

### SSTP

- **Certificats non valides** : utiliser des certificats émis par une AC de confiance.
- **TLS 1.0/1.1** : désactiver les versions obsolètes.

## 12.3 Bonnes pratiques de sécurité

1. **Choisir des protocoles modernes** : WireGuard, OpenVPN, IKEv2/IPsec, SSTP.
2. **Désactiver PPTP** et les algorithmes obsolètes.
3. **Utiliser des certificats** plutôt que des mots de passe simples.
4. **Activer la MFA** pour tous les accès VPN.
5. **Segmenter l'accès** : un utilisateur ne doit accéder qu'aux ressources nécessaires.
6. **Journaliser et surveiller** : SIEM, alertes, détection d'anomalies.
7. **Maintenir à jour** : noyaux, logiciels, bibliothèques cryptographiques.
8. **Restreindre les ports** : limiter l'accès au serveur VPN via ACLs.
9. **Configurer correctement le DNS** : éviter les fuites DNS.
10. **Tester régulièrement** : audits, scans de vulnérabilités, tests de pénétration.

## 12.4 Checklist de durcissement

- [ ] Protocole à jour et version stable.
- [ ] Algorithmes forts (AES-GCM, ChaCha20-Poly1305, ECDHE, SHA-256+).
- [ ] Certificats valides et renouvelés.
- [ ] Authentification forte + MFA.
- [ ] Journalisation activée.
- [ ] Pare-feu configuré (limiter les sources si possible).
- [ ] Anti-fuite DNS/IPv6/WebRTC configuré.
- [ ] MTU/MSS ajustés.
- [ ] Surveillance et alerting en place.
- [ ] Plan de révocation des certificats/clés.

---

# 13. Ressources, bibliographie et liens

## 13.1 RFCs essentielles

- RFC 2784 - Generic Routing Encapsulation (GRE)
- RFC 2890 - Key and Sequence Number Extensions to GRE
- RFC 2637 - Point-to-Point Tunneling Protocol (PPTP)
- RFC 2661 - Layer Two Tunneling Protocol "L2TP"
- RFC 3931 - Layer Two Tunneling Protocol - Version 3 (L2TPv3)
- RFC 4301 - Security Architecture for IPsec
- RFC 4302 - IP Authentication Header (AH)
- RFC 4303 - IP Encapsulating Security Payload (ESP)
- RFC 7296 - Internet Key Exchange Protocol Version 2 (IKEv2)
- RFC 4555 - IKEv2 Mobility and Multihoming Protocol (MOBIKE)
- RFC 7421 - Analysis of the 64-bit Boundary in IPv6
- RFC 8446 - TLS 1.3
- RFC 5246 - TLS 1.2
- RFC 8031 - Curve25519 and Curve448 for IKEv2

## 13.2 Documentation OpenVPN

- https://openvpn.net/community-resources/
- https://openvpn.net/community-resources/reference-manual-for-openvpn-2-6/
- https://github.com/OpenVPN/openvpn

## 13.3 Documentation WireGuard

- https://www.wireguard.com/
- https://www.wireguard.com/quickstart/
- https://www.wireguard.com/formal-verification/
- https://git.zx2c4.com/wireguard-tools/about/src/man/wg.8

## 13.4 Documentation strongSwan / IPsec

- https://www.strongswan.org/
- https://wiki.strongswan.org/projects/strongswan/wiki/UserDocumentation
- https://libreswan.org/

## 13.5 Documentation Microsoft

- https://docs.microsoft.com/en-us/windows-server/remote/remote-access/
- https://docs.microsoft.com/en-us/windows-server/remote/remote-access/vpn/always-on-vpn/
- https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-sstp/

## 13.6 Livres recommandés

- *Network Security with OpenSSL* - Viega, Messier, Chandra
- *Bulletproof TLS and PKI* - Ivan Ristić
- *Real-World Cryptography* - David Wong
- *The WireGuard VPN: A Practical Guide* - community editions
- *IPsec VPN Design* - Vijay Bollapragada, Mohamed Khalid, Scott Wainner
- *Building Internet Firewalls* - Elizabeth D. Zwicky, Simon Cooper, D. Brent Chapman
- *VPNs Illustrated: Tunnels, VPNs, and IPsec* - Jon C. Snader
- *Applied Cryptography* - Bruce Schneier
- *Cryptography Engineering* - Ferguson, Schneier, Kohno
- *Hacking Exposed: Network Security Secrets & Solutions* - Stuart McClure, Joel Scambray, George Kurtz

## 13.7 Communautés et liens utiles

- https://www.reddit.com/r/VPN/
- https://www.reddit.com/r/WireGuard/
- https://www.reddit.com/r/OpenVPN/
- https://serverfault.com/questions/tagged/vpn
- https://security.stackexchange.com/
- https://letsencrypt.org/
- https://www.ssllabs.com/ssltest/
- https://testssl.sh/
- https://mullvad.net/fr/help/
- https://www.ivpn.net/knowledgebase/

## 13.8 Auteur et crédits

**Auteur :** hackers_tchad  
**Date de publication :** 2026  
**Contact :** hackers_tchad (réseaux sociaux et communautés techniques)  
**Remerciements :** IETF, OpenVPN Inc., WireGuard project, strongSwan team, Microsoft Open Specifications, MIT, communautés open source.

---

# 14. Fiches de synthèse rapide

## Fiche OpenVPN

- **Objectif :** VPN SSL/TLS flexible et robuste.
- **Mode :** TUN (routé) ou TAP (bridge), UDP/TCP.
- **Port :** 1194 par défaut.
- **Cryptographie :** OpenSSL, AES-GCM, ChaCha20-Poly1305, TLS-Auth/Crypt.
- **Authentification :** Certificats X.509, user/pass, MFA.
- **Outils :** easy-rsa, OpenVPN Connect, Tunnelblick.
- **Cas d'usage :** VPN généraliste, site-à-site, contournement.

## Fiche WireGuard

- **Objectif :** VPN moderne, simple et rapide.
- **Mode :** Couche 3 routé, UDP uniquement.
- **Port :** 51820 par défaut.
- **Cryptographie :** Curve25519, ChaCha20-Poly1305, BLAKE2s, Noise_IK.
- **Authentification :** Clés publiques.
- **Outils :** wg, wg-quick, Tailscale, Headscale.
- **Cas d'usage :** VPN haute performance, IoT, mesh, cloud.

## Fiche L2TP/IPsec

- **Objectif :** VPN natif multiplateforme.
- **Mode :** Couche 2 (PPP) sur IPsec.
- **Ports :** UDP 500, 4500, 1701.
- **Cryptographie :** IPsec ESP (AES, 3DES).
- **Authentification :** PSK, certificats, EAP, MS-CHAPv2.
- **Outils :** strongSwan + xl2tpd, clients natifs.
- **Cas d'usage :** Accès distant simple, compatibilité native.

## Fiche PPTP

- **Objectif :** VPN hérité (obsolète).
- **Mode :** PPP sur GRE.
- **Ports :** TCP 1723 + GRE (IP proto 47).
- **Cryptographie :** MPPE-RC4 (cassé).
- **Authentification :** MS-CHAPv2 (cassé).
- **Recommandation :** Ne pas utiliser en production.

## Fiche GRE

- **Objectif :** Tunnelisation générique de couche 3.
- **Mode :** Tunnel IP dans IP.
- **Protocole :** IP 47.
- **Cryptographie :** Aucune (utiliser GRE over IPsec).
- **Authentification :** Aucune native.
- **Outils :** ip tunnel, routeurs Cisco.
- **Cas d'usage :** Overlays, MPLS, DMVPN, multicast.

## Fiche IKEv2/IPsec

- **Objectif :** VPN IPsec moderne avec mobilité.
- **Mode :** Couche 3, tunnel ou transport.
- **Ports :** UDP 500, 4500.
- **Cryptographie :** AES-GCM, ChaCha20-Poly1305, DH éphémère.
- **Authentification :** Certificats, EAP-MSCHAPv2, EAP-TLS.
- **Outils :** strongSwan, clients natifs Windows/macOS/iOS.
- **Cas d'usage :** Mobilité, entreprise, smartphones.

## Fiche SSTP

- **Objectif :** VPN sur HTTPS pour pare-feux restrictifs.
- **Mode :** PPP sur TLS, TCP 443.
- **Cryptographie :** TLS/SSL.
- **Authentification :** MS-CHAPv2, EAP-TLS, certificats.
- **Outils :** Windows Server RRAS, SoftEther, sstp-client.
- **Cas d'usage :** Environnements Windows, proxies web.

---

# 15. Annexes

## Annexe A : Tableau des ports et protocoles

| Protocole | Protocole transport | Port(s) / Protocole IP | Description |
|-----------|--------------------|------------------------|-------------|
| OpenVPN | UDP/TCP | 1194 (par défaut) | VPN SSL/TLS |
| WireGuard | UDP | 51820 (par défaut) | VPN moderne |
| L2TP | UDP | 1701 (encapsulé dans IPsec) | Tunnel couche 2 |
| IPsec IKEv1/v2 | UDP | 500, 4500 (NAT-T) | Négociation SA |
| IPsec ESP | IP | 50 | Encapsulation sécurisée |
| IPsec AH | IP | 51 | En-tête d'authentification |
| PPTP | TCP + IP | 1723 + GRE (47) | VPN obsolète |
| GRE | IP | 47 | Tunnel générique |
| SSTP | TCP | 443 | VPN sur HTTPS |

## Annexe B : Génération de certificats avec easy-rsa pour OpenVPN

```bash
cd /etc/openvpn/easy-rsa
./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa gen-dh
./easyrsa build-server-full server nopass
./easyrsa build-client-full client1 nopass
./easyrsa gen-crl
```

## Annexe C : Génération de clés WireGuard en masse

```bash
#!/bin/bash
for i in {1..10}; do
    wg genkey | tee client${i}.key | wg pubkey > client${i}.pub
    echo "[Peer]" >> wg0.conf
    echo "PublicKey = $(cat client${i}.pub)" >> wg0.conf
    echo "AllowedIPs = 10.200.200.$((i+1))/32" >> wg0.conf
    echo "" >> wg0.conf
done
```

## Annexe D : Configuration nftables pour serveur VPN

```bash
#!/usr/sbin/nft -f

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        ct state established,related accept
        iif lo accept
        udp dport 1194 accept
        udp dport 51820 accept
        udp dport { 500, 4500 } accept
        tcp dport 443 accept
    }
    chain forward {
        type filter hook forward priority 0; policy drop;
        iifname { "tun0", "wg0" } accept
        ct state established,related accept
    }
    chain nat {
        type nat hook postrouting priority 100;
        oifname "eth0" masquerade
    }
}
```

## Annexe E : Configuration systemd pour démarrage automatique

### OpenVPN

```bash
sudo systemctl enable openvpn@server
sudo systemctl start openvpn@server
```

### WireGuard

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

### strongSwan

```bash
sudo systemctl enable strongswan-starter
sudo systemctl start strongswan-starter
```

## Annexe F : Modèle de monitoring Prometheus pour WireGuard

```yaml
# Exemple de métriques personnalisées via script
- name: wireguard_peers
  help: Number of configured WireGuard peers
  type: gauge
  command: "wg show wg0 peers | wc -l"

- name: wireguard_latest_handshake
  help: Latest handshake per peer in seconds
  type: gauge
  command: "wg show wg0 latest-handshakes"
```

## Annexe G : Glossaire

- **VPN** : Virtual Private Network.
- **TUN/TAP** : Interfaces virtuelles de couche 3 (TUN) ou couche 2 (TAP).
- **PKI** : Public Key Infrastructure.
- **SA** : Security Association (IPsec).
- **ESP** : Encapsulating Security Payload.
- **AH** : Authentication Header.
- **IKE** : Internet Key Exchange.
- **MOBIKE** : Mobility and Multihoming Protocol for IKEv2.
- **PSK** : Pre-Shared Key.
- **MFA** : Multi-Factor Authentication.
- **DoS** : Denial of Service.
- **NAT-T** : NAT Traversal.
- **GRE** : Generic Routing Encapsulation.
- **PPP** : Point-to-Point Protocol.
- **L2TP** : Layer 2 Tunneling Protocol.
- **SSTP** : Secure Socket Tunneling Protocol.
- **AEAD** : Authenticated Encryption with Associated Data.
- **Forward secrecy** : Confidentialité persistante après compromission d'une clé.

## Annexe H : Feuille de route de migration PPTP → moderne

1. Inventaire des clients PPTP.
2. Choix du remplaçant (OpenVPN, WireGuard ou IKEv2).
3. Déploiement du nouveau serveur en parallèle.
4. Tests avec un groupe pilote.
5. Communication et formation des utilisateurs.
6. Migration progressive.
7. Désactivation de PPTP au pare-feu.
8. Audit final et suppression des configurations PPTP.

## Annexe I : Modèles de politique VPN pour entreprise

### Politique d'accès

- Tout accès distant doit passer par un VPN approuvé.
- L'authentification par certificat ou MFA est obligatoire.
- Les accès sont segmentés par rôle.
- Les journaux de connexion sont conservés 12 mois.

### Politique de chiffrement

- Interdiction de PPTP et L2TP sans IPsec.
- AES-256-GCM ou ChaCha20-Poly1305 obligatoires.
- Forward secrecy requise.
- Certificats émis par une AC interne de confiance.

---

**Fin du guide ultra-avancé. Pour toute question, contribution ou correction, contactez hackers_tchad.**
