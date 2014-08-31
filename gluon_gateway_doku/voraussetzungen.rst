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

.. _gateway_schema:

Gateway-Schema
--------------

Bevor wir ein Gateway aufsetzen definieren wir einen Namen (leichter zu merken) und ziehen eine Nummer (1 <= x < 255).

Wir haben von der Freifunkcommunity folgende Netze zugewiesen bekommen:

* Mainz - ``10.37.0.0/16`` = **37**
* Wiesbaden - `10.56.0.0/16`` = **56**

Mit der uns zugewiesenen Netznummer sowie der Gateway-Nummer und dem -Namen werden alle benötigten Informationen abgeleitet:

* IPv4
    * Das Netz von unten auffüllen (x.x.0.0/24 ist für Gateways)

* MAC-Adresse
    * Privates Prefix (02:00) + IPv4-Adresse in hexadezimal

* IPv6
    * Range-Prefix (``fd37:b4dc:4b1e`` bzw. ``fd56:b4dc:4b1e``) + IPv4 Adresse in hexadezimal, Doppelpunkte anpassen, führende Nullen streichen

* DNS
    * ``xxxx.freifunk-mainz.de`` -> A- + AAAA-Record
    * ``xxxx.freifunk-wiesbaden.de`` -> A- + AAAA-Record
    * ``gateXX.freifunk-mainz.de`` -> CNAME auf s.o.
    * ``gateXX.freifunk-wiesbaden.de`` -> CNAME auf s.o.

    * Reverse DNS Eintrag korrekt setzen für Haupt DNS Namen, es geht nur einer -> Mainz


Beispiel
^^^^^^^^

Gateway: **Lotuswurzel** - Nummer: **23**

Zahlen umwandeln:

==== =====
dec  hex
==== =====
10   0a
37   25
 0   00
23   17
56   38
==== =====

und einsetzen:

=========== ================================= =====================================
Lotuswurzel Mainz                             Wiesbaden
=========== ================================= =====================================
IPv4        ``10.37.0.23``                    ``10.56.0.23``
IPv6        ``fd37:b4dc:4b1e:0a25:00017``     ``fd37:b4dc:4b1e:a38:17``
MAC         ``02:00:0a:25:00:17``             ``02:00:0a:38:00:17``
DNS1        ``lotuswurzel.freifunk-mainz.de`` ``lotuswurzel.freifunk-wiesbaden.de``
DNS2        ``lotuswurzel.ffmz.org``          ``lotuswurzel.ffmz.org``
DNS1        ``gate23.freifunk-mainz.de``      ``gate23.freifunk-wiesbaden.de``
DNS2        ``gate23.ffmz.org``               ``gate23.ffmz.org``
=========== ================================= =====================================

Interfaces
^^^^^^^^^^

Wir vergeben unsere Interface-Bezeichnungen einheitlich!
Dies erleichtert das Scripten und Debuggen.

+-----------+------+--------------+--------+-------------+---------------+----------+
|           | eth0 | Mesh (Nodes) | Bridge | B.A.T.M.A.N | Intercity-VPN | Exit VPN |
+-----------+------+--------------+--------+-------------+---------------+----------+
| Mainz     |      | mzVPN        | mzBR   | mzBAT       |               |          |
+-----------+ eth0 +--------------+--------+-------------+ icVPN         + exitVPN  +
| Wiesbaden |      | wiVPN        | wiBR   | wiBAT       |               |          |
+-----------+------+--------------+--------+-------------+---------------+----------+
