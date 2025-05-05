README

Ce document décrit la mise en place de diverses tâches planifiées sur un système Linux conformément aux spécifications.

Prérequis

Système Linux avec cron, systemd et at installés.

Droits root ou sudo pour créer des scripts et modifier /etc/crontab ou les fichiers systemd.

Installation et configuration

1. Tâche aléatoire (intervalle de 30 minutes)

Créer le script /usr/local/bin/random_task.sh :

sudo tee /usr/local/bin/random_task.sh << 'EOF'
#!/bin/bash
# Attendre aléatoirement entre 0 et 1800 s (30 min)
sleep $(( RANDOM % 1800 ))
/usr/bin/logger "tâche1 ok"
EOF
sudo chmod +x /usr/local/bin/random_task.sh

Ajouter dans cron via /etc/crontab pour exécution au démarrage :

sudo bash -c "echo '@reboot root /usr/local/bin/random_task.sh' >> /etc/crontab"

2. Vérification des mises à jour chaque dimanche à 12:00

Créer le script /usr/local/bin/check_updates.sh :

sudo tee /usr/local/bin/check_updates.sh << 'EOF'
#!/bin/bash
DATE=$(/bin/date +%Y-%m-%d)
# Mise à jour de la base de paquets
/usr/bin/apt-get update >/dev/null 2>&1
# Simulation d'upgrade pour lister les MAJ dispo
/usr/bin/apt-get -s upgrade > /var/log/update-$DATE.log 2>&1
EOF
sudo chmod +x /usr/local/bin/check_updates.sh

Ajouter dans /etc/crontab :

sudo bash -c "echo '0 12 * * 0 root /usr/local/bin/check_updates.sh' >> /etc/crontab"

3. Tâche ponctuelle créant un fichier dans /opt/mytask

Préparer le répertoire :

sudo mkdir -p /opt/mytask
sudo chown $(whoami):$(whoami) /opt/mytask

Planifier l’exécution unique (ex. +1 minute) :

echo "mkdir -p /opt/mytask; touch /opt/mytask/one_time_$(/bin/date +%Y%m%d_%H%M%S).txt" | at now + 1 minute

4. Exécution toutes les secondes ("computer started")

Créer le service /etc/systemd/system/computer-started.service :

[Unit]
Description=Echo computer started

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo "computer started"'

Créer le timer /etc/systemd/system/computer-started.timer :

[Unit]
Description=Run computer-started.service every second

[Timer]
OnUnitActiveSec=1
AccuracySec=1us

[Install]
WantedBy=timers.target

Activer et démarrer :

sudo systemctl daemon-reload
sudo systemctl enable --now computer-started.timer

5. Logger la date chaque jour dans le journal système

Ajouter dans /etc/crontab avec la macro @daily :

sudo bash -c "echo '@daily root /usr/bin/logger "\$(/bin/date +%d/%m/%Y)"' >> /etc/crontab"

Toutes les tâches sont maintenant configurées selon les bonnes pratiques FHS et utilisent les outils cron, at et systemd-timer pour répondre aux besoins énoncés.