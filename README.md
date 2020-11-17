# elefan-install

Notes technique concernant l'installation d'elefan.

## version actuellement installée

- `v1.27`
- détails : https://github.com/elefan-grenoble/gestion-compte/releases/tag/v1.27

## services web exposés

- gestion membres prod : https://membres.poplacoop.fr
- gestion membres test : https://membres-test.poplacoop.fr
- outil gestion base de données : https://adminer.poplacoop.fr

## architecture

- installé sur VPS SSH
- nginx, mariadb, php

## connexion en SSH au server

`ssh -p 4422 debian@vps-189ab9d8.vps.ovh.net`

## scénario complet d'installation du VPS

[doc/install-server.md](docs/install-server.md)

## commandes utiles pour la maintenance

**nginx : tester et recharger la conf**
sudo nginx -t
sudo systemctl reload nginx

**nginx : logs**
less /var/log/nginx/membres.poplacoop.fr_access.log
less /var/log/nginx/membres.poplacoop.fr_error.log

**app logs**
tail -f -n 50 /var/www/membres.poplacoop.fr/gestion-compte/var/logs/prod.log

**php tool to debug app**
cd /var/www/membres.poplacoop.fr/gestion-compte
sudo -u www-data php bin/console

**update app after changes in parameter.yml**
cd /var/www/membres.poplacoop.fr/gestion-compte
sudo -u www-data php bin/console cache:clear --env=prod --no-debug

**update app if you changes assets like images**
cd /var/www/membres.poplacoop.fr/gestion-compte
sudo -u www-data php bin/console assetic:dump

**edit parameter.yml**
sudo nano /var/www/membres.poplacoop.fr/gestion-compte/app/config/parameters.yml

