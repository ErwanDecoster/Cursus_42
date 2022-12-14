## Login VM

user : edecoste

pass : 26Erwan@gmal

user : root

pass : 26R00tsalut

## Clé chiffrage disque

pass : edecosteedecoste


# Commamdes utilisées

## Instalations

Instalation de Sudo

```sh
apt install sudo
```

Instalation de SSH

```sh
apt install ssh
```

Instalation de UFW

```sh
apt install ufw
```

Instalation de libpam-pwquality
(ceci nous servira pour mettre en place une politique de mot de passe)
```sh
apt install libpam-pwquality
```


# Configuration

## SSH
Changement de port du SSH

fichier : etc/ssh/sshd_config
```
port 4242
```
fichier : etc/ssh/ssh_config
```
port 4242
```

Redémarrage du SSH
```bach
/etc/init.d/ssh reload
```
Vérification du status du SSH
```bach
systemctl status sshd
```
Connection au SSH
```bach
ssh edecoste@127.0.0.1 -p 4242
```

## Sudo
Configuration de Sudo
```
sudo visudo
```
Ajout des lignes suivante
```
Defaults		secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
Defaults        badpass_message="Wrong password"
Defaults        requiretty
Defaults        log_input, log_output
Defaults        iolog_dir="/var/log/sudo"
Defaults        logfile="/var/log/sudo/sudo.log"
```

## UFW

Activation du firewall
```
sudo ufw enable 
```

Ouverture du port 4242
```
sudo ufw allow 4242
```

Lancement de UFW au démarrage
```
sudo systemctl enable ufw
```

## Libpam-pwquality

Changement des règles d'expiration
fichier : /etc/login.defs
```
PASS_MAX_DAYS	30
PASS_MIN_DAYS	2
PASS_WARM_AGE	7
```
Changement des règles de complexité
fichier : /etc/pam.d/common-password
```
password	requisite	pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 difok=7 reject_username enforce_for_root
```

Changement du mot de passe de "root"
```
sudo passwd root
```

Changement des regles des utilisateurs deja creer
```
sudo chage -m 2 -M 30 edecoste
sudo chage -m 2 -M 30 root
```


# Création des groupes
Création du groupe user42
```
sudo groupadd user42 
```

Ajout de l'utilisateur "edecoste" au groupe "user42"
```
sudo adduser edecoste user42 
```


# Script 

Création du script

fichier : /bin/monitoring.sh 
```sh
#!/bin/bash
wall $'#Architecture:' `uname -a` $'\n'\
$'#CPU physical:' `lscpu | grep "Cœur(s)" | awk '{print $4}'` $'\n'\
$'#vCPU:' `lscpu | grep "Thread(s)" | awk '{print $4}'` $'\n'\
$'#Memory Usage:' `free -m | grep Mem: | awk '{printf "%d/%dMB (%.2f%%)", $3, $2, $3/$2*100}'` $'\n'\
$'#Disk Usage:' `df -h --total | grep total | awk '{print $3 "/" $2 "  (" $5 ")"}'` $'\n'\
$'#CPU load:' `top -bn1 | sed -n 3p | awk '{print $2 "%"}'` $'\n'\
$'#Last boot:' `who | awk '{print $3 " " $4}'` $'\n'\
$'#LVM use:' `lsblk | grep lvm | wc -l | awk '{if ($1) print "yes"; else print "no"}'` $'\n'\
$'#Connexions TCP:' `ss -tunlp | grep LISTEN | wc -l` $'ESTABLISHED\n'\
$'#User log:' `who | wc -l` $'\n'\
$'#Network:' `ip a | grep -v 127 | grep "inet " | awk '{print $2}' | awk -F "/" '{print "IP " $1}' ` `ip a | grep ether | awk '{printf "("$2")"}'`  $'\n'\
$'#Sudo:' `journalctl -q _COMM=sudo | grep COMMAND | wc -l` $'cmd' $'\n'\
```

Création d'une tâche Cron
```
sudo crontab -e
```
Ajout de :
```
*/10 * * * * /bin/monitoring.sh
```
Rendre notre script executable par root
```
sudo chown root:root /bin/monitoring.sh
sudo chmod 700 /bin/monitoring.sh
```

Vérifier le status de Cron
```
sudo systemctl status cron.service 
```
Lancer Cron
```
sudo systemctl enable cron.service
```

# Commande utile

Changer le mot de passe
```sh
sudo passwd USERNAME
```

Voir le nom de notre machine
```sh
hostname
```

Changer le nom de notre machine
```sh
sudo hostnamectl set-hostname HOSTNAME
```
et 

fichier : /etc/hostname
```
HOSTNAME
```

Voir les informations de mot de passe d'un compte
```sh
sudo chage USERNAME -l
```
Changer les informations de mot de passe d'un compte
```sh
sudo chage USERNAME
```

Ajouter un utilisateur
```sh
sudo adduser USERNAME
```
Supprimer un utilisateur
```sh
sudo deluser USERNAME
```

Creer un groupe
```sh
sudo groupadd GROUP_NAME
```
Ajouter un utilisateur a un groupe
```sh
sudo adduser USERNAME GROUP_NAME
```
Supprimer un utilisateur d'un groupe
```sh
sudo deluser USERNAME GROUP_NAME
```
Voir les groupes
```sh
groups
```
Voir tous les groupes
```sh
getent group
```

Ajouter unw règle UFW
```
sudo ufw allow PORT_NUMBER
```
Voir les règles UFW
```
sudo ufw status
```
Supprimer une règle UFW
```
sudo ufw delete allow PORT_NUMBER
```
