.. _voraussetzungen:

Voraussetzungen
===============

* voller Root Zugriff für alle Admins
* öffentliche IPv4 Adresse
* öffentliche IPv6 Adresse
* voller Kernelzugriff
    * Linux direkt installiert,
    * oder als KVM-Instanz (OpenVZ o.ä. nicht möglich)
* Momentan: Ubuntu 14.04 LTS Server

Betrieb und Zugang
------------------

Wir stellen mit einem oder mehreren Gateways in Vereinshand eine Basisversorgung sicher.

Gerne sollen weitere Gateways aufgesetzt werden. Redundanz schafft Vertrauen!

Wer ein weiteres Gate betreiben möchte, muss sich vor der Installation in einigen technischen Punkten mit der Admin-Gruppe abstimmen. Möglichst viele Informationen sollen auch in dieser Dokumentation zu finden sein.

Am Besten wird der Gate-Betreiber gleich Teil des Admin-Teams :)

Jeder Admin muss die Möglichkeit haben jederzeit an jeder Stelle jedes beliebige Problem fixen zu können (und zu dürfen). Die technischen Details zur Umsetzung sind der Admin-Gruppe bekannt.

Funktionen & dafür benötigte Software
-------------------------------------

Momentan kommen auf allen Gateways eine Ubuntu Server 14.04 LTS Installation zum Einsatz. Jeweils angepasst auf die entsprechenden Plattformen (Direkt auf dem Host, oder als KVM-Instanz).

Für viele der Gate-Funktionen wird weitere Software benötigt, die größtenteils als Pakete aus den Ubuntu- und weiteren Repositories nachinstalliert werden kann.

:see:
    - :ref:`packages`
    - :ref:`repositories`

Verbindung der Teilwolken
^^^^^^^^^^^^^^^^^^^^^^^^^

Die Nodes bauen ihr Freifunk mit dem Mesh-Protokoll *B.A.T.M.A.N* selbständig auf.

Da das Netz aber noch nicht dicht genug ist, dass alle Nodes sich über ihre Funk-Interfaces erreichen können, helfen wir über eine VPN-Verbindung über das Internet nach, um einzelne Wolken zu überbrücken.

So wie über ihre Funk-Interfaces, sprechen die Nodes das *B.A.T.M.A.N* Protokoll auch über ihre VPN-Verbindungen.

Die Gates agieren als Fokuspunkte für dieses VPN. d.h. alle am VPN teilnehmenden Nodes verbinden sich zu den Gates und Nodes und Gates spannen so das VPN auf.

Alle Gates verbinden sich für das VPN auch vollständig untereinander. So entsteht eine Multi-Stern-Architektur. Alle Sternmittelpunkte sind untereinander voll vermascht.

Auf den Gates werden hierzu **fastd** sowie das Kernel Modul **batman_adv** benötigt.

:see:
    - :ref:`packages`
    - :ref:`fastd`

Mainz und Wiesbaden
^^^^^^^^^^^^^^^^^^^

*Freifunk Mainz* und *Freifunk Wiesbaden* werden als zwei getrennte Netze betrieben.

Das soll von Beginn an die Mesh-Wolken kleiner halten. Mesh-Protokolle skalieren nicht sehr gut, wenn viele Knoten teilnehmen. (Grenze ab ca. 200 Nodes)

Das bedeutet, dass es zwei *B.A.T.M.A.N* Wolken gibt, was wiederum bedeutet, dass es auch zwei **fastd** VPNs pro Gateway gibt.

Aus Gründen der Effizienz unterstützen die Gates aktuell jeweils beide Netze.

:see:
    - :ref:`netzplan`

Sollten die Netze in Zukunft stark wachsen, könnten weitere Aufteilungen nötig werden.


DHCP für die Clients
^^^^^^^^^^^^^^^^^^^^

Nodes, Clients und Gates sind über ein Layer-2-Netz miteinander verbunden (*B.A.T.M.A.N*, s. o.).

.. note:: Oben ist zu lesen, dass es zwei Layer-2-Netze gibt. Im Folgenden wird immer eines davon betrachtet; das andere funktioniert analog.

Die Gates halten auch als DHCP-Server für die Clients her.

Verfügbare IPv4-Ranges, aus denen der DHCP-Server vergeben darf, müssen innerhalb bzw. mit der Admin-Gruppe abgestimmt werden, und werden im :ref:`netzplan` festgehalten.

Hierfür wird **isc-dhcp-server** genutzt. Für das vorbereitende Ausrollen von IPv6 Adressen benötigen wir hier auch **radvd**.

:see:
    - :ref:`dhcp`
    - :ref:`radvd`

Übergang ins restliche Internet
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Der Übergang ins Internet wird durch einen VPN-Tunnel nach Schweden oder in die Niederlande (ipredator.se, mullvad.net) getunnelt - im Falle von IPv4 ist das auch kaum anders zu realisieren, da die verwendeten Netze 10.37.0.0/16 und 10.56.0.0/16 im Internet nicht geroutet werden.

Zu diesem Zweck wird ein weiteres VPN zu einem Anbieter aufgebaut und aller Freifunk-Traffic dort entlang geschickt.

Damit dies gelingt muss auch dem Gate, in Richtung des Anbieters auch ein NATing (masquerading) erfolgen. Zur besseren Administrierbarkeit wird jedes *B.A.T.M.A.N*-Interface noch in jeweils einer Netzwerk-Bridge gekapselt.

An dieser Stelle wird einiges an zusätzlicher Software gebraucht: **bridge-utils**, **iproute**, **iptables** & **openvpn**.

:see:
    - :ref:`packages`
    - :ref:`interfaces`
    - :ref:`routing_tables`
    - :ref:`openvpn`

Übergang zu anderen Freifunk-Communities (InterCityVPN)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Wie auch bei uns, so sind auch die IPv4-Netze der anderen Freifunk-Communities nicht über das restliche Internet zu erreichen.

Damit interne Dienste auch aus anderen Städten genutzt werden können, wurde das IC-VPN als Verbindung der Freifunk-Communities untereinander in's Leben gerufen.

Als Software benutzen wir hier **tinc** und **bird6**.

:see:
    - :ref:`packages`
    - :ref:`interfaces`
    - :ref:`routing_tables`
    - icvpn
        - :ref:`tinc`
        - :ref:`bird`

Datenschutz auf dem Gateway
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Unsere Gateways loggen keinen Traffic!

Alles was existiert sind die zur Laufzeit benötigten Verbindungsdaten. DHCP-Leases, Batman Protokolldaten und die ARP-Tabelle.

Diese werden nur im Arbeitsspeicher vorgehalten, ist das Gateway aus (z.B. die Herren in Grün nehmen den Server mit), sind diese weg.

