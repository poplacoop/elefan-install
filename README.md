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

[doc/maintenance-server.md](docs/maintenance-server.md)
