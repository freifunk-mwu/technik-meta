.. _scripts:

Backend Scripte
===============

Um sich das Leben einfacher zu machen, werden immer wiederkehrende Probleme mittels Scripte gelöst.

Diese werden in einem Repository gesammelt und nutzen selbst Photon_ als Backend.

:Repository: `freifunk-mwu/backend-scripts <https://github.com/freifunk-mwu/backend-scripts>`_
:Backend: `Photon <https://github.com/spookey/photon>`_

Installation
------------

Photon liegt im neuesten Release im `Python Package Index <https://pypi.python.org/pypi/photon_core/>`_ und wird wiefolgt installiert::

    sudo pip3 install -U photon_core --pre

Die eigentlichen Backend Scripte werden normal geklont::

    mkdir -p ~/clones/backend-scripts
    git clone https://github.com/freifunk-mwu/backend-scripts.git ~/clones/backend-scripts

Einrichten
----------

Um den Backend Scripten (Schreib-) Zugriff auf die benötigten Repos unter GitHub zu gewähren müssen diese erst eingerichtet werden. Dazu dient das Script :file:`bootstrap_git_all.py`::

    python3 ~/clones/backend-scripts/bootstrap_git_all.py.py

Dieses erzeugt ein SSH-Schlüsselpaar und legt es unter :file:`~/.ssh/%hostname%_rsa` ab sofern dieses noch nicht existiert. Die :file:`~/.ssh/config` wird um einen Eintrag erweitert (sofern dieser noch nicht existiert), so dass ein Zugriff auf GitHub mittels frisch erzeugtem Schlüssel möglich ist.

Momentan: ``github_mwu``

Den Öffentlichen Teil des Schlüssels bekommt ein extra erstellter GitHub Account hinterlegt, der duch eine Gruppen Zugriff auf die entsprechenden Repos erhält.

Für uns ist dies der Nutzer `freifunkmwu <https://github.com/freifunkmwu>`_ der innerhalb der Gruppe `machines <https://github.com/orgs/freifunk-mwu/teams/machines>`_ verweilt.

Ob alles korrekt funktioniert lässt sich durch einen einfachen Aufruf zeigen::

    ssh github_mwu

Die Ausgabe sollte folgendes enthalten::

    Hi freifunkmwu! You've successfully authenticated, but GitHub does not provide shell access.
    Connection to github.com closed.

Die restlichen Scripte funktionieren dann analog.
