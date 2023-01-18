---
title: 'Colorer la barre de navigation de Chrome mobile (Android)'
date: '2016-02-22T14:09:10-05:00'
author: Adrien Poupa
url: /colorer-la-barre-de-navigation-de-chrome-mobile-android/
categories:
    - Android
tags:
    - 'google chrome'
    - navigation
---

![Screenshot_20160222-140029](https://cdn.poupa.net/uploads/2016/02/Screenshot_20160222-140029-576x1024.png)Depuis Chrome 39 et Android Lollipop, il est possible de colorer la barre de navigation de Chrome comme dans l’impression d’écran ci-dessus. Pour le faire, c’est (très) simple, il suffit de rajouter la ligne suivante entre les balises de votre site :

```
<meta name="theme-color" content="#ffffff">
```

Bien sûr, remplacez #ffffff par le code couleur attendu. Vous pouvez voir le résultat ici-même, ce n’est pas grand chose mais c’est une fonctionnalité sympathique pour une seule ligne de code 🙂

Vous pouvez aussi rajouter une icône à votre site avec la ligne :

```
<link rel="icon" sizes="192x192" href="icone.png">
```

[Source](https://developers.google.com/web/updates/2014/11/Support-for-theme-color-in-Chrome-39-for-Android)