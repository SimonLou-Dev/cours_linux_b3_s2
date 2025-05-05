# Configuration des tâches planifiées sous Linux

Ce document présente la mise en place de diverses tâches planifiées sur un système Linux, utilisant différentes méthodes de planification (`cron`, `systemd` et `at`).

## 📋 Prérequis

- Système Linux avec `cron`, `systemd` et `at` installés
- Droits administrateur (root ou sudo) pour créer les scripts et modifier les fichiers de configuration

## 🚀 Tâches configurées

### 1️⃣ Tâche aléatoire au démarrage

Exécute une tâche qui démarre après un délai aléatoire (entre 0 et 30 minutes) après le démarrage du système.

```bash
# Création du script
sudo tee /usr/local/bin/random_task.sh << 'EOF'
#!/bin/bash
sleep $(( RANDOM % 1800 ))
/usr/bin/logger "tâche1 ok"
EOF
sudo chmod +x /usr/local/bin/random_task.sh

# Configuration dans cron
sudo bash -c "echo '@reboot root /usr/local/bin/random_task.sh' >> /etc/crontab"
```

### 2️⃣ Vérification hebdomadaire des mises à jour

Vérifie les mises à jour disponibles tous les dimanches à 12h00 et enregistre le résultat dans un fichier de log daté.

```bash
# Création du script
sudo tee /usr/local/bin/check_updates.sh << 'EOF'
#!/bin/bash
DATE=$(/bin/date +%Y-%m-%d)
# Mise à jour de la base de paquets
/usr/bin/apt-get update >/dev/null 2>&1
# Simulation d'upgrade pour lister les MAJ disponibles
/usr/bin/apt-get -s upgrade > /var/log/update-$DATE.log 2>&1
EOF
sudo chmod +x /usr/local/bin/check_updates.sh

# Configuration dans cron
sudo bash -c "echo '0 12 * * 0 root /usr/local/bin/check_updates.sh' >> /etc/crontab"
```

### 3️⃣ Tâche ponctuelle avec `at`

Crée un fichier horodaté dans le répertoire `/opt/mytask` à un moment précis (exemple : 1 minute après l'exécution de la commande).

```bash
# Préparation du répertoire
sudo mkdir -p /opt/mytask
sudo chown theo:theo /opt/mytask

# Planification de la tâche unique
echo "mkdir -p /opt/mytask; touch /opt/mytask/one_time_$(/bin/date +%Y%m%d_%H%M%S).txt" | at now + 1 minute
```

### 4️⃣ Tâche toutes les secondes avec systemd

Affiche le message "computer started" toutes les secondes grâce à systemd.

```bash
# Création du service
sudo tee /etc/systemd/system/computer-started.service << 'EOF'
[Unit]
Description=Echo loop

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo "computer started"'
EOF

# Création du timer
sudo tee /etc/systemd/system/computer-started.timer << 'EOF'
[Unit]
Description=Loop computer-started.service

[Timer]
OnUnitActiveSec=1
AccuracySec=1us

[Install]
WantedBy=timers.target
EOF

# Activation et démarrage
sudo systemctl daemon-reload
sudo systemctl enable --now computer-started.timer
```

### 5️⃣ Journalisation quotidienne de la date

Enregistre la date du jour dans le journal système tous les jours à minuit.

```bash
# Configuration dans cron
sudo bash -c "echo '@daily root /usr/bin/logger \"\$(/bin/date +%d/%m/%Y)\"' >> /etc/crontab"
```

## 🔍 Vérification de ce que nous avons fait

- **Tâches cron** : `sudo cat /etc/crontab`
- **Tâches at** : `atq`
- **Timers systemd** : `systemctl list-timers`
- **Journal système** : `journalctl -f`