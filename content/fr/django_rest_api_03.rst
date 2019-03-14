Django REST API #3 : Authentification / Permissions
###################################################

:date: 2019-02-28 20:35
:modified: 2019-02-28 20:35
:tags: Python, Django, REST API
:category: Développement
:slug: django-rest-api-03
:authors: Marc-Aurele Coste
:lang: fr

.. role:: red

Notre API de Todo(s) fonctionne bien. En plus de pouvoir *lister*, *créer*, *modifier*, *supprimer* des éléments de notre base de données elle nous permet également de filtrer les résultats selon divers paramètres (e.g. le titre, la description, ...).

Mais un fondamental est manquant, la gestion des utilisateurs, sauf si bien sûr vous faites une API ouverte avec de la donnée libre d'accès. Cette gestion permettra surtout de faire l'association entre une donnée et son *propriétaire*.

Settings
========

On commence par changer nos settings. On ajoute l'application "rest_framework.authtoken" qui permettra l'authentification via un token (ce sera utile en production) et dans les settings de DRF nous allons ajouter une restriction : les utilisateurs doivent être authentifiés pour pouvoir utiliser notre API.

.. code-block:: python

    # common.py
    ...
    INSTALLED_APPS = [
        ...
        'rest_framework.authtoken',
        ...
    ]
    ...
    REST_FRAMEWORK = {
        'DEFAULT_PERMISSION_CLASSES': (
            'rest_framework.permissions.IsAuthenticated',
        ),
        'DEFAULT_AUTHENTICATION_CLASSES': (
            'rest_framework.authentication.TokenAuthentication',
        ),
        ...
    }

Dans le fichier de settings pour le développement nous ajoutons ceci pour autoriser l'utilisation de la 'basic authentication'. Cela simplifiera nos tests.

.. code-block:: python

    # development.py
    ...
    # REST Framework
    REST_FRAMEWORK['DEFAULT_AUTHENTICATION_CLASSES'] = (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    )

Création du modèle TodoUser
===========================

Nous allons créer un utilisateur pour notre application. Django fournit un modèle User de base mais nous allons wrapper ce modèle avec notre propre classe et en utilisant une liaison OneToOne. On peut voir cet "utilisateur" plus comme un profile. Cela nous offrira plus de flexibilité si nous voulons ajouter des informations à notre utilisateur dans le futur.

On va dans notre fichier :red:`models.py`

.. code-block:: python

    # models.py
    ...
    from django.contrib.auth.models import User
    ...
    class Todo(models.Model):
        ...
        owner = models.ForeignKey('TodoUser', on_delete=models.CASCADE, null=False)
    ...
    class TodoUser(models.Model):
        d_user = models.OneToOneField(User, on_delete=models.CASCADE)

        def __str__(self):
            return self.d_user.username

Afin de ne pas être limité par le modèle user fourni par Django nous allons commencer par créer le nôtre.

Creation d'un todo: assignation du propriétaire
===============================================

La création de nos Todo est maintenant légèrement modifiée. En plus des informations **title**, **description** et **finished** il nous faut maintenant définir le **owner**. Pour cela on va se baser sur l'utilisateur qui envoie la requête étant donnée que maintenant pour utiliser notre API il faut être authentifié. Pour faire cela nous allons overwrite la méthode create du serializer.

.. code-block:: python

    # serializers.py
    ...
    class TodoSerializer(ModelSerializer):
        ...
        def create(self, validated_data):
            request_user = self.context['request'].user
            todo_user = TodoUser.objects.get(d_user=request_user)
            todo = Todo.objects.create(**validated_data, owner=todo_user)
            todo.save()
            return todo

Filter les résultats
====================

Tout comme la création, la récupération des résultats doit être modifiée. En effet je ne veux voir que les todo que j'ai créé et pas ceux des autres. Il faut donc filtrer les résultats qui vont être retournés. C'est dans le viewset que l'on a défini la queryset c'est donc dans cette classe que nous allons agir pour filtrer les résultats.

.. code-block:: python

    # views.py
    ...
    class TodoViewset(viewsets.ModelViewSet):
        ...
        def filter_queryset(self, queryset):
            todo_user = TodoUser.objects.get(d_user=self.request.user)
            return queryset.filter(owner=todo_user)

Voilà maintenant les résultats retournés sont différents en fonction de l'utilisateur qui fait la requête.

Les sources : `GitHub <https://github.com/MarcAureleCoste/DjangoRestApiTutorial/tree/S03-filters>`_
