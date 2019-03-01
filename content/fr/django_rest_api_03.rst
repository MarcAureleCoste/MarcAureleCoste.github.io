Django REST API #3 : Authentification / Permissions
###################################################

:date: 2019-02-28 20:35
:modified: 2019-02-28 20:35
:tags: Python, Django, REST API
:category: Développement
:slug: django-rest-api-03
:authors: Marc-Aurele Coste
:lang: fr
:status: draft

Notre api de todo fonctionne plutôt bien mais il reste un problème, tout le monde peut accéder à tous les todos. Nous allons remédier à ça en ajoutant une partie authentification et la gestion des droits sur les résultats retournés.

Création d'un modèle user
=========================

Afin de ne pas etre limiter par le modèle user fourni par Django nous allons commencer par créer le notre.

Ajouter l'authentification et authtoken settings

creation d'un modèle todouser + lien avec les todo

Filter les résultats en se basant sur l'utilisateur qui a fait la requête
