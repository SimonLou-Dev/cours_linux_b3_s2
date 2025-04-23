# LINUX - LPIC 102 - semestre 2
## Mise en place de Rsyslog avec RELP et TLS (optionnel)

# Création des boxs avec Vagrant

Depuis la racine
```bash
vagrant up
```
# Coté serveur

## Connexion au serveur 

```bash
vagrant ssh rsyslog-server
```

## Installation

Installation des paquets
```bash
sudo dnf install rsyslog rsyslog-relp openssl
```

```text
Upgraded:
  openssl-1:3.2.2-6.el9_5.1.x86_64                                                                     openssl-libs-1:3.2.2-6.el9_5.1.x86_64
Installed:
  librelp-1.10.0-5.el9.x86_64                                                                         rsyslog-relp-8.2310.0-4.el9.x86_64

Complete!
```

Démarage du service rsyslog
```bash
sudo systemctl enable --now rsyslog
```

## Ajout d'un serveur ntp

Installation
```bash
sudo dnf install chrony
sudo systemctl enable chronyd --now
```

Set de la timezone 
```bash
sudo timedatectl set-timezone Europe/Paris
```

Redémarage & force le tracking & status
```bash
sudo systemctl restart chronyd
sudo chronyc -a makestep
chronyc tracking
```

```text
Reference ID    : D9B689D0 (passion.bitschine.fr)
Stratum         : 3
Ref time (UTC)  : Wed Apr 23 15:59:31 2025
System time     : 0.000000000 seconds slow of NTP time
Last offset     : +0.000031641 seconds
RMS offset      : 0.000031641 seconds
Frequency       : 2.919 ppm slow
Residual freq   : +50.479 ppm
Skew            : 0.517 ppm
Root delay      : 0.028731361 seconds
Root dispersion : 0.002061127 seconds
Update interval : 2.0 seconds
Leap status     : Normal
```

## Mise en place des certificats TLS

```bash
sudo mkdir -p /etc/rsyslog.d/tls
cd /etc/rsyslog.d/tls
```

Création de l'autorité de certification (CA)

```bash
sudo openssl genrsa -out ca.key 2048
sudo openssl req -x509 -new -nodes -key ca.key -out ca.pem -days 365 -subj "/CN=KevinRsyslogCA"
```
Contenue de `/etc/rsyslog.d/tls`
```text
ca.key
ca.pem
```

Création du certificat

```bash
sudo openssl genrsa -out server-key.pem 2048
sudo openssl req -new -key server-key.pem -out server.csr -subj "/CN=rsyslog-server"
sudo openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key -CAcreateserial -out server-cert.pem -days 365
```

Retour de la dernière commande

```text
Certificate request self-signature ok
subject=CN=rsyslog-server
```

Contenue de `/etc/rsyslog.d/tls`
```text
ca.key
ca.pem
ca.srl
server-cert.pem
server.csr
server-key.pem
```

Configuration des permissions
```bash
sudo chmod 644 ca.pem server-cert.pem client-cert.pem
sudo chmod 600 client-key.pem server-key.pem
```

## Configuration du serveur rsyslog 

dans `/etc/rsyslog.d/10-relp-server.conf`
```conf
module(load="imrelp")

input(type="imrelp"
      port="20515"
      tls="on"
      tls.caCert="/etc/rsyslog.d/tls/ca.pem"
      tls.myCert="/etc/rsyslog.d/tls/server-cert.pem"
      tls.myPrivKey="/etc/rsyslog.d/tls/server-key.pem"
      tls.authMode="name"
      tls.permittedPeer=["rsyslog-client"])
```

## Sécurité avec firewalld et SELinux

Changement du contexte SELinux
```bash
sudo semanage fcontext -a -t etc_t \"/etc/rsyslog.d/tls(/.*)?\"
sudo restorecon -Rv /etc/rsyslog.d/tls
```


Ouverture des port firewalld

```bash
sudo firewall-cmd --permanent --add-port=20515/tcp
sudo firewall-cmd --reload
```

Ouverture du port via SELINUX

```bash
sudo semanage port -a -t syslogd_port_t -p tcp 20515

```

## Fin de l'installation

Redémarage du serveur

```bash
sudo systemctl restart rsyslog
```

Vérification du service

```bash
sudo systemctl status rsyslog
```
```text
● rsyslog.service - System Logging Service
     Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-04-23 15:28:09 UTC; 5s ago
       Docs: man:rsyslogd(8)
             https://www.rsyslog.com/doc/
   Main PID: 29228 (rsyslogd)
      Tasks: 4 (limit: 5816)
     Memory: 2.1M
        CPU: 29ms
     CGroup: /system.slice/rsyslog.service
             └─29228 /usr/sbin/rsyslogd -n

Apr 23 15:28:09 rsyslog-server systemd[1]: Starting System Logging Service...
Apr 23 15:28:09 rsyslog-server systemd[1]: Started System Logging Service.
Apr 23 15:28:09 rsyslog-server rsyslogd[29228]: [origin software="rsyslogd" swVersion="8.2310.0-4.el9" x-pid="29228" x-info="https://www.rsyslog.com"] start
Apr 23 15:28:09 rsyslog-server rsyslogd[29228]: imjournal: journal files changed, reloading...  [v8.2310.0-4.el9 try https://www.rsyslog.com/e/0 ]
```
# Installation coté client


## Connection au serveur
```bash
vagrant ssh rsyslog-client
```
## Installation

Installation des paquets
```bash
sudo dnf install rsyslog rsyslog-relp openssl
```

```text
Upgraded:
  openssl-1:3.2.2-6.el9_5.1.x86_64                                                                     openssl-libs-1:3.2.2-6.el9_5.1.x86_64
Installed:
  librelp-1.10.0-5.el9.x86_64                                                                         rsyslog-relp-8.2310.0-4.el9.x86_64

Complete!
```

Démarage du service rsyslog
```bash
sudo systemctl enable --now rsyslog
```

## Ajout d'un serveur ntp

Installation
```bash
sudo dnf install chrony
sudo systemctl enable chronyd --now
```

Set de la timezone 
```bash
sudo timedatectl set-timezone Europe/Paris
```

Redémarage & force le tracking & status
```bash
sudo systemctl restart chronyd
sudo chronyc -a makestep
chronyc tracking
```

```text
Reference ID    : 4EC24E31 (moz75-1-78-194-78-49.fbxo.proxad.net)
Stratum         : 3
Ref time (UTC)  : Wed Apr 23 15:59:32 2025
System time     : 0.000000000 seconds slow of NTP time
Last offset     : +0.001645083 seconds
RMS offset      : 0.001645083 seconds
Frequency       : 31762.979 ppm fast
Residual freq   : -27.348 ppm
Skew            : 0.449 ppm
Root delay      : 0.017764248 seconds
Root dispersion : 0.030412035 seconds
Update interval : 2.0 seconds
Leap status     : Normal
```

## Génération des certificats

1. Création des répertoires
```bash
sudo mkdir -p /etc/rsyslog.d/tls
cd /etc/rsyslog.d/tls
```

2. Génération de clé et du CSR

```bash
sudo openssl genrsa -out client-key.pem 2048
sudo openssl req -new -key client-key.pem -out client.csr -subj "/CN=rsyslog-client"
```

3. copier le `client.csr` sur `rsyslog-server` pour le signer

```bash
scp client.csr rsyslog-server:/home/vagrant
```

4. Sur le serveur signer le client

```bash
sudo mv ~/client.csr /etc/rsyslog.d/tls/
sudo openssl x509 -req -in client.csr -CA ca.pem -CAkey ca.key -CAcreateserial -out client-cert.pem -days 365
```

5. Depuis le serveur renvoyer le `ca.pem` et `client-cert.pem` sur `rsyslog-client`

```bash
scp ca.pem client-cert.pem rsyslog-client:/home/vagrant
```

6. Sur le client déplacer le `ca.pem` et `client-cert.pem`  

```bash
sudo mv ~/ca.pem /etc/rsyslog.d/tls/
sudo mv ~/client-cert.pem /etc/rsyslog.d/tls/
```

## Configuration du serveur rsyslog

dans `/etc/rsyslog.d/10-relp-server.conf`
```conf
module(load="omrelp")

*.* action(
  type="omrelp"
  target="rsyslog-server"
  port="20515"
  tls="on"
  tls.caCert="/etc/rsyslog.d/tls/ca.pem"
  tls.myCert="/etc/rsyslog.d/tls/client-cert.pem"
  tls.myPrivKey="/etc/rsyslog.d/tls/client-key.pem"
  tls.authmode="name"
  tls.permittedPeer=["rsyslog-server"]
)

```

Configuration des permissions
```bash
sudo chmod 644 ca.pem client-cert.pem
sudo chmod 600 client-key.pem
```



## Gestion de firewalld et SELinux

Changement du context SELinux

```bash
sudo semanage fcontext -a -t etc_t \"/etc/rsyslog.d/tls(/.*)?\"
sudo restorecon -Rv /etc/rsyslog.d/tls
```

Ouverture des port firewalld

```bash
sudo firewall-cmd --permanent --add-port=20515/tcp
sudo firewall-cmd --reload
```

Ouverture du port via SELINUX

```bash
sudo semanage port -a -t syslogd_port_t -p tcp 20515
```

Redémarage du serveur

```bash
sudo systemctl restart rsyslog
```

Vérification du service

```bash
sudo systemctl status rsyslog
```
```text
● rsyslog.service - System Logging Service
     Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-04-23 15:28:09 UTC; 5s ago
       Docs: man:rsyslogd(8)
             https://www.rsyslog.com/doc/
   Main PID: 29228 (rsyslogd)
      Tasks: 4 (limit: 5816)
     Memory: 2.1M
        CPU: 29ms
     CGroup: /system.slice/rsyslog.service
             └─29228 /usr/sbin/rsyslogd -n

Apr 23 15:28:09 rsyslog-server systemd[1]: Starting System Logging Service...
Apr 23 15:28:09 rsyslog-server systemd[1]: Started System Logging Service.
Apr 23 15:28:09 rsyslog-server rsyslogd[29228]: [origin software="rsyslogd" swVersion="8.2310.0-4.el9" x-pid="29228" x-info="https://www.rsyslog.com"] start
Apr 23 15:28:09 rsyslog-server rsyslogd[29228]: imjournal: journal files changed, reloading...  [v8.2310.0-4.el9 try https://www.rsyslog.com/e/0 ]
```

# Régler les erreur d'horloges

```bash
echo tsc | sudo tee /sys/devices/system/clocksource/clocksource0/current_clocksource
sudo grubby --update-kernel=ALL --args="clocksource=tsc"
sudo reboot
```


# Test des logs

Sur le serveur 

```bash
sudo tail -f /var/log/messages
```

Sur le client

```bash
logger -p local0.info "Test message from client"
```

Sur le serveur doit apparaitre 
```log
Apr 23 18:05:17 rsyslog-client vagrant[870]: Test message from client
```