# LINUX - LPIC 102 - semestre 2

## 🎓 ESGI Reims - Semestre 2

### Théo Kazak, Rémi Renault, Simon Bidet, Lucas Bonnaire

---

## 🧾 Objectifs pédagogiques du TP

### 🖥️ Serveur rsyslog

- Créer un serveur Rsyslog pour recevoir les logs via **RELP**.
- Le client et le serveur doivent être correctement configurés pour l’envoi et la réception des logs.
- ⚠️ Le chiffrement TLS est **optionnel**. Sa mise en place fonctionnelle rapporte **+3 points**.

---

### 🕒 Tâches planifiées (Cron)

- Tâche aléatoire toutes les **30 minutes max**, exécutant :
  `logger tâche1 ok`
- Tâche **le dimanche à 12:00** : vérifie les mises à jour disponibles, et écrit le résultat dans un fichier de log nommé `update-<date>` dans un emplacement conforme au FHS.
- Tâche ponctuelle : exécute une commande de votre choix **et crée un fichier dans `/opt/mytask`**.
- Tâche toutes les **secondes** :
  `echo "computer started"`
- Tâche **quotidienne** (tous les jours, toutes heures) : exécute un script Bash qui envoie dans les logs système la date du jour au format `DD/MM/YYYY`.

---

### 🛠️ Automatisation avec Bash

Projet sous forme de script `user-and-ui` placé dans `/usr/local/sbin` :

- Shell : Bash uniquement, avec shebang correct.
- Vérification d’exécution **en tant que root**.
- Création de 10 utilisateurs `utilisateur1` à `utilisateur10` dans `/home`, avec `bash` comme shell.
- Vérification des utilisateurs via `/etc/passwd`, avec messages :
  - `"Ok pour username."` si tout est bon.
  - `"Utilisateur $user déjà créé"` si relancé.
- Interface utilisateur : choix entre **KDE (1)**, **XFCE (2)**, **MATE (3)** ou **quitter (4)** via boucle interactive.
  - Si erreur, la boucle recommence.
  - En cas d’échec d’installation, relancer le script.
  - Si l’environnement choisi est **déjà installé**, informer l’utilisateur et quitter proprement.

---

## ⚙️ Fonctionnement du Vagrantfile

Ce projet déploie **2 VM Rocky Linux 9** :

| Nom             | IP              | CPU | RAM     | Disque |
|----------------|------------------|-----|---------|--------|
| rsyslog-server | 192.168.56.10    | 1   | 1024 Mo | 10 Go  |
| rsyslog-client | 192.168.56.11    | 1   | 1024 Mo | 10 Go  |

### 📦 Provisioning automatique

Chaque machine :

- Met à jour les paquets (`yum`).
- Installe : `unzip`, `curl`, `vim`, `git`, `python3-pip`, etc.
- Modifie `sshd_config` pour activer ChallengeResponseAuthentication.
- Redémarre le service SSH.
- Ajoute les IPs et hostnames des VMs dans `/etc/hosts`.

### 📁 Dossier partagé

Le dossier courant est **monté automatiquement dans `/vagrant`** sur toutes les VMs.

### 🌐 Réseau

Chaque VM est connectée en **`public_network`**, avec une IP fixe.

---

## 📚 Documentation

- 👉 [Documentation officielle de Vagrant](https://developer.hashicorp.com/vagrant)

---

## 📁 Structure du projet

| Dossier         | Description                                                           | Lien                                  |
|----------------|-----------------------------------------------------------------------|---------------------------------------|
| `./rsyslog/`   | TP configuration `rsyslog`, avec ou sans TLS.                         | [`./rsyslog/readme.md`](./rsyslog) |
| `./cron/`      | TP sur les tâches planifiées (tâches aléatoires, planifiées, etc.)    | [`./cron/readme.md`](./cron)       |
| `./auto_bash/` | Script Bash `user-and-ui` pour automatiser utilisateurs & UI.         | [`./auto_bash/readme.md`](./auto_bash) |

---

## ✅ Prérequis

- [VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://developer.hashicorp.com/vagrant/)
- Système compatible : Linux, macOS, ou Windows

---

---

## 🚀 Lancement

```bash
vagrant up
```

## 🛜 Connexion

```bash
vagrant ssh rsyslog-client
```

```bash
vagrant ssh rsyslog-server
```

## 🛑 Arret

```bash
vagrant destroy
```
