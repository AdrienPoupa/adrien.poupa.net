---
title: 'Sauvegardez vos sites et bases de données sur Google Drive'
date: '2015-12-24T18:30:31-05:00'
author: Adrien Poupa
url: /sauvegarde-ftp-mysql-google-drive/
categories:
    - Linux
tags:
    - debian
    - ftp
    - 'google drive'
    - mysql
---

![Google Drive](https://cdn.poupa.net/uploads/2015/12/cloudd-283x300.png)J’ai de plus en plus de mal à comprendre les sites web, mais le plus souvent de petites communautés, qui ne rouvrent qu’après un certain temps hors-ligne, ou pire, qu’elles ferment en raison de l’absence d’une sauvegarde après un problème avec leur hébergeur. Comment cela peut-il être possible à l’heure du cloud, où toutes nos données nous sont accessibles d’un bout du monde à l’autre, via des services gratuits ?

Si je conçois qu’il est contraignant de faire une sauvegarde régulière de son FTP et des bases de données composant un site web, il est primordial de mettre en place un système le faisant pour nous, un bon informaticien <span style="text-decoration: line-through;">étant de base fainéant</span> optimisant chacune de ses actions.

Pour ce faire, quoi de plus naturel qu’utiliser la solution de sauvegarde cloud la plus répandue, [Google Drive](http://drive.google.com/) ? A raison de 15 Go par compte gratuit, il serait dommage de s’en priver.

Nous considérerons que le serveur où vous hébergez vos données est un serveur Debian et que vous avez les privilèges administrateurs sur votre machine.

Pour ce faire, nous allons utiliser [Grive2](http://yourcmc.ru/wiki/Grive2), petit utilitaire écrit en C++ avec la librairie Boost pour synchroniser les dossiers local et distant dans lesquels se trouveront les sauvegardes.

Installation des dépendances :

```
sudo apt-get install git cmake build-essential libgcrypt11-dev libyajl-dev libboost-all-dev libcurl4-openssl-dev libexpat1-dev libcppunit-dev binutils-dev
```

Récupération du script sur GitHub, compilation et installation :

```
cd /var/dossier_ou_seront_les_sauvegardes
git clone https://github.com/vitalif/grive2
mkdir build
cd build
cmake ..
make -j4
sudo make install
grive -a
```

La dernière ligne vous donnera un lien à visiter avec le compte sur lequel vous souhaitez effectuer la sauvegarde pour récupérer une clé API pour sauvegarder dans votre compte Google Drive. Copiez la clé obtenue. Le “-a” n’est nécessaire que la première fois.

Maintenant, vous pouvez mettre en place vos tâches cron, par exemple en copiant le fichier suivant dans /etc/cron.daily

```
#!/bin/bash
cd /var/dossier_ou_seront_les_sauvegardes

for i in nom_de_la_bdd nom_de_la_seconde_bdd; do

## On sauvegarde nos bdd en SQL
mysqldump --user=Utilisateur_BDD --password=Mdp_Utilisateur $i &amp;amp;gt; ${i}_`date +"%Y-%m-%d"`.sql

## On compresse le fichier obtenu
tar jcf ${i}_`date +"%Y-%m-%d"`.sql.tar.bz2 ${i}_`date +"%Y-%m-%d"`.sql

## On supprime les SQL
rm ${i}_`date +"%Y-%m-%d"`.sql

done

## Grive
/usr/local/bin/grive
```

On peut reprendre le même modèle pour une sauvegarde de FTP :

```
#!/bin/bash
#
## on se place dans le repertoire
#
cd /var/dossier_ou_seront_les_sauvegardes

## Compression récursive
tar -zcvf ftp_`date +"%Y-%m-%d"`.tar.gz /var/www

## Grive
/usr/local/bin/grive
```

où /var/www est le répertoire de vos sites web.

Pensez bien à rendre les dossiers exécutables par l’utilisateur qui lancera les cron et à ne pas donner d’extension aux fichiers :).