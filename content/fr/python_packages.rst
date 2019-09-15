Packager ses applications en python
###################################

:date: 2019-09-15 19:15
:tags: Python, Packaging
:category: Développement
:slug: python-packages
:authors: Marc-Aurele Coste
:lang: fr

.. role:: red

Lorsque l'on se lance dans le développement en général la partie packaging n'est pas une priorité,
c'est même souvent la partie que l'on garde pour la fin. C'est malheureusement une erreur et python
n'échappe pas à la règle. Nous verrons dans cet article les nombreux avantages que peut nous apporter
le fichier :red:`setup.py` durant pour le développement.

Un package basic
================

Commençons en douceur avec un package des plus basic. Une fois installé, ce package nous permettra
d'importer la fonction que l'on aura définie et on pourra l'utiliser facilement dans d'autres projets.

Pour vous aider, vous trouverez le code de cette étape sur GitHub
`step-01 <https://github.com/MarcAureleCoste/packages/tree/step-01>`_.

La pièce maitresse dans la création d'un paquet python, comme expliqué en introduction, c'est le
:red:`setup.py`, et voilà à quoi cela peut ressembler.

.. code-block:: python

    # setup.py
    from setuptools import setup, find_packages

    NAME: str = 'simple_package'
    VERSION: str = '0.1.0'
    DESCRIPTION: str = 'A simple python package.'

    REQUIRES: list = []


    setup(
        name=NAME,
        version=VERSION,
        description=DESCRIPTION,

        zip_safe=False,

        install_requires=REQUIRES,

        packages=find_packages('src'),
        package_dir={'': 'src'},
    )

Ce n’est finalement que l’appel à une fonction setup avec les bons paramètres rien de plus. Ici
nous passons à cette fonction le nom du package que nous voulons créer, sa version, une description
et ses requirements. Nous expliquons aussi à cette fonction où trouver les sources de notre
package.

Notre projet est structuré de la manière suivante :

::

    root/
    |-- src/
    |   |-- simple_package/
    |       |-- __init__.py
    |-- README.md
    |-- setup.py

Nous expliquons donc à la fonction setup de prendre tous les packages qu’elle trouvera dans le
dossier src (packages=find_packages('src')) et nous lui précisons aussi que le dossier qui contient
les packages est src (package_dir={'': 'src'}).

Ok c'est super tout ça mais on en fait quoi de ce :red:`setup.py` ? Et bien vous pouvez maintenant
créer votre package, l'installer et même le mettre sur `pypi <https://pypi.org/>`_ (le repository
des packages python). Nous verrons cette dernière étape à la fin de l’article.

.. code-block:: sh

    # build
    python setup.py build

    # build distribution package
    python setup.py sdist

    # installation
    pip install .

    # installation dev mode
    pip install -e .

Gestion des requirements
========================

Maintenant, voyons comment gérer facilement les dépendances. Les fichiers requirements*.txt peuvent
sembler assez limités, surtout dans le cas où l'on veut séparer ses requirements de
**developement**. Même s'ils sont limités, nous n'allons pas supprimer #####nos fichiers de
requirements mais plutôt essayer de les intégrer dans notre :red:`setup.py`.

Le fichier de setup est un fichier python comme un autre et par conséquent on peut faire tout ce que
le langage nous autorise, notamment lire un fichier pour en extraire ce que l'on veut (dans notre
cas ce sera les requirements)

On ajoute les éléments suivants pour permettre la gestion des dépendances

.. code-block:: python

    # setup.py
    import os
    # ...

    def _get_requirements(filename):
    """Return the requirements as a list of string."""
    requirements_path = os.path.join(os.path.dirname(__file__), filename)
    with open(requirements_path) as f:
        return f.readlines()

    # ...

    REQUIRES: list = _get_requirements('requirements.txt')
    REQUIRES_DEV: list = _get_requirements('requirements-dev.txt')

    setup(
        # ...

        extras_require={"dev": REQUIRES_DEV}
    )

On peut maintenant installer notre package comme avant, ce qui installera aussi ses dependances,
mais on peut également l'installer avec ses dépendances de développement, pour cela rien de plus
simple, on utilise la commande suivante.

.. code-block:: sh

    # installation dev mode with dev dependencies
    pip install -e .[dev]

Vérifions que notre package est bien installé en utilisant une console python.

.. code-block:: python

    >>> from simple_package import str_to_datetime
    >>> str_to_datetime('2019-01-01')
    # datetime.datetime(2019, 1, 1, 0, 0, tzinfo=tzutc())

Et voilà on maîtrise maintenant la création du paquet python avec une gestion de ses dépendances
grâce au :red:`setup.py`. Nous allons voir dans la suite de ce post un autre atout que peut nous
apporter notre script de packaging à savoir la mise en place de commandes qui seront directement
accessibles depuis votre terminal.

Comme toujours le code de cette partie est disponible ici
`step-02 <https://github.com/MarcAureleCoste/packages/tree/step-02>`_.

Ajouter une commande
====================

Nous avons déjà une super librairie qu'on peut installer facilement avec une
gestion des dépendances mais on va voir que packager notre application peut
également nous permettre de créer des outils directement accessibles en ligne de
commande.

On va modifier notre fichier :red:`setup.py` pour lui ajouter ce que l'on
appelle les **entry_points**

.. code-block:: python

    # setup.py
    # ...
    setup(
        # ...

        entry_points={
            "console_scripts": [
                "now=simple_package.dates:now",
                "hour+1=simple_package.dates:plus_hour",
                "hour-1=simple_package.dates:minus_hour",
            ]
        }
    )

Pour expliquer simplement, les **entry_points** sont les fonctions de notre paquet python que nous
voulons appeler depuis la console lorsque celui-ci est installé. Les entrées se composent de la
manière suivante:

``<command_name>=<package>.<module>:<function>``

Nous ajoutons donc ici trois nouvelles commandes (now, hour+1, hour-1) qui vont appeler les fonctions
now, plus_hour, minus_hour qui se trouvent dans le package simple_package, dans le module date;
là où nous avons défini notre fonction str_to_datetime.

Ajoutons donc ces trois nouvelles fonctions

.. code-block:: python

    # dates.py
    # ...

    def now():
    return arrow.now()


    def plus_hour():
        return arrow.now().shift(hours=1)


    def minus_hour():
        return arrow.now().shift(hours=-1)

On peut maintenant lancer ces fonctions directement depuis notre shell.

Les sources c'est ici `step-03 <https://github.com/MarcAureleCoste/packages/tree/step-03>`_.

Uploader notre package sur PyPi
===============================

.. _`PyPi de test`: https://test.pypi.org/



La dernière étape de cet article est la mise à disposition de notre package via le repository officiel
`PyPi <https://pypi.org/>`_. Il existe également un `PyPi de test`_ pour vérifier que tout se
passe bien lorsque l'on essay de mettre notre package en ligne. Dans les deux cas il vous faudra un compte si vous n'en
avait pas déjà un.

Pour nous aider nous allons utiliser l'utilitaire **twine**.

.. code-block:: sh

    pip install twine

Maintenant on va build notre **distribution package**. C'est le résultat de cette opération
qui sera ensuite mis sur PyPi. On lance donc la commande suivante:

.. code-block:: sh

    python setup.py sdist bdist_wheel

Nous somme prets a uploader tout ca et on va commencer par faire un test sur le PyPi prévu à cet effet.

.. code-block:: sh

    twine upload --repository-url https://test.pypi.org/legacy/ dist/*



Et voilà notre paquet est maintenant disponible, enfin presque. On peut quand se rendre sur le `PyPi de test`_ pour voir
si notre paquet est bien disponible. Normalement on devrait voir une page qui ressemble à ça :

.. image:: {static}/static/images/python_packages/no_description.png
    :alt: pypi no description
    :align: center
    :class: responsive-images

Notre paquet est là c'est une bonne chose mais on remarque vite qu'il y a un problème, aucune description n'est présente
bien que dans notre :red:`setup.py` on ait bien défini la **description**. En réalité pour voir apparaitre cette fameuse
description il faut définir la **long_description**.

Nous avons vu plus haut que le fichier :red:`setup.py` est un fichier python comme les autres, nous allons une nouvelle
fois utiliser cet avantage pour avoir rapidement cette fameuse **long_description**.

En effet si vous avez plusieurs projets sur GitHub vous avez probablement créer des README.md afin d'expliquer aux
personnes qui arrivent sur votre repository en quoi consiste votre projet. Nous allons utiliser ce fichier README comme
**long_description**, pour cela on fait les modifications suivantes.


.. code-block:: python

    # setup.py
    # ...

    def read(file_path: str):
        """Simply return the content of a file."""
        with open(file_path) as f:
            return f.read()


    # ...
    VERSION: str = "0.1.1"
    # ...

    setup(
        # ...
        long_description=read(os.path.join(os.path.dirname(__file__), 'README.md')),
        long_description_content_type='text/markdown',
        # ...
    )

Conclusion
==========

Nous savons maintenant créer des paquets python mais nous avons surtout compris en quoi leur création nous simplifie la
vie. Maintenant lorsque vous commencerez un développement en python j'espère que vous ferez le meilleur usage possible
du setup.