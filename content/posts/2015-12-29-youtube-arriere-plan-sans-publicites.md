---
title: 'YouTube en arrière-plan et sans publicités'
date: '2015-12-29T18:32:43-05:00'
author: Adrien Poupa
url: /youtube-arriere-plan-sans-publicites/
categories:
    - Android
tags:
    - xposed
    - 'youtube adaway'
    - 'youtube background playback'
---

![YouTube](https://cdn.poupa.net/uploads/2015/12/YouTube-logo-full_color-300x187.png)Si l’application Android officielle de YouTube n’est pas trop mal faite, ce qui m’embête le plus est sans conteste l’arrêt forcé du son lors du passage de l’application en arrière-plan et les publicités incessantes avant le début de chaque vidéo. Je me suis renseigné et j’ai essayé plusieurs applications sans jamais être vraiment comblé (suppression de l’application du Play Store, fonctionnement bancal, support douteux…)

Le salut est venu d’un module Xposed. Xposed, pour ceux qui ne sont pas familiers avec ce terme, c’est un framework permettant l’installation d’applications non officielles qui peuvent affecter le téléphone plus en profondeur que des applications disponibles par le Play Store, même si elles requièrent le root, en forçant la machine virtuelle d’Android à charger un fichier supplémentaire et en permettant le remplacement du code de n’importe quelle application par un autre.

Xposed s’installe sur un téléphone rooté, et plusieurs méthodes d’installations s’offrent à vous. La plus simple, c’est de [récupérer la dernière version de l’application ici](http://dl.xposed.info/latest.apk), l’installer puis la lancer en cliquant sur Framework puis Install/Update. Après un reboot du téléphone, c’est fonctionnel.

![Screenshot_20151229-182328](https://cdn.poupa.net/uploads/2015/12/Screenshot_20151229-182328-169x300.png)Sinon, vous pouvez [flasher directement le .zip](http://forum.xda-developers.com/showthread.php?t=3034811) adapté depuis votre recovery. Attention à sélectionner la bonne version selon le processeur de votre téléphone. Ensuite, il faudra installer l’APK comme expliqué ci-dessus.

Une fois Xposed installé, on installe les deux modules “YouTube Adaway” et “YouTube Background Playback” depuis la section “Download” (cliquer sur le nom, puis onglet “Version” et “Download”). Il faudra ensuite activer les modules s’ils ne le sont pas et redémarrer.

![Screenshot_20151229-182659](https://cdn.poupa.net/uploads/2015/12/Screenshot_20151229-182659-169x300.png)Et c’est bon !

![Screenshot_20151229-182608](https://cdn.poupa.net/uploads/2015/12/Screenshot_20151229-182608-169x300.png)