---
title: >
    H5ai : remplacez l'explorateur par défaut de Nginx
date: '2016-01-31T22:55:14-05:00'
author: Adrien Poupa
url: /h5ai-remplacez-lexplorateur-par-defaut-de-nginx/
categories:
    - Linux
tags:
    - apache
    - h5ai
    - nginx
---

![h5ai](https://cdn.poupa.net/uploads/2016/01/h5ai.png)L’index de Nginx par défaut permettant l’affichage des fichiers est quelque peu spartiate. Pas vraiment l’idéal si on veut avoir un affichage de miniatures ou encore un rappel de l’arborescence des dossiers.

Heureusement, [H5ai](https://larsjung.de/h5ai/) existe ! Originellement pensé pour Apache, d’où son nom (HTML5 Apache Index), il est désormais compatible avec les serveurs web les plus populaires (Apache, Nginx, Lighttpd, Cherokee). De [bons guides](http://www.it-connect.fr/remplacer-lindex-apache-par-h5ai/) existent pour Apache, mais peu pour Nginx, d’où le fait que je ne m’attarderai pas sur les autres serveurs (ça, et le fait que Nginx est [bien supérieur](https://www.nginx.com/blog/nginx-vs-apache-our-view/) de toute façon 😉 )

Histoire de vous faire une idée, une démo est [disponible ici](https://larsjung.de/h5ai/demo/).

L’installation sous Nginx est rapide et facile. Vous téléchargez l’archive dans le répertoire que vous voulez afficher, vous la dézippez et on passe à la configuration.

Vous pouvez tester la compatibilité de votre serveur et voir que tout est en ordre à l’adresse /votredossier/\_h5ai/public (le mot de passe par défaut est vide) :

![h5ai configuration](https://cdn.poupa.net/uploads/2016/01/2016-01-31_224345.png)Dans votre votre fichier de configuration propre au site dans lequel vous voulez afficher l’index (/etc/nginx/sites-available), vous devez rajouter (ou modifier) la ligne index de cette façon :

```
index  index.html index.htm index.php /cheminvers/_h5ai/public/index.php;
```

On n’oublie pas d’inclure PHP dans le dossier si ce n’est pas fait :

```
location ~ \.php$ {
 root /mondossier;
 fastcgi_pass 127.0.0.1:9000;
 fastcgi_index   index.php;
 include fastcgi_params;
 fastcgi_param   SCRIPT_FILENAME /mondossier$fastcgi_script_name;
 }
```

Maintenant, la sécurité. Si vous ne voulez pas ouvrir votre index au public, le plus simple est de le [protéger par mot de passe](https://blog.bini.io/mettre-en-place-une-authentification-http-sur-nginx/):

```
 auth_basic            "Acces interdit";
 auth_basic_user_file  /etc/nginx/motsdepasse;
```

Cependant, si vous voulez le passer en accès public, il faut faire attention à plusieurs petites choses… déjà on peut commencer par interdire l’accès au répertoire private de h5ai :

```
 location ^~ /votredossier/_h5ai/private {
 deny all;
 return 403;
 }
```

Je vous recommande bien sûr de changer le mot de passe par défaut, ça se passe dans /votredossier/\_h5ai/private/options.json, à la ligne “passhash”. Vous devez [générer une chaîne en sha512](http://md5hashing.net/hashing/sha512) et la remplacer. Vous pouvez voir au passage quelques options sympathiques, comme activer le téléchargement des fichiers, renseigner votre compte Google Analytics…

Enfin, je vous recommande de placer un fichier index.html vide dans le dossier /votredossier/\_h5ai, faute de quoi n’importe qui se rendant à cette adresse verrait l’interface h5ai affichant les deux dossiers public et private de votre dossier \_h5ai. Pas terrible.

Et voilà ! Vous pouvez désormais afficher un bel index, pratique pour [mettre ses cours](http://adrien.poupa.net/efrei/) par exemple 😉