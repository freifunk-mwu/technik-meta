Gluon Gateway Doku
==================

In dieser Anleitung wird beschrieben, wie wir unsere Gateway-Server für Freifunk aufsetzen. Die Community Freifunk Mainz, Wiesbaden & Umgebung versorgt mit der Gateway Infrastruktur mehrere Communities, aktuell Mainz und Wiesbaden. Aus diesem Grund sind alle Konfigurationen darauf ausgelegt, sofern möglich und sinnvoll, die Communities logisch zu trennen. An dieser Stelle sei darauf hingewiesen, dass es sich also nicht um eine Gateway Anleitung für den Infrastrukturaufbau für eine einzige Community handelt. Es ist aber durchaus möglich die hier aufgezeigten Konfigurationen dahingehende zu adaptieren.

Ziel des Ganzen soll *nicht* eine Auflistung von Konfigurationen sein, eher wollen wir am Beispiel derer unsere Gedankengänge und Hintergründe dazu beleuchten.

.. note:: Diese Dokumentation ist noch nicht komplett.

Die aktuelle Konfiguration der Gates ist jederzeit im gateway-configs.git_ zu finden.

Grundsätzlich dienen bei der Konfiguration eines neuen Gateways die bekannte Konfiguration der bestehenden Gates als Vorbilder. Trotzdem werden hier wichtige Punkte explizit beschrieben und zusammengefasst.

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
   prerequisites/gluon
   configuration/repos_und_pakete
   configuration/netzwerk
   daemons/fastd
   configuration/interfaces
   daemons/alfred
   daemons/ip
   daemons/bind
   daemons/openvpn
   daemons/tinc
   daemons/bird
   configuration/logging
   operations/workflows
