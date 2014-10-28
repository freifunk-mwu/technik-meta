.. _icvpn:

IC-VPN
======

Das IC-VPN beruht darauf, dass zwischen den teilnehmenden communities ein
IP-Routing (layer 3) stattfindet. Dafür wird über :ref:`tinc` ein Transfernetz
zwischen einigen Gates aufgebaut. Über dieses Transfernetz wird dann
ge-route-t.

Die Routing-Einträge auf diesen vielen Routern (Gates) werden nicht
manuell, sondern per `BGP`_ gepflegt (die `ASN`_ für Wiesbaden und Mainz sind 65036 und 65037).
Es gibt zwei etablierte BGP-Implmentationen:
quagga_ und den `bird daemon`_; wir haben uns für letztere entschieden. Auch hier folgen
wir grob der zentralen `Dokumentation`_ und es sei auf das im Aufbau befindliche
`IC-VPN-Meta-repository`_ für die Metainformationen sowie auf das `IC-VPN-scripts-repository`_ für die Erzeugung der bgp peers sowie DNS Config verwiesen.

.. _bird:

BIRD
----

dir structure
^^^^^^^^^^^^^

bird wird für IPv4 und IPv6 gesondert konfiguriert, wobei sich die config files allerdings sehr
ähneln. Da die Einträge für die Nachbarrouter im IC-VPN (*peers*) in Kürze halbautomatisch
gepflegt werden sollen und die bird-Konfiguration das Einbinden von config files in config
files erlaubt, werden die peers schon jetzt ausgelagert. Damit ergibt sich diese Dateistruktur::

  /etc/bird/
  /etc/bird/bird.conf
  /etc/bird/ebgp_peers_v4.inc
  /etc/bird/bird6.conf
  /etc/bird/ebgp_peers_v6.inc

peer include files
^^^^^^^^^^^^^^^^^^

In den beiden files ``ebgp_peers_v4.inc`` und ``ebgp_peers_v6.inc`` gibt es jeweils einen Eintrag pro
peer. Nicht jeder peer muss v4 **und** v6 anbieten. Die grundlegenden Paramter für die
BGP-Verbindung sind für alle (externen) peers identisch, so dass sie in einem template
(namens ``ebgp_ic``) zusammengefasst sind. So ist jeder einzelne Eintrag recht kurz und folgt dem
Muster::

  protocol bgp [name_of_peer] from ebgp_ic {
      neighbor [IP_of_peer] as [AS_of_peer];
  };

Die Adresse des peer (=neighbour) ist in der v4-config eine v4-Adresse und entsprechend in der
v6-config eine v6-Adresse.

bird config 
^^^^^^^^^^^

Im Großen und Ganzen handelt es sich bei uns um eine recht normale bird-BGP-Konfiguration
(nachdem der Versuch, in bird eine gleichberechtigte config für zwei AS hinzubekommen
gescheitert war). Die Routen zu den anderen communities werden über BGP abgeglichen. Die eigenen
Netze, die ins IC-VPN bekannt zu geben sind, werden über einen ``protocol direkt``-Eintrag
bestimmt.

Das config file wird mit den üblichen Standards eröffnet:

* Die ``router-id`` muss bei uns explizit gesetzt werden und entspricht der IP des Gates im
  IC-VPN-Transfernetz. Als ``router-id`` kommt in beiden Konfigurationen die v4-(sic!)-Adresse
  zum Einsatz.
* Wenn wir zwei kernel routing tables beschicken wollen, brauchen wir auch in bird dafür
  zwei routing ``table``s. Die zweite ist eine einfache Kopie der ersten, auf der ausschließlich
  gearbeitet wird.
* Die Definition von Konstanten erleichtert das Leben ein wenig.

Es folgt jeweils ein Block mit ein paar Funktionen, die beim Filtern der zu sendenden und
der empfangenen Routen eingesetzt werden, um beides aus unserer Sicht zu kontrollieren (wir
nehmen nicht jede angebotene Route an und schicken auch nur Routen auf unsere eigenen Netze
raus).

Die dann folgenden ``device``-, ``direct``-, ``kernel``- und ``pipe``-Protokolldefinitionen
dienen der Kommunikation von bird in Richtung des Kernels des hosts: Ohne ``device``-Protokoll
kann bird fast nichts. Über das ``direct``-Protokoll werden die aktiven mwu-eigenen Netze
gefunden, die den peers gegenüber beworben werden sollen und über die
``kernel``-Protokollinstanzen wird der host mit den von den peers erhaltenen routing-Informationen
versorgt.

Abgesehen von der mittels ``include`` eingebundenen Liste der peers, bilden die ``template``s
für die BGP-Verbindungen den Abschluss. Es gibt je ein ``template`` für internal BGP und für
external BGP. Jeweils werden die eigene ASN, die eigene IP-Adresse für abgehende Verbindungen,
die anzuwendenden Filter und ein paar flags definiert. Alle diese Einstellungen sind für
jeweils alle iBGP- und alle eBGP-Verbindungen gleich; es ändern sich immer nur die Daten der
entsprechenden peers. Die peers werden in eingebundenen file (für eBGP) bzw. im Anschluss
(für iBGP) unter Bezug auf diese ``template``s definiert.

Ein erwähnenswerter Punkt sind die ``export filter``-Definitionen im eBGP. Jedes Gate kann im
IC-VPN nur im Namen **einer** community auftreten und auch nur **eine** ASN nach dort anbieten.
So nennen sich Lotuswurzel, Hinterschinken und Spinat im IC-VPN z.B. ``wiesbaden1``,
``wiesbaden2`` und ``mainz2`` (resp.). Während letzteres die ASN 65037 bewirbt, geben die
beiden anderen 65036 an. Intern können alle Gates aber Pakete an alle communities ausliefern.
Deshalb gibt z.B. Spinat an, **hinter** seiner 65037 auch die 65036 erreichen zu können
(liefert evtl. Pakete dann aber natürlich direkt aus); die beiden anderen Gates verfahren
entsprechend anders herum ebenso. Damit der Spinat gegenüber den beiden anderen Gates
beim routing gen Wiesbaden nicht benachteiligt wird, geben letztere bekannt, dass die
Routen über sie nach dem ASN 65036 auch noch in das ASN 65036 müssen (via Spinat:
65037-65036); ebenso anders herum wieder respektive. => Bei Übernahme der configs von
einer community in die andere ist also auch an dieser Stelle Änderungsbedarf!

Das iBGP wir **nur** innerhalb einer community gefahren (als für gates, die im IC-VPN als
Wiesbadener gates in Erscheinung treten nur zu anderen Wiesbadener Gates; analog für Mainzer
gates) ! Dagegen bauen wir eBGP sessions aber weder zu Mainzer, noch zu Wiesbadener Gates auf,
als nur zu mwu-externen. Die Erzeugung der include files soll bald mal - unter Verwendung der
Daten aus dem ``icvpn-meta`` repo automagisiert werden, ist aber aktuell noch Handarbeit.

.. _BGP: http://de.wikipedia.org/wiki/Border_Gateway_Protocol
.. _ASN: http://wiki.freifunk.net/AS-Nummern
.. _quagga: http://www.nongnu.org/quagga/
.. _bird daemon: http://bird.network.cz/
.. _Dokumentation: http://wiki.freifunk.net/IC-VPN#BGP_Einrichten
.. _IC-VPN-Meta-repository: https://github.com/freifunk/icvpn-meta
.. _IC-VPN-Scripts-repository: https://github.com/freifunk/icvpn-scripts

.. _tinc:

tinc
----

.. note:: Bisher ist leider noch nicht bewiesen, dass das icVPN-setup für mehrere
    communities auf einem Gate funktioniert.

Alle Gates der am IC-VPN teilnehmenden communities verbinden sich in einem
Transfernetz untereinander. Um ihre virtuellen Kabel zusammenstecken zu können,
bauen sie sich dafür einen virtuellen switch über das Internet auf. Hierbei
kommt `tinc vpn`_ zum Einsatz, ein Protokoll ganz ähnlich dem von uns intern genutzten
:ref:`fastd` .

Jedes telnehmende Gate soll einen Eintrag in der zentralen Gate-Liste_ haben, um
IP-Adress-Kollisionen zu vermeiden. Aus Repräsentationsgründen sollte dort
jede community auch mit mindestens einem Eintrag vertreten sein. Die Lotuswurzel
besetzt dort wiesbaden1 und Spinat soll mainz2 sein. Wie jedes diese Gates
später auch die andere community vertreten kann, sehen wir später (:ref:`bird`).

Diese Liste soll offenbar durch ein IC-VPN-Meta-repository_ auf github abgelöst
werden. In dieses  repository müssen dann Einträge in die beiden files
``wiesbaden`` und ``mainz`` vorgenommen werden. Aktuell empfiehlt es sich,
**beide** Informationsquellen zu pflegen: die Liste vor Beginn der
Gate-Einrichtung, das repository im Anschluss.

Unsere Art, das tinc für das IC-VPN einzurichten ist angelehnt an die
entsprechende freifunk-weite Beschreibung_.
Die Ordnerstruktur für die Konfiguration sieht so aus::

  /etc/tinc/ - Basisverzeichnis
  /etc/tinc/icVPN/ - Config für das IC-VPN Netz
  /etc/tinc/icVPN/hosts/ - keys aller Gates aller teilnehmenden communities

Dafür benötigen wir::

  mkdir /etc/tinc/icVPN
  chown admin:admin /etc/tinc/icVPN   # !!!

Die öffentlichen Schlüssel aller Gates aller teilnehmenden communities verwaltet
Freifunk ebenfalls in einem IC-VPN-repository_ auf github. Dieses wird einfach
auf unser neues Gate dupliziert (kennen wir ja schon)::

  git clone https://github.com/freifunk/icvpn /etc/tinc/icVPN/

Anschließend muss das file ``/etc/tinc/icVPN/tinc.conf`` angepasst (oder
angelegt) werden. Es soll dann wie folgt aussehen, der ``Name`` ist natürlich
der passende für das entsprechende Gate (in diesem Beispiel für die
Lutuswurzel)::

  Name = wiesbaden1
  PrivateKeyFile = /etc/tinc/icVPN/rsa_key.priv
  Mode = Switch
  PingTimeout = 30
  Port = 10656
  Hostnames = yes
  Interface = icVPN

  ConnectTo = berlin1
  ConnectTo = [...]

Für jede Datei im directory ``/etc/tinc/icVPN/hosts`` soll es eine entsprechende
``ConnectTo =``-Zeile geben; nur für den eigenen Eintrag nicht.

.. note:: Die Pflege der ``ConnectTo =``-Einträge sollte baldmöglichst
  automagisiert werden. Im den git repos von ``freifunk`` sollte es dafür auch
  schon was geben...

Nun legen wir noch die ``/etc/tinc/icVPN/tinc-up`` an::

  #!/bin/sh
  /sbin/ip link set dev $INTERFACE up
  /sbin/ip addr add dev $INTERFACE 10.207.X.Y/16 broadcast 10.207.255.255 scope link
  /sbin/ip -6 addr add dev $INTERFACE fec0::a:cf:X:Y/96 preferred_lft 0

  # ip rules
  /sbin/ip rule add lookup icvpn priority 10042
  /sbin/ip -6 rule add lookup icvpn priority 10042

Dabei sind ``X`` und ``Y`` die entsprechenden Stellen aus der Adresse des
Gates im Transfernetz; in der v4-Adresse zur
Basis 10 und in der v6-Adresse zur Basis 16.

Das passende ``/etc/tinc/icVPN/tinc-down``::

  #!/bin/sh
  /sbin/ip addr del dev $INTERFACE 10.207.X.Y/16 broadcast 10.207.255.255
  /sbin/ip -6 addr del dev $INTERFACE fec0::a:cf:X:Y/96

  # ip rules
  /sbin/ip rule del lookup icvpn priority 10042
  /sbin/ip -6 rule del lookup icvpn priority 10042

  # shutdown interface
  /sbin/ip link set dev $INTERFACE down

Rechte anpassen:

  chmod 755 /etc/tinc/icVPN/tinc-*

Ebenso, wie alle Partnergates ihre öffentlichen Schlüssel in
``/etc/tinc/icVPN/hosts/`` liegen haben, braucht auch unser neues Gate so etwas.
Sollen die Schlüssel von einer alten Installation übernommen werden, können wir
den folgenden Schlüssel-Generierungs-Schritt auslassen und die bestehenden
einfach nach ``/etc/tinc/icVPN/rsa_key.priv`` kopiert bzw. per pull request
in das repository transportiert.

Ein neues Schlüsselpaar wird mit einem Aufruf erzeugt::

  tincd -n icvpn -K

die vorgeschlagenen defaults passen. Unter ``/etc/tinc/icVPN/wiesbaden1``
(oder dem entsprechenden Namen) findet sich der public key, der in das
repository wandern muss. Vorher müssen allerdings die Kontaktinformationen
des tinc daemon auf diesem Gate hinzugefügt werden. An den Anfang der Datei:

  Address = [fqdn oder IP-Adresse]
  Port = 10656
  [...]

.. note:: Solange unsere Domains im Schwebestatus hängen, sollten wir als
  Adresse eine IP-Adresse des Gates verwenden. Später sollte es ein extra
  CNAME (nur für diesen Zweck) auf das gate werden.

Als Letztes ist noch die Zeile ``icVPN`` der Datei ``/etc/tinc/nets.boot``
hinzuzufügen. Nun kann tinc gestartet werden.


.. _tinc vpn: http://www.tinc-vpn.org/
.. _IC-VPN-Meta-repository: https://github.com/freifunk/icvpn_meta
.. _Beschribung: http://wiki.freifunk.net/IC-VPN#Tinc_einrichten
.. _IC-VPN-repository: https://github.com/freifunk/icvpn
