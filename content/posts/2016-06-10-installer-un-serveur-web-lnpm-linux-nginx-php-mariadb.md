---
title: 'Installer un serveur web LNPM (Linux, Nginx, PHP, MariaDB)'
date: '2016-06-10T13:47:58-04:00'
author: Adrien Poupa
url: /installer-un-serveur-web-lnpm-linux-nginx-php-mariadb/
categories:
    - Linux
---

![](https://www.unixmen.com/wp-content/uploads/2014/08/lemp.jpg)  
*Source: unixmen.com*

Cet article est surtout là pour me servir de pense-bête en cas de réinstallation d’un serveur, mais sait-on jamais, il peut être utile à d’autres 🙂

Le but est de mettre en place un serveur web qui soit le plus léger possible ; pour ce faire, j’utilise Linux et sa distribution Debian (8 Jessie pour le tutoriel ci-dessous, mais facilement adaptable pour d’autres version), Nginx en remplacement d’Apache, PHP 7 en FPM et MariaDB en remplacement de MySQL.

La première étape consiste en la modification de

```
/etc/apt/sources.list
```

auquel il faut ajouter les repositories nécessaires pour Nginx, MariaDB et Dotdeb pour PHP7 :

```
# Nginx
deb http://nginx.org/packages/debian/ jessie nginx
deb-src http://nginx.org/packages/debian/ jessie nginx

# MariaDB
deb [arch=amd64,i386] http://ftp.igh.cnrs.fr/pub/mariadb/repo/10.1/debian jessie main
deb-src http://ftp.igh.cnrs.fr/pub/mariadb/repo/10.1/debian jessie main

# Dotdeb
deb http://packages.dotdeb.org jessie all
```

Ajout des clés nécessaires à l’authentification des repositories :

```
wget https://www.dotdeb.org/dotdeb.gpg
apt-key add dotdeb.gpg

wget http://nginx.org/keys/nginx_signing.key
apt-key add nginx_signing.key

apt-get install software-properties-common
apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xcbcb082a1bb943db
```

Mise à jour des listes

```
apt-get update
```

Installation des paquets

```
apt-get install nginx
apt-get install php7.0-fpm
apt-get install php7.0-mysql
apt-get install mariadb-server
```

Si on n’installe que php7.0-fpm sans php7.0-mysql, l’extension mysqli ne sera pas installée et on ne pourra pas se servir de phpMyAdmin par exemple.

J’aime bien mettre mes sites dans /var/www et utiliser des dossiers sites-available et sites-enabled pour Nginx, ainsi je modifie

```
/etc/nginx/nginx.conf
```

de la façon suivante :

```
user www-data;
worker_processes 4;
pid /run/nginx.pid;

events {
 worker_connections 8192;
 multi_accept on;
 use epoll;
}

http {

 ##
 # Basic Settings
 ##

 sendfile on;
 tcp_nopush on;
 tcp_nodelay on;
 keepalive_timeout 4;
 types_hash_max_size 2048;
 server_tokens off;
 client_max_body_size 20m;
 client_body_buffer_size 128k;

 include /etc/nginx/mime.types;
 default_type application/octet-stream;

 ##
 # Logging Settings
 ##

 access_log /var/log/nginx/access.log;
 error_log /var/log/nginx/error.log;

    gzip  on;
    gzip_static on;
    gzip_comp_level 9;
    gzip_min_length 1400;
    gzip_types  text/plain text/css image/png image/gif image/jpeg application/x-javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_vary  on;
    gzip_http_version 1.1;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";

 ##
 # nginx-naxsi config
 ##
 # Uncomment it if you installed nginx-naxsi
 ##

 #include /etc/nginx/naxsi_core.rules;

 ##
 # Virtual Host Configs
 ##

 include /etc/nginx/conf.d/*.conf;
 include /etc/nginx/sites-enabled/*;
}
```

Ensuite, dans /etc/nginx, on crée les deux dossiers sites-available et sites-enabled. On crée le fichier :

```
/etc/nginx/sites-available/default
```

avec

```
server {
 listen 80 default_server;
 listen [::]:80 default_server ipv6only=on;

 root /var/www;
 index index.html index.htm index.php;

 # Make site accessible from http://localhost/
 server_name localhost;

 # PHP
 location ~ \.php$ {
 root /var/www;
 fastcgi_pass unix:/run/php/php7.0-fpm.sock;
 fastcgi_index   index.php;
 include fastcgi_params;
 fastcgi_param   SCRIPT_FILENAME /var/www$fastcgi_script_name;
 }
}
```

Puis on active la configuration par défaut créée ci-dessus en faisant un lien symbolique :

```
ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/default
```

On recharge la configuration de Nginx et tout doit fonctionner :

```
nginx -s reload
```

Chaque nouveau site devra être ajouté dans sites-available avec un lien symbolique dans enabled et un server\_name correspondant au nom du site web, par exemple :

```
server {
    listen       80;
    server_name  www.test.com;
    return       301 http://test.com$request_uri;
}

server {

 listen       80;
 server_name test.com;
 root /var/www/test.com;

 location / {

 root /var/www/test.com;
 index index.php index.html index.htm;
 try_files $uri $uri/ /index.php?$args;
 access_log /var/log/nginx/test.com.access;
 error_log /var/log/nginx/test.com.error error;

 location ~ \.php$ {
 root /var/www/test.com;
 fastcgi_pass unix:/run/php/php7.0-fpm.sock;
 fastcgi_index   index.php;
 include fastcgi_params;
 fastcgi_param   SCRIPT_FILENAME /var/www/test.com$fastcgi_script_name;
 }
 }
}
```

Ici, on crée le site test.com avec www.test.com redirigeant vers test.com, le site se trouvant dans /var/www/test.com.