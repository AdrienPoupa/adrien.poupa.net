---
title: 'Autocomplétion des adresses avec la Base Adresse Nationale'
date: '2016-10-14T14:53:45-04:00'
author: Adrien Poupa
url: /autocompletion-des-adresses-avec-la-base-adresse-nationale/
categories:
    - Unclassified
---

![ban](https://cdn.poupa.net/uploads/2016/10/ban.png)Lors de mon stage à [Diagamter ](http://www.diagamter.com/)cet été, j’ai eu pour tâche de créer un formulaire d’autocomplétion des adresses. Après avoir fait un tour d’horizon des outils disponibles, un m’a marqué : la Base Adresse Nationale. Tout d’abord parce cette API est officielle, qu’elle peut faire office de référence ; toutes les modifications de communes y sont répertoriées rapidement. Ensuite, elle simple, bien documentée et n’a pas de quota de requête à ma connaissance.

Le but est donc de mettre en place un formulaire simple qui va envoyer des requêtes à l’API lorsqu’on saisit d’abord le code postal, puis la ville et enfin l’adresse en elle-même. On va donc se servir de jQuery et jQuery UI pour mettre en place l’autocomplétion :

```
<script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
<script src="https://code.jquery.com/ui/1.12.1/jquery-ui.min.js"></script>
<link rel="stylesheet" href="https://code.jquery.com/ui/1.7.3/themes/base/jquery-ui.css">
```

Ensuite, on met en place les inputs en eux-mêmes :

```
<input name="cp" id="cp" type="text" placeholder="CP">
<input name="ville" id="ville" type="text" placeholder="Ville">
<input name="adresse" id="adresse" type="text" placeholder="Adresse">
```

Enfin, on s’attaque aux fonctions d’autocomplétion : tout d’abord le code postal. Le but est de saisir un code et d’afficher la ville à laquelle il est rattaché. Par exemple, si on tape “31000”, on obtiendra “31000 – Toulouse”. Un clic dessus permettra de remplir les deux premiers champs. De la même façon, si on tape “Toulouse” dans le deuxième champ, on aura le même résultat.

On décode ensuite l’objet json GeoCode retourné pour exploiter le code postal, le nom de la ville et l’adresse.

Code des trois autocomplétions :

```
<script>
$("#cp").autocomplete({
    source: function (request, response) {
        $.ajax({
            url: "https://api-adresse.data.gouv.fr/search/?postcode="+$("input[name='cp']").val(),
            data: { q: request.term },
            dataType: "json",
            success: function (data) {
                var postcodes = [];
                response($.map(data.features, function (item) {
                    // Ici on est obligé d'ajouter les CP dans un array pour ne pas avoir plusieurs fois le même
                    if ($.inArray(item.properties.postcode, postcodes) == -1) {
                        postcodes.push(item.properties.postcode);
                        return { label: item.properties.postcode + " - " + item.properties.city, 
                                 city: item.properties.city,
                                 value: item.properties.postcode
                        };
                    }
                }));
            }
        });
    },
    // On remplit aussi la ville
    select: function(event, ui) {
        $('#ville').val(ui.item.city);
    }
});
$("#ville").autocomplete({
    source: function (request, response) {
        $.ajax({
            url: "https://api-adresse.data.gouv.fr/search/?city="+$("input[name='ville']").val(),
            data: { q: request.term },
            dataType: "json",
            success: function (data) {
                var cities = [];
                response($.map(data.features, function (item) {
                    // Ici on est obligé d'ajouter les villes dans un array pour ne pas avoir plusieurs fois la même
                    if ($.inArray(item.properties.postcode, cities) == -1) {
                        cities.push(item.properties.postcode);
                        return { label: item.properties.postcode + " - " + item.properties.city, 
                                 postcode: item.properties.postcode,
                                 value: item.properties.city
                        };
                    }
                }));
            }
        });
    },
    // On remplit aussi le CP
    select: function(event, ui) {
        $('#cp').val(ui.item.postcode);
    }
});
$("#adresse").autocomplete({
    source: function (request, response) {
        $.ajax({
            url: "https://api-adresse.data.gouv.fr/search/?postcode="+$("input[name='cp']").val(),
            data: { q: request.term },
            dataType: "json",
            success: function (data) {
                response($.map(data.features, function (item) {
                    return { label: item.properties.name, value: item.properties.name};
                }));
            }
        });
    }
});
</script>
```

Fiddle de démonstration : https://jsfiddle.net/8apgu63n/embedded