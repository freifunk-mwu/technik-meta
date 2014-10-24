.. _tinc:

tinc (IC-VPN)
=============

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
  /sbin/ip rule add to 10.0.0.0/8 lookup icvpn priority 10042

Dabei sind ``X`` und ``Y`` die entsprechenden Stellen aus der Adresse des
Gates im Transfernetz; in der v4-Adresse zur
Basis 10 und in der v6-Adresse zur Basis 16.

Das passende ``/etc/tinc/icVPN/tinc-down``::

  #!/bin/sh
  /sbin/ip addr del dev $INTERFACE 10.207.X.Y/16 broadcast 10.207.255.255
  /sbin/ip -6 addr del dev $INTERFACE fec0::a:cf:X:Y/96

  # ip rules
  /sbin/ip rule del to 10.0.0.0/8 lookup icvpn priority 10042

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
