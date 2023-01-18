---
title: >
    Laravel : envoi du token CSRF lors d'un appel Ajax
date: '2016-04-25T11:07:19-04:00'
author: Adrien Poupa
url: /laravel-envoi-du-token-csrf-lors-dun-appel-ajax/
categories:
    - PHP
tags:
    - ajax
    - csrf
    - jquery
    - Laravel
---

Lors d’une requête POST en Ajax sous Laravel 5, il faut passer le jeton CSRF sous peine de recevoir une erreur de TokenMismatch, la protection contre les failles CSRF s’activant. Pour ce faire, je ne trouve pas la documentation très simple, alors qu’il suffit de passer l’attribut ‘\_token’ dans le champ data (sous jQuery).

Ainsi, il n’y a qu’une ligne à rajouter :

```
$.ajax({
 type: "POST",
 url: "{{ url('/votre-url') }}",
 data: {
 ...
 _token: "{!! csrf_token() !!}"
 }
})
```

Une alternative consiste à rajouter le champ complet contenant le jeton dans le fichier de template adéquat :

```
{!! csrf_field() !!}
```

Puis de recopier un code similaire au précédent, à ceci près qu’il ira chercher le code rajouté plus haut dans la page :

```
$.ajax({
 type: "POST",
 url: "{{ url('/votre-url') }}",
 data: {
 ...
 _token: $('meta[name="csrf-token"]').attr('content')
 }
})
```