Django REST API #2 : Les filtres
################################

:date: 2019-02-25 20:35
:modified: 2019-02-25 20:35
:tags: Python, Django, REST API
:category: Développement
:slug: django-rest-api-02
:authors: Marc-Aurele Coste
:lang: fr

.. role:: red
.. role:: blue
.. role:: green

On a vu comment faire une API REST avec Django et Django REST Framework. On va maintenant améliorer ce que nous avons en ajoutant la possibilité de filtrer les éléments que nous retourne notre API.

Pour cela il nous faut tout d'abord installer le paquet :blue:`django-filter`.

.. code-block:: sh

    pip install django-filter

On fait les changements nécessaires dans les settings.

:green:`INSTALLED_APPS`: :red:`common.py`

    .. code-block:: python

        INSTALLED_APPS = [
            ...
            'django_filters',
        ]

:green:`REST_FRAMEWORK`: :red:`common.py`

    .. code-block:: python

        REST_FRAMEWORK = {
            ...
            'DEFAULT_FILTER_BACKENDS': ('django_filters.rest_framework.DjangoFilterBackend',)
        }

Filtrage basique
================

Maintenant on ajoute dans notre application un nouveau fichier **filters.py** dans lequel on va déclarer notre classe qui servira à filtrer les résultats.

.. code-block:: python

    # filters.py
    from django_filters import FilterSet

    from .models import Todo


    class TodoFilter(FilterSet):
        class Meta():
            model = Todo
            fields = [
                'title', 'description', 'finished'
            ]

On ajoute ensuite cette classe dans notre viewset.

.. code-block:: python

    # views.py
    ...
    from .filters import TodoFilter

    class TodoViewset(viewsets.ModelViewSet):
        ...
        filter_class = TodoFilter

Voilà on peut se rendre sur `locahost:8000/api/todo/ <http://locahost:8000/api/todo/>`_ et voir en haut à droite un nouveau bouton **Filters**

.. image:: {static}/static/images/django_rest_api/filters_button.png
    :width: 804 px
    :height: 184 px
    :scale: 90 %
    :alt: bouton Filters
    :align: center

On peut maintenant filtrer nos résultats.

Filtrage avec des booléens
==========================

Nous avons dans la liste des champs (fields) :red:`finished` qui est de type booléen. Le problème est que l'URL final ressemble à ceci lorsque l'on essaie de filtrer sur ce champ.

    127.0.0.1:8000/api/todo/?finished=2

On aimerait plutôt quelque chose dans ce genre:

    127.0.0.1:8000/api/todo/?finished=true

Nous allons donc préciser le type de ce champ dans notre classe de filtre. Pour cela on déclare notre champ comme un attribut de notre classe comme ci-dessous.

.. code-block:: python

    # filters.py
    ...
    class TodoFilter(FilterSet):
        finished = filters.BooleanFilter()

        class Meta():
            model = Todo
            fields = [
                'title', 'description', 'finished'
            ]

Filtrage avec des listes
========================

On aimerait maintenant filtrer nos résultats en utilisant une liste d'ids. On ne peut pas passer directement **id** dans la liste des champs (fields) car on ne pourrait filtrer que sur un seul id et notre API nous permet déjà de faire ça en ajoutant l'id à la fin de notre URL.

Le package django_filters dispose déjà de ce que nous recherchons, il s'agit du **BaseInFilter**

.. code-block:: python

    # filters.py
    ...
    class TodoFilter(FilterSet):
        ids = filters.BaseInFilter(field_name='id', lookup_expr='in')
        finished = filters.BooleanFilter()

        class Meta():
            model = Todo
            fields = [
                'ids', 'title', 'description', 'finished'
            ]

Ici notre attribut ne portant pas le même nom que l'attribut du modèle il nous faut préciser le **field_name**. Nous pouvons maintenant utiliser facilement notre API avec du JavaScript par exemple.

.. code-block:: javascript

    ids = [1, 3]
    console.log(`localhost:8000/api/todo/?ids=${ids}`)
    // return localhost:8000/api/todo/?ids=1,3

Petit plus
==========

Nous avons une classe qui nous permet de filtrer nos résultats. On peut notamment chercher une tâche en entrant son titre complet. Dans la vraie vie, en revanche, le plus souvent on ne se souvient pas du titre exact de ce que nous cherchons mais seulement des mots-clés.

On va modifier légèrement notre classe de filtre afin de permettre ce type de recherches.

.. code-block:: python

    # filters.py
    ...
    class TodoFilter(FilterSet):
        ids = filters.BaseInFilter(field_name='id', lookup_expr='in')
        finished = filters.BooleanFilter()

        class Meta():
            model = Todo

            fields = {
                'ids': [],
                'title': ['icontains'],
                'description': ['icontains'],
                'finished': []
            }

Dans cet exemple nous précisons que nous cherchons à faire un test d'inclusion pour le **title** et pour la **description**.

Voici quelques possibilités de tests offertes par Django (la liste complète se trouve `ici <https://docs.djangoproject.com/fr/2.1/ref/models/querysets/#field-lookups>`_)

:blue:`exact`
    Correspondance exacte.

:blue:`iexact`
    Correspondance exacte insensible à la casse.

:blue:`contains`
    Test d’inclusion sensible à la casse.

:blue:`icontains`
    Test d’inclusion insensible à la casse.

:blue:`gt`
    Plus grand que.

:blue:`lt`
    Plus petit que.

:blue:`startswith`
    Commence par (sensible à la casse).

:blue:`endswith`
    Se termine par (sensible à la casse).

Voilà c'est tout pour la partie filtrage. Cela vous permettra d'obtenir une API assez flexible sur les données retournées et vous évitera un traitement côté client.

-----

**Partie 03** : `Authentification / Permissions <{filename}/fr/django_rest_api_03.rst>`_