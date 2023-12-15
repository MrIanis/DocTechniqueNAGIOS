# Installation et Configuration de Nagios 4 sur Debian 11/Debian 10

## Introduction
Ce guide explique comment installer et configurer Nagios 4 sur un système Debian 11/Debian 10. Nagios est un outil puissant de surveillance qui permet de détecter et de résoudre les problèmes informatiques avant qu'ils n'impactent les opérations critiques.

## Étape 1 : Mise à jour du système
Mettez à jour votre système pour obtenir les dernières mises à jour logicielles :

```bash
sudo apt update && sudo apt -y full-upgrade
[ -f /var/run/reboot-required ] && sudo reboot -f
 ```
 
## Étape 2 : Installation des packages requis
Installez les outils nécessaires pour que Nagios fonctionne correctement :

```bash
sudo apt install vim wget curl build-essential unzip openssl libssl-dev apache2 php libapache2-mod-php php-gd libgd-dev
```

## Étape 3 : Récupération et extraction des fichiers Nagios
Téléchargez et extrayez les fichiers de Nagios depuis le site officiel :

```bash
cd ~
NAGIOS_VER=$(curl -s https://api.github.com/repos/NagiosEnterprises/nagioscore/releases/latest|grep tag_name | cut -d '"' -f 4)
wget https://github.com/NagiosEnterprises/nagioscore/releases/download/$NAGIOS_VER/$NAGIOS_VER.tar.gz
tar xvzf $NAGIOS_VER.tar.gz
```

## Étape 4 : Compilation des fichiers extraits
Compilez les fichiers de Nagios que vous avez extraits :

```bash
cd $NAGIOS_VER
./configure --with-httpd-conf=/etc/apache2/sites-enabled
```

## Étape 5 : Création d'un utilisateur et d'un groupe
Créez un utilisateur et un groupe spécifiques pour Nagios et ajoutez l'utilisateur www-data à ce groupe :

```bash
sudo make install-groups-users
sudo usermod -a -G nagios www-data
sudo make all
sudo make install
```

## Étapes 6 à 8 : Installation du démon, ajout d'un mode de commande, installation des fichiers de configuration
Installez le démon Nagios, ajoutez un mode de commande, et installez les fichiers de configuration nécessaires :

```bash
sudo make install-daemoninit
sudo make install-commandmode
sudo make install-config
```

## Étapes 9 et 10 : Configuration du serveur Web Apache et authentification Nagios Apache
Configurez Apache pour servir les pages Web Nagios et mettez en place l'authentification pour Nagios :

```bash
sudo make install-webconf
sudo a2enmod rewrite cgi
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
sudo chown www-data:www-data /usr/local/nagios/etc/htpasswd.users
sudo chmod 640 /usr/local/nagios/etc/htpasswd.users
```

## Étape 11 : Installation des plugins Nagios essentiels
Téléchargez, extrayez, et installez les plugins Nagios essentiels :

```bash
cd ~
wget https://github.com/nagios-plugins/nagios-plugins/releases/download/2.4.8/nagios-plugins-2.4.8.tar.gz
tar xvf nagios-plugins-$VER.tar.gz
cd nagios-plugins-$VER
./configure --with-nagios-user=nagios --with-nagios-group=nagios
sudo make
sudo make install
```

## Étape 12 : Autorisation des ports sur le pare-feu et démarrage de Nagios
Autorisez les ports nécessaires sur le pare-feu et démarrez les services Nagios et Apache :

```bash
sudo ufw allow 80
sudo ufw reload
sudo systemctl restart apache2
sudo systemctl start nagios.service
```

## Étape 13 : Connexion à l'interface Web de Nagios
Ouvrez votre navigateur et accédez à l'interface Web Nagios en utilisant l'adresse IP ou le nom de domaine de votre serveur :


http://<Adresse IP/FQDN>/nagios

Connectez-vous avec le nom d'utilisateur "nagiosadmin" que vous avez défini à l'étape 10.

