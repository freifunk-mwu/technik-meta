.. _bird:

bird für IC-VPN einrichten
==========================

Das IC-VPN beruht darauf, dass zwischen den teilnehmenden communities ein
IP-Routing (layer 3) stattfindet. Dafür wird über :ref:`tinc` ein Transfernetz
zwischen einigen Gates aufgebaut. Über dieses Transfernetz wird dann
ge-route-t.

Die Routing-Einträge auf diesen vielen Routern (Gates) werden nicht
manuell, sondern per BGP_ gepflegt. Es gibt zwei etablierte BGP-Implmentationen:
quagga_ und bird_; wir haben uns für Letzteres entschieden. Auch hier folgen
wir grob der zentralen Dokumentation_ uns es sei auf das im Aufbau befindliche
IC-VPN-Meta-repository_ für die Konfigurationsdaten verwiesen.

tbc...










.. _BGP: http://de.wikipedia.org/wiki/Border_Gateway_Protocol
.. _quagga: http://www.nongnu.org/quagga/
.. _bird: http://bird.network.cz/
.. _IC-VPN-Meta-repository: https://github.com/freifunk/icvpn_meta
