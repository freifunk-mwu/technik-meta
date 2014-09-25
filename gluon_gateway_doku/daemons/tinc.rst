.. _tinc:

tinc für IC-VPN einrichten
==========================

Note

Bisher ist leider noch nicht bewiesen, dass das icVPN-setup für mehrere
communities auf einem Gate funktioniert.

Alle Gates der am IC-VPN teilnehmenden communities verbinden sich in einem
Transfernetz untereinander. Um ihre virtuellen Kabel zusammenstecken zu können,
bauen sie sich dafür einen virtuellen switch über das Internet auf. Hierbei
kommt tinc_ zum Einsatz, ein Protokoll ganz ähnlich dem von uns intern genutzten
:ref:`fastd` .

Jedes telnehmende Gate soll einen Eintrag in der zentralen Gate-Liste_ haben, um
IP-Adress-Kollisionen zu vermeiden. Durch unseren dual-community-Ansatz braucht
jedes unserer Gates **zwei** Einträge in dieser Liste; so gehören z.B. sowohl
wiesbaden1, als auch mainz11 zur Lotuswurzel.

Diese Liste soll offenbar durch ein IC-VPN-Meta-repository_ auf github abgelöst
werden. In dieses  repository müssen dann Einträge in die beiden files
``wiesbaden`` und ``mainz`` vorgenommen werden. Aktuell empfiehlt es sich,
**beide** Informationsquellen zu pflegen: die Liste vor Beginn der
Gate-Einrichtung, das repository im Anschluss.

Unsere Art, das tinc für das IC-VPN einzurichten ist angelehnt an die
entsprechende freifunk-weite Beschreibung_. An dieser Stelle wird der Zugang
zum IC-VPN aber nur für den primären Namen (vergl. oben; im Beispiel
``wiesbaden1`` für die Lotusblüte, **nicht** für mainz11) eingerichtet. Der
sekundäre wird später lediglich als sekundäre IP-Adressen auf das interface
konfiguriert (s.u.).

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
angelegt) werden. Es soll dann wie folgt aussehen, der ``Name`` ist dabei der
oben erwähnte primäre Name (im Beispiel ``wiesbaden1``::

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
``ConnectTo =``-Zeile geben.

Note

Die Pflege der ``ConnectTo =``-Einträge sollte baldmöglichst automagisiert
werden. Im den git repos von ``freifunk`` sollte es dafür auch schon was
geben...

Nun legen wir noch die ``/etc/tinc/icVPN/tinc-up`` an::

  #!/bin/sh
  /sbin/ip link set dev $INTERFACE up
  /sbin/ip addr add dev $INTERFACE 10.207.V.W/16 broadcast 10.207.255.255 scope link
  /sbin/ip addr add dev $INTERFACE 10.207.X.Y/16 broadcast 10.207.255.255 scope link
  /sbin/ip -6 addr add dev $INTERFACE fec0::a:cf:V:W/96 preferred_lft 0
  /sbin/ip -6 addr add dev $INTERFACE fec0::a:cf:X:Y/96 preferred_lft 0

Dabei sind ``V`` und ``W`` die entsprechenden Stellen aus den Adressen der
primären Namens und ``X`` und ``Y`` die des sekundären; in der v4-Adresse zur
Basis 10 und in der v6-Adresse zur Basis 16.

Das passende ``/etc/tinc/icVPN/tinc-up``::

  #!/bin/sh
  /sbin/ip addr del dev $INTERFACE 10.207.V.W/16 broadcast 10.207.255.255
  /sbin/ip addr del dev $INTERFACE 10.207.X.Y/16 broadcast 10.207.255.255
  /sbin/ip -6 addr del dev $INTERFACE fec0::a:cf:V:W/96
  /sbin/ip -6 addr del dev $INTERFACE fec0::a:cf:X:Y/96
  /sbin/ip link set dev $INTERFACE down

Rechte anpassen:

  chmod 755 /etc/tinc/icVPN/tinc-*

Ebenso, wie alle Partnergates ihre öffentlichen Schlüssel in
``/etc/tinc/icVPN/hosts/`` liegen haben, barucht auch unser neues Gate so etwas.
Sollen die Schlüssel von einer alten Installation übernommen werden, können wir
den folgenden Schlüssel-Generierungs-Schritt auslassen und sie bestehenden
einfach nach   ... *tbc*



.. _tinc: http://www.tinc-vpn.org/
.. _IC-VPN-Meta-repository: https://github.com/freifunk/icvpn_meta
.. _Beschribung: http://wiki.freifunk.net/IC-VPN#Tinc_einrichten
.. _IC-VPN-repository: https://github.com/freifunk/icvpn
