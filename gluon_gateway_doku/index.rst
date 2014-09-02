Gluon Gateway Doku
==================

In dieser Anleitung wird beschrieben, wie wir unsere Gateway-Server für Freifunk aufsetzen.

Ziel des Ganzen soll *nicht* eine Auflistung von Konfigurationen sein, eher wollen wir am Beispiel derer unsere Gedankengänge und Hintergründe dazu beleuchten.

.. note:: Diese Dokumentation ist noch nicht komplett.

Die aktuelle Konfiguration der Gates ist jederzeit im gateway-configs.git_ zu finden.

Grundsätzlich dienen bei der Konfiguration eines neuen Gate die bekannten Konfiguration der bestehenden Gates als Vorbilder. Trotzdem werden hier wichtige Punkte explizit beschrieben und zusammengefasst.

Geschrieben in Zusammenarbeit von Freifunk-Mainz_ and Freifunk-Wiesbaden_ als freifunk-mwu_ auf GitHub.

Eine augenfreundliche Version dieser Dokumentation findet sich unter gluon-gateway-doku.readthedocs.org_!

.. _gateway-configs.git: https://github.com/freifunk-mwu/gateway-configs/
.. _Freifunk-Mainz: http://www.freifunk-mainz.de/
.. _Freifunk-Wiesbaden: http://www.freifunk-wiesbaden.de/
.. _freifunk-mwu: https://github.com/freifunk-mwu/
.. _gluon-gateway-doku.readthedocs.org: http://gluon-gateway-doku.readthedocs.org/de/latest/

Inhalt
------

.. toctree::
   :maxdepth: 2

   prerequisites/voraussetzungen
   prerequisites/netzplan
   configuration/repos_und_pakete
   configuration/netzwerk
   daemons/fastd
   configuration/interfaces
   daemons/alfred
   daemons/ip
   daemons/bind
