# Scénario complet d'install elefan pour poplacoop sur un VPS OVH

Entre autre, cette procédure :

- configure la machine (ssh pare-feu ...)
- installe les paquets nécéssaire (nginx, php, mariadb...)
- installe l'app elefan et adminer

Pour ne pas trop charger j'ai retiré les actions pour la version "membres-test" Elles sont similaire à "membres"

## ovh start guide

**change password**

- passwd (change password for debian user)
- sudo passwd root (change root password)

**update system**

- sudo apt update
- sudo apt upgrade
  **ssh change port and refuse root login for security**
- sudo nano /etc/ssh/sshd_config > change Port 4422
- sudo nano /etc/ssh/sshd_config > PermitRootLogin no
- sudo /etc/init.d/ssh restart

**install Fail2ban**

- sudo apt install fail2ban
- sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.conf.backup
- sudo nano /etc/fail2ban/jail.conf (to change config)
- sudo /etc/init.d/fail2ban restart

**firewall iptable**

- sudo iptables -L
- bloquer les requetes non souhaitées
- ovh propose aussi un parefeu web : https://docs.ovh.com/fr/dedicated/firewall-network/
- => on va configurer le parefeu avec 'ufw' juste après

## other set up

**add my ssh key to login esaly without password**

- cat ~/.ssh/id_rsa.pub | ssh -p 4422 debian@vps-189ab9d8.vps.ovh.net "cat - >> ~/.ssh/authorized_keys"

**add the server in my .ssh/config**

- Host pop-la-coop
- HostName vps-189ab9d8.vps.ovh.net
- User debian
- Port 4422

**change server timezone**

- timedatectl
- timedatectl list-timezones
- sudo timedatectl set-timezone Europe/Paris # <=====
- timedatectl

## install firewall + nginx + mariaddb + php + composer + git

**firewall**

- sudo apt install ufw
- sudo ufw default deny incoming
- sudo ufw default allow outgoing
- sudo ufw allow 4422 # the right SSH port or you'll be autolock
- sudo ufw allow 80
- sudo ufw allow 443
- sudo ufw enable
- sudo ufw status

**nginx**

- sudo apt install nginx
- visit http://vps-189ab9d8.vps.ovh.net

**nginx : remove default website**

- sudo rm /etc/nginx/sites-enabled/default

**mariadb**

- sudo apt install mariadb-server
- sudo mysql_secure_installation
- press enter (empty mariadb root pass because just installed)
- n , ENTER => no root password
- Y => remove anonymous login
- Y => disallow root remote login
- Y => remove test databases
- Y => reload privilege tables
- sudo mariadb (to connect)

**php**

- sudo apt install php-fpm php-mysql php-gd php-xml php-mbstring

**composer**

- sudo apt install php-cli php-zip unzip
- wget -O composer-setup.php https://getcomposer.org/installer
- sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer --version=1.6.5
- composer
- rm composer-setup.php

**git**

- sudo apt install git

**certbot for let's encrypt**

- sudo apt install snapd
- sudo snap install core; sudo snap refresh core
- sudo snap install --classic certbot
- sudo ln -s /snap/bin/certbot /usr/bin/certbot

## nginx config

**nginx conf : default to reject other requests**

- scp -P 4422 nginx-config/0-default debian@vps-189ab9d8.vps.ovh.net:~
- sudo mv ~/0-default /etc/nginx/sites-available
- sudo ln -s /etc/nginx/sites-available/0-default /etc/nginx/sites-enabled/

**nginx conf : membres.poplacoop.fr**

- scp -P 4422 nginx-config/membres.poplacoop.fr debian@vps-189ab9d8.vps.ovh.net:~
- sudo mv ~/membres.poplacoop.fr /etc/nginx/sites-available
- sudo ln -s /etc/nginx/sites-available/membres.poplacoop.fr /etc/nginx/sites-enabled/a-membres.poplacoop.fr
- sudo mkdir /var/www/membres.poplacoop.fr
- visit http://membres.poplacoop.fr

**SSL certificate : membres.poplacoop.fr**

- sudo certbot --nginx -d membres.poplacoop.fr
- email : informatique@poplacoop.fr / A : agree terms / N : disagree share email
- sudo certbot renew --dry-run (to test auto renewal)
- all traffic from 80 will be redirect to 443
- sudo nano /etc/nginx/sites-available/membres.poplacoop.fr (to check modified nginx conf)

**check and reload nginx**

- sudo nginx -t
- sudo systemctl reload nginx

## mariadb config

**mariadb conf : prepare DB users for membres**

- sudo mariadb
- GRANT ALL ON gestion_compte.\* TO 'gestion_compte_user'@'localhost' IDENTIFIED BY 'complex-password' WITH GRANT OPTION;
- FLUSH PRIVILEGES;
- exit
- sudo mariadb -u gestion_compte_user -p
- SHOW DATABASES;
- SHOW GRANTS;
- exit

## php config

**change timezone**

- sudo nano /etc/php/7.3/cli/php.ini
- sudo nano /etc/php/7.3/fpm/php.ini
- => `date.timezone="Europe/Paris"`

## install adminer

- sudo mkdir /var/www/adminer
- sudo wget "http://www.adminer.org/latest.php" -O /var/www/adminer/adminer.php
- scp -P 4422 nginx-config/adminer.poplacoop.fr debian@vps-189ab9d8.vps.ovh.net:~
- sudo mv ~/adminer.poplacoop.fr /etc/nginx/sites-available/
- sudo ln -s /etc/nginx/sites-available/adminer.poplacoop.fr /etc/nginx/sites-enabled/z-adminer.poplacoop.fr
- sudo certbot --nginx -d adminer.poplacoop.fr
- sudo nano /etc/nginx/sites-available/adminer.poplacoop.fr (to check new lines for ssl)
- sudo nginx -t
- sudo systemctl reload nginx
- visit https://adminer.poplacoop.fr

## gestion-compte app

**gestion-compte app : get source**

- cd /var/www/membres.poplacoop.fr
- sudo git clone https://github.com/elefan-grenoble/gestion-compte.git
- sudo chown -R www-data:www-data /var/www/membres.poplacoop.fr
- cd gestion-compte
- sudo git fetch --all --tags
- sudo git checkout tags/v1.27

**gestion-compte app : replace elephant logo**

- scp -P 4422 gestion-compte-config/cute-elefan.png debian@vps-189ab9d8.vps.ovh.net:~
- sudo mv ~/cute-elefan.png /var/www/membres.poplacoop.fr/gestion-compte/src/AppBundle/Resources/public/img/cute-elefan.png

**gestion-compte app : replace elephant favicon**

- scp -P 4422 gestion-compte-config/favicon-lelefan.png debian@vps-189ab9d8.vps.ovh.net:~
- sudo mv ~/favicon-lelefan.png /var/www/membres.poplacoop.fr/gestion-compte/web/favicon-lelefan.png

**gestion-compte app : add specific parameters.yml**

- scp -P 4422 gestion-compte-config/parameters.yml debian@vps-189ab9d8.vps.ovh.net:~
- sudo mv ~/parameters.yml /var/www/membres.poplacoop.fr/gestion-compte/app/config/parameters.yml
- change `database_port, database_user, database_password, super_admin.*, mailer*, emails, [...]`

**gestion-compte app : prepare source**

- sudo chown -R www-data:www-data /var/www/membres.poplacoop.fr
- sudo -u www-data composer install
- sudo -u www-data php bin/console doctrine:database:create
- sudo -u www-data php bin/console doctrine:migration:migrate
- sudo -u www-data php bin/console assetic:dump
- visit http://membres.poplacoop.fr

**gestion-compte app : create super admin**

- https://membres.poplacoop.fr/user/install_admin
- connect with admin / (see password in parameters.yml)

**gestion-compte app : create a user**

- creer un adhérant manuellement
- john@mail.com John Hal 01 02 03 04 05 rue 35000 Rennes montant:15

**gestion-compte app : activate user by email or with this command**

- sudo -u www-data php bin/console fos:user:activate jhal
- sudo -u www-data php bin/console fos:user:change-password jhal pass
- connect with jhal / pass

## gestion-compte app : configs specifiques supplémentaires pop la coop

**pas de renouvellement des adhérant chaque année**

- in parameters.yml : `registration_duration: '99 year'`

**adhérant expert dès le début**

- in parameters.yml : `new_users_start_as_beginner: false`

**set up crontab**

- sudo -u www-data crontab -l
- sudo -u www-data crontab -e
- sudo -u www-data EDITOR=nano crontab -e

```bash
0 6 * * * php /var/www/membres.poplacoop.fr/gestion-compte/bin/console app:shift:reminder $(date -d "+2 days" +\%Y-\%m-\%d)
55 5 * * * php /var/www/membres/bin/console app:shift:generate $(date -d "+27 days" +\%Y-\%m-\%d)
```

**ajout des services**

- https://membres.poplacoop.fr/services/
- site web / cagette
