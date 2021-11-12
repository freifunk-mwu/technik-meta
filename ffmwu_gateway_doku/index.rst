Freifunk MWU Gateway Doku
=========================

.. warning::

   Diese Dokumentation ist **veraltet** und entspricht nicht mehr unserer aktuellen Gateway-Konfiguration. Sie sollte nur als Anhaltspunkt genutzt werden um eigene Konzepte für Freifunk-Gateways zu entwickeln!

   Wir konfigurieren alle unsere Server mittlerweile per Ansible. Die Rollen liegen unter https://github.com/freifunk-mwu/ansible-ffmwu

.. include:: Readme.rst

Intro
-----

.. toctree::
   :maxdepth: 2

   intro/prerequisites
   intro/netzplan
   intro/gluon

Konfiguration
-------------

.. toctree::
   :maxdepth: 2

   configuration/basics
   configuration/interfaces
   configuration/firewall
   configuration/policyrouting
   configuration/daemons/ddi
   configuration/daemons/fastd
   configuration/daemons/alfred
   configuration/daemons/apache
   configuration/daemons/icvpn
   configuration/internetexit
   configuration/cleanup

Betrieb
-------

.. toctree::
   :maxdepth: 2

   operations/tests
   operations/workflows
   operations/scripts

Mitwirken
---------

Du hast eine Stelle entdeckt, die Schreibfehler enthält, was unsauber oder falsch erklärt, nicht existiert aber sollte, oder veraltet ist?

Gerne nehmen wir sinnvolle Vorschläge und Ergänzungen mit auf, über den `Issue-Tracker <http://github.com/freifunk-mwu/technik-meta/issues>`_ (mit dem `Label 'Doku' <https://github.com/freifunk-mwu/technik-meta/labels/Doku>`_) oder am besten gleich per Pull Request.
