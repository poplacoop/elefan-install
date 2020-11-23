# Commandes utiles pour la maintenance

**connect with SSH**

- ssh -p 4422 debian@vps-189ab9d8.vps.ovh.net

**nginx : tester et recharger la conf**

- sudo nginx -t
- sudo systemctl reload nginx

**nginx : logs**

- less /var/log/nginx/membres.poplacoop.fr_access.log
- less /var/log/nginx/membres.poplacoop.fr_error.log

**app logs**

- tail -f -n 50 /var/www/membres.poplacoop.fr/gestion-compte/var/logs/prod.log

**php tool to debug app**

- cd /var/www/membres.poplacoop.fr/gestion-compte
- sudo -u www-data php bin/console

**update app after changes in parameter.yml**

- cd /var/www/membres.poplacoop.fr/gestion-compte
- sudo -u www-data php bin/console cache:clear --env=prod --no-debug

**update app if you changes assets like images**

- cd /var/www/membres.poplacoop.fr/gestion-compte
- sudo -u www-data php bin/console assetic:dump

**edit parameter.yml**

- sudo nano /var/www/membres.poplacoop.fr/gestion-compte/app/config/parameters.yml

**create new super admin**
- cd /var/www/membres.poplacoop.fr/gestion-compte
- sudo -u www-data php bin/console fos:user:create
- sudo -u www-data php bin/console fos:user:promote
  - role : ROLE_SUPER_ADMIN