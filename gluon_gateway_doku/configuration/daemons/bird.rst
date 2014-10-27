.. _bird:

BIRD (IC-VPN)
=============

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
