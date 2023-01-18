---
title: >
    Mise en place d'un environnement chroot avec identification en clés SSH
date: '2017-01-31T21:11:58-05:00'
author: Adrien Poupa
url: /mise-en-place-dun-environnement-chroot-avec-identification-en-cles-ssh/
categories:
    - Linux
---

![chroot](https://cdn.poupa.net/uploads/2017/02/chroot.jpg)Sur mon serveur dédié, je voulais mettre en place un environnement chroot afin de pouvoir donner un accès limité à leur espace à des utilisateurs externes.

La mise en place d’un tel espace n’est pas complexe en soit, mais j’ai en revanche eu beaucoup plus de mal à activer la connexion par un jeu de clés SSH.

Nous allons partir du principe que le système est déjà configuré pour la connexion avec des clés SSH.

On crée notre utilisateur “utilisateur”

```
useradd utilisateur
```

Ici, on va donc appliquer notre prison à l’utilisateur “utilisateur”, partir du principe que son répertoire de chroot est /home/www/utilisateur (pour de l’hébergement par exemple).

Il faut modifier si besoin son répertoire d’accueil en /home/www/utilisateur si besoin, on vérifie avec:

```
nano /etc/passwd
```

Puis la ligne de notre utilisateur

```
utilisateur:x:1000:1000::/home/www/utilisateur:/usr/sbin/nologin
```

Notez la dernière partie, /usr/sbin/nologin pour empêcher le login en SSH.

On crée le répertoire utilisateur, qui appartient au root et n’est pas modifiable par l’utilisateur:

```
mkdir /home/www/utilisateur
chmod 755 /home/www/utilisateur
```

L’utilisateur aura accès en modification à un autre répertoire, contenu dans celui qu’on vient de créer:

```
mkdir /home/www/utilisateur/upload
chmod 755 /home/www/utilisateur/upload
chown utilisateur /home/www/utilisateur/upload
```

Dans /etc/ssh/sshd\_config, rajouter à la fin:

```
Match User utilisateur
    ChrootDirectory /home/www/utilisateur
    ForceCommand internal-sftp
    AllowTcpForwarding no
```

On force la connexion en SFTP avec le ForceCommand et on interdit le TcpForwarding.

Toujours dans le sshd\_config, il faut changer le répertoire des clés SSH (et c’est là que les ressources sur internet commencent à se faire mince…).

On remplace AuthorizedKeysFile par:

```
AuthorizedKeysFile      /etc/ssh/authorized_keys/%u/authorized_keys
```

On va donc avoir toutes nos clés dans un répertoire appartenant à root (très important) et en dehors du chroot (important aussi).

Création du dossier:

```
mkdir /etc/ssh/authorized_keys
```

Création des dossiers utilisateurs:

```
mkdir /etc/ssh/authorized_keys/root
mkdir /etc/ssh/authorized_keys/utilisateur
```

Il faut copier toutes les clés existantes dans ce nouveau répertoire:

```
cp /root/.ssh/authorized_keys /etc/ssh/authorized_keys/root
cp /utilisateur/.ssh/authorized_keys /etc/ssh/authorized_keys/utilisateur
```

La deuxième ligne est à ajuster si nécessaire pour l’emplacement du fichier de clés de l’utilisateur. L’important est de les copier dans le nouveau répertoire.

Enfin, point crucial : paramétrer les permissions correctement. Il faut que le répertoire de base /authorized\_keys soit en 755 pour que le daemon SSH y accède sans problème, ainsi que le répertoire utilisateur. J’ai lu partout qu’il fallait être en 700… Or cela ne fonctionne pas !

Enfin, plus classiquement le fichier de clés doit être en 600 et appartenir à l’utilisateur.

```
chmod 755 /etc/ssh/authorized_keys
chmod 755 /etc/ssh/authorized_keys/utilisateur
chmod 600 /etc/ssh/authorized_keys/utilisateur/authorized_keys
chown utilisateur /etc/ssh/authorized_keys/utilisateur/authorized_keys
```

Et c’est bon! 🙂

Notre utilisateur peut désormais se connecter en SFTP seulement avec sa clé SSH, voir le répertoire /home/www/utilisateur en lecture seule et écrire dans /home/www/utilisateur/upload

Il est possible de généraliser cette méthode en l’appliquant non à un utilisateur mais un groupe.