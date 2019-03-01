Django REST API #1 : Installation / API basique
###############################################

:date: 2019-02-20 20:35
:modified: 2019-02-20 20:35
:tags: Python, Django, REST API
:category: Développement
:slug: django-rest-api-01
:authors: Marc-Aurele Coste
:lang: fr

.. role:: red

Nous allons voir ici comment mettre en place les bases de la réalisation d'une API REST avec Django et Django REST Framework (DRF).

Nous verrons ici:

 - La création d'un nouveau projet Django avec Django REST Framework installé
 - La création d'un modèle
 - La création d'un sérialiser pour ce modèle
 - La création d'un viewset

Installation et création du projet
==================================

On commence par la création d'un virtualenv dans lequel on va installer Django et Django REST Framework. Lorsque l'installation est terminée on crée le projet.

.. code-block:: sh

    pip install Django djangorestframework
    django-admin startproject <site name>


.. note:: Pour la suite, le nom du projet sera **DjangoRestApiTutorial**

Settings
========

Actuellement un unique fichier de settings est présent. Nous allons le remplacer par un dossier avec le même nom et nous repartirons ensuite les settings dans plusieurs fichiers:

- :red:`common.py` : Settings communs entre production et développement
- :red:`development.py` : Settings utilisés en développement
- :red:`production.py` : Settings utilisés en production

.. code-block:: sh

    cd DjangoRestApiTutorial/DjangoRestApiTutorial
    mkdir settings && cd settings
    touch __init__.py common.py development.py production.py
    cat ../settings.py > common.py
    rm ../settings.py

On supprime de :red:`common.py`

- DEBUG
- ALLOWED_HOSTS
- DATABASES

que l'on ajoute à :red:`development.py` et :red:`production.py`.

.. code-block:: python

    # development.py
    """Development settings"""
    from .common import *


    # SECURITY WARNING: don't run with debug turned on in production!
    DEBUG = True

    ALLOWED_HOSTS = []

    # Database
    # https://docs.djangoproject.com/en/2.1/ref/settings/#databases
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        }
    }

.. code-block:: python

    # production.py
    """Production settings"""
    from .common import *


    DEBUG = False

    ALLOWED_HOSTS = []

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        }
    }

Pour finir nous ajoutons **rest_framework** dans la liste des **INSTALLED_APPS** dans note fichier :red:`common.py`. On peut maintenant lancer notre application et verifier qu'il n'y a aucune erreur.

.. code-block:: sh

    ./manage.py runserver --settings=DjangoRestApiTutorial.settings.development

Rendez-vous sur `localhost:8000 <http://localhost:8000>`_ et vous devriez voir une page ressemblant à l'image ci-dessous.

.. image:: {static}/static/images/django_rest_api/install_success.png
    :width: 2373 px
    :height: 1215 px
    :scale: 40 %
    :alt: install success
    :align: center

Les sources pour le mise en place du project sont disponibles `ici <https://github.com/MarcAureleCoste/DjangoRestApiTutorial/tree/S01-installation>`_, branche **S01-installation**.

Création de l'API
=================

.. code-block:: sh

    # Je crée un dossier dans lequel je mettrais toutes mes applications.
    # Vous pouvez, si vous préférez, tout mettre à la racine dans ce cas la sautez les 3 lignes qui suivent.
    django-admin startapp apps
    cd apps
    rm -r migrations admin.py models.py tests.py views.py
    
    django-admin startapp todo

Dans le fichier :red:`models.py` on va définir note modèle de Todo.

.. code-block:: python

    class Todo(models.Model):
        title = models.CharField(max_length=150, null=False, blank=False)
        description = models.TextField()
        finished = models.BooleanField(default=False)

On ajoute notre nouvelle application dans les **INSTALLED_APPS** du fichier :red:`common.py`.

.. code-block:: python

    # ...
    INSTALLED_APPS = [
        #...

        # REST Framework
        'rest_framework',

        # Our Todo app
        'apps.todo',
    ]
    # ...

On peut maintenant créer notre première migration pour notre application *todo* et appliquer cette migration (cela appliquera également les migrations que nous n'avons pas faite dans la première partie).

.. code-block:: sh

    ./manage.py makemigrations --settings=DjangoRestApiTutorial.settings.development
    ./manage.py migrate --settings=DjangoRestApiTutorial.settings.development

Il nous reste trois étapes pour finir la création de notre API.

- Création d'un sérialiser

*serializers.py*

.. code-block:: python

    from rest_framework.serializers import ModelSerializer
    from .models import Todo

    class TodoSerializer(ModelSerializer):
        class Meta:
            model = Todo
            fields = (
                'id',
                'title',
                'description',
                'finished'
            )
            extra_kwargs = {
                'id': {'read_only': True}
            }

- Création d'un viewset

*views.py*

.. code-block:: python

    from rest_framework import viewsets

    from .models import Todo
    from .serializers import TodoSerializer


    # Create your views here.
    class TodoViewset(viewsets.ModelViewSet):

        queryset = Todo.objects.all()
        serializer_class = TodoSerializer

- Mise en place des URLs

*urls.py*

.. code-block:: python

    from rest_framework.routers import DefaultRouter
    from .views import TodoViewset


    TODO_ROUTER = DefaultRouter()
    TODO_ROUTER.register(r'', TodoViewset)

Voilà pour finir nous importons ce router dans le fichier :red:`urls.py` qui se situe dans le dossier **DjangoRestApiTutorial** qui est notre fichier principal d'url.

.. code-block:: python

    from django.contrib import admin
    from django.urls import path, include

    from apps.todo.urls import TODO_ROUTER


    API_URL = [
        path(r'todo', include(TODO_ROUTER.urls))
    ]

    urlpatterns = [
        path('admin/', admin.site.urls),

        path(r'api/', include(API_URL)),
    ]

Vous pouvez maintenant vous rendre sur `localhost:8000/api/todo <http://localhost:8000/api/todo>` afin de tester votre API.

:GET /api/todo/: Récupération de la liste des todos
:POST /api/todo/: Création d'un nouveau todo
:GET /api/todo/<id>: Récupération du todo avec l'id *<id>*
:PUT /api/todo/<id>: Update du todo avec l'id *<id>*
:DELETE /api/todo/<id>: Suppression du todo avec l'id *<id>*

-----

**Partie 02** : `Les filtres <{filename}/fr/django_rest_api_02.rst>`_