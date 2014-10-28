.. _netzplan:

Netzplan
========

Freifunk Mainz & Freifunk Wiesbaden sind TCP/IP-basierte Netzwerke. Deshalb haben die beteiligten Computer (Clientes, Gates und die Knoten) jeweils IP-Adressen, die eindeutig sein müssen.

Wird (z. B. aus Versehen) eine Adresse mehrfach verwendet, so führt das zu Problemen im Betrieb.

Andererseits müssen alle Adressen im *Freifunk Mainz* oder *Freifunk Wiesbaden* zum dem selben Adressbereich, sowie zum selben Netz gehören.

IPv4
----

Wir haben von der Freifunkcommunity folgende Netze zugewiesen bekommen:

* Mainz - ``10.37.0.0/16`` = **37**
* Wiesbaden - ``10.56.0.0/16`` = **56**

Datenpakete aus diesen Adressbereichen werden innerhalb des Freifunks vermittelt, im großen weiten Internet aber nicht geroutet (Hintergrundinformationen dazu `gibt es hier`_).

.. _gibt es hier: http://de.wikipedia.org/wiki/Private_IP-Adresse#Adressbereiche

Unser großes ``10.X.0.0/16`` (mit 65536 Adressen) teilen wir uns ein wenig ein:

``10.X.0.0/16`` wird nicht komplett genutzt, um in Zukunft noch was auf Halde zu haben. Wachsen ist immer einfacher als schrumpfen.

Wir nutzen von diesem Netz erstmal das untere Viertel: ``10.X.0.0/18``:

================ =============== ================= =============== ==========
Netz             (bis)           Verwendung        verteilt durch  status
================ =============== ================= =============== ==========
``10.X.0.0/24``  ``10.X.0.255``  Gateways          fix             in Betrieb
``10.X.1.0/24``  ``10.X.1.255``  ¬                 ¬               frei
``10.X.2.0/24``  ``10.X.2.255``  ¬                 ¬               frei
``10.X.3.0/24``  ``10.X.3.255``  ¬                 ¬               frei
``10.X.4.0/22``  ``10.X.7.255``  User Services     fix             in Betrieb
``10.X.8.0/22``  ``10.X.15.255`` ¬                 ¬               frei
``10.X.16.0/20`` ``10.X.31.255`` Client DHCP-Range Lotuswurzel     in Betrieb
``10.X.32.0/20`` ``10.X.47.255`` Client DHCP-Range Spinat          in Betrieb
``10.X.48.0/20`` ``10.X.63.254`` Client DHCP-Range Hinterschinken  in Betrieb
================ =============== ================= =============== ==========

IPv6
----

Wir nutzen das `IPv6 ULA Prefix`_ ``fdXX:b4dc:4b1e::/48``

Dieses ist bei SixXS_ an zentraler Stelle registriert, siehe `fd37:b4dc:4b1e\:\:/48`_ oder `fd56:b4dc:4b1e\:\:/48`_

IPv6 Subnetze haben immer eine Prefix-Länge von *64 Bit*. Durch das /48 Subnetz stehen uns also 2^16 = 65536 /64 IPv6 Subnetze zur Verfügung.

Fürs erste wird allein dieses IPv6 Subnetz verwendet: ``fdXX:b4dc:4b1e:0000::/64`` (abgekürzt: ``fdXX:b4dc:4b1e::/64``, siehe `Address Notation`_).

.. _IPv6 ULA Prefix: http://de.wikipedia.org/wiki/IPv6#Unique_Local_Unicast
.. _SixXS: https://www.sixxs.net/
.. _fd37:b4dc:4b1e\:\:/48:  https://www.sixxs.net/tools/whois/?fd37:b4dc:4b1e\:\:/48
.. _fd56:b4dc:4b1e\:\:/48:  https://www.sixxs.net/tools/whois/?fd56:b4dc:4b1e\:\:/48
.. _Address Notation: http://de.wikipedia.org/wiki/IPv6#Adressnotation


.. _interface_bezeichnung:

Interface Bezeichnung
---------------------

Wir vergeben unsere Interface-Bezeichnungen einheitlich!

+-----------+------+--------------+-------------+--------+---------------+----------+
|           | eth0 | Mesh (Nodes) | B.A.T.M.A.N | Bridge | Intercity-VPN | Exit VPN |
+-----------+------+--------------+-------------+--------+---------------+----------+
| Mainz     |      | mzVPN        | mzBAT       | mzBR   |               |          |
+-----------+ eth0 +--------------+-------------+--------+ icVPN         + exitVPN  +
| Wiesbaden |      | wiVPN        | wiBAT       | wiBR   |               |          |
+-----------+------+--------------+-------------+--------+---------------+----------+

Dies erleichtert das Scripten und Debuggen.

Namenskonvention
----------------

Als *Hostname* der Gateways kommt **"irgendwas mit Nahrung"** zum Einsatz :)


.. _next_node:

Next Node Adressen
------------------

Die Next Node Adressen sind dafür da, um sich im Fehler- oder Troublehootingfall mit einem Freifunk Knoten zu verbinden.

Diese Adressen sind auf jedem Knoten gleich. Der Freifunker muss sich nur diese Adresse(n) merken und seine Netzwerkkarte für ein Subnetz konfigurieren, in dem diese Adresse(n) liegt.

Wir nutzen dazu die jeweils niedrigsten Adressen

* Mainz:
    * IPv4: ``10.37.0.1``
    * IPv6: ``fd37:b4dc:4b1e::1``

* Wiesbaden:
    * IPv4: ``10.56.0.1``
    * IPv6: ``fd56:b4dc:4b1e::1``

    ..

.. _gateway_schema:

Gateway-Schema
--------------

Bevor wir ein Gateway aufsetzen definieren wir einen Namen (leichter zu merken) und ziehen eine Nummer (1 <= x < 255).

Mit den uns zugewiesenen Netznummern sowie der Gateway-Nummer und dem Gateway-Namen werden alle benötigten Informationen abgeleitet:

* IPv4
    * Das Netz von unten auffüllen (``10.x.0.0/24`` ist für Gateways)

* MAC-Adresse
    * Privates Prefix (``02:00``) + IPv4-Adresse in hexadezimal

    * Beispiele (für Mainz):
        * 10.37.0.1 -> ``02:00:0a:25:00:01``
        * 10.37.23.42 -> ``02:00:0a:25:17:2a``
        * 10.37.254.2 -> ``02:00:0a:25:fe:02``

    * Beispiele (für Wiesbaden):
        * 10.56.0.1 -> ``02:00:0a:38:00:01``
        * 10.56.23.42 -> ``02:00:0a:38:17:2a``
        * 10.56.254.2 -> ``02:00:0a:38:fe:02``

    * Wer zu faul zum rechnen ist, darf auch gerne den IP2MAC-Konverter_ nutzen.

.. _IP2MAC-Konverter: http://www.freifunk-mainz.de/mac.html

* IPv6
    * Range-Prefix (``fd37:b4dc:4b1e`` bzw. ``fd56:b4dc:4b1e``) + IPv4 Adresse in hexadezimal, Doppelpunkte anpassen, führende Nullen streichen

    * Beispiele (für Mainz):
        * gate02 -> ``fd37:b4dc:4b1e::0a25:0002/64``
        * gate05 -> ``fd37:b4dc:4b1e::0a25:0005/64``

    * Beispiele (für Wiesbaden, abgekürzt):
        * gate02 -> ``fd56:b4dc:4b1e::a38:2/64``
        * gate23 -> ``fd56:b4dc:4b1e::a38:17/64``

* DNS
    * ``xxxx.freifunk-mwu.de`` -> A- + AAAA-Record
    * ``gateXX.freifunk-mwu.de`` -> CNAME auf s.o.
    * Reverse DNS Eintrag korrekt setzen für Haupt DNS Namen: ``xxxx.freifunk-mwu.de``


Beispiel
--------

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
DNS1        ``lotuswurzel.freifunk-mwu.de``   ``lotuswurzel.freifunk-mwu.de``
DNS2        ``lotuswurzel.ffmz.org``          ``lotuswurzel.ffwi.org``
CNAME1      ``gate23.freifunk-mwu.de``        ``gate23.freifunk-mwu.de``
CNAME2      ``gate23.ffmz.org``               ``gate23.ffwi.org``
=========== ================================= =====================================
