---
title: >
    Certificat SSL avec Let's Encrypt, Nginx et CloudFlare
date: '2017-08-13T18:54:10-04:00'
author: Adrien Poupa
url: /certificat-ssl-avec-lets-encrypt-nginx-et-cloudflare/
categories:
    - Linux
---

![](https://cdn.poupa.net/uploads/2017/08/le-logo-twitter-300x300.png)Avec Google qui fait ressortir les sites en HTTPS prioritairement et Firefox qui affiche désormais un joli message d’avertissement lors du remplissage d’un formulaire de connexion en HTTP, il est temps d’installer un certificat SSL sur vos sites, d’autant plus que c’est gratuit ! Nous allons en effet nous servir de Let’s Encrypt, fondation à but non lucratif qu’on ne présente plus, qui a l’avantage de fournir gratuitement des certificats SSL acceptés par tous les navigateurs modernes.

On commence par installer Let’s Encrypt

```
git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
```

Ensuite, on va modifier le virtual host Nginx du domaine dont on veut créer un certificat en ajoutant ceci :

```
location ~ /\.well-known/acme-challenge {
    root /tmp/well-known;
}
```

Et on crée le répertoire :

```
mkdir /tmp/well-known
```

Explication : le script de Let’s Encrypt va devoir vérifier que nous possédons bien le domaine pour lequel on veut un certificat. Pour ce faire, il va ping domaine.fr/.well-known/acme-challenge/blabla où blabla représente une chaîne générée à la volée par le script au moment de la vérification. On le met donc dans /tmp vu que c’est du temporaire, et on dit à Nginx de rediriger toutes les requêtes à cet endroit. Si jamais votre domaine est protégé par un htaccess, il faut s’assurer que ce répertoire en particulier n’est pas protégé avec un allow all;.

Rechargement de Nginx pour appliquer la modification :

```
nginx -t reload
```

On peut enfin lancer Let’s Encrypt et générer nos certificats :

```
cd /opt/letsencrypt
./letsencrypt-auto certonly -a webroot --webroot-path=/tmp/well-known -d domaine.fr -d www.domaine.fr
```

Ici on génère les certificats pour domaine.fr et le sous-domaine www. Vous ne pouvez en faire qu’un seul à la fois. Si vous avez des sous-domaines, il faut répéter l’opération.

On vous demandera un mail qui servira à vous envoyer des rappels d’expiration de certificats puisque ces derniers expirent tous les 90 jours. Mais ce n’est pas un problème, on verra plus tard.

Il faut également que vous acceptiez les ToS.

Quand tout s’est déroulé avec succès, vous obtenez le message suivant :

```
IMPORTANT NOTES:
- Congratulations! Your certificate and chain have been saved at
/etc/letsencrypt/live/domaine.fr/fullchain.pem. Your cert will
expire on 2017-11-13. To obtain a new or tweaked version of this
certificate in the future, simply run letsencrypt-auto again. To
non-interactively renew *all* of your certificates, run
"letsencrypt-auto renew"
- If you like Certbot, please consider supporting our work by:

Donating to ISRG / Let's Encrypt: https://letsencrypt.org/donate
Donating to EFF: https://eff.org/donate-le
```

On va générer quelques fichiers : commençons par le paramètre Diffie-Hellman

```
mkdir /etc/letsencrypt/certs
openssl dhparam -out /etc/letsencrypt/certs/dhparam.pem 2048
```

On utilise 2048 bits, on pourrait mettre 4096 pour plus de sécurité mais cela prend vraiment beaucoup de temps sur un serveur peu puissant.

On crée aussi un ticket de session :

```
openssl rand 48 -out /etc/letsencrypt/certs/ticket.key
```

On peut passer à la configuration Nginx 🙂 Dans le bloc server :

```
listen 443 ssl http2;
listen [::]:443 ssl http2;

server_name domain.fr;
ssl on;
ssl_certificate /etc/letsencrypt/live/domain.fr/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/domain.fr/privkey.pem;
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/letsencrypt/live/domain.fr/fullchain.pem;
resolver 8.8.8.8 8.8.4.4 208.67.222.222 208.67.220.220 216.146.35.35 216.146.36.36 valid=300s;
resolver_timeout 3s;
ssl_session_cache shared:SSL:100m;
ssl_session_timeout 24h;
ssl_session_tickets on;
ssl_session_ticket_key /etc/letsencrypt/certs/ticket.key;
ssl_dhparam /etc/letsencrypt/certs/dhparam.pem;
ssl_ecdh_curve secp384r1;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!3DES:!MD5:!PSK';
```

Il faut aussi penser à forcer le https depuis le http. On peut faire comme ceci, en rajoutant un nouveau bloc server :

```
]server {
    listen 80;
    listen [::]:80;
    server_name domaine.fr;
    location ~ /\.well-known/acme-challenge {
        allow all;
        root /tmp/well-known;
    }
    location / {
        return 301 https://domaine.fr$request_uri;
    }
}
```

On remarque qu’on a rajouté la section challenge, on peut donc la supprimer de l’autre bloc serveur qui, lui, écoute le 443 pour le https.

Et on peut relancer le serveur :

```
nginx -t reload
```

Pour ce qui est du renouvellement, votre certificat est valide trois mois. C’est peu, mais on peut le renouveler en ligne de commande. Et donc rajouter une tâche cron pour le faire automatiquement :

```
crontab -e

30 2 * * 1 /opt/letsencrypt/letsencrypt-auto renew >> /var/log/lets-encrypt-renew.log
```

Si vous n’utilisez pas CloudFlare, vous pouvez vous arrêter là. Sinon, c’est parti 🙂

Commencez, dans l’onglet Crypto, par activer le mode Full SSL (strict), qu’on peut activer puisqu’on a bien un certificat valide côté serveur. CloudFlare se connectera donc de façon sécurisée à votre serveur, en étant sûr que le certificat est valide.

![](https://cdn.poupa.net/uploads/2017/08/crypto1.png)Je vous recommande d’activer les autres options qui permettent d’améliorer la sécurité.

![](https://cdn.poupa.net/uploads/2017/08/crypto2-628x1024.png)L’option Authenticated Origin Pulls permet de s’assurer que les visiteurs viendront par CloudFlare et CloudFlare seulement, en utilisant un certificat fourni par eux.

Le certificat se situe en bas de cette page : https://support.cloudflare.com/hc/en-us/articles/204899617

Vous devrez enregistrer ce fichier et modifier la configuration Nginx comme suit :

```
ssl_client_certificate /etc/nginx/certs/cloudflare.crt;
ssl_verify_client on;
```

où /etc/nginx/certs/cloudflare.crt est le chemin complet pour accéder au certificat. N’oubliez pas de relancer Nginx.

Enfin, si vous passez par CloudFlare, vous pouvez aussi créer des Page Rules pour gérer la redirection http =&gt; https et éviter d’ajouter le bloc serveur vu plus haut.

Voilà, bravo, votre site est désormais derrière un certificat SSL valide 🙂